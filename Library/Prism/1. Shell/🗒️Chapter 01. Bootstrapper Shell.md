
# ✍️ How to use

### BootstrapperShell
---
##### App.xaml.cs의 Startup 이벤트에서 PrismBootstrapper를 상속/구현한 클래스를 호출하여 App 실행

```ad-note
collapse: open
title: App.xaml.cs
~~~csharp
namespace BootstrapperShell;

/// <summary>
/// Interaction logic for App.xaml
/// </summary>
public partial class App : Application
{
	protected override void OnStartup(StartupEventArgs e)
	{
		base.OnStartup(e);

		var bootstrapper = new Bootstrapper();
		bootstrapper.Run();
	}
}
~~~
```

```ad-note
collapse: open
title: Bootstrapper.cs
~~~csharp
using Prism.Ioc;
using Prism.Unity;
using BootstrapperShell.Views;

namespace BootstrapperShell;

class Bootstrapper : PrismBootstrapper
{
    /// <summary>
	/// Call from PrismBootstrapperBase.Initialize (Second)
	/// </summaray>
	protected override DependencyObject CreateShell()
	{
		return Container.Resolve<MainWindow>();
	}

    /// <summary>
	/// Call from PrismBootstrapperBase.Initialize (First)
	/// </summaray>
	protected override void RegisterTypes(IContainerRegistry containerRegistry)
	{
		
	}
}
~~~
```
