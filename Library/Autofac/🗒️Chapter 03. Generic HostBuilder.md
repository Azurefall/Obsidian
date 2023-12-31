### When using Microsoft.Extensions.Hosting.IHostingService

# ✍️ How to use

### 🧩Logging unit
---
##### Plugin interface
```ad-info
collapse: close
title: ILogger.cs
~~~csharp
namespace GenericHostBuilderExample;

internal interface ILogger
{
	Task Log(string value);
}
~~~
```

```ad-note
collapse: close
title: Logger.cs
~~~csharp
namespace GenericHostBuilderExample;

internal class Logger : ILogger
{
	public async Task Log(string value)
	{
		await Console.Out.WriteLineAsync($"Logger: {value}");
	}
}
~~~
```

```ad-note
collapse: close
title: HostedService.cs
~~~csharp
using Microsoft.Extensions.Hosting;

namespace GenericHostBuilderExample;

internal class HostedService : IHostedService
{
	private readonly ILogger _logger;

	// ReSharper disable once UnusedMember.Global
	public HostedService(ILogger logger)
	{
		_logger = logger;
	}

	public async Task StartAsync(CancellationToken cancellationToken)
	{
		await _logger.Log("Starting hosted service");
	}

	public async Task StopAsync(CancellationToken cancellationToken)
	{
		await _logger.Log("Stopping hosted service");
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
using Autofac.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace GenericHostBuilderExample;

internal static class Program
{
	public static async Task Main(string[] args)
	{
		// The `UseServiceProviderFactory(new AutofacServiceProviderFactory())`
		// call here allows for ConfigureContainer to be supported with a
		// strongly-typed ContainerBuilder. If you don't have the call to
		// `UseServiceProviderFactory(new AutofacServiceProviderFactory())`
		// here, you won't get ConfigureContainer support. This also
		// automatically calls Populate to put services you register during
		// ConfigureServices into Autofac.
		await Host.CreateDefaultBuilder(args)
			.UseServiceProviderFactory(new AutofacServiceProviderFactory())
			.ConfigureServices(ConfigureServices)
			.ConfigureContainer<ContainerBuilder>(ConfigureContainer)
			.RunConsoleAsync();
	}


	private static void ConfigureServices(IServiceCollection services)
		// Use extensions from libraries to register services in the
		// collection. These will be automatically added to the Autofac
		// container.
		=> services.AddHostedService<HostedService>();

	private static void ConfigureContainer(ContainerBuilder containerBuilder)
		// Add any Autofac modules or registrations. This is called AFTER
		// ConfigureServices so things you register here OVERRIDE things
		// registered in ConfigureServices.
		//
		// You must have the call to `UseServiceProviderFactory(new AutofacServiceProviderFactory())`
		// when building the host or this won't be called.
		=> containerBuilder.RegisterType<Logger>().As<ILogger>();
}
~~~
```
