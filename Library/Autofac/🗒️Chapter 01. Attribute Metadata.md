
# ✍️ Define use case

### 🧩Logging unit
---
```ad-info
collapse: close
title: ILogAppender.cs
~~~csharp
namespace AttributeMetadataExample;

public interface ILogAppender
{
	void Write(string message);
}
~~~
```

```ad-note
collapse: close
title: Log.cs
~~~csharp
using Autofac.Features.Metadata;

namespace AttributeMetadataExample;

public class Log
{
    private readonly IEnumerable<Meta<ILogAppender>> _appenders;

    public Log(IEnumerable<Meta<ILogAppender>> appenders)
    {
        _appenders = appenders;
    }

    public void Write(string destination, string message)
    {
        var appender = _appenders.First(a => a.Metadata["AppenderName"].Equals(destination));
        appender.Value.Write(message);
    }
}
~~~
```

```ad-note
collapse: close
title: LogWithFilter.cs
~~~csharp
using Autofac.Features.AttributeFilters;

namespace AttributeMetadataExample;

public class LogWithFilter
{
    private readonly ILogAppender _appender;

    public LogWithFilter([MetadataFilter("AppenderName", "attributed")]ILogAppender appender)
    {
        _appender = appender;
    }

    public void Write(string message)
    {
        _appender.Write(message);
    }
}
~~~
```


##### Using type 1. Attach string-based metadata manually through registration
---
```ad-note
collapse: close
title: StringManualMetadataAppender.cs
~~~csharp
namespace AttributeMetadataExample;

public class StringManualMetadataAppender : ILogAppender
{
    public void Write(string message)
    {
        Console.WriteLine("String Metadata on Registration: {0}", message);
    }
}
~~~
```


##### Using type 2. Attach strongly-typed metadata manually through registration
---
```ad-note
collapse: close
title: AppenderMetadata.cs
~~~csharp
namespace AttributeMetadataExample;

public class AppenderMetadata
{
	public string AppenderName { get; set; }
}
~~~
```

```ad-note
collapse: close
title: TypedManualMetadataAppender.cs
~~~csharp
namespace AttributeMetadataExample;

public class TypedManualMetadataAppender : ILogAppender
{
    public void Write(string message)
    {
        Console.WriteLine("Strongly-Typed Metadata on Registration: {0}", message);
    }
}
~~~
```


##### Using type 3. Attach interface-based metadata manually through registration
---
```ad-info
collapse: close
title: IAppenderMetadata.cs
~~~csharp
namespace AttributeMetadataExample;

public interface IAppenderMetadata
{
	string AppenderName { get; }
}
~~~
```

```ad-note
collapse: close
title: InterfaceManualMetadataAdapter.cs
~~~csharp
namespace AttributeMetadataExample;

public class InterfaceManualMetadataAdapter : ILogAppender
{
    public void Write(string message)
    {
        Console.WriteLine("Interface Metadata on Registration: {0}", message);
    }
}
~~~
```


##### Using type 4. Attribute-based metadata
---
```ad-note
collapse: close
title: AppenderNameAttribute.cs
~~~csharp
namespace AttributeMetadataExample;

[MetadataAttribute]
[AttributeUsage(AttributeTargets.Class)]
public class AppenderNameAttribute : Attribute
{
    public AppenderNameAttribute(string appenderName)
    {
        AppenderName = appenderName;
    }

    public string AppenderName { get; private set; }
}
~~~
```

```ad-note
collapse: close
title: AttributeMetadataAppender.cs
~~~csharp
namespace AttributeMetadataExample;

[AppenderName("attributed")]
public class AttributeMetadataAppender : ILogAppender
{
    public void Write(string message)
    {
        Console.WriteLine("Attribute Metadata: {0}", message);
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
using Autofac.Extras.AttributeMetadata;
using Autofac.Features.AttributeFilters;
using Autofac.Integration.Mef;

namespace AttributeMetadataExample;

public class Program
{
    private static void Main()
    {
        // Attribute metadata documentation can be found here:
        // https://autofac.readthedocs.io/en/latest/advanced/metadata.html
        var builder = new ContainerBuilder();
        builder.RegisterType<Log>();

		// Using type 1. Attach string-based metadata manually through registration.
        builder.RegisterType<StringManualMetadataAppender>()
            .As<ILogAppender>()
            .WithMetadata("AppenderName", "string-manual");

		// Using type 2. Attach strongly-typed metadata manually through registration.
        builder.RegisterType<TypedManualMetadataAppender>()
            .As<ILogAppender>()
            .WithMetadata<AppenderMetadata>(m => m.For(am => am.AppenderName, "typed-manual"));

		// Using type 3. Attach interface-based metadata manually through registration.
        //               Requires use of the RegisterMetadataRegistrationSources extension from Autofac.Mef.
        builder.RegisterType<InterfaceManualMetadataAdapter>()
            .As<ILogAppender>()
            .WithMetadata<IAppenderMetadata>(m => m.For(am => am.AppenderName, "interface-manual"));
        builder.RegisterMetadataRegistrationSources();

		// Using type 4. Attribute-based metadata.
        //               Requires registration of the AttributedMetadataModule.
        builder.RegisterType<AttributeMetadataAppender>()
            .As<ILogAppender>();
        builder.RegisterModule<AttributedMetadataModule>();

        // Example: Consuming component using metadata filter attribute in constructor.
        builder.RegisterType<LogWithFilter>()
            .WithAttributeFiltering();

        using (var container = builder.Build())
        {
            using var scope = container.BeginLifetimeScope();

            // The standard logger will choose which appender to use
            // based on the specified metadata value.
            var log = scope.Resolve<Log>();
            log.Write("string-manual", "Message 1");
            log.Write("typed-manual", "Message 2");
            log.Write("interface-manual", "Message 3");
            log.Write("attributed", "Message 4");

            // This logger selects the appropriate appender by
            // applying a filter attribute on the constructor parameter.
            var filteredLog = scope.Resolve<LogWithFilter>();
            filteredLog.Write("Message from filtered log");
        }

        if (Debugger.IsAttached)
        {
            Console.WriteLine("[press any key to exit]");
            Console.ReadKey();
        }
    }
}
~~~
```
