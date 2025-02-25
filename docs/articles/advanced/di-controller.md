<div class="article">

# Controllers and Dependency Injection in Our Project

In our project, we use `Controllers` to separate concerns and organize our application logic. Controllers act as the entry point for handling specific functionalities, such as processing chat messages or managing commands. Additionally, we leverage `Dependency Injection (DI)` to manage dependencies and promote loose coupling, making our code more modular, testable, and maintainable.

## Why Use Dependency Injection (DI)?

Dependency Injection is used to:
1. `Decouple Dependencies`: DI allows us to inject dependencies (e.g., services, actors, loggers) into controllers, reducing tight coupling between components.
2. `Improve Testability`: With DI, dependencies can be easily mocked or replaced during unit testing, making it simpler to test individual components in isolation.
3. `Promote Reusability`: Services and dependencies registered in the DI container can be reused across multiple controllers or components.
4. `Simplify Maintenance`: DI makes it easier to manage and update dependencies, as changes to a service only need to be made in one place.

For a deeper understanding of Dependency Injection, check out this video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/J1f5b4vcxCQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Why Use Controllers?

Controllers are used to:
1. `Separate Concerns`: Each controller is responsible for a specific domain or functionality (e.g., chat, commands, etc.), ensuring that the code is organized and adheres to the Single Responsibility Principle (SRP).
2. `Centralize Logic`: Controllers centralize the logic for handling specific tasks, such as processing incoming chat messages or registering commands.
3. `Improve Readability`: By isolating functionality into controllers, the codebase becomes easier to navigate and understand.

## Automatic Registration

In our project, we use the `[RegisterSingleton]` attribute from the [AutoRegisterInject](https://github.com/patrickklaeren/AutoRegisterInject) NuGet package to automatically register controllers and other services into the Dependency Injection (DI) container during application startup. This eliminates the need for manual registration and ensures that all marked classes are available for injection.



**How It Works**:
1. `Attribute Usage`: Classes (such as controllers) are decorated with the `[RegisterSingleton]` attribute. This tells the `AutoRegisterInject` package to register the class as a singleton in the DI container.
   ```c#
   [RegisterSingleton]
   public sealed class ChatController
   {
       // Controller implementation
   }
   ```

2. `Automatic Registration`
During application startup, the `AutoRegisterInject` package scans the assembly for classes marked with the `[RegisterSingleton]` attribute and registers them in the DI container. This ensures that all controllers and services are automatically available for dependency injection.

3. `Singleton Lifetime`
The `[RegisterSingleton]` attribute ensures that only one instance of the controller is created and reused throughout the application's lifetime. This is ideal for stateless controllers or services that do not need to be re-instantiated.

4. `Benefits of [RegisterSingleton]`
- **Reduced Boilerplate**: Automates the registration process, reducing the need for manual configuration in the DI container.
- **Consistency**: Ensures that all controllers and services are registered in a consistent manner.
- **Ease of Use**: Simplifies the onboarding process for new developers by abstracting away the DI registration logic.

## Bootstrapper Lifecycle

The `Bootstrapper` class is responsible for initializing and configuring the application's dependencies, services, and logging. It acts as the entry point for setting up the application's environment and ensuring all necessary components are ready before the application starts running.

# Overview

The `Bootstrapper` class performs the following tasks:
1. `Configuration Setup`: Loads configuration files (e.g., `*settings.json`) and configures the application settings.
2. `Logging Configuration`: Sets up logging using Serilog and configures log levels based on the application's debug mode.
3. `Dependency Injection (DI) Setup`: Registers all necessary services and dependencies in the DI container.
4. `Service Initialization`: Scans and instantiates services marked with custom attributes (e.g., `RegisterSingletonAttribute`).
5. `Application Startup`: Initializes the database and other controllers, then starts the application host.

# Lifecycle Summary
1. `Configuration`: Loads settings and configures the application.
2. `Logging`: Sets up logging and registers settings in the DI container.
3. `Services`: Registers and initializes services.
4. `Startup`: Initializes the database, controllers, and starts the application host.

This ensures the application is fully configured and ready to run before any requests are processed.

## IController Interface

Defines a contract for controllers requiring asynchronous initialization.

# Purpose

1. `Asynchronous Initialization`: Perform async setup tasks during startup.
2. `Consistency`: Standardizes initialization across controllers that need to be instantiated asynchronously.

# Why We Need This

In dependency injection (DI), constructors are called automatically when a service is resolved. However, **constructors cannot be asynchronous**, meaning you cannot use `await` inside them. This is a limitation because many initialization tasks (e.g., database connections, API calls, or file loading) are inherently asynchronous.

By using the `IController` interface, we decouple the asynchronous initialization logic from the constructor. Instead, we move it to the `InitializeAsync` method, which can be called explicitly after the object is constructed. This ensures:

1. `Asynchronous Operations`: You can safely use `await` in `InitializeAsync` for tasks like connecting to a database or calling an API.
2. `Explicit Initialization`: The initialization process is explicit and controlled, making it easier to manage and debug.
3. `Error Handling`: Errors during initialization can be handled gracefully, rather than causing the constructor to fail.

# Example Scenario

Imagine a `DatabaseController` that needs to connect to a database during startup. Without `IController`, you’d have to block the thread (e.g., using `.Result` or `.Wait()`), which is bad practice and can lead to deadlocks. With `IController`, you can write:

```c#
public sealed class DatabaseController : IController
{
    public async Task InitializeAsync()
    {
        await ConnectToDatabaseAsync(); // Safe async operation
    }

    private async Task ConnectToDatabaseAsync()
    {
        // Simulate database connection
        await Task.Delay(1000); // Example delay
        Console.WriteLine("Database connected!");
    }
}
```

# Circular Dependencies

A `circular dependency` occurs when two or more controllers depend on each other directly or indirectly, creating a loop in the dependency graph. This makes it impossible to determine the correct initialization order, leading to runtime errors or crashes.

# Example of a Circular Dependency

Imagine two controllers:
- `ControllerA` depends on `ControllerB`.
- `ControllerB` depends on `ControllerA`.

When the `BootStrapper` tries to resolve the initialization order, it will get stuck in an infinite loop, eventually causing a `StackOverflowException`.

# How to Avoid Circular Dependencies

- Redesign the controllers to remove the circular dependency.
- For example, if `ControllerA` and `ControllerB` depend on each other, introduce a third service or controller to break the cycle.

## The Dependency Graph Helper

The `DependencyGraphHelper` class is responsible for resolving and initializing controllers in the correct order based on their dependencies. It ensures that controllers are initialized only after their dependencies have been initialized.

# Purpose

1. `Dependency Resolution`: Resolves the dependencies of controllers by inspecting their constructor parameters.
2. `Ordered Initialization`: Ensures controllers are initialized in the correct order based on their dependency graph.
3. `Asynchronous Initialization`: Calls the `InitializeAsync` method on each controller after resolving its dependencies.

# Why We Need It

In a complex application, controllers often depend on other controllers or services. For example:
- A `DatabaseController` might need to be initialized before an `ApiController` that depends on it.
- Without proper dependency resolution, controllers might try to initialize before their dependencies are ready, leading to runtime errors.

The `DependencyGraphHelper` solves this problem by:
1. `Building a Dependency Graph`: It analyzes the constructors of all controllers to determine their dependencies.
2. `Topological Sorting`: It orders the controllers so that dependencies are initialized before the controllers that depend on them.
3. `Initialization`: It initializes the controllers in the correct order, ensuring a smooth startup process.

# Why This Approach?

1. `Ensures Correct Initialization Order`:
   - Controllers are initialized only after their dependencies are ready.

2. `Scalable`:
   - As the application grows, new controllers can be added without breaking the initialization process.


## TODOS
* Factories für die Zukunft
---
* BootStrapper Lifecycle, DependencyGraph erklären
* `IController` und `InitializeAsync` erklären, eventuell mit beispiel (warning: Circular Dependency)
* Controller / `[RegisterSingleton]` erklären
* Dependency Injection erklären / zu video linken https://www.youtube.com/watch?v=J1f5b4vcxCQ

</div>