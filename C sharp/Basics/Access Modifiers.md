There are **six** access levels in modern C#.

| Modifier             | Accessible from                                                 | Who can see it                             | Most common use case                                                      | Real-world frequency        |
| -------------------- | --------------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------- | --------------------------- |
| `public`             | Everywhere                                                      | Any code in any assembly                   | Public API, interfaces, base classes                                      | Very high                   |
| `protected`          | Same class + derived classes                                    | Only inside the class and its children     | Template method pattern, customizable behavior                            | Medium                      |
| `internal`           | Same assembly                                                   | Any code inside the same .dll / .exe       | "Friend" classes, internal utilities                                      | High (especially libraries) |
| `private`            | Only the containing type                                        | Only code inside the **same class/struct** | Implementation details, private fields/methods                            | Very high                   |
| `protected internal` | Same assembly **OR** derived classes (even in other assemblies) | Combination of internal + protected        | When you want to allow inheritance but keep tight control inside assembly | Medium-low                  |
| `private protected`  | Same class **OR** derived classes **in the same assembly**      | Very restrictive protected                 | When derived classes in the same assembly need access, but not outsiders  | Low                         |
| `file` (C# 11+)      | Only within the same **.cs file**                               | Extremely narrow — file-scoped types only  | Top-level statements helper types, mini-DTOs                              | Growing (newer feature)     |

### Quick visibility cheat-sheet (from most → least visible)

```text
public
    ↓
protected internal
    ↓
internal
    ↓
protected
    ↓
private protected
    ↓
private
    ↓
file           ← only C# 11+
```

### Concrete examples – all important cases

```csharp
// File: Shapes.cs
file class FileLocalHelper     // C# 11+ — visible ONLY in this file
{
    public int SecretNumber = 42;
}

public class Shape
{
    public int PublicId = 1;                    // everyone
    protected int ProtectedThickness = 2;       // this class + children
    internal int InternalLayer = 3;             // same assembly
    private int PrivateColor = 4;               // only this class

    protected internal int ProtectedOrInternal = 5;   // same assembly OR derived (any assembly)
    private protected int PrivateProtectedData = 6;   // this class OR derived classes in same assembly

    private void PrivateMethod() { }
    protected virtual void DoSomething() { }
}

public class Circle : Shape
{
    void Test()
    {
        // All of these compile:
        Console.WriteLine(PublicId);
        Console.WriteLine(ProtectedThickness);       // because we inherit
        Console.WriteLine(InternalLayer);            // same assembly
        Console.WriteLine(ProtectedOrInternal);

        // Console.WriteLine(PrivateColor);          // ERROR - private
        // Console.WriteLine(PrivateProtectedData);  // OK only because Circle and Shape are in same assembly

        // FileLocalHelper f = new();                // ERROR - file-local type
    }
}

// Different assembly (imagine this is in AnotherProject.dll)
public class ExternalSquare : Shape
{
    void Test()
    {
        Console.WriteLine(PublicId);                 // OK
        Console.WriteLine(ProtectedThickness);       // OK - inheritance
        Console.WriteLine(ProtectedOrInternal);      // OK - protected internal

        // Console.WriteLine(InternalLayer);         // ERROR - internal
        // Console.WriteLine(PrivateProtectedData);  // ERROR - private protected
    }
}
```

### Very common real-world patterns (2024–2025 style)

```csharp
// Library author perspective (most common today)

public abstract class RepositoryBase
{
    private readonly DbContext _context;           // hidden

    protected RepositoryBase(DbContext context)    // children can call this ctor
    {
        _context = context ?? throw new ArgumentNullException();
    }

    protected DbContext Context => _context;       // children can use it

    public abstract Task SaveChangesAsync();
}

internal class SqlServerUserRepository : RepositoryBase   // internal → consumers can't create directly
{
    public SqlServerUserRepository(DbContext ctx) : base(ctx) { }

    public override Task SaveChangesAsync() => Context.SaveChangesAsync();
}

// Public factory – clean public API
public static class Repositories
{
    public static IUserRepository CreateUserRepository(DbContext ctx)
        => new SqlServerUserRepository(ctx);           // returns interface
}
```

### Quick cheat-sheet – what people most often get wrong

| You wrote this                              | What actually happens                          | Correct action if wrong                     |
|---------------------------------------------|------------------------------------------------|---------------------------------------------|
| `protected private`                         | Means **private protected** (order doesn't matter) | Write `private protected` for clarity       |
| `internal protected`                        | Means **protected internal**                   | Same as above                               |
| `private` field in `sealed` class           | Still private — no benefit                     | Usually use `private readonly` instead      |
| No modifier on top-level class              | `internal` by default                          | Explicitly write `public` or `internal`     |
| No modifier on class member                 | `private` by default                           | Be explicit in team code                    |
| `file class` inside namespace               | Allowed, but still file-scoped                 | Usually used at file top-level              |

### One-liner reminders (interview style)

- `public` → world  
- `internal` → my project  
- `private` → my class only  
- `protected` → my family (children)  
- `protected internal` → my project + my children anywhere  
- `private protected` → my children, but only if they live in my house (same assembly)

**Assembly** -> A single compiled .NET unit — basically one `.dll` or one `.exe` file. It's the **smallest deployable, versionable, and securable chunk** of .NET code.