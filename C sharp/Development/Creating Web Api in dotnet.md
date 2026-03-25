You can create web api in dotnet which can serve requests to your frontend. Steps to do so:

#### 1. Create Project
```bash
dotnet new webapi -n MyWebApi
cd MyWebApi
```
`MyWebApi` can be any name of your choice.
To open in Visual Studio, `start devenv MyWebApi.csproj`, In VS code `code .`

By only doing the above you will have a basic api at some endpoint which you can get after `dotnet run`. 

Your `program.cs` will look like below:
```csharp
using Microsoft.EntityFrameworkCore;
using MyWebApi.Models;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();    

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.UseHttpsRedirection(); 

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};    

app.MapGet("/weatherforecast", () =>
{
    var forecast =  Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
        
    return forecast;
})
.WithName("GetWeatherForecast");

app.Run();  

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```
Above we have `/weatherforecast` route which we can append to our `localhost:<port>` and you a web api running.

#### 2. Create models and model context
If you want to serve some specific data you can create model in a `/Models` folder. Say we are creating `Product.cs`
```csharp
// '/Models/Product.cs'

namespace MyWebApi.Models;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```
 Create a context.
 ```csharp
 // '/Models/ProductContext.cs'
 
 using Microsoft.EntityFrameworkCore;  

namespace MyWebApi.Models
{
    public class ProductContext : DbContext
    {
        public ProductContext(DbContextOptions<ProductContext> options)
        : base(options)
        {
        }

        // Your 'Product' model.
        public DbSet<Product> Products { get; set; } = null!;
    }
}
 ```

#### 3. Create Controller
You can scaffold your controller using steps that are according to either visual studio (`/controllers -> right click -> new Scaffold item`) or in vs code use below commands:
```bash
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet tool uninstall -g dotnet-aspnet-codegenerator
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet tool update -g dotnet-aspnet-codegenerator
```
Above commands will add all the necessary packages.

Now you can scaffold the controller using the following command:
```bash
dotnet aspnet-codegenerator controller -name ProductsController -async -api -m Product -dc ProdcutContext -outDir Controllers
```

The created controller will look something like follows:
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using MyWebApi.Models; 

namespace MyWebApi.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ProductsController : ControllerBase
    {
        private readonly ProductContext _context;  

        public ProductsController(ProductContext context)
        {
            _context = context;
        } 

        // GET: api/Products
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Product>>> GetProducts()
        {
            return await _context.Products.ToListAsync();
        } 

        // GET: api/Products/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Product>> GetProduct(int id)
        {
            var product = await _context.Products.FindAsync(id);

            if (product == null)
            {
                return NotFound();
            } 

            return product;
        }

        // PUT: api/Products/5
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPut("{id}")]
        public async Task<IActionResult> PutProduct(int id, Product product)
        {
            if (id != product.Id)
            {
                return BadRequest();
            } 

            _context.Entry(product).State = EntityState.Modified; 

            try
            {
                await _context.SaveChangesAsync();
            }
            catch (DbUpdateConcurrencyException)
            {
                if (!ProductExists(id))
                {
                    return NotFound();
                }
                else
                {
                    throw;
                }
            }  

            return NoContent();
        }  

        // POST: api/Products
        // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
        [HttpPost]
        public async Task<ActionResult<Product>> PostProduct(Product product)
        {
            _context.Products.Add(product);
            await _context.SaveChangesAsync();

            // return CreatedAtAction("GetProduct", new { id = product.Id }, product);
            return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
        }

        // DELETE: api/Products/5
        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteProduct(int id)
        {
            var product = await _context.Products.FindAsync(id);
            if (product == null)
            {
                return NotFound();
            } 

            _context.Products.Remove(product);
            await _context.SaveChangesAsync();  

            return NoContent();
        }  

        private bool ProductExists(int id)
        {
            return _context.Products.Any(e => e.Id == id);
        }
    }
}
```

#### 4. Update your `Program.cs`
Now you have to add `Swagger` for testing of your api in your `Program.cs` file. It should something like this:
```csharp
using Microsoft.EntityFrameworkCore;
using MyWebApi.Models;  

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

// Learn more about configuring OpenAPI at https://aka.ms/aspnet/openapi
builder.Services.AddOpenApi(); 

builder.Services.AddControllers(); 

builder.Services.AddEndpointsApiExplorer();

builder.Services.AddSwaggerGen(); 

// Using in memory database
builder.Services.AddDbContext<ProductContext>(opt => opt.UseInMemoryDatabase("Products")); 

var app = builder.Build();  

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseSwagger();
app.UseSwaggerUI(); 

app.UseHttpsRedirection();  

app.MapControllers();
app.Run();
```

#### 5. Run your program using `dotnet run`
Now, you can simply do `dotnet run` and you will get a url `localhost:<port>` in your terminal. You can append `/swagger` to visualize your api.

#### Short notes
1. Create project
2. Create models and context
3. Create controller
4. Update `Program.cs`
5. Run your app

[Connecting Razor Pages to web Api](Connecting%20Razor%20Pages%20to%20web%20Api.md)