
# ✍️ Define object

### 🧩Dependency unit
---
```ad-info
collapse: close
title: IDependency.cs
~~~csharp
namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Demonstration dependency interface that allows you to inspect the unique
/// ID on a specific resolved instance of the dependency.
/// </summary>
public interface IDependency
{
    /// <summary>
    /// Gets the unique instance ID for the dependency.
    /// </summary>
    /// <value>
    /// A <see cref="System.Guid"/> that indicates the unique ID for the
    /// instance.
    /// </value>
    Guid InstanceId { get; }
}
~~~
```

```ad-note
collapse: close
title: BaseDependency.cs
~~~csharp
using System;

namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Base class for dependencies. Used simply to avoid redundant code; it's not
/// actually required to have a common derivation chain.
/// </summary>
public class BaseDependency : IDependency
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BaseDependency"/> class.
    /// </summary>
    public BaseDependency()
    {
        InstanceId = Guid.NewGuid();
    }

    /// <summary>
    /// Gets the unique instance ID for the dependency.
    /// </summary>
    /// <value>
    /// A <see cref="System.Guid"/> that indicates the unique ID for the
    /// instance.
    /// </value>
    public Guid InstanceId { get; private set; }
}
~~~
```

```ad-info
collapse: close
title: IDependencyConsumer.cs
~~~csharp
namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Demonstration interface for a consumer of a dependency. Enables the ability
/// to illustrate resolving components across global and tenant-specific scopes.
/// </summary>
public interface IDependencyConsumer
{
    /// <summary>
    /// Gets the dependency for this consumer.
    /// </summary>
    /// <value>
    /// An <see cref="MultitenantExample.ConsoleApplication.IDependency"/>
    /// held by this specific consumer.
    /// </value>
    IDependency Dependency { get; }
}
~~~
```

```ad-note
collapse: close
title: Consumer.cs
~~~csharp
namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Simple dependency consumer that takes a dependency as a constructor parameter.
/// </summary>
public class Consumer : IDependencyConsumer
{
    /// <summary>
    /// Gets the dependency for this consumer.
    /// </summary>
    /// <value>
    /// An <see cref="MultitenantExample.ConsoleApplication.IDependency"/>
    /// held by this specific consumer.
    /// </value>
    public IDependency Dependency { get; private set; }

    /// <summary>
    /// Initializes a new instance of the <see cref="MultitenantExample.ConsoleApplication.Consumer"/> class.
    /// </summary>
    /// <param name="dependency">The dependency this class consumes.</param>
    public Consumer(IDependency dependency)
    {
        Dependency = dependency;
    }
}
~~~
```


##### BaseDependency object
---
```ad-note
collapse: close
title: DefaultTenantDependency.cs
~~~csharp
namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Tenant-specific dependency for the default tenant.
/// </summary>
public class DefaultTenantDependency : BaseDependency
{
}
~~~
```

```ad-note
collapse: close
title: Tenant1Dependency.cs
~~~csharp
namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Tenant-specific dependency for Tenant 1.
/// </summary>
public class Tenant1Dependency : BaseDependency
{
}
~~~
```

```ad-note
collapse: close
title: Tenant2Dependency.cs
~~~csharp
namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Tenant-specific dependency for Tenant 2.
/// </summary>
public class Tenant2Dependency : BaseDependency
{
}
~~~
```


##### Implement Autofac.Multitenant.ITenantIdentificationStrategy
---
```ad-note
collapse: close
title: ManualTenantIdentificationStrategy.cs
~~~csharp
using Autofac.Multitenant;

namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Simple tenant ID strategy that allows you to manually specify what the
/// current tenant ID is.
/// </summary>
/// <remarks>
/// <para>
/// While this simple implementation requires you to manually specify the
/// current tenant ID, in a more "real world" application, you could
/// get the tenant ID from an environment variable, a role on the current
/// thread principal, a value from an incoming message, or anywhere else
/// that would work for your application.
/// </para>
/// </remarks>
public class ManualTenantIdentificationStrategy : ITenantIdentificationStrategy
{
    /// <summary>
    /// Gets or sets the current tenant ID. "0" is the "default tenant."
    /// </summary>
    /// <value>
    /// An <see cref="System.Object"/> that will be returned by
    /// <see cref="MultitenantExample.ConsoleApplication.ManualTenantIdentificationStrategy.TryIdentifyTenant"/>
    /// when the current tenant ID is requested.
    /// </value>
    public object CurrentTenantId { get; set; }

    /// <summary>
    /// Returns the tenant ID that was manually specified in
    /// <see cref="MultitenantExample.ConsoleApplication.ManualTenantIdentificationStrategy.CurrentTenantId"/>.
    /// </summary>
    /// <param name="tenantId">The current tenant identifier.</param>
    /// <returns>
    /// This implementation always returns <see langword="true"/>.
    /// </returns>
    public bool TryIdentifyTenant(out object tenantId)
    {
        if (CurrentTenantId.ToString() == "0")
        {
            // 0 is the "default tenant ID"
            tenantId = null;
        }
        else
        {
            // If the current tenant isn't default, return the actual ID.
            tenantId = CurrentTenantId;
        }
        return true;
    }
}
~~~
```


# ✍️ How to use

```ad-note
collapse: open
title: Program.cs
~~~csharp
using Autofac;
using Autofac.Multitenant;

namespace MultitenantExample.ConsoleApplication;

/// <summary>
/// Demonstration console application illustrating multitenant dependency injection.
/// </summary>
/// <remarks>
/// <para>
/// While there is not much of a use case for a console application to be
/// multitenant, what this illustrates is that non-ASP.NET, non-WCF applications
/// can also be multitenant. Windows services, for example, could make use
/// of multitenancy.
/// </para>
/// </remarks>
public class Program
{
    /// <summary>
    /// The container from which dependencies will be resolved.
    /// </summary>
    private static IContainer _container;

    /// <summary>
    /// Strategy used for identifying the current tenant with multitenant DI.
    /// </summary>
    private static ManualTenantIdentificationStrategy _tenantIdentifier;

    /// <summary>
    /// Demo program entry point.
    /// </summary>
    public static void Main()
    {
        // Initialize the tenant identification strategy.
        _tenantIdentifier = new ManualTenantIdentificationStrategy();

        // Set the application container to the multitenant container.
        _container = ConfigureDependencies();

        // Explain what you're looking at.
        WriteInstructions();

        // Start listening for input.
        ListenForInput();
    }

    /// <summary>
    /// Configures the multitenant dependency container.
    /// </summary>
    private static IContainer ConfigureDependencies()
    {
        // Register default dependencies in the application container.
        var builder = new ContainerBuilder();
        builder.RegisterType<Consumer>().As<IDependencyConsumer>().InstancePerDependency();
        builder.RegisterType<BaseDependency>().As<IDependency>().SingleInstance();
        var appContainer = builder.Build();

        // Create the multitenant container.
        var mtc = new MultitenantContainer(_tenantIdentifier, appContainer);

        // Configure overrides for tenant 1. Tenant 1 registers their dependencies
        // as instance-per-dependency.
        mtc.ConfigureTenant('1', b => b.RegisterType<Tenant1Dependency>().As<IDependency>().InstancePerDependency());

        // Configure overrides for tenant 2. Tenant 2 registers their dependencies
        // as singletons.
        mtc.ConfigureTenant('2', b => b.RegisterType<Tenant2Dependency>().As<IDependency>().SingleInstance());

        // Configure overrides for the default tenant. That means the default
        // tenant will have some different dependencies than other unconfigured
        // tenants.
        mtc.ConfigureTenant(null, b => b.RegisterType<DefaultTenantDependency>().As<IDependency>().SingleInstance());

        return mtc;
    }

    /// <summary>
    /// Loops and listens for input until the user quits.
    /// </summary>
    private static void ListenForInput()
    {
        ConsoleKeyInfo input;
        do
        {
            Console.Write("Select a tenant (1-9, 0=default) or 'q' to quit: ");
            input = Console.ReadKey();
            Console.WriteLine();

            if (input.KeyChar >= 48 && input.KeyChar <= 57)
            {
                // Set the "contextual" tenant ID based on the input, then
                // put it to the test.
                _tenantIdentifier.CurrentTenantId = input.KeyChar;
                ResolveDependencyAndWriteInfo();
            }
            else if (input.Key != ConsoleKey.Q)
            {
                Console.WriteLine("Invalid key pressed.");
            }
        } while (input.Key != ConsoleKey.Q);
    }

    /// <summary>
    /// Resolves the dependency from the container and displays some information
    /// about it.
    /// </summary>
    private static void ResolveDependencyAndWriteInfo()
    {
        var consumer = _container.Resolve<IDependencyConsumer>();
        Console.WriteLine("Tenant ID:       {0}", _tenantIdentifier.CurrentTenantId);
        Console.WriteLine("Dependency Type: {0}", consumer.Dependency.GetType().Name);
        Console.WriteLine("Instance ID:     {0}", consumer.Dependency.InstanceId);
        Console.WriteLine();
    }

    /// <summary>
    /// Writes the application instructions to the screen.
    /// </summary>
    private static void WriteInstructions()
    {
        Console.WriteLine("Multitenant Example: Console Application");
        Console.WriteLine("----------------------------------------");
        Console.WriteLine("Select a tenant ID (1 - 9) to see the dependencies that get resolved for that tenant. You will see the dependency type as well as the instance ID so you can verify the proper type and lifetime scope registration is being used.");
        Console.WriteLine();
        Console.WriteLine("* Tenant 1 has an override registered as InstancePerDependency.");
        Console.WriteLine("* Tenant 2 has an override registered as SingleInstance.");
        Console.WriteLine("* The default tenant has an override registered as SingleInstance.");
        Console.WriteLine("* Tenants that don't have overrides fall back to the application/base singleton.");
        Console.WriteLine();
    }
}
~~~
```
