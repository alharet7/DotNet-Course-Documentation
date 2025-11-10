# Dependency Injection 

## Tight Coupling:
Definition: When one class directly depends on another specific class.

Why it's bad:

1. Hard to change or replace components.

2. Difficult to test (you can't easily mock dependencies).

3. Code becomes rigid and fragile.

```
public class OrderService {
    private PaymentService _paymentService = new PaymentService(); // tightly coupled
}

```

## Dependency Hell:
Definition: A messy situation where managing dependencies becomes chaotic—especially when many classes depend on many others.

Symptoms:

1. Long constructor chains (new A(new B(new C(...))))

2. Hard-to-track bugs due to hidden dependencies.

3. Painful refactoring and testing.

```
var service = new A(new B(new C(new D(new E())))); // 😵
```

---

## Dependency Inversion Principle (DIP):

**DIP** = High-level modules should not depend on low-level modules. Both should depend on abstractions.

High-level modules = Your business logic (e.g. `OrderService`)

Low-level modules = Utility classes or services (e.g. `SqlOrderRepository`)

Abstraction = Interfaces or abstract classes (e.g. `IOrderRepository`)

####  ***The Problems Before DIP***
1. **Rigid Dependencies**

- High-level modules (like OrderService) directly depend on low-level modules (like SqlOrderRepository).

- If you want to switch to MongoOrderRepository, you have to change the high-level code.

2. **Hard to Test**

- You can’t easily mock or stub the low-level module.

- Unit testing becomes a pain because the high-level module is glued to a real implementation.

3. **Code Breaks Easily**

- A small change in the low-level module can ripple up and break the high-level logic.

- You lose control over how modules evolve independently.

4. **No Reusability**

- You can’t reuse the high-level module in another context unless the low-level module fits too.

- Your code becomes context-locked.

####  ***How DIP Solves These***
1. **Decouples Logic from Implementation**

- High-level modules depend on **interfaces**, **not concrete** classes.

- You can swap implementations without touching business logic.

```
public interface IOrderRepository {
    void Save(Order order);
}
```

```
public class OrderService {
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) {
        _repo = repo;
    }
}
```

2. **Boosts Testability**

- You can inject a fake or mock implementation during tests.

- No need to spin up a real database or external service.

```
public class FakeOrderRepository : IOrderRepository {
    public void Save(Order order) {
        // pretend to save
    }
}
```


3. **Improves Flexibility & Maintenance**

- You can evolve low-level modules independently.

- High-level modules stay clean and focused on business rules.

4. **Enables Dependency Injection**

- DIP sets the rule: “Depend on abstractions.”

- DI (Dependency Injection) is the tool that injects the right implementation at runtime.

`services.AddScoped<IOrderRepository, SqlOrderRepository>();
`

---


## Inversion Of Control Principle (IoC Containers):

### 🧠 What Is an IoC Container?

An **IoC (Inversion of Control) Container** is a framework or tool that:
- **Creates** objects (like services or repositories)
- **Resolves** their dependencies (injects what they need)
- **Manages** their lifecycle (singleton, scoped, transient)

In short: **You tell it what you need, and it wires everything up for you.**


### 🧩 Why Do We Need It?

Before IoC containers, you had to do this manually:

```
var repo = new SqlOrderRepository();
var service = new OrderService(repo);

OR

var service = new EmailService();
var controller = new UserController(service);
```

That’s fine for small apps, but in real-world projects:

- You have dozens of services

- Each has multiple dependencies

- You need to control lifetimes (singleton, etc.)

- You want clean, testable, maintainable code

IoC containers automate all of this.

### How It Works in .NET
1. Register Services

- You tell the container what to inject and how long to keep it alive.


```
services.AddScoped<IOrderRepository, SqlOrderRepository>();
services.AddTransient<OrderService>();

```

- ***AddScoped***: One instance per request
- ***AddTransient***: New instance every time
- ***AddSingleton***: One instance for the whole app

2. Inject Dependencies

- .NET will automatically inject dependencies into constructors.

```
public class OrderService {
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) {
        _repo = repo;
    }
}
```

You don’t new anything—.NET ***does it*** for you.

3. Use It Anywhere

- In controllers, background services, etc.

```
public class OrdersController : Controller {
    private readonly IOrderService _orderService;
    public OrdersController(IOrderService orderService) {
        _orderService = orderService;
    }
}
```

## 🔄 Lifecycle Recap

| Method         | Lifetime Scope                      | Use Case                         |
|----------------|--------------------------------------|----------------------------------|
| `AddSingleton` | One instance for entire app          | Shared config, caching, etc.     |
| `AddScoped`    | One per HTTP request                 | DB contexts, business services   |
| `AddTransient` | New instance every time it's needed | Lightweight, stateless services  |

🎯 Summary
- IoC Principle says: “Let someone else control object creation.”

- IoC Container is that “someone else”—it wires up dependencies for you.

- In .NET, it’s built-in and super easy to use via Startup.cs or Program.cs.

---
