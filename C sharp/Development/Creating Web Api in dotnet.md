You can create web api in dotnet which can serve requests to your frontend. Steps to do so:

#### 1. Setup

**1. Create the Project**
```bash
dotnet new webapi -n BlogApi
cd MyWebApi
```
`BlogApi` can be any name of your choice.
To open in Visual Studio, `start devenv BlogApi.csproj`, In VS code `code .`

By only doing the above you will have a basic api at some endpoint which you can get after `dotnet run`. 

**2. Install NuGet packages**
```bash
# Entity Framework Core + SQL Server
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools

# JWT Authentication
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer

# AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection

# Swagger
dotnet add package Swashbuckle.AspNetCore

# Password hashing
dotnet add package BCrypt.Net-Next
```

Our folder structure should something like this:
```text
BlogApi/
├── Controllers/
│   ├── AuthController.cs
│   ├── PostsController.cs
│   └── CommentsController.cs
├── Models/          # EF Core entities
│   ├── User.cs
│   ├── Post.cs
│   └── Comment.cs
├── DTOs/            # Request/Response shapes
│   ├── Auth/
│   ├── Posts/
│   └── Comments/
├── Data/
│   └── AppDbContext.cs
├── Services/
│   ├── IAuthService.cs
│   └── AuthService.cs
├── Mappings/
│   └── MappingProfile.cs
├── Middleware/
│   └── ExceptionMiddleware.cs
└── Program.cs
```

#### 2. models and DB
If you want to serve some specific data you can create model in a `/Models` folder. Say we are creating `User.cs`
```csharp
// '/Models/User.cs'

using System.ComponentModel.DataAnnotations;

namespace BlogApi.Models;

public class User
{
    public int Id { get; set; }

    [Required]
    [MaxLength(50)]
    public string Username { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Required]
    public string PasswordHash { get; set; } = string.Empty;

    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;

    // Navigation property for related posts
    public ICollection<Post> Posts { get; set; } = new List<Post>();
}
```
One thing to make sure is that `Post` should exist in your model folder for migration to work above. In case you have some circular dependency you can migrate them one by one.

 Create a context.
 ```csharp
 // '/Models/ProductContext.cs'
 
using BlogApi.Models;
using Microsoft.EntityFrameworkCore;

namespace BlogApi.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }

    public DbSet<User> Users => Set<User>();

    public DbSet<Post> Posts => Set<Post>();

    public DbSet<Comment> Comments => Set<Comment>();
	
	// Below we are creating indexes
    protected override void OnModelCreating(ModelBuilder builder)
    {
        // Unique email
        builder.Entity<User>().HasIndex(user => user.Email).IsUnique();

        // Unique username
        builder.Entity<User>().HasIndex(user => user.Username).IsUnique();
    }
}
 ```

Add `ConnectionStrings__DefaultConnection` to your `appsettings.json` and run migrations.

After adding `ConnectionStrings__DefaultConnection`, we can register `DbContext` to our `Program.cs`.
```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration
               .GetConnectionString("DefaultConnection")));
```

Create and apply migrations:
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

#### 3. DTOs & AutoMapper
DTOs (Data Transfer Objects) are simple objects used to pass data between the client and the server or between different layers of an application. They act as a 'contract' that defines exactly what data is being sent over the network, separate from the complex business logic or database models. In short you don't have to expose your entire models to the client and your client can just interact with DTOs only.

Sample DTOs for Auth:
```csharp
using System.ComponentModel.DataAnnotations;

namespace BlogApi.DTOs;

public class RegisterDto
{
    [Required]
    [MaxLength(50)]
    public string Username { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Required]
    [MinLength(6)]
    public string Password { get; set; } = "";
}

// DTOs/Auth/LoginDto.cs
public class LoginDto
{
    [Required]
    public string Email { get; set; } = "";

    [Required]
    public string Password { get; set; } = "";
}

// DTOs/Auth/AuthResponseDto.cs
public class AuthResponseDto
{
    public string Token { get; set; } = "";
    public string Username { get; set; } = "";
    public string Email { get; set; } = "";
}
```
Note that you have applied Validations on DTO.

Create mapping or `automapper`. `AutoMapper` will basically convert one dto to another dto or to a model and vice-versa. In below conversions, `CreateMap<A, B>()` `A` is converted to `B`.

```csharp
using AutoMapper;
using BlogApi.DTOs;
using BlogApi.Models;
using Microsoft.Data.SqlClient;

namespace BlogApi.Mappings;

// 'Profile' is an AutoMapper class used to define mappings.
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        // User maps
        CreateMap<RegisterDto, User>();
        CreateMap<User, AuthResponseDto>();

        // Post maps
        CreateMap<CreatePostDto, Post>();
        CreateMap<Post, PostResponseDto>()
            .ForMember(dest => dest.AuthorUsername, opt => opt.MapFrom(src => src.User.Username))
            .ForMember(dest => dest.CommentCount, opt => opt.MapFrom(src => src.Comments.Count));

        CreateMap<CreateCommentDto, Comment>();
        CreateMap<Comment, CommentResponseDto>()
            .ForMember(dest => dest.AuthorUsername, opt => opt.MapFrom(src => src.User.Username));
    }
}

```

#### 4. CRUD controllers
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
dotnet aspnet-codegenerator controller -name ProductController -async -api -m Product -dc ProdcutContext -outDir Controllers
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

You can also create your own custom controller with things like authorization and custom return type.
```c#
using System.Security.Claims;
using AutoMapper;
using BlogApi.Data;
using BlogApi.DTOs;
using BlogApi.Mappings;
using BlogApi.Models;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace BlogApi.Controllers;

[ApiController]
[Route("api/[controller]")] // api/posts
public class PostsController : ControllerBase
{
    // ControllerBase -> no views, only JSON responses

    // Lets us access the database
    private readonly AppDbContext _db;

    // converts between entity models and DTOs
    private readonly IMapper _mapper;

    public PostsController(AppDbContext db, IMapper mapper)
    {
        _db = db;
        _mapper = mapper;
    }

    // Get api/posts
    [HttpGet]
    public async Task<ActionResult<List<PostResponseDto>>> GetAll()
    {
        // Fetch all post and include the user and comments, than sort by 'CreatedAt' in ascending order.
        var posts = await _db
            .Posts.Include(p => p.User)
            .Include(p => p.Comments)
            .OrderByDescending(p => p.CreatedAt)
            .ToListAsync();

        // Convert 'posts' to List<dto> ('PostResponseDto')
        return Ok(_mapper.Map<List<PostResponseDto>>(posts));
    }

    // Get api/posts/5
    [HttpGet("{id}")]
    public async Task<ActionResult<PostResponseDto>> GetById(int id)
    {
        // Find first post with given ID.
        var post = await _db
            .Posts.Include(p => p.User)
            .Include(p => p.Comments)
            .FirstOrDefaultAsync(p => p.Id == id);

        // Returns 404 (not found)
        if (post == null)
            return NotFound();

        // Converts 'post' to dto ('PostResponseDto')
        return Ok(_mapper.Map<PostResponseDto>(post));
    }

    // Post api/posts [Authorize]
    [HttpPost]
    [Authorize]
    public async Task<ActionResult<PostResponseDto>> Create(CreatePostDto dto)
    {
        // Extract logged-in user ID from JWT token
        var userId = int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);

        // converts incoming dto to post
        var post = _mapper.Map<Post>(dto);
        // assigns current user as owner
        post.UserId = userId;

        // adds post to database
        _db.Posts.Add(post);
        await _db.SaveChangesAsync();

        // explicitly loads related user data
        await _db.Entry(post).Reference(p => p.User).LoadAsync();

        // returns 201 (created)
        // includes location of new resource
        // created post data as 'PostResponseDto'
        return CreatedAtAction(
            nameof(GetById),
            new { id = post.Id },
            _mapper.Map<PostResponseDto>(post)
        );
    }

    // Put api/posts/5 [Authorize]
    [HttpPut("{id}")]
    [Authorize]
    public async Task<IActionResult> Update(int id, UpdatePostDto dto)
    {
        var userId = int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);

        var post = await _db.Posts.FindAsync(id);

        if (post == null)
            return NotFound();

        // Only the user that created this post can modify this, else forbid 403
        if (post.UserId != userId)
            return Forbid();

        if (dto.Title != null)
            post.Title = dto.Title;
        if (dto.Content != null)
            post.Content = dto.Content;

        post.UpdatedAt = DateTime.UtcNow;

        await _db.SaveChangesAsync();
        return NoContent();
    }

    // Delete api/posts/5 [Authorize]
    [HttpDelete("{id}")]
    [Authorize]
    public async Task<IActionResult> Delete(int id)
    {
        var userId = int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);

        var post = await _db.Posts.FindAsync(id);
        if (post == null)
            return NotFound();
        if (post.UserId != userId)
            return Forbid();

        _db.Posts.Remove(post);
        await _db.SaveChangesAsync();
        return NoContent();
    }
}
```

Important things to know about above code:
##### • `var userId = int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);`:
Above line retrieves the currently logged-in user's ID from the JWT/authentication claims and converts it to an integer. If you have added more than one claim to your JWT (will add in later step), you can also access them.
Say you have added `Username` and `Email`, you can access them using:
```c#
new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
new Claim(ClaimTypes.Name, user.Username),
new Claim(ClaimTypes.Email, user.Email),
```

##### • `ActionResult` VS `IActionResult`:
##### • `Controller` VS `ControllerBase`:

### 5. JWT auth
First we will create services which will handle authentication work (register and login).
```c#
// Services/IAuthService.cs
using BlogApi.DTOs;

namespace BlogApi.Services;

public interface IAuthService
{
    Task<AuthResponseDto> Register(RegisterDto dto);
    Task<AuthResponseDto> Login(LoginDto dto);
}
```

```c#
// Services/AuthService.cs
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Security.Cryptography;
using System.Text;
using AutoMapper;
using BlogApi.Data;
using BlogApi.DTOs;
using BlogApi.Models;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;

namespace BlogApi.Services;

public class AuthService : IAuthService
{
	// A connection to database
    private readonly AppDbContext _db;
    // A way to read appsettings
    private readonly IConfiguration _config;
    // AutoMapper
    private readonly IMapper _mapper;

    public AuthService(AppDbContext db, IConfiguration config, IMapper mapper)
    {
        _db = db;
        _config = config;
        _mapper = mapper;
    }

    public async Task<AuthResponseDto> Register(RegisterDto dto)
    {
        // checks if user already exists
        if (await _db.Users.AnyAsync(u => u.Email == dto.Email))
            throw new InvalidOperationException("User already exists.");

        // map dto to user entity
        var user = _mapper.Map<User>(dto);
        // hash the password
        user.PasswordHash = BCrypt.Net.BCrypt.HashPassword(dto.Password);

        // save user to database
        _db.Users.Add(user);
        await _db.SaveChangesAsync();

        // return response with jwt token
        var response = _mapper.Map<AuthResponseDto>(user);
        response.Token = GenerateToken(user);
        return response;
    }

    public async Task<AuthResponseDto> Login(LoginDto dto)
    {
        // find user by email, if not found, throw error
        var user =
            await _db.Users.FirstOrDefaultAsync(u => u.Email == dto.Email)
            ?? throw new UnauthorizedAccessException("Invalid credentials");

        // matches hashed password and the password given to us now. done by BCrypt
        if (!BCrypt.Net.BCrypt.Verify(dto.Password, user.PasswordHash))
            throw new UnauthorizedAccessException("Invalid credentials");

        // if valid, generate token and return response
        var response = _mapper.Map<AuthResponseDto>(user);
        response.Token = GenerateToken(user);
        return response;
    }

    private string GenerateToken(User user)
    {
        // get secret key from config.
        var secret = _config["JwtSettings:Secret"];
        // create signing key
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secret!));

        // create claims, user info inside token.
        // these claims are stored inside the jwt, and can be used for authorization.
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Email, user.Email),
        };

        // create token
        var token = new JwtSecurityToken(
            issuer: _config["JwtSettings:Issuer"],
            audience: _config["JwtSettings:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(int.Parse(_config["JwtSettings:ExpiryMinutes"]!)),
            signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256)
        );

        // convert token to string
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

In general cases your Services folder will contain your business logic. The main ideology is to keep your controllers thin and pass down the work to services and repositories.

Add controller that will use the above `AuthService`.
```c#
// Controller/AuthController
using BlogApi.DTOs;
using BlogApi.Services;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;

    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }

    [HttpPost("register")]
    public async Task<ActionResult<AuthResponseDto>> Register(RegisterDto dto)
    {
        var result = await _authService.Register(dto);

        return Ok(result);
    }

    [HttpPost("login")]
    public async Task<ActionResult<AuthResponseDto>> Login(LoginDto dto)
    {
        var result = await _authService.Login(dto);

        return Ok(result);
    }
}
```

Somethings to know about above controller:
##### • Why inject interface only and not the implementation:
You inject the interface instead of the implementation class because of:
- abstraction
- loose coupling
- easier testing
- flexibility
- maintainability
If you inject the implementation directly, your controller becomes tightly coupled to that exact class. Controller depends on a specific implementation. The controller only cares about `I need something that behaves like a post service`. 
It is also easy to swap implementation, you just have to change your `Program.cs`
```csharp
AddScoped<IPostService, PostService>();
```
to 
```csharp
AddScoped<IPostService, CachedPostService>();
```

##### • Why `private readonly`?
`private` means that only this class can access it. `readonly` means that value can be assigned once. Value assignment is usually done in constructor, this prevents accidental reassignment after constructor finishes.
Another reason for `readonly` is that dependencies should not change while controller is running.

Now you need to add JWT auth in your `Program.cs`
Use dotnet 9, 10 have many issues regarding Swagger.
```c#
builder.Services.AddScoped<IAuthService, AuthService>();

builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters =
            new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration[
                "JwtSettings:Issuer"],
            ValidAudience = builder.Configuration[
                "JwtSettings:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(
                    builder.Configuration[
                        "JwtSettings:Secret"]!))
        };
    });

// In the middleware pipeline (order matters!)
app.UseAuthentication();   // before
app.UseAuthorization();    // after
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