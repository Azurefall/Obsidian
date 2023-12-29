
# ✍️ How to use

### 🧩ConfigurationExampleInterface
---
##### Plugin interface
```ad-info
collapse: close
title: IPlugin.cs
~~~csharp
namespace ConfigurationExampleInterface
{
    /// <summary>
    /// Simple plugin interface used to illustrate assembly loading/plugin handling
    /// via configuration. See the "ConfigurationExample" project for usage.
    /// </summary>
    public interface IPlugin
    {
        /// <summary>
        /// Gets the name of the plugin.
        /// </summary>
        /// <value>
        /// A <see cref="string"/> with the plugin name. Likely just the type name for illustrative purposes.
        /// </value>
        string Name { get; }
    }
}
~~~
```


### 🧩ConfigurationExamplePlugin 
---
##### External plugin class (inherit IPlugin)
```ad-note
collapse: close
title: ExternalPlugin.cs
~~~csharp
using ConfigurationExampleInterface;

namespace ConfigurationExamplePlugin
{
    /// <summary>
    /// Implementation of the plugin interface that will be loaded from an external assembly. See the "ConfigurationExample"
    /// project for more details.
    /// </summary>
    public class ExternalPlugin : IPlugin
    {
        /// <summary>
        /// Gets the name of the plugin.
        /// </summary>
        /// <value>
        /// Always returns <c>ExternalPlugin</c>.
        /// </value>
        public string Name
        {
            get
            {
                return "ExternalPlugin";
            }
        }
    }
}
~~~
```


### 🧩ConfigurationExample
---
##### Configuration file
##### "type": [Class or Interface], [Module]
```ad-tip
collapse: close
title: autofac.json
~~~json
{
  "defaultAssembly": "ConfigurationExample",
  "components": [
    {
      "type": "ConfigurationExample.InternalPlugin",
      "services": [
        {
          "type": "ConfigurationExample.InternalPlugin"
        },
        {
          "type": "ConfigurationExampleInterface.IPlugin, ConfigurationExampleInterface"
        }
      ]
    },
    {
      "type": "ConfigurationExamplePlugin.ExternalPlugin, ConfigurationExamplePlugin",
      "services": [
        {
          "type": "ConfigurationExampleInterface.IPlugin, ConfigurationExampleInterface"
        }
      ]
    }
  ]
}
~~~
```

##### Internal plugin class (inherit IPlugin)
```ad-note
collapse: close
title: InternalPlugin.cs
~~~csharp
using ConfigurationExampleInterface;

namespace ConfigurationExample;

/// <summary>
/// Implementation of the plugin interface that will be loaded from a default assembly. This will be registered
/// via configuration rather than code.
/// </summary>
public class InternalPlugin : IPlugin
{
    /// <summary>
    /// Gets the name of the plugin.
    /// </summary>
    /// <value>
    /// Always returns <c>InternalPlugin</c>.
    /// </value>
    public string Name
    {
        get
        {
            return "InternalPlugin";
        }
    }
}
~~~
```

##### Implementation
```ad-note
collapse: open
title: Program.cs
~~~csharp
using Autofac;
using Autofac.Configuration;
using Microsoft.Extensions.Configuration;

using ConfigurationExampleInterface;

namespace ConfigurationExample;

public class Program
{
    /* Plugins and configuration can be challenging to figure out in .NET Core since assembly
     * loading and type handling has slightly changed. This example shows how to use configuration
     * to selectively load things. Additional examples and resources:
     * - Unit tests for Autofac.Configuration: https://github.com/autofac/Autofac.Configuration/tree/develop/test/Autofac.Configuration.Test
     * - Documentation for Autofac.Configuration: https://autofac.readthedocs.io/en/latest/configuration/xml.html
     *
     * To avoid referencing the external plugin assembly, the solution has a build dependency on the ConfigurationExamplePlugin
     * project and this project uses a post-build copy to get the plugin assembly into the ConfigurationExample bin directory.
     */
    public static void Main()
    {
        // THIS IS THE MAGIC!
        // .NET Core assembly loading is confusing. Things that happen to be in your bin folder don't just suddenly
        // qualify with the assembly loader. If the assembly isn't specifically referenced by your app, you need to
        // tell .NET Core where to get it EVEN IF IT'S IN YOUR BIN FOLDER.
        // https://stackoverflow.com/questions/43918837/net-core-1-1-type-gettype-from-external-assembly-returns-null
        //
        // The documentation says that any .dll in the application base folder should work, but that doesn't seem
        // to be entirely true. You always have to set up additional handlers if you AREN'T referencing the plugin assembly.
        // https://github.com/dotnet/core-setup/blob/master/Documentation/design-docs/corehost.md
        //
        // To verify, try commenting this out and you'll see that the config system can't load the external plugin type.
        var executionFolder = Path.GetDirectoryName(typeof(Program).Assembly.Location);
        AssemblyLoadContext.Default.Resolving += (AssemblyLoadContext context, AssemblyName assembly) => context.LoadFromAssemblyPath(Path.Combine(executionFolder, $"{assembly.Name}.dll"));

        var config = new ConfigurationBuilder()
            .AddJsonFile("autofac.json")
            .Build();
        var configModule = new ConfigurationModule(config);
        var builder = new ContainerBuilder();
        builder.RegisterModule(configModule);
        var container = builder.Build();

        try
        {
            // Always resolve from a scope.
            // https://autofac.readthedocs.io/en/latest/best-practices/index.html#always-resolve-dependencies-from-nested-lifetimes
            using var scope = container.BeginLifetimeScope();
            var plugin = scope.Resolve<InternalPlugin>();
            Console.WriteLine("Resolved specific plugin type: {0}", plugin.Name);

            Console.WriteLine("All available plugins:");
            var allPlugins = scope.Resolve<IEnumerable<IPlugin>>();
            foreach (var resolved in allPlugins)
            {
                Console.WriteLine("- {0}", resolved.Name);
            }
        }
        catch (Exception ex)
        {
            Console.Error.WriteLine("Error during configuration demonstration: {0}", ex);
        }

        if (Debugger.IsAttached)
        {
            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
        }
    }
}
~~~
```
