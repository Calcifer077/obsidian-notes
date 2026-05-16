## 1. Simple Explanation — What is Testing?

Before we touch a single framework, understand _why_ testing exists.

**Without tests**, every time you change code, you must manually verify nothing broke. In a system with 200 features, that means checking 200 things by hand — every single time. That's not engineering, that's gambling.

**With tests**, you write code that _verifies your code_. You run it in milliseconds. If something breaks, you know immediately, exactly where, and why.

---

## 2. Real-World Analogy

Think of a **car manufacturing plant**.

Before a car leaves the factory:
- Individual parts are tested (unit testing — "does this brake pad work?")
- Parts assembled together are tested (integration testing — "do the brakes work with the wheels?")
- The whole car is test-driven (end-to-end testing — "does the car stop when you press the brake at 100 km/h?")

You don't ship the car and hope the brakes work. You _prove_ they work before shipping.

Your software is the car. xUnit/NUnit are the testing machines. Moq is the _simulator_ — it simulates road conditions without needing a real road.

---

## 3. Technical Deep Dive — The Three Layers

```
┌─────────────────────────────────────────────────────────┐
│                    Testing Pyramid                      │
│                         /\                              │
│                        /  \                             │
│                       /E2E \        ← Few, Slow, Costly │
│                      /──────\                           │
│                     /Integra-\                          │
│                    / tion     \    ← Medium             │
│                   /────────────\                        │
│                  /  Unit Tests  \  ← Many, Fast, Cheap  │
│                 /________________\                      │
└─────────────────────────────────────────────────────────┘
```

**Unit Tests** — Test ONE thing in complete isolation. No database. No HTTP. No file system. Just logic.

**Integration Tests** — Test how components work _together_. May hit real DB or real HTTP.

**E2E Tests** — Test the whole system as a user would. Slow. Brittle. Few.

We will focus heavily on **Unit Tests** because that's where xUnit, NUnit, and Moq shine.

---

## 4. xUnit vs NUnit — Why Do Both Exist?

### Historical Context (IMPORTANT)

```
NUnit (2002) ──► was THE standard for years
     │
     ▼
xUnit (2007) ──► created by the original NUnit author
     │           to fix fundamental design flaws in NUnit
     ▼
MSTest (Microsoft) ──► built into Visual Studio, but
                        historically inferior to both
```

**Why did xUnit author abandon NUnit and create xUnit?**

Because NUnit had _architectural mistakes_ baked in:
- Test classes were reused across tests (shared state = bugs)
- `[SetUp]` and `[TearDown]` encouraged bad patterns
- Not designed for modern async/await

xUnit was built from scratch with _clean architecture principles in mind_.

---

## 5. xUnit — Deep Dive

### Installation

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add package Microsoft.NET.Test.Sdk
```

### Your First Test — Every Keyword Explained

```csharp
using Xunit; // ← imports xUnit attributes and assertions

public class CalculatorTests // ← just a plain C# class, no base class needed
{                            //   WHY? xUnit uses reflection to discover tests
                             //   it doesn't need you to inherit anything
                             //   NUnit required [TestFixture] attribute on class
                             //   xUnit doesn't — cleaner, less ceremony

    [Fact] // ← marks this method as a test
           // WHY "Fact"? Because it represents a fact about your system
           // "Adding 2 and 3 always equals 5" — that's a fact, not an opinion
           // This is philosophical naming — a Fact has no parameters
           // Compare: a Theory has parameters (see below)
    public void Add_TwoNumbers_ReturnsCorrectSum()
    //  ↑ naming convention: MethodUnderTest_Scenario_ExpectedResult
    //  WHY this naming? It's documentation. When it fails, you instantly
    //  know what broke, under what condition, and what was expected.
    //  Bad name: "Test1()" → tells you NOTHING when it fails
    {
        // ARRANGE — set up the world
        var calculator = new Calculator();

        // ACT — perform the action
        var result = calculator.Add(2, 3);

        // ASSERT — verify the outcome
        Assert.Equal(5, result);
        // WHY Assert.Equal(expected, actual) and not Assert.Equal(actual, expected)?
        // Convention: EXPECTED comes first, ACTUAL comes second
        // When it fails: "Expected 5 but got 6" — reads naturally
    }
}
```

```csharp
public class Calculator // ← the class being tested (System Under Test = SUT)
{
    public int Add(int a, int b) => a + b;
}
```

---

### xUnit Test Lifecycle — CRITICAL to understand

```csharp
public class OrderServiceTests
{
    private readonly OrderService _sut; // sut = System Under Test

    public OrderServiceTests() // ← xUnit calls this constructor BEFORE EVERY test
    {                          //   WHY? Fresh instance = no shared state between tests
                               //   NUnit used [SetUp] method on a SHARED instance
                               //   That caused subtle bugs from state leaking between tests
                               //   xUnit forces isolation by design
        _sut = new OrderService();
    }

    [Fact]
    public void Test1() { /* _sut is a fresh instance */ }

    [Fact]
    public void Test2() { /* _sut is ANOTHER fresh instance, completely separate */ }
}
```

**WHY is this important?**

Imagine Test1 modifies `_sut` internal state. In NUnit with `[SetUp]`, if the setup ran once per class, Test2 would see dirty state. xUnit prevents this by constructing a new instance per test.

This is the **Single Responsibility** and **Test Isolation** principle in action.

---

### [Theory] and [InlineData] — Parameterized Tests

```csharp
[Theory] // ← "Theory" because we're testing a theory with multiple examples
         // A theory says: "For ANY valid input, this rule holds"
[InlineData(2, 3, 5)]   // ← data set 1
[InlineData(0, 0, 0)]   // ← data set 2
[InlineData(-1, 1, 0)]  // ← data set 3
[InlineData(int.MaxValue - 1, 1, int.MaxValue)] // ← edge case
public void Add_VariousInputs_ReturnsCorrectSum(int a, int b, int expected)
{
    var calculator = new Calculator();
    var result = calculator.Add(a, b);
    Assert.Equal(expected, result);
}
```

**WHY not just write 4 separate [Fact] tests?**

You could. But that's duplication. [Theory] + [InlineData] gives you:
- One test definition
- Multiple data sets
- Each runs independently — one failure doesn't stop others
- Easy to add edge cases later

**Alternative: [MemberData] for complex objects**

```csharp
public static IEnumerable<object[]> OrderTestData =>
    new List<object[]>
    {
        new object[] { new Order { Items = 3, Total = 150m }, true },
        new object[] { new Order { Items = 0, Total = 0m }, false },
    };

[Theory]
[MemberData(nameof(OrderTestData))]
public void ProcessOrder_WithData_ReturnsExpected(Order order, bool expectedResult)
{
    // test body
}
```

---

## 6. NUnit — Deep Dive

### Key Differences From xUnit

```csharp
using NUnit.Framework; // ← different namespace

[TestFixture] // ← NUnit requires this on the class (more ceremony)
              // xUnit doesn't need this
public class CalculatorTests
{
    private Calculator _calculator;

    [SetUp] // ← runs before EACH test (similar to xUnit constructor)
            // BUT: on the SAME class instance — potential state issues
    public void Setup()
    {
        _calculator = new Calculator();
    }

    [TearDown] // ← runs after EACH test
               // useful for cleanup: closing connections, deleting files
    public void Cleanup()
    {
        // dispose resources
    }

    [OneTimeSetUp] // ← runs ONCE before ALL tests in class
                   // Danger zone: shared state across tests
    public void OneTimeSetup() { }

    [Test] // ← NUnit equivalent of xUnit's [Fact]
    public void Add_TwoNumbers_ReturnsSum()
    {
        var result = _calculator.Add(2, 3);
        Assert.That(result, Is.EqualTo(5)); // ← NUnit assertion style
        // Is.EqualTo is more readable English
        // "Assert THAT result IS EQUAL TO 5"
    }

    [TestCase(2, 3, 5)]   // ← NUnit equivalent of xUnit's [InlineData]
    [TestCase(0, 0, 0)]
    public void Add_TestCases(int a, int b, int expected)
    {
        Assert.That(_calculator.Add(a, b), Is.EqualTo(expected));
    }
}
```

### NUnit Assertion Constraint Model

```csharp
// NUnit's "constraint model" is very expressive:
Assert.That(result, Is.EqualTo(5));
Assert.That(name, Is.Not.Null);
Assert.That(list, Has.Count.EqualTo(3));
Assert.That(value, Is.GreaterThan(0).And.LessThan(100));
Assert.That(collection, Contains.Item("admin"));
Assert.That(str, Does.StartWith("Hello").And.EndWith("World"));

// vs xUnit style:
Assert.Equal(5, result);
Assert.NotNull(name);
Assert.Equal(3, list.Count);
// xUnit is simpler but less fluent/readable
```

---

## 7. xUnit vs NUnit — Side-by-Side Comparison

|Concept|xUnit|NUnit|
|---|---|---|
|Mark test class|(none needed)|`[TestFixture]`|
|Mark test method|`[Fact]`|`[Test]`|
|Parameterized test|`[Theory]` + `[InlineData]`|`[TestCase]`|
|Before each test|Constructor|`[SetUp]`|
|After each test|`IDisposable.Dispose()`|`[TearDown]`|
|Before all tests|`IClassFixture<T>`|`[OneTimeSetUp]`|
|Assertion style|`Assert.Equal(expected, actual)`|`Assert.That(actual, Is.EqualTo(expected))`|
|Instance per test|YES (by design)|NO (shared instance)|

**Which should you use in 2025?**

**xUnit** is the industry standard for:

- ASP.NET Core team uses it internally
- .NET runtime tests use it
- Better isolation by design
- Better async support

NUnit is still valid and widely used. Many enterprise codebases have it. Know both.

---

## 8. Moq — The Dependency Simulator

This is where it gets _really_ powerful.

### WHY Moq Exists

Consider this service:

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;     // sends real emails
    private readonly IPaymentGateway _paymentGateway; // charges real credit cards
    private readonly IOrderRepository _repo;          // hits real database

    public OrderService(
        IEmailService emailService,
        IPaymentGateway paymentGateway,
        IOrderRepository repo)
    {
        _emailService = emailService;
        _paymentGateway = paymentGateway;
        _repo = repo;
    }

    public async Task<bool> PlaceOrder(Order order)
    {
        var saved = await _repo.SaveAsync(order);
        if (!saved) return false;

        var charged = await _paymentGateway.ChargeAsync(order.Total);
        if (!charged) return false;

        await _emailService.SendConfirmationAsync(order.CustomerEmail);
        return true;
    }
}
```

**Problem**: How do you unit test `PlaceOrder`?

- You can't hit a real database in unit tests (slow, state, setup)
- You can't charge real credit cards
- You can't send real emails

**Solution: Mock the dependencies** — create fake versions that _simulate_ real behavior.

---

### Installation

```bash
dotnet add package Moq
```

---

### Moq — Every Concept Explained

```csharp
using Moq; // ← Moq library
using Xunit;

public class OrderServiceTests
{
    // STEP 1: Create Mock objects
    private readonly Mock<IEmailService> _mockEmailService;
    //                  ↑ Mock<T> where T must be an interface or virtual class
    //                  WHY interface? Because Moq uses runtime proxy generation
    //                  It creates a fake class at runtime that implements IEmailService
    //                  You CANNOT mock sealed classes or non-virtual methods
    //                  WHY? Because the proxy needs to OVERRIDE methods
    //                  sealed = no inheritance allowed = no override = no mock

    private readonly Mock<IPaymentGateway> _mockPaymentGateway;
    private readonly Mock<IOrderRepository> _mockRepo;
    private readonly OrderService _sut;

    public OrderServiceTests()
    {
        // STEP 2: Instantiate the mocks
        _mockEmailService = new Mock<IEmailService>();
        _mockPaymentGateway = new Mock<IPaymentGateway>();
        _mockRepo = new Mock<IOrderRepository>();

        // STEP 3: Pass mock OBJECTS (not Mock<T> itself) to the SUT
        _sut = new OrderService(
            _mockEmailService.Object,  // .Object gives you the fake implementation
            _mockPaymentGateway.Object,
            _mockRepo.Object
        );
        // WHY .Object? Because Mock<T> is a wrapper/configuration object
        // .Object is the actual fake instance that implements the interface
    }

    [Fact]
    public async Task PlaceOrder_WhenAllServicesSucceed_ReturnsTrue()
    {
        // ARRANGE

        var order = new Order
        {
            Total = 100m,
            CustomerEmail = "test@example.com"
        };

        // SETUP: Tell the mocks what to return when called
        _mockRepo
            .Setup(x => x.SaveAsync(order)) // ← "when SaveAsync is called with this order"
            .ReturnsAsync(true);             // ← "return true"
        //   ↑ Setup() takes a lambda expression
        //   x represents the mock object itself
        //   x.SaveAsync(order) specifies WHICH method call to configure
        //   ReturnsAsync() because it's an async method returning Task<bool>

        _mockPaymentGateway
            .Setup(x => x.ChargeAsync(100m))
            .ReturnsAsync(true);

        _mockEmailService
            .Setup(x => x.SendConfirmationAsync(It.IsAny<string>()))
            //                                   ↑ It.IsAny<string>()
            //                                   = "match ANY string argument"
            //                                   We don't care what email string is passed
            //                                   WHY? Because we want to test the FLOW
            //                                   not the exact email formatting
            .Returns(Task.CompletedTask); // ← void async returns Task.CompletedTask

        // ACT
        var result = await _sut.PlaceOrder(order);

        // ASSERT
        Assert.True(result);

        // VERIFY: Check that methods were ACTUALLY called
        _mockRepo.Verify(x => x.SaveAsync(order), Times.Once());
        //                                         ↑ Times.Once() = must have been called exactly once
        //                                         Times.Never() = must NOT have been called
        //                                         Times.Exactly(3) = called 3 times
        //                                         Times.AtLeastOnce() = called 1 or more times

        _mockPaymentGateway.Verify(x => x.ChargeAsync(100m), Times.Once());

        _mockEmailService.Verify(
            x => x.SendConfirmationAsync(It.IsAny<string>()),
            Times.Once()
        );
    }

    [Fact]
    public async Task PlaceOrder_WhenRepositoryFails_ReturnsFalseAndNeverChargesPayment()
    {
        // ARRANGE
        var order = new Order { Total = 100m };

        _mockRepo
            .Setup(x => x.SaveAsync(order))
            .ReturnsAsync(false); // ← simulate DB failure

        // ACT
        var result = await _sut.PlaceOrder(order);

        // ASSERT
        Assert.False(result);

        // Critical: If repo fails, we should NEVER charge the customer
        _mockPaymentGateway.Verify(
            x => x.ChargeAsync(It.IsAny<decimal>()),
            Times.Never() // ← THIS is the important assertion
        );                //   Verifying behavior under failure conditions
    }
}
```

---

### Moq — It.Is vs It.IsAny vs Exact Values

```csharp
// Exact value match
_mockRepo.Setup(x => x.GetByIdAsync(42)).ReturnsAsync(someOrder);
// Only matches when called with exactly 42

// Match ANY value
_mockRepo.Setup(x => x.GetByIdAsync(It.IsAny<int>())).ReturnsAsync(someOrder);
// Matches GetByIdAsync(1), GetByIdAsync(999), GetByIdAsync(-5) — anything

// Match with condition
_mockRepo.Setup(x => x.GetByIdAsync(It.Is<int>(id => id > 0)))
         .ReturnsAsync(someOrder);
// Only matches positive integers
// GetByIdAsync(-1) would NOT match this setup

// Match on object properties
_mockEmailService.Setup(x => x.SendConfirmationAsync(
    It.Is<string>(email => email.Contains("@"))))
    .Returns(Task.CompletedTask);
// Only matches if email contains "@"
```

---

### Moq — Throwing Exceptions

```csharp
[Fact]
public async Task PlaceOrder_WhenPaymentThrows_PropagatesException()
{
    var order = new Order { Total = 100m };

    _mockRepo.Setup(x => x.SaveAsync(order)).ReturnsAsync(true);

    _mockPaymentGateway
        .Setup(x => x.ChargeAsync(It.IsAny<decimal>()))
        .ThrowsAsync(new PaymentException("Card declined")); // ← simulate exception

    await Assert.ThrowsAsync<PaymentException>(
        () => _sut.PlaceOrder(order)
    );
}
```

---

### Moq MockBehavior — Strict vs Loose

```csharp
// Default: MockBehavior.Loose
var looseMock = new Mock<IEmailService>();
// If you call a method that has NO setup:
// → Returns default value (null for reference types, 0 for int, false for bool)
// → Does NOT throw

// MockBehavior.Strict
var strictMock = new Mock<IEmailService>(MockBehavior.Strict);
// If you call a method that has NO setup:
// → THROWS MockException immediately
// → Forces you to explicitly set up EVERY call

// WHY Strict? Forces explicit test setup. No accidental implicit behavior.
// WHY Loose? Less setup code. Good for tests where you don't care about some calls.
// Industry prefers Loose for most tests, Strict for critical workflow validation.
```

---

## 9. Complete Real-World Example — Putting It All Together

```csharp
// Domain
public record Order(int Id, decimal Total, string CustomerEmail);

// Interfaces (always mock interfaces, not implementations)
public interface IOrderRepository
{
    Task<bool> SaveAsync(Order order);
    Task<Order?> GetByIdAsync(int id);
}

public interface IPaymentGateway
{
    Task<bool> ChargeAsync(decimal amount);
}

public interface IEmailService
{
    Task SendConfirmationAsync(string email);
}

// Service under test
public class OrderService
{
    private readonly IOrderRepository _repo;
    private readonly IPaymentGateway _payment;
    private readonly IEmailService _email;

    public OrderService(
        IOrderRepository repo,
        IPaymentGateway payment,
        IEmailService email)
    {
        _repo = repo ?? throw new ArgumentNullException(nameof(repo));
        _payment = payment ?? throw new ArgumentNullException(nameof(payment));
        _email = email ?? throw new ArgumentNullException(nameof(email));
    }

    public async Task<OrderResult> PlaceOrderAsync(Order order)
    {
        ArgumentNullException.ThrowIfNull(order);

        var saved = await _repo.SaveAsync(order);
        if (!saved)
            return OrderResult.Failed("Could not save order");

        var charged = await _payment.ChargeAsync(order.Total);
        if (!charged)
            return OrderResult.Failed("Payment failed");

        await _email.SendConfirmationAsync(order.CustomerEmail);

        return OrderResult.Success(order.Id);
    }
}

// Tests
public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _repoMock = new();
    //                                                    ↑ C# 9 target-typed new
    //                                                    compiler infers type from left side
    private readonly Mock<IPaymentGateway> _paymentMock = new();
    private readonly Mock<IEmailService> _emailMock = new();
    private readonly OrderService _sut;

    public OrderServiceTests()
    {
        _sut = new OrderService(
            _repoMock.Object,
            _paymentMock.Object,
            _emailMock.Object);
    }

    // Test 1: Happy path
    [Fact]
    public async Task PlaceOrderAsync_ValidOrder_ReturnsSuccess()
    {
        // Arrange
        var order = new Order(1, 99.99m, "user@test.com");
        _repoMock.Setup(r => r.SaveAsync(order)).ReturnsAsync(true);
        _paymentMock.Setup(p => p.ChargeAsync(99.99m)).ReturnsAsync(true);
        _emailMock.Setup(e => e.SendConfirmationAsync(It.IsAny<string>()))
                  .Returns(Task.CompletedTask);

        // Act
        var result = await _sut.PlaceOrderAsync(order);

        // Assert
        Assert.True(result.IsSuccess);
        Assert.Equal(1, result.OrderId);
    }

    // Test 2: Null guard
    [Fact]
    public async Task PlaceOrderAsync_NullOrder_ThrowsArgumentNullException()
    {
        await Assert.ThrowsAsync<ArgumentNullException>(
            () => _sut.PlaceOrderAsync(null!));
    }

    // Test 3: Repository failure — short circuit
    [Fact]
    public async Task PlaceOrderAsync_RepoFails_NeverChargesCard()
    {
        var order = new Order(1, 99.99m, "user@test.com");
        _repoMock.Setup(r => r.SaveAsync(order)).ReturnsAsync(false);

        await _sut.PlaceOrderAsync(order);

        _paymentMock.Verify(
            p => p.ChargeAsync(It.IsAny<decimal>()),
            Times.Never);
    }

    // Test 4: Payment failure — email not sent
    [Fact]
    public async Task PlaceOrderAsync_PaymentFails_NeverSendsEmail()
    {
        var order = new Order(1, 99.99m, "user@test.com");
        _repoMock.Setup(r => r.SaveAsync(order)).ReturnsAsync(true);
        _paymentMock.Setup(p => p.ChargeAsync(99.99m)).ReturnsAsync(false);

        await _sut.PlaceOrderAsync(order);

        _emailMock.Verify(
            e => e.SendConfirmationAsync(It.IsAny<string>()),
            Times.Never);
    }
}
```

---

## 10. Common Mistakes

```csharp
// ❌ MISTAKE 1: Mocking concrete classes
var mock = new Mock<OrderService>(); // Dangerous — can only mock virtual members

// ✅ Always mock interfaces
var mock = new Mock<IOrderService>();

// ❌ MISTAKE 2: Not verifying important behaviors
// Just asserting return value isn't enough for side effects
Assert.True(result); // Did email actually get sent? We don't know!

// ✅ Verify side effects explicitly
_emailMock.Verify(e => e.SendConfirmationAsync("user@test.com"), Times.Once);

// ❌ MISTAKE 3: One test method, 10 assertions
// If assertion 3 fails, you never know if 4-10 would pass

// ✅ One logical assertion per test (or closely related assertions)

// ❌ MISTAKE 4: Setup returning null without thinking
_repoMock.Setup(r => r.GetByIdAsync(1)); // Returns null by default (Loose mock)
// Your code then NullReferenceExceptions — test passes but for wrong reason

// ✅ Always be explicit
_repoMock.Setup(r => r.GetByIdAsync(1)).ReturnsAsync((Order?)null);
```

---

## Interview Questions You Should Know

1. What is the difference between `[Fact]` and `[Theory]` in xUnit?
2. Why does xUnit create a new class instance per test?
3. What is the difference between `Mock.Setup()` and `Mock.Verify()`?
4. Why can't you mock sealed classes with Moq?
5. What is `MockBehavior.Strict` and when would you use it?
6. What is the AAA pattern in testing?
7. What is the difference between a Mock, Stub, and Fake?
8. Why should unit tests never hit a real database?
9. What does `It.IsAny<T>()` do and when would you prefer it over exact values?
10. How does Moq generate fake implementations at runtime?

---

## Your Cross-Examination — Answer These Before Moving On

**Conceptual Questions:**

1. I have a class `EmailService` (concrete, not interface) that sends emails. A junior dev says "just mock `EmailService` directly." What are the problems with this? What would you tell them?
    
2. In our `OrderService`, what happens if I call `_paymentMock.Verify(...)` but I _forgot_ to set up `_paymentMock.Setup(...)` first — and the mock is `MockBehavior.Loose`? Will the test crash during Setup, during Act, or during Verify? Why?
    
3. Why does xUnit use the constructor for setup instead of a `[SetUp]` attribute like NUnit? What fundamental design principle does this enforce?
    
4. If `PlaceOrderAsync` is called and repo returns `false` — our test verifies payment is `Times.Never`. But what if I had a bug where payment was called BEFORE checking the repo result? Would our test catch it?
    

**Scenario:**

You're reviewing a PR. A developer wrote this test:

```csharp
[Fact]
public async Task ProcessOrder_Test()
{
    var mock = new Mock<IOrderRepository>();
    var service = new OrderService(mock.Object);
    var result = await service.ProcessOrder(new Order());
    Assert.NotNull(result);
}
```

**Identify every problem with this test.**

**Mini Assignment:**

Build a `UserService` with a method `RegisterUserAsync(User user)` that:

1. Validates email is not already taken (via `IUserRepository.ExistsAsync`)
2. Hashes password (via `IPasswordHasher.Hash`)
3. Saves user (via `IUserRepository.SaveAsync`)
4. Sends welcome email (via `IEmailService.SendWelcomeAsync`)
5. Returns `RegistrationResult` (success or failure reason)

Write **5 unit tests** covering: happy path, duplicate email, save failure, and verify email is never sent if save fails.