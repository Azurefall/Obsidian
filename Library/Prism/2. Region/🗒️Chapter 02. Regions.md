##### App 실행

```ad-info
title: PrismApplication Class
collapse: close
~~~csharp
namespace Prism.Unity;

/// <summary>
/// Base application class that uses <see cref="UnityContainerExtension"/> as it's container.
/// </summary>
public abstract class PrismApplication : PrismApplicationBase
{
	/// <summary>
	/// Create a new <see cref="UnityContainerExtension"/> used by Prism.
	/// </summary>
	/// <returns>A new <see cref="UnityContainerExtension"/>.</returns>
	protected override IContainerExtension CreateContainerExtension()
	{
		return new UnityContainerExtension();
	}

	/// <summary>
	/// Registers the <see cref="Type"/>s of the Exceptions that are not considered 
	/// root exceptions by the <see cref="ExceptionExtensions"/>.
	/// </summary>
	protected override void RegisterFrameworkExceptionTypes()
	{
		ExceptionExtensions.RegisterFrameworkExceptionType(typeof(ResolutionFailedException));
	}
}
~~~
```

# ✍️ How to use

### Regions
---
##### App.xaml.cs에서 PrismApplication을 상속하여 App 실행

```ad-note
collapse: open
title: App.xaml.cs
~~~csharp
using Prism.Ioc;
using Prism.Unity;
using Regions.Views;

namespace Regions;

/// <summary>
/// Interaction logic for App.xaml
/// </summary>
public partial class App : PrismApplication
{
    /// <summary>
	/// Call from PrismApplicationBase.Initialize (Second)
	/// </summaray>
	protected override Window CreateShell()
	{
		return Container.Resolve<MainWindow>();
	}

    /// <summary>
	/// Call from PrismApplicationBase.Initialize (First)
	/// </summaray>
	protected override void RegisterTypes(IContainerRegistry containerRegistry)
	{
		
	}
}
~~~
```
