Let's start from the absolute beginning.

---

## 1. THE PROBLEM BEFORE ORM EXISTED

### The Real World Analogy

Imagine you're a chef. You need ingredients from a warehouse. The warehouse stores everything in **boxes with a specific labeling system** (rows, columns, data types). But your kitchen works with **objects** — a `Recipe` has `Ingredients`, quantities, steps.

Every time you need ingredients, you have to:

1. Go to the warehouse
2. Read the box labels
3. Manually translate label language to kitchen language
4. Carry things back
5. Do the reverse when returning

That's exactly what developers did **before ORM** — manually translating between the **relational world** (tables, rows, SQL) and the **object world** (classes, properties, references).

---

## 2. ADO.NET — THE FOUNDATION LAYER

### What Is ADO.NET?

ADO.NET is Microsoft's **low-level data access technology**, introduced with .NET Framework 1.0 (around 2002). It sits **directly above the database driver**.

Think of it as a **thin wrapper** over raw SQL — it gives you:

- Connection management
- Command execution
- Result reading
- Transaction control

But it gives you **nothing else**. No object mapping. No change tracking. No query generation. You do everything manually.

### Let's See Raw ADO.NET

```csharp
using System.Data;
using Microsoft.Data.SqlClient; // The SQL Server driver

public class ProductRepository
{
    // Why private readonly? 
    // private = nobody outside this class should touch it
    // readonly = once assigned in constructor, it CANNOT be reassigned
    // This protects the connection string from mutation at runtime
    private readonly string _connectionString;

    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public Product? GetProductById(int id)
    {
        // WHY "using"?
        // SqlConnection implements IDisposable
        // A database connection is an EXPENSIVE, LIMITED resource
        // The database server has a fixed connection pool (default ~100 connections)
        // If you don't dispose, the connection stays open FOREVER
        // "using" guarantees Dispose() is called even if an exception occurs
        // Internally: C# compiler transforms this into try/finally
        using var connection = new SqlConnection(_connectionString);

        // WHY open explicitly?
        // Creating the connection object doesn't open the physical connection
        // Open() actually acquires a connection from the CONNECTION POOL
        // (We'll deep dive connection pooling shortly)
        connection.Open();

        // SqlCommand = represents a SQL statement to execute against the database
        using var command = new SqlCommand(
            "SELECT Id, Name, Price, Stock FROM Products WHERE Id = @Id", 
            connection
        );

        // WHY @Id parameter instead of string concatenation?
        // CRITICAL: SQL Injection prevention
        // If you did: $"WHERE Id = {id}" -- an attacker sends id = "1; DROP TABLE Products"
        // Parameters are sent SEPARATELY to the database driver
        // The driver treats them as DATA, never as SQL code
        // This is a non-negotiable security rule in production
        command.Parameters.AddWithValue("@Id", id);

        // ExecuteReader = execute a SELECT and get back a stream of rows
        // WHY a reader and not a List?
        // The reader is a FORWARD-ONLY, READ-ONLY stream
        // It reads ONE ROW AT A TIME from the database
        // This is memory-efficient: a 10 million row table doesn't load into RAM
        using var reader = command.ExecuteReader();

        // reader.Read() moves the cursor to the next row
        // Returns true if a row exists, false if no more rows
        if (reader.Read())
        {
            // Manual mapping: you're doing what ORM does automatically
            // reader["Name"] = looks up column by name (slower)
            // reader.GetString(1) = looks up by index (faster, more fragile)
            return new Product
            {
                Id = reader.GetInt32(0),           // column index 0 = Id
                Name = reader.GetString(1),         // column index 1 = Name
                Price = reader.GetDecimal(2),       // column index 2 = Price
                Stock = reader.GetInt32(3)          // column index 3 = Stock
            };
        }

        return null; // Product not found
    }
}
```

### What Just Happened Internally?

Let me walk you through the **full execution stack**:

```
Your C# code
    ↓
SqlConnection.Open()
    ↓
ADO.NET asks Connection Pool: "Do you have a free connection to this server?"
    ↓ (if yes, reuses it. if no, creates new TCP connection)
Physical TCP connection to SQL Server established
    ↓
SqlCommand.ExecuteReader() sends SQL text over the wire
    ↓
SQL Server parses SQL → builds execution plan → executes → sends rows back
    ↓
SqlDataReader receives rows ONE AT A TIME via TDS protocol (Tabular Data Stream)
    ↓
Your code reads each row
    ↓
"using" block exits → Dispose() called → connection returned to pool
```

### Connection Pooling — The Hidden Mechanism

This is critical and almost nobody explains it properly.

```
First request:  Your app → creates real TCP connection → sends to pool
Second request: Your app → pool says "here's a free one" → reuses TCP connection
...
1000 requests:  pool manages ~100 real connections for thousands of virtual ones
```

**Why does this matter?**

- Opening a real TCP connection takes **~100-300ms**
- Without pooling, a high-traffic app would be **unusably slow**
- Connection string controls pooling: `"Max Pool Size=100;Min Pool Size=5"`

**What breaks it:**

```csharp
// DISASTER: connection never returned to pool
var conn = new SqlConnection(connStr);
conn.Open();
// forgot to dispose - connection LEAKS
// After ~100 leaks: new requests TIMEOUT waiting for a pool connection
```

---

## 3. THE PROBLEM WITH RAW ADO.NET AT SCALE

Let me show you what production code looks like without ORM:

```csharp
// BAD: What production code WITHOUT ORM looks like
// Imagine doing this for 50 tables...

public async Task<Order> GetOrderWithDetailsAsync(int orderId)
{
    using var connection = new SqlConnection(_connectionString);
    await connection.OpenAsync();

    // Query 1: Get Order
    using var orderCmd = new SqlCommand(
        "SELECT * FROM Orders WHERE Id = @Id", connection);
    orderCmd.Parameters.AddWithValue("@Id", orderId);
    
    Order order = null;
    using (var reader = await orderCmd.ExecuteReaderAsync())
    {
        if (await reader.ReadAsync())
        {
            order = new Order
            {
                Id = reader.GetInt32(0),
                CustomerId = reader.GetInt32(1),
                OrderDate = reader.GetDateTime(2),
                TotalAmount = reader.GetDecimal(3)
                // imagine 20 more columns...
            };
        }
    }

    // Query 2: Get Order Items (separate query!)
    using var itemsCmd = new SqlCommand(
        "SELECT * FROM OrderItems WHERE OrderId = @OrderId", connection);
    itemsCmd.Parameters.AddWithValue("@OrderId", orderId);
    
    order.Items = new List<OrderItem>();
    using (var reader = await itemsCmd.ExecuteReaderAsync())
    {
        while (await reader.ReadAsync())
        {
            order.Items.Add(new OrderItem
            {
                // manual mapping again...
            });
        }
    }

    return order;
    // Problems:
    // 1. ZERO compile-time safety - typos in column names = runtime crash
    // 2. Schema changes = grep through 500 SQL strings manually
    // 3. No change tracking - you have to manually detect what changed
    // 4. N+1 problem built-in if you're not careful
    // 5. Testing? You're testing SQL strings...
}
```

**This is why ORM was invented.** Not to be slower. Not to be "easier." But to solve real, expensive engineering problems.

---

## 4. ENTER ENTITY FRAMEWORK CORE

### What EF Core Actually Is

EF Core is NOT magic. It is a **code generator and runtime** that:

1. **Reads your C# classes** (your model)
2. **Understands relationships** (via configuration or conventions)
3. **Generates SQL** at runtime
4. **Tracks changes** to objects
5. **Maps results back** to objects

Think of it as a **translator** who speaks both C# and SQL fluently — and also remembers what you told them earlier (change tracking).

### The Three Core Concepts You Must Understand

```
1. DbContext    = Your session with the database
2. DbSet<T>     = Represents a TABLE as a queryable collection
3. Entities     = Your C# classes that map to tables
```

### Let's Build It Properly

```csharp
// The Entity - a plain C# class that EF Core will map to a table
// WHY no [Required] attributes here? We'll use Fluent API (cleaner, separates concerns)
public class Product
{
    // WHY int Id specifically?
    // EF Core CONVENTION: a property named "Id" or "ProductId" is automatically
    // treated as the PRIMARY KEY. This is convention-over-configuration.
    // Internally: EF Core scans your properties during model building
    // and applies rules to decide PK, FK, index, etc.
    public int Id { get; set; }
    
    public string Name { get; set; } = string.Empty;
    
    public decimal Price { get; set; }
    
    public int Stock { get; set; }
    
    // Navigation property - this is how EF Core understands relationships
    // WHY virtual? In EF6, this enabled LAZY LOADING via proxy classes
    // In EF Core, lazy loading requires explicit setup, but virtual is still common
    // WHY ICollection and not List?
    // ICollection = read, write, count. IEnumerable = read only. List = full control
    // Using the interface = looser coupling, EF Core can inject its own proxy type
    public virtual ICollection<OrderItem> OrderItems { get; set; } 
        = new List<OrderItem>();
}

public class Order
{
    public int Id { get; set; }
    
    public DateTime OrderDate { get; set; }
    
    public decimal TotalAmount { get; set; }
    
    // Foreign key property - EF Core will use this to create FK constraint
    public int CustomerId { get; set; }
    
    // Navigation property to the related Customer
    // WHY nullable (Customer?)? Because when you load an Order without
    // including Customer, this will be null. Null tells you "not loaded"
    public Customer? Customer { get; set; }
    
    public ICollection<OrderItem> Items { get; set; } = new List<OrderItem>();
}

// The DbContext - THE most important class in EF Core
// Think of it as your "Unit of Work" + database session combined
public class AppDbContext : DbContext  // WHY inherit DbContext? It provides ALL the machinery
{
    // WHY DbContextOptions<AppDbContext> in constructor?
    // This is how ASP.NET Core's DI system passes configuration (connection string,
    // database provider, etc.) to the context
    // The generic <AppDbContext> ensures the options are TYPE-SAFE to THIS context
    public AppDbContext(DbContextOptions<AppDbContext> options) 
        : base(options)  // Pass options to the parent DbContext class
    { 
    }

    // DbSet<T> = represents the Products TABLE
    // WHY DbSet and not List<T>?
    // DbSet implements IQueryable<T> which means queries are TRANSLATED TO SQL
    // A List<T> would load EVERYTHING into memory first, then filter in C#
    // DbSet = lazy, translated. List = eager, in-memory.
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<Customer> Customers { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    // OnModelCreating = where you configure your model using Fluent API
    // WHY override this? To tell EF Core things it can't figure out by convention
    // WHY Fluent API over Data Annotations?
    // 1. Keeps your entity classes PURE (no infrastructure concerns)
    // 2. More powerful - can configure things annotations can't
    // 3. Centralized configuration
    // 4. SOLID: Single Responsibility - entity class only describes the shape
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure Product entity
        modelBuilder.Entity<Product>(entity =>
        {
            // Table name (by convention it would be "Products" anyway)
            entity.ToTable("Products");
            
            // Primary key (by convention "Id" is already PK, but explicit is clearer)
            entity.HasKey(p => p.Id);
            
            // Column configuration
            entity.Property(p => p.Name)
                .IsRequired()           // NOT NULL in SQL
                .HasMaxLength(200)      // VARCHAR(200)
                .HasColumnType("nvarchar(200)");  // exact SQL type
            
            entity.Property(p => p.Price)
                .HasPrecision(18, 2);  // DECIMAL(18,2) - critical for money!
                // WHY? DECIMAL vs FLOAT: float has rounding errors
                // 0.1 + 0.2 in float = 0.30000000000000004
                // For money, ALWAYS use decimal

            // Index for performance on commonly searched column
            entity.HasIndex(p => p.Name)
                .HasDatabaseName("IX_Products_Name");
        });

        // Configure Order-Customer relationship
        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasOne(o => o.Customer)        // Order has ONE Customer
                  .WithMany(c => c.Orders)         // Customer has MANY Orders
                  .HasForeignKey(o => o.CustomerId) // FK is CustomerId
                  .OnDelete(DeleteBehavior.Restrict); // Don't cascade delete orders when customer deleted
                  // WHY Restrict? Cascade delete = customer deleted → all orders deleted
                  // In most business scenarios, you NEVER want to lose order history
        });
    }
}
```

### Registering in ASP.NET Core

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// WHY AddDbContext and not AddDbContextFactory?
// AddDbContext = registers with SCOPED lifetime (one context per HTTP request)
// This is the standard choice for web apps
// WHY scoped? DbContext is NOT thread-safe. One request = one thread = safe.
// If you made it Singleton, multiple requests sharing ONE context = race conditions
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        sqlOptions =>
        {
            // Retry on transient failures (network blips, brief SQL Azure unavailability)
            // WHY? Cloud databases occasionally have brief interruptions
            // Without retry: one network hiccup = HTTP 500 for the user
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 3,
                maxRetryDelay: TimeSpan.FromSeconds(5),
                errorNumbersToAdd: null
            );
        }
    );
    
    // In development only: log the actual SQL being generated
    // NEVER in production: leaks query structure, sensitive data, slows things down
    if (builder.Environment.IsDevelopment())
    {
        options.EnableSensitiveDataLogging();  // logs parameter values
        options.LogTo(Console.WriteLine, LogLevel.Information);
    }
});
```

---

## 5. CHANGE TRACKING — EF CORE'S SUPERPOWER (AND DANGER)

This is where most developers have no idea what's happening internally.

### How Change Tracking Works

When you load an entity, EF Core does something behind the scenes:

```csharp
var product = await context.Products.FindAsync(1);
// At this point, EF Core has:
// 1. Created a Product object with the data
// 2. Created an INTERNAL SNAPSHOT (a copy of the original values)
// 3. Registered the entity in its IdentityMap (dictionary of tracked objects)

product.Price = 999.99m; // You modify the price

await context.SaveChangesAsync();
// EF Core internally:
// 1. Scans ALL tracked entities
// 2. For each entity, compares CURRENT values vs SNAPSHOT values
// 3. Detects: Price changed from 50.00 to 999.99
// 4. Generates: UPDATE Products SET Price = 999.99 WHERE Id = 1
// 5. Executes the SQL
// 6. Updates the snapshot to reflect new values
```

The internal state machine for each entity:

```
Detached    → not tracked by context (new object from 'new' keyword)
Added       → will be INSERTed on SaveChanges
Unchanged   → loaded from DB, no modifications
Modified    → loaded from DB, at least one property changed
Deleted     → will be DELETEd on SaveChanges
```

```csharp
// Seeing the state machine in action
var product = new Product { Name = "Widget", Price = 9.99m };

Console.WriteLine(context.Entry(product).State); 
// → Detached (we haven't told context about it yet)

context.Products.Add(product);
Console.WriteLine(context.Entry(product).State); 
// → Added

await context.SaveChangesAsync();
Console.WriteLine(context.Entry(product).State); 
// → Unchanged (saved, now tracked as clean)

product.Price = 19.99m;
Console.WriteLine(context.Entry(product).State); 
// → Modified (change tracking detected the difference!)

context.Products.Remove(product);
Console.WriteLine(context.Entry(product).State); 
// → Deleted
```

### The Performance Trap Everyone Falls Into

```csharp
// BAD: Loading 10,000 products for a report
// Change tracking creates 10,000 snapshots in memory - WASTEFUL
var products = await context.Products.ToListAsync();

// GOOD: When you only need to READ (no updates), disable tracking
var products = await context.Products
    .AsNoTracking()  // Tell EF: "don't track these, I won't save changes"
    .ToListAsync();
// Result: ~40% less memory usage, faster query execution
// WHY? No snapshot creation, no IdentityMap registration

// RULE OF THUMB:
// Reading data for API responses / display → AsNoTracking()
// Reading data you intend to modify → tracked (default)
```

---

## 6. QUERYING — IQueryable vs IEnumerable (CRITICAL CONCEPT)

This is one of the most misunderstood things in .NET and causes massive performance bugs.

```csharp
// Understanding what IQueryable REALLY means

// IQueryable<Product> = a QUERY DEFINITION, not data
// The SQL has NOT been sent to the database yet!
IQueryable<Product> query = context.Products.Where(p => p.Price > 100);

// At this point: query is just an EXPRESSION TREE stored in memory
// Expression tree ≈ "a description of the query" in C# object form
// EF Core will translate this tree to SQL when you materialize it

// Adding more conditions BEFORE materialization = all in ONE SQL query
if (someCondition)
{
    query = query.Where(p => p.Stock > 0);  // Adds AND Stock > 0 to SQL
}

query = query.OrderBy(p => p.Name);  // Adds ORDER BY to SQL

// MATERIALIZATION: this is when the SQL is actually sent
var products = await query.ToListAsync();
// Generated SQL: SELECT * FROM Products WHERE Price > 100 AND Stock > 0 ORDER BY Name

// -----------------------------------------------------------

// THE DANGER: Switching to IEnumerable too early

// This materializes the query (loads ALL products into memory)
IEnumerable<Product> enumerable = context.Products.AsEnumerable();

// NOW this filter runs in C#, not SQL
// ALL rows were already loaded from the database!
var filtered = enumerable.Where(p => p.Price > 100); // IN-MEMORY, SLOW, WASTEFUL

// If Products has 1 million rows: you just loaded 1 million rows
// into application server memory to filter 50 of them.
// This is a production disaster.
```

### Expression Trees — The Secret Behind LINQ to SQL

```csharp
// When you write this LINQ:
var query = context.Products.Where(p => p.Price > 100);

// The C# compiler does NOT compile p => p.Price > 100 as executable code
// Instead, it builds an EXPRESSION TREE:
// BinaryExpression(
//   Left: MemberExpression(p.Price),
//   Operator: GreaterThan,
//   Right: ConstantExpression(100)
// )

// EF Core's query translator walks this tree and generates:
// "WHERE Price > 100"

// This is why LINQ works against databases.
// If it were compiled to a Func<T,bool> (a real delegate), 
// EF Core couldn't inspect it — it would just be machine code.
// Expression trees = inspectable, translatable code descriptions.
```

---

## 7. THE N+1 PROBLEM — THE #1 PERFORMANCE KILLER

Every .NET developer needs to understand this deeply.

```csharp
// THE N+1 PROBLEM - this will destroy your app's performance

// You load 100 orders
var orders = await context.Orders.ToListAsync(); // 1 SQL query

// Now for each order, you access the Customer (navigation property)
foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name); 
    // EACH ACCESS = a separate SQL query!
    // "SELECT * FROM Customers WHERE Id = 1"
    // "SELECT * FROM Customers WHERE Id = 2"
    // ...100 times
}
// Total: 1 + 100 = 101 queries. THIS IS THE N+1 PROBLEM.

// -----------------------------------------------------------

// SOLUTION 1: Eager Loading with Include()
var orders = await context.Orders
    .Include(o => o.Customer)      // JOIN with Customers in ONE query
    .Include(o => o.Items)         // JOIN with OrderItems
        .ThenInclude(i => i.Product) // JOIN OrderItems with Products
    .ToListAsync();
// Total: 1 query with JOINs. 
// Generated SQL:
// SELECT o.*, c.*, i.*, p.*
// FROM Orders o
// LEFT JOIN Customers c ON o.CustomerId = c.Id
// LEFT JOIN OrderItems i ON i.OrderId = o.Id
// LEFT JOIN Products p ON i.ProductId = p.Id

// -----------------------------------------------------------

// SOLUTION 2: Explicit Loading (when you need control)
var order = await context.Orders.FindAsync(orderId);

// Load related data explicitly, on demand
await context.Entry(order)
    .Reference(o => o.Customer)  // single reference
    .LoadAsync();

await context.Entry(order)
    .Collection(o => o.Items)    // collection
    .LoadAsync();

// -----------------------------------------------------------

// SOLUTION 3: Projection (most efficient for read-only scenarios)
var orderDtos = await context.Orders
    .Select(o => new OrderDto  // SELECT only what you need
    {
        OrderId = o.Id,
        CustomerName = o.Customer.Name,  // EF Core translates this to a JOIN
        ItemCount = o.Items.Count         // Translates to COUNT(*)
    })
    .ToListAsync();
// Generated SQL: SELECT o.Id, c.Name, COUNT(i.Id) FROM...
// Only the COLUMNS YOU NEED are fetched. Minimal data transfer.
```

---

## 8. MIGRATIONS — DATABASE SCHEMA VERSIONING

```csharp
// How migrations work internally:

// 1. You modify your entity class
// 2. EF Core compares your model to the last known "snapshot"
// 3. It generates a migration file describing the DIFF

// Command: dotnet ef migrations add AddProductCategory

// EF Core generates:
public partial class AddProductCategory : Migration
{
    // Up() = apply the migration (schema change forward)
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "Category",
            table: "Products",
            type: "nvarchar(100)",
            nullable: true);  // EF Core defaulted to nullable
                              // IMPORTANT: if you made it required with a default,
                              // existing rows need a value or the migration FAILS
    }

    // Down() = rollback (undo the migration)
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "Category",
            table: "Products");
    }
}

// WHY is Down() important?
// If you deploy v2 and it breaks, you need to rollback
// dotnet ef database update PreviousMigrationName
// This calls Down() on each migration between current and target

// PRODUCTION REALITY: Never run migrations from app startup
// context.Database.Migrate() on startup is an antipattern because:
// 1. Multiple instances race to migrate simultaneously
// 2. Long migrations hold up app startup
// 3. Failed migrations crash ALL instances
// Instead: run migrations in CI/CD pipeline or dedicated migration job
```

---

## 9. ADO.NET vs EF CORE — WHEN TO USE WHICH

This is a nuanced engineering decision, not a holy war:

```csharp
// USE EF CORE when:
// - Standard CRUD operations
// - Relationship navigation
// - Rapid development
// - Model-first or code-first design
// - Change tracking is valuable

// USE ADO.NET / Dapper when:
// - Complex reporting queries with many JOINs and aggregations
// - Stored procedures that return multiple result sets
// - Bulk operations (10,000+ rows)
// - You need EXACT control over SQL
// - Maximum performance is critical for a specific hot path

// HYBRID APPROACH (what production apps actually do):
public class ProductRepository
{
    private readonly AppDbContext _context;         // EF Core
    private readonly string _connectionString;      // For raw ADO.NET

    // EF Core for standard operations
    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products.FindAsync(id);
    }

    // Raw SQL via EF Core (best of both worlds)
    public async Task<List<ProductSalesReport>> GetSalesReportAsync(DateTime from, DateTime to)
    {
        // EF Core can execute raw SQL and map results to objects
        // Use when EF Core's LINQ can't express the query efficiently
        return await _context.Database
            .SqlQuery<ProductSalesReport>(
                $"EXEC sp_GetProductSalesReport {from}, {to}"
            )
            .ToListAsync();
    }
    
    // ADO.NET for bulk operations
    public async Task BulkInsertProductsAsync(IEnumerable<Product> products)
    {
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        // SqlBulkCopy = specialized ADO.NET for massive inserts
        // EF Core AddRange on 50,000 items = slow (individual INSERT statements)
        // SqlBulkCopy = one network round-trip for ALL rows (bulk protocol)
        using var bulkCopy = new SqlBulkCopy(connection);
        bulkCopy.DestinationTableName = "Products";
        
        var dataTable = ConvertToDataTable(products);
        await bulkCopy.WriteToServerAsync(dataTable);
    }
}
```

---

## Interview Questions You Should Know Cold

1. What is the difference between `FirstOrDefault()` and `Find()` in EF Core?
2. What happens if you call `SaveChangesAsync()` with no changes?
3. Explain what happens internally when EF Core executes a LINQ query.
4. Why is `DbContext` scoped and not singleton?
5. What is the difference between eager loading, lazy loading, and explicit loading?
6. What is a "tracking query" vs a "no-tracking query"?

---

## 🔥 Cross Questions For YOU

Think carefully before answering. Don't Google. Reason through it:

**Q1.** You have a `DbContext` registered as **Singleton** instead of Scoped in a web application. Two users make simultaneous requests. User A loads a Product and modifies the price. User B simultaneously loads the same Product. What happens? What are the specific failure modes?

**Q2.** Consider this code:

```csharp
var products = context.Products
    .Where(p => p.Price > 100)
    .ToList()
    .Where(p => p.Name.StartsWith("A"));
```

Where exactly does the database query end and the in-memory filter begin? How would you fix it?

**Q3.** You load 1,000 entities with EF Core to generate a PDF report. No changes are made. What unnecessary work is EF Core doing? How do you fix it?

**Q4.** Why does EF Core generate `UPDATE Products SET Price = @p0, Name = @p1, Stock = @p2 WHERE Id = @p3` and update ALL columns, even if you only changed `Price`? Is this a problem? How can you optimize it?

**Q5.** What is the difference between `context.Products.Add(product)` and `context.Entry(product).State = EntityState.Added`?

---

## 🎯 Practical Scenario

You're building an **e-commerce order processing system**. You receive a batch of 500 orders every minute. Each order has 1-20 line items. You need to:

1. Save all orders to the database
2. Update inventory (reduce stock for each product)
3. Generate a summary report of total revenue per product category

**Design question:** Which parts would you use EF Core for? Which parts would you use raw ADO.NET or stored procedures for? What would your `DbContext` lifetime look like? How would you handle the inventory update to prevent race conditions?

---

## 📝 Mini Assignment

Write a `ProductService` class that:

1. Uses `AppDbContext` via constructor injection
2. Has a method `GetTopSellingProductsAsync(int topN, DateTime since)` that returns the top N products by total units sold since a given date — using a single efficient query (no N+1)
3. Has a method `UpdatePriceAsync(int productId, decimal newPrice)` that updates only the price column
4. Uses `AsNoTracking()` where appropriate
5. Handles the case where the product doesn't exist

Tags:
#csharp 
#dotnet 
#development 
#database 