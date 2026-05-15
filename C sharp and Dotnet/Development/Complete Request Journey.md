## PART 1: THE BIG PICTURE (Simple Explanation First)

Imagine you're sending a **letter through a postal system**.

- You (the user) write a letter (HTTP Request)
- The post office (Web Server / Kestrel) receives it
- It gets sorted and routed (Middleware Pipeline + Routing)
- A department handles it (Controller)
- That department asks a specialist (Service Layer)
- The specialist queries the records office (Repository Layer)
- Records office goes to the filing cabinet (Database via EF Core)
- The response travels back the same way, packaged as HTML/JSON

Now let's make this real and technical.

---

## PART 2: THE FULL STACK — LAYERS WE WILL COVER

```
BROWSER
  │
  │  HTTP Request (GET/POST)
  ▼
KESTREL (Web Server)
  │
  ▼
MIDDLEWARE PIPELINE
  │  [Logging → Exception → Auth → Routing → CORS → ...]
  ▼
ROUTING ENGINE
  │
  ▼
MODEL BINDING
  │
  ▼
FILTERS (Action Filters, Auth Filters, etc.)
  │
  ▼
CONTROLLER / ACTION METHOD
  │
  ▼
SERVICE LAYER (Business Logic)
  │
  ▼
REPOSITORY LAYER (Data Abstraction)
  │
  ▼
ENTITY FRAMEWORK CORE (ORM)
  │
  ▼
DATABASE (SQL Server / PostgreSQL)
  │
  ▼  (Response travels back up)
VIEWMODEL / DTO
  │
  ▼
VIEW ENGINE (Razor)
  │  [Tag Helpers, View Components, Partial Views]
  ▼
HTML RESPONSE → BROWSER
```

Every single one of these is a deliberate design decision. Let's go through each.

---

## PART 3: THE BROWSER — WHERE IT ALL STARTS

User clicks a button. What actually happens?

```html
<!-- This is a Razor View — a .cshtml file on the server -->
<form asp-controller="Product" asp-action="Create" method="post">
    <input asp-for="ProductName" />
    <button type="submit">Save Product</button>
</form>
```

### What is `asp-controller`, `asp-action`, `asp-for`?

These are **Tag Helpers**.

**Tag Helpers** are C# classes that run **at render time on the server** and transform HTML elements. They are NOT JavaScript. They execute when Razor renders the page, BEFORE the HTML is sent to the browser.

```csharp
// What asp-controller and asp-action do internally:
// They call IUrlHelper.Action("Create", "Product")
// And generate: action="/Product/Create"

// What asp-for does:
// It reads the ModelExpression for "ProductName"
// Generates: name="ProductName" id="ProductName"
// AND generates validation attributes like data-val="true"
```

So when the button is clicked, the browser sends:

```
POST /Product/Create HTTP/1.1
Host: localhost:5000
Content-Type: application/x-www-form-urlencoded

ProductName=Laptop&Price=999.99&CategoryId=3
```
	`application/x-www-form-urlencoded` is a MIME(Multipurpose Internet Mail Extensions tells browser and email clients how to interpret and display a file content.) type used in HTML forms to encode data into a string of key-value pairs.

This is a raw **HTTP Request** — text over TCP. Nothing magical yet.

---

## PART 4: KESTREL — THE WEB SERVER

### What is Kestrel?

Kestrel is ASP.NET Core's **built-in, cross-platform web server** written in C#. It listens on a TCP port, accepts connections, and reads raw HTTP bytes.

**Why Kestrel exists:** Old ASP.NET was tied to IIS (Windows only). ASP.NET Core needed to run on Linux/Mac/Docker. So Microsoft built Kestrel from scratch using `System.IO.Pipelines` — a high-performance I/O abstraction.

```csharp
// Program.cs — This is where Kestrel starts
var builder = WebApplication.CreateBuilder(args);
// ^^^^ Internally, this configures Kestrel, sets up the DI container,
//      loads configuration from appsettings.json, environment variables, etc.

var app = builder.Build();
// ^^^^ This BUILDS the middleware pipeline

app.Run();
// ^^^^ This starts Kestrel listening on the port
```

**Internal flow inside Kestrel:**
1. Kestrel receives raw TCP bytes
2. It parses HTTP/1.1 or HTTP/2 protocol
3. Creates an `HttpContext` object — **this is the most important object in ASP.NET Core**
4. Passes `HttpContext` to the middleware pipeline
### What is HttpContext?

`HttpContext` is the **god object**(kami) of a single request. It carries everything:

```csharp
HttpContext
├── Request          → URL, headers, body, query string, cookies, form data
├── Response         → status code, headers, body stream
├── User             → ClaimsPrincipal (who is logged in)
├── Session          → session data
├── Items            → dictionary to pass data between middleware
├── RequestServices  → the DI container scoped to THIS request
├── Connection       → IP address, port
└── TraceIdentifier  → unique ID for this request (for logging)
```

Every middleware, every controller, every service called within a request — they all share this ONE `HttpContext`.

---

## PART 5: THE MIDDLEWARE PIPELINE — THE MOST IMPORTANT CONCEPT

### Simple Explanation

Think of middleware as a **series of security checkpoints at an airport**.
- Checkpoint 1: Passport control (Authentication)
- Checkpoint 2: Baggage scan (Request logging)
- Checkpoint 3: Customs (Authorization)
- Checkpoint 4: Gate check (Routing)

Each checkpoint can:
1. Let you through
2. Stop you completely (short-circuit)
3. Modify you and let you through

### The Technical Reality

[Middleware](Middlewares.md) is a **chain of delegates**. Each middleware is literally a function:

```csharp
// What middleware IS internally — a function that takes HttpContext
// and a "next" delegate to call the next middleware
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    // RequestDelegate = delegate Task(HttpContext context)

    public RequestLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
        // _next IS the rest of the pipeline — everything after this middleware
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // ═══ BEFORE — executes on the way IN (request going down)
        Console.WriteLine($"Request: {context.Request.Method} {context.Request.Path}");
        var stopwatch = Stopwatch.StartNew();

        await _next(context);
        // ^^^^ THIS LINE is where the entire rest of the pipeline executes
        // After this returns, we're on the way BACK UP (response going up)

        // ═══ AFTER — executes on the way OUT (response going up)
        stopwatch.Stop();
        Console.WriteLine($"Response: {context.Response.StatusCode} in {stopwatch.ElapsedMilliseconds}ms");
    }
}
```

### The Pipeline is Built as Nested Functions

```csharp
// In Program.cs, the ORDER YOU REGISTER MATTERS CRITICALLY
app.UseExceptionHandler("/Error");   // Middleware 1 — outermost
app.UseHttpsRedirection();           // Middleware 2
app.UseStaticFiles();                // Middleware 3
app.UseRouting();                    // Middleware 4 — MUST come before Auth
app.UseAuthentication();             // Middleware 5 — MUST come before Authorization
app.UseAuthorization();              // Middleware 6
app.MapControllers();                // Terminal middleware — routes to controllers
```

**Internally, ASP.NET Core compiles this into:**

```
Request comes in →
  ExceptionHandler(
    HttpsRedirection(
      StaticFiles(
        Routing(
          Authentication(
            Authorization(
              ControllerDispatcher(HttpContext)
            )
          )
        )
      )
    )
  )
← Response goes back out through the same chain in reverse
```

It's **nested function calls**. Not a loop. Not a list. Each `_next(context)` call goes one level deeper.

### Why Order Matters — A Real Bug

```csharp
// ❌ WRONG ORDER — This is a production bug waiting to happen
app.UseAuthorization();   // Authorization runs BEFORE Authentication
app.UseAuthentication();  // User identity not set yet!
// Result: User.Identity.IsAuthenticated is always false
// Protected endpoints appear accessible or always reject everyone

// ✅ CORRECT ORDER
app.UseAuthentication();  // First: Who are you? (sets User/ClaimsPrincipal)
app.UseAuthorization();   // Then: Are you allowed? (reads User)
```

### Static Files — An Important Short-Circuit

```csharp
app.UseStaticFiles();
```

When a request comes in for `/images/logo.png`, `UseStaticFiles` middleware checks if that file exists in `wwwroot/`. If it does, it **short-circuits** — returns the file directly and never calls `_next(context)`. The request never reaches your controllers. This is a major performance optimization.

---

## PART 6: ROUTING ENGINE

After middleware processes the request, [**Routing**](Routing%20in%20MVC%20and%20API.md)  figures out which Controller + Action to call.

```csharp
// Two ways to define routes:

// 1. Conventional Routing (older style, MVC)
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
// URL: /Product/Create → ProductController.Create()
// URL: /Product/Details/5 → ProductController.Details(id: 5)

// 2. Attribute Routing (preferred in modern apps)
[Route("api/products")]
[HttpPost("create")]
public IActionResult Create([FromBody] CreateProductDto dto) { }
// URL: POST /api/products/create
```

**How routing works internally:**
1. `UseRouting()` middleware scans all registered routes at startup and builds a **Route Table** (a decision tree)
2. When a request comes in, it matches the URL against this tree — O(log n) lookup
3. It stores the matched `RouteData` in `HttpContext.Request.RouteValues`
4. `UseAuthorization()` can now read the matched endpoint's `[Authorize]` attributes
5. The terminal middleware (`MapControllers`) actually invokes the controller

**Why are routing and endpoint execution separated?**
Because Authorization middleware needs to know WHICH endpoint was matched (to check `[Authorize]` attributes) BEFORE executing it. If routing and execution were one step, you couldn't authorize before executing.

---

## PART 7: MODEL BINDING — MAGIC THAT MAPS HTTP TO C Sharp

HTTP is text. C# works with strongly-typed objects. **Model Binding bridges this gap.**
```csharp
// The HTTP request body is:
// ProductName=Laptop&Price=999.99&CategoryId=3

// Your action method expects:
[HttpPost]
public async Task<IActionResult> Create(CreateProductViewModel model)
//                                      ^^^^ How does ASP.NET Core
//                                           populate this object from HTTP text?
```

### Model Binding Sources (in priority order):

```csharp
// [FromRoute]   → URL segments: /products/5  → int id = 5
// [FromQuery]   → Query string: ?page=2      → int page = 2
// [FromForm]    → Form body (POST)           → object model
// [FromBody]    → JSON body                  → object dto
// [FromHeader]  → HTTP headers               → string correlationId
// [FromServices] → DI container              → IService service

public async Task<IActionResult> Create(
    [FromBody] CreateProductDto dto,        // from JSON body
    [FromRoute] int categoryId,             // from URL
    [FromQuery] string returnUrl,           // from ?returnUrl=...
    [FromServices] IEmailService emailSvc)  // from DI container
```

**Internal mechanics of model binding:**
1. ASP.NET Core reads the Action's parameters via **Reflection**
2. For each parameter, it finds the appropriate `IModelBinder`
3. For complex objects, it uses `ComplexObjectModelBinder` which recursively binds each property
4. It calls **type converters** to convert strings → int, DateTime, Guid, etc.
5. It runs **Model Validation** (DataAnnotations) after binding

```csharp
// ViewModel with validation
public class CreateProductViewModel
{
    [Required(ErrorMessage = "Product name is required")]
    [StringLength(100, MinimumLength = 2)]
    public string ProductName { get; set; }

    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }
}

// In Controller — checking if binding + validation succeeded
[HttpPost]
public async Task<IActionResult> Create(CreateProductViewModel model)
{
    if (!ModelState.IsValid)
    // ^^^^ ModelState was populated by model binding
    // If any [Required]/[Range] etc. failed, IsValid = false
    {
        return View(model); // Return the form with errors
    }
    // ... proceed
}
```

---

## PART 8: FILTERS — CROSS-CUTTING CONCERNS

[Filters](Filters.md) run around your action method. Think of them as middleware but **action-aware** — they know which controller and action they're wrapping.

```
Request
  │
  ▼
[Authorization Filter]    ← Can short-circuit (e.g., redirect to login)
  │
  ▼
[Resource Filter]         ← Can short-circuit (e.g., cache hit)
  │
  ▼
Model Binding
  │
  ▼
[Action Filter - Before]  ← Runs before action executes
  │
  ▼
ACTION METHOD EXECUTES
  │
  ▼
[Action Filter - After]   ← Runs after action, has access to result
  │
  ▼
[Exception Filter]        ← Only runs if exception was thrown
  │
  ▼
[Result Filter]           ← Wraps IActionResult execution
  │
  ▼
Response
```

```csharp
// A real-world Action Filter — logging execution time
public class ExecutionTimingFilter : IActionFilter
{
    private Stopwatch _stopwatch;
    private readonly ILogger<ExecutionTimingFilter> _logger;

    public ExecutionTimingFilter(ILogger<ExecutionTimingFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    // ^^^^ Runs BEFORE action method
    {
        _stopwatch = Stopwatch.StartNew();
        _logger.LogInformation(
            "Executing {Controller}.{Action}",
            context.RouteData.Values["controller"],
            context.RouteData.Values["action"]);
    }

    public void OnActionExecuted(ActionExecutedContext context)
    // ^^^^ Runs AFTER action method (even if it threw an exception)
    {
        _stopwatch.Stop();
        _logger.LogInformation(
            "Executed in {ElapsedMs}ms", _stopwatch.ElapsedMilliseconds);
    }
}
```

**Filters vs Middleware — when to use which?**

|Concern|Use Middleware|Use Filters|
|---|---|---|
|Logging all requests|✅|❌|
|Auth for specific actions|❌|✅|
|CORS|✅|❌|
|Caching a specific action|❌|✅|
|Exception handling globally|✅|❌|
|Validation logic per action|❌|✅|

---

## PART 9: THE CONTROLLER

The controller is the **traffic cop** — it receives a request, orchestrates the work, and returns a response. It should have NO business logic.

```csharp
[Authorize]                          // Filter: only authenticated users
[Route("products")]                  
public class ProductController : Controller
{
    // Why private?  Encapsulation — external code shouldn't touch these
    // Why readonly? After construction, dependencies don't change.
    //               Protects against accidental reassignment.
    //               Communicates intent to other developers.
    // Why interface (IProductService, not ProductService)?
    //               Loose coupling. Testability. Swappability.
    private readonly IProductService _productService;
    private readonly ILogger<ProductController> _logger;

    // Constructor injection — the DI container calls this
    // and provides the implementations
    public ProductController(
        IProductService productService,
        ILogger<ProductController> logger)
    {
        _productService = productService;
        _logger = logger;
    }

    [HttpPost("create")]
    public async Task<IActionResult> Create(CreateProductViewModel viewModel)
    {
        if (!ModelState.IsValid)
            return View(viewModel);

        // Map ViewModel → Domain DTO (don't pass ViewModels into services)
        // WHY? ViewModels are UI concerns. Services shouldn't know about UI.
        var dto = new CreateProductDto
        {
            Name = viewModel.ProductName,
            Price = viewModel.Price,
            CategoryId = viewModel.CategoryId
        };

        // Delegate ALL business logic to the service
        var result = await _productService.CreateProductAsync(dto);

        _logger.LogInformation("Product created with ID {ProductId}", result.Id);

        return RedirectToAction("Details", new { id = result.Id });
    }
}
```

**Why `async Task<IActionResult>`?**

- `async` — enables `await` inside the method
- `Task<>` — the method returns a promise; the thread is released while waiting for DB
- `IActionResult` — a polymorphic return type: can be `View()`, `Json()`, `RedirectToAction()`, `NotFound()`, `BadRequest()`, etc.

**If you used `IActionResult` without `async`:** You'd block a thread while the database query runs. Under load, you'd exhaust the thread pool. Your app would queue requests or crash. This is why async is not optional in production.

---

## PART 10: THE SERVICE LAYER — WHERE BUSINESS LOGIC LIVES

```csharp
// Interface in Application layer
public interface IProductService
{
    Task<ProductDto> CreateProductAsync(CreateProductDto dto);
}

// Implementation
public class ProductService : IProductService
{
    private readonly IProductRepository _productRepository;
    private readonly ICategoryRepository _categoryRepository;
    private readonly IUnitOfWork _unitOfWork;
    private readonly ILogger<ProductService> _logger;

    public ProductService(
        IProductRepository productRepository,
        ICategoryRepository categoryRepository,
        IUnitOfWork unitOfWork,
        ILogger<ProductService> logger)
    {
        _productRepository = productRepository;
        _categoryRepository = categoryRepository;
        _unitOfWork = unitOfWork;
        _logger = logger;
    }

    public async Task<ProductDto> CreateProductAsync(CreateProductDto dto)
    {
        // ══ BUSINESS RULES LIVE HERE ══

        // 1. Validate business rules (not just data validation — actual rules)
        var category = await _categoryRepository.GetByIdAsync(dto.CategoryId);
        if (category == null)
            throw new BusinessException($"Category {dto.CategoryId} does not exist");

        if (category.IsDiscontinued)
            throw new BusinessException("Cannot add products to discontinued categories");

        // 2. Check for duplicate products
        var existing = await _productRepository
            .GetByNameAndCategoryAsync(dto.Name, dto.CategoryId);
        if (existing != null)
            throw new DuplicateProductException(dto.Name);

        // 3. Create domain entity
        var product = new Product
        {
            Name = dto.Name,
            Price = dto.Price,
            CategoryId = dto.CategoryId,
            CreatedAt = DateTime.UtcNow,
            // Note: DateTime.UtcNow not DateTime.Now
            // UTC avoids timezone bugs in international apps
        };

        // 4. Persist
        await _productRepository.AddAsync(product);
        await _unitOfWork.SaveChangesAsync();
        // ^^^^ ONE save call — atomic operation
        //      Either everything saves or nothing does, kinda like transactions

        // 5. Return DTO (not domain entity — never expose entities to upper layers)
        return new ProductDto { Id = product.Id, Name = product.Name };
    }
}
```

**Why not just call the repository directly from the controller?**

If you call the repository from the controller:
- Business rules get scattered across controllers
- Multiple controllers might implement the same rule differently
- Testing requires setting up HTTP context
- You can't reuse the logic in a background job, a CLI, or an API endpoint

The service layer is **reusable, testable, and owns the rules**.

---

## PART 11: THE REPOSITORY PATTERN

```csharp
// Why this pattern exists:
// EF Core is a powerful ORM. But if you write EF queries directly in services:
// - Your services are tightly coupled to EF Core
// - Unit testing services requires a real database or complex mocking
// - Switching ORMs (rare but happens) means rewriting services
// - LINQ queries get duplicated across multiple services

public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<Product?> GetByNameAndCategoryAsync(string name, int categoryId);
    Task AddAsync(Product product);
    Task<IEnumerable<Product>> GetAllByCategoryAsync(int categoryId);
}

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;
    // ^^^^ Only the repository knows about DbContext
    //      Nothing above this layer touches EF Core

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products
            .Include(p => p.Category)      // Eager load related data
            .FirstOrDefaultAsync(p => p.Id == id);
            // ^^^^ Returns null if not found (safer than .First() which throws error)
    }

    public async Task<Product?> GetByNameAndCategoryAsync(string name, int categoryId)
    {
        return await _context.Products
            .AsNoTracking()
            // ^^^^ CRITICAL: For read-only queries, don't track the entity
            // EF Core's change tracker adds overhead
            // When you only READ data and won't save changes, AsNoTracking()
            // is 30-40% faster and uses less memory
            .FirstOrDefaultAsync(p =>
                p.Name == name &&
                p.CategoryId == categoryId);
    }

    public async Task AddAsync(Product product)
    {
        await _context.Products.AddAsync(product);
        // Note: This does NOT hit the database yet
        // It just marks the entity as "Added" in EF Core's change tracker
        // The actual INSERT happens when SaveChangesAsync() is called
    }
}
```

---

## PART 12: ENTITY FRAMEWORK CORE — HOW IT REALLY WORKS

```csharp
public class AppDbContext : DbContext
{
	// 'base(options)' calls the parent constructor.
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Fluent API configuration — preferred over DataAnnotations on entities
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(p => p.Id);
            entity.Property(p => p.Name)
                  .IsRequired()
                  .HasMaxLength(100);
            entity.Property(p => p.Price)
                  .HasColumnType("decimal(18,2)");
            entity.HasOne(p => p.Category)
                  .WithMany(c => c.Products)
                  .HasForeignKey(p => p.CategoryId)
                  .OnDelete(DeleteBehavior.Restrict);
                  // ^^^^ Don't cascade delete products when category is deleted
        });
    }
}
```

**What happens when `SaveChangesAsync()` is called:**

1. EF Core's **Change Tracker** inspects all tracked entities
2. For each entity with state `Added` → generates `INSERT` SQL
3. For each entity with state `Modified` → generates `UPDATE` SQL (only changed columns)
4. For each entity with state `Deleted` → generates `DELETE` SQL
5. Wraps everything in a **database transaction**
6. Sends the SQL to the database
7. Updates entity primary keys with database-generated values

```sql
-- The INSERT EF Core generates for our product:
INSERT INTO [Products] ([Name], [Price], [CategoryId], [CreatedAt])
VALUES (@p0, @p1, @p2, @p3);
SELECT [Id]  -- Gets the auto-generated ID back
FROM [Products]
WHERE @@ROWCOUNT = 1 AND [Id] = scope_identity();
```

---

## PART 13: THE RESPONSE JOURNEY BACK UP

The database returns data. Now it travels back up through the layers:

```
Database → EF Core materializes entities → Repository returns entities
→ Service maps to DTOs → Controller receives DTOs → Controller creates ViewModel
→ Passes ViewModel to View → Razor Engine renders HTML → Response sent to browser
```

### ViewModels vs DTOs vs Entities — The Distinction Matters

```csharp
// Domain Entity — what EF Core works with, maps to database table
// Never expose this to the UI or API clients
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    public Category Category { get; set; }
    public bool IsActive { get; set; }
    public string InternalNotes { get; set; } // sensitive — never expose!
}

// DTO (Data Transfer Object) — crosses layer boundaries
// Used between Service and Controller
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string CategoryName { get; set; }
}

// ViewModel — specific to one View, can have UI-specific properties
// This is what the Razor view receives
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string FormattedPrice { get; set; }  // "$999.99" — UI formatting
    public string CategoryName { get; set; }
    public bool CanEdit { get; set; }           // UI state
    public bool CanDelete { get; set; }         // UI state
    public string BreadcrumbPath { get; set; }  // UI navigation
}
```

---

## PART 14: RAZOR VIEW ENGINE, TAG HELPERS, VIEW COMPONENTS

```csharp
// The Razor View — a .cshtml file
// It's C# + HTML. Razor compiles this into a C# class at build/runtime.

@model ProductDetailsViewModel
// ^^^^ Declares the type of the ViewModel this view expects, you need to declare this otherwise it will give some dynamic error.
//      Now @Model is strongly typed

<h1>@Model.Name</h1>
<p>Price: @Model.FormattedPrice</p>

@* Tag Helper — generates <a href="/products/edit/5"> *@
<a asp-controller="Product" asp-action="Edit" asp-route-id="@Model.Id"
   class="btn btn-primary">Edit</a>

@* View Component — a self-contained, reusable component with its own logic *@
@await Component.InvokeAsync("ShoppingCart")
@* This calls ShoppingCartViewComponent.InvokeAsync()
   which has its own dependencies injected, queries its own data,
   returns its own view. It's like a mini-controller inside a view. *@
```

### View Components — Why They Exist

```csharp
// Problem: You need a "Shopping Cart Summary" widget that appears on EVERY page.
// It needs to query the database for the current user's cart.
// You can't do this in every controller action — it's repeated code.
// You can't do this in the layout — layouts don't have C# logic easily.
// Solution: ViewComponent

public class ShoppingCartViewComponent : ViewComponent
{
    private readonly ICartService _cartService;

    public ShoppingCartViewComponent(ICartService cartService)
    // ^^^^ DI works here too! ViewComponents participate in DI.
    {
        _cartService = cartService;
    }

    public async Task<IViewComponentResult> InvokeAsync()
    {
        var userId = UserClaimsPrincipal.FindFirstValue(ClaimTypes.NameIdentifier);
        var cart = await _cartService.GetCartSummaryAsync(userId);

        return View(cart); // Renders Views/Shared/Components/ShoppingCart/Default.cshtml
    }
}
```

---

## PART 15: DEPENDENCY INJECTION — THE GLUE HOLDING EVERYTHING TOGETHER

How does `ProductController` GET an `IProductService`? How does `ProductService` GET an `IProductRepository`? **The DI Container.**

```csharp
// Registration in Program.cs
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<ICategoryRepository, CategoryRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// AddScoped means: ONE instance per HTTP request
// The SAME instance is shared across everything within that request
// This is critical for EF Core — you want ONE DbContext per request
```

**The Service Lifetime rules — this is a common source of bugs:**

```
Singleton: ONE instance for the entire app lifetime
  │ App starts → instance created → lives until app shuts down
  │ Shared across ALL requests, ALL threads
  │ MUST be thread-safe

Scoped: ONE instance per HTTP request
  │ Request starts → instance created → lives until request ends
  │ Shared within a single request
  │ This is the default for services, repositories, DbContext

Transient: NEW instance every time it's requested from DI
  │ Lightweight, stateless services
  │ Every injection point gets a fresh instance
```

**The Captive Dependency Bug — a production nightmare:**

```csharp
// ❌ WRONG: Singleton holding a Scoped dependency
public class MySingletonService
{
    private readonly IProductRepository _repo;
    // IProductRepository is Scoped, but MySingletonService is Singleton
    // The Singleton was created ONCE and captured the Scoped instance
    // After the first request ends, the Scoped instance is DISPOSED
    // But the Singleton still holds a reference to it
    // Second request: BOOM — ObjectDisposedException

    public MySingletonService(IProductRepository repo)
    {
        _repo = repo; // Captured forever — the scoped instance is now a zombie
    }
}

// ASP.NET Core will actually throw at startup:
// InvalidOperationException: Cannot consume scoped service 'IProductRepository'
// from singleton 'MySingletonService'
```

---

## THE COMPLETE FLOW — PUTTING IT ALL TOGETHER

```
1. User clicks "Save Product" button
   └── Browser sends: POST /products/create + form data

2. Kestrel receives TCP bytes
   └── Parses HTTP, creates HttpContext

3. Middleware Pipeline (request phase):
   └── ExceptionHandler wraps everything
   └── HttpsRedirection (if HTTP → redirect to HTTPS)
   └── StaticFiles (not a static file, pass through)
   └── Routing: matches /products/create → ProductController.Create
   └── Authentication: reads JWT/Cookie, sets HttpContext.User
   └── Authorization: checks [Authorize] on controller/action

4. Model Binding:
   └── Reads form body
   └── Maps to CreateProductViewModel
   └── Runs DataAnnotations validation
   └── Populates ModelState

5. Action Filters (OnActionExecuting):
   └── Logging filter starts stopwatch
   └── (Any custom filters run here)

6. ProductController.Create() executes:
   └── Checks ModelState.IsValid
   └── Maps ViewModel → CreateProductDto
   └── Calls _productService.CreateProductAsync(dto)

7. ProductService.CreateProductAsync():
   └── Validates business rules
   └── Calls _categoryRepository.GetByIdAsync()
   └── Calls _productRepository.AddAsync()
   └── Calls _unitOfWork.SaveChangesAsync()

8. EF Core SaveChangesAsync():
   └── Change Tracker generates INSERT SQL
   └── Opens transaction
   └── Executes SQL against database
   └── Commits transaction
   └── Returns generated ID

9. Response travels back up:
   └── Service returns ProductDto
   └── Controller calls RedirectToAction("Details", new { id = result.Id })

10. Action Filters (OnActionExecuted):
    └── Logging filter records elapsed time

11. Middleware Pipeline (response phase):
    └── Each middleware gets to process the response on the way out
    └── ExceptionHandler (no exception, passes through)

12. Kestrel sends HTTP 302 Redirect response to browser

13. Browser follows redirect to /products/details/42

14. New request begins for the Details page...
    └── Controller gets product from service
    └── Maps to ProductDetailsViewModel
    └── Returns View(viewModel)
    └── Razor renders HTML with Tag Helpers, View Components
    └── HTML sent to browser
```

---

## INTERVIEW QUESTIONS YOU SHOULD BE ABLE TO ANSWER

1. Why is middleware order important? What breaks if `UseAuthentication` comes after `UseAuthorization`?
2. What is the difference between a Filter and Middleware?
3. Why should Controllers not contain business logic?
4. What is the difference between `AsNoTracking()` and tracked queries?
5. Explain the captive dependency problem with DI lifetimes
6. Why do we map entities to DTOs at the service boundary?
7. What does `ModelState.IsValid` actually check?
8. Why is `async/await` important in controller actions?

---

## CROSS-QUESTIONS

**Question 1:** If I register `AppDbContext` as **Singleton** instead of Scoped (which happens automatically with `AddDbContext`), what would go wrong in a multi-user web application?

**Question 2:** A junior developer says: _"Why do we need a Service layer? I'll just call the Repository directly from the Controller — it's less code."_ How do you convince them this is wrong? Give me a concrete scenario where skipping the service layer causes a real problem.

**Question 3:** Look at this code:

```csharp
public async Task<IActionResult> Create(CreateProductViewModel model)
{
    var product = new Product { Name = model.ProductName };
    await _context.Products.AddAsync(product);
    await _context.SaveChangesAsync();
    return RedirectToAction("Index");
}
```

I can see **at least 4 architectural violations** here. Can you spot them?

**Question 4:** What is the difference between a **ViewModel** and a **DTO**? Can they be the same class? Should they be?

**Question 5 (Scenario):** You have a `[Authorize]` attribute on a Controller. A request comes in with an **expired JWT token**. Walk me through exactly what happens — which middleware fires, in what order, and what response the user gets.

---

## YOUR MINI ASSIGNMENT

Build mentally (or on paper) the **complete layer structure** for this scenario:

> A user submits a form to **transfer money** from Account A to Account B.

Define:

- What the ViewModel looks like
- What the Controller action does
- What business rules the Service must enforce
- What Repository methods are needed
- What EF Core operations happen
- What should happen if the database fails mid-transaction
- What the response is on success and on failure