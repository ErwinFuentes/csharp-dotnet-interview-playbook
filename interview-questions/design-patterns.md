# 10 Most Popular Design Patterns in C#

A practical guide for developers, with clear examples for each pattern.

Design patterns are **reusable solutions to common problems in software design**.  
They help developers write **cleaner, more maintainable, scalable, and flexible code**.

---

## 1. Singleton

**Category:** Creational

Ensures a class has only **one instance** and provides a global access point to it.

### When to use
- Managing shared resources (e.g., config, logging, DB connections)

### Example

```csharp
public sealed class Logger
{
    private static readonly Lazy<Logger> _instance =
        new Lazy<Logger>(() => new Logger());

    private Logger() { }

    public static Logger Instance => _instance.Value;

    public void Log(string message) =>
        Console.WriteLine($"[LOG] {message}");
}

// Usage
Logger.Instance.Log("Application started");
Logger.Instance.Log("Another message from the same instance");
```

---

## 2. Factory Method

**Category:** Creational

Defines an interface for creating objects, but lets **subclasses decide** which class to instantiate.

### When to use
- When the exact type of object to create isn't known until runtime

### Example

```csharp
public abstract class Notification
{
    public abstract void Send(string message);
}

public class EmailNotification : Notification
{
    public override void Send(string message) =>
        Console.WriteLine($"Email: {message}");
}

public class SmsNotification : Notification
{
    public override void Send(string message) =>
        Console.WriteLine($"SMS: {message}");
}

public abstract class NotificationFactory
{
    public abstract Notification CreateNotification();

    public void Notify(string message)
    {
        var notification = CreateNotification();
        notification.Send(message);
    }
}

public class EmailFactory : NotificationFactory
{
    public override Notification CreateNotification() => new EmailNotification();
}

public class SmsFactory : NotificationFactory
{
    public override Notification CreateNotification() => new SmsNotification();
}

// Usage
NotificationFactory factory = new EmailFactory();
factory.Notify("Welcome aboard!");
```

---

## 3. Builder

**Category:** Creational

Separates the **construction** of a complex object from its **representation**.

### When to use
- Creating objects with many optional parameters

### Example

```csharp
public class Pizza
{
    public string Size { get; set; }
    public bool HasCheese { get; set; }
    public bool HasPepperoni { get; set; }
    public bool HasMushrooms { get; set; }

    public override string ToString() =>
        $"{Size} pizza | Cheese: {HasCheese} | Pepperoni: {HasPepperoni} | Mushrooms: {HasMushrooms}";
}

public class PizzaBuilder
{
    private readonly Pizza _pizza = new Pizza();

    public PizzaBuilder SetSize(string size) { _pizza.Size = size; return this; }
    public PizzaBuilder AddCheese() { _pizza.HasCheese = true; return this; }
    public PizzaBuilder AddPepperoni() { _pizza.HasPepperoni = true; return this; }
    public PizzaBuilder AddMushrooms() { _pizza.HasMushrooms = true; return this; }
    public Pizza Build() => _pizza;
}

// Usage
var pizza = new PizzaBuilder()
    .SetSize("Large")
    .AddCheese()
    .AddPepperoni()
    .Build();

Console.WriteLine(pizza);
// Output: Large pizza | Cheese: True | Pepperoni: True | Mushrooms: False
```

---

## 4. Adapter

**Category:** Structural

Converts the interface of a class into **another interface** that clients expect. Bridges incompatible interfaces.

### When to use
- Integrating legacy or third-party code with a new system

### Example

```csharp
// Legacy class we can't modify
public class LegacyPaymentSystem
{
    public void MakePayment(double amount) =>
        Console.WriteLine($"Legacy payment processed: ${amount}");
}

// Target interface the rest of the app expects
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}

// Adapter
public class PaymentAdapter : IPaymentProcessor
{
    private readonly LegacyPaymentSystem _legacy = new LegacyPaymentSystem();

    public void ProcessPayment(decimal amount) =>
        _legacy.MakePayment((double)amount);
}

// Usage
IPaymentProcessor processor = new PaymentAdapter();
processor.ProcessPayment(99.99m);
// Output: Legacy payment processed: $99.99
```

---

## 5. Decorator

**Category:** Structural

Adds **new responsibilities** to an object dynamically without modifying its class.

### When to use
- Extending functionality at runtime (e.g., logging, caching, compression)

### Example

```csharp
public interface ICoffee
{
    string GetDescription();
    double GetCost();
}

public class SimpleCoffee : ICoffee
{
    public string GetDescription() => "Simple Coffee";
    public double GetCost() => 1.00;
}

public abstract class CoffeeDecorator : ICoffee
{
    protected readonly ICoffee _coffee;
    protected CoffeeDecorator(ICoffee coffee) => _coffee = coffee;
    public virtual string GetDescription() => _coffee.GetDescription();
    public virtual double GetCost() => _coffee.GetCost();
}

public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }
    public override string GetDescription() => _coffee.GetDescription() + ", Milk";
    public override double GetCost() => _coffee.GetCost() + 0.25;
}

public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee) { }
    public override string GetDescription() => _coffee.GetDescription() + ", Sugar";
    public override double GetCost() => _coffee.GetCost() + 0.10;
}

// Usage
ICoffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

Console.WriteLine(coffee.GetDescription()); // Simple Coffee, Milk, Sugar
Console.WriteLine($"${coffee.GetCost()}");  // $1.35
```

---

## 6. Observer

**Category:** Behavioral

Defines a **one-to-many dependency** so that when one object changes state, all its dependents are notified automatically.

### When to use
- Event systems, UI bindings, pub/sub models

### Example

```csharp
public interface IObserver
{
    void Update(string eventName, object data);
}

public class EventBus
{
    private readonly Dictionary<string, List<IObserver>> _subscribers = new();

    public void Subscribe(string eventName, IObserver observer)
    {
        if (!_subscribers.ContainsKey(eventName))
            _subscribers[eventName] = new List<IObserver>();
        _subscribers[eventName].Add(observer);
    }

    public void Publish(string eventName, object data)
    {
        if (_subscribers.TryGetValue(eventName, out var observers))
            observers.ForEach(o => o.Update(eventName, data));
    }
}

public class EmailAlerter : IObserver
{
    public void Update(string eventName, object data) =>
        Console.WriteLine($"[Email] Event '{eventName}' received: {data}");
}

public class DashboardWidget : IObserver
{
    public void Update(string eventName, object data) =>
        Console.WriteLine($"[Dashboard] Updating for '{eventName}': {data}");
}

// Usage
var bus = new EventBus();
bus.Subscribe("OrderPlaced", new EmailAlerter());
bus.Subscribe("OrderPlaced", new DashboardWidget());

bus.Publish("OrderPlaced", "Order #1042");
```

---

## 7. Strategy

**Category:** Behavioral

Defines a **family of algorithms**, encapsulates each one, and makes them interchangeable at runtime.

### When to use
- When you need to switch between different algorithms or behaviors

### Example

```csharp
public interface ISortStrategy
{
    void Sort(List<int> data);
}

public class BubbleSortStrategy : ISortStrategy
{
    public void Sort(List<int> data)
    {
        // Simplified bubble sort
        data.Sort(); // placeholder for illustration
        Console.WriteLine("Sorted using Bubble Sort");
    }
}

public class QuickSortStrategy : ISortStrategy
{
    public void Sort(List<int> data)
    {
        data.Sort();
        Console.WriteLine("Sorted using Quick Sort");
    }
}

public class Sorter
{
    private ISortStrategy _strategy;

    public Sorter(ISortStrategy strategy) => _strategy = strategy;

    public void SetStrategy(ISortStrategy strategy) => _strategy = strategy;

    public void Sort(List<int> data) => _strategy.Sort(data);
}

// Usage
var data = new List<int> { 5, 2, 8, 1, 9 };
var sorter = new Sorter(new BubbleSortStrategy());
sorter.Sort(data);

sorter.SetStrategy(new QuickSortStrategy());
sorter.Sort(data);
```

---

## 8. Command

**Category:** Behavioral

Encapsulates a **request as an object**, allowing you to parameterize, queue, log, or undo operations.

### When to use
- Undo/redo functionality, job queues, transaction logs

### Example

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class TextEditor
{
    private string _content = "";
    public void Append(string text) => _content += text;
    public void RemoveLast(int length) =>
        _content = _content.Length >= length
            ? _content[..^length]
            : "";
    public string GetContent() => _content;
}

public class AppendCommand : ICommand
{
    private readonly TextEditor _editor;
    private readonly string _text;

    public AppendCommand(TextEditor editor, string text)
    {
        _editor = editor;
        _text = text;
    }

    public void Execute() => _editor.Append(_text);
    public void Undo() => _editor.RemoveLast(_text.Length);
}

public class CommandHistory
{
    private readonly Stack<ICommand> _history = new();

    public void Execute(ICommand command)
    {
        command.Execute();
        _history.Push(command);
    }

    public void Undo()
    {
        if (_history.TryPop(out var command))
            command.Undo();
    }
}

// Usage
var editor = new TextEditor();
var history = new CommandHistory();

history.Execute(new AppendCommand(editor, "Hello"));
history.Execute(new AppendCommand(editor, ", World!"));
Console.WriteLine(editor.GetContent()); // Hello, World!

history.Undo();
Console.WriteLine(editor.GetContent()); // Hello
```

---

## 9. Repository

**Category:** Architectural

Abstracts the **data layer**, providing a collection-like interface for accessing domain objects.

### When to use
- Separating business logic from data access (Entity Framework, Dapper, etc.)

### Example

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public interface IProductRepository
{
    Product GetById(int id);
    IEnumerable<Product> GetAll();
    void Add(Product product);
    void Delete(int id);
}

public class InMemoryProductRepository : IProductRepository
{
    private readonly List<Product> _products = new();

    public Product GetById(int id) =>
        _products.FirstOrDefault(p => p.Id == id);

    public IEnumerable<Product> GetAll() => _products;

    public void Add(Product product) => _products.Add(product);

    public void Delete(int id) =>
        _products.RemoveAll(p => p.Id == id);
}

// Usage
IProductRepository repo = new InMemoryProductRepository();
repo.Add(new Product { Id = 1, Name = "Laptop", Price = 999.99m });
repo.Add(new Product { Id = 2, Name = "Mouse", Price = 29.99m });

foreach (var product in repo.GetAll())
    Console.WriteLine($"{product.Id}: {product.Name} - ${product.Price}");
```

---

## 10. Dependency Injection

**Category:** Architectural / Creational

**Injects dependencies** from the outside rather than creating them internally. Core to modern C# and ASP.NET Core.

### When to use
- Decoupling components, enabling unit testing, using IoC containers

### Example

```csharp
// Abstraction
public interface IMessageService
{
    void SendMessage(string to, string message);
}

// Concrete implementation
public class SmtpMessageService : IMessageService
{
    public void SendMessage(string to, string message) =>
        Console.WriteLine($"SMTP -> To: {to} | Message: {message}");
}

// Consumer — depends on abstraction, not concrete class
public class OrderService
{
    private readonly IMessageService _messageService;

    // Dependency is injected via constructor
    public OrderService(IMessageService messageService)
    {
        _messageService = messageService;
    }

    public void PlaceOrder(string customerEmail, string item)
    {
        Console.WriteLine($"Order placed for: {item}");
        _messageService.SendMessage(customerEmail, $"Your order for '{item}' is confirmed!");
    }
}

// Usage (manual DI)
IMessageService service = new SmtpMessageService();
var orderService = new OrderService(service);
orderService.PlaceOrder("user@example.com", "Mechanical Keyboard");

// In ASP.NET Core, register in Program.cs:
// builder.Services.AddScoped<IMessageService, SmtpMessageService>();
// builder.Services.AddScoped<OrderService>();
```

---

## Quick Reference

| # | Pattern | Category | Core Idea |
|---|---------|----------|-----------|
| 1 | Singleton | Creational | One instance only |
| 2 | Factory Method | Creational | Delegate instantiation to subclasses |
| 3 | Builder | Creational | Step-by-step object construction |
| 4 | Adapter | Structural | Bridge incompatible interfaces |
| 5 | Decorator | Structural | Add behavior dynamically |
| 6 | Observer | Behavioral | Notify dependents on state change |
| 7 | Strategy | Behavioral | Swap algorithms at runtime |
| 8 | Command | Behavioral | Encapsulate requests as objects |
| 9 | Repository | Architectural | Abstract the data layer |
| 10 | Dependency Injection | Architectural | Inject dependencies from outside |

---

*These patterns form the backbone of clean, maintainable, and testable C# applications. Master them, and you'll write code that scales.*
