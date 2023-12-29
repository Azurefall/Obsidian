# ✍️ How to use

### ModuleA
---
##### [ModuleAModule.cs] 호출되는 모듈에서 호출하는 모듈 내 표시 영역인 Region("ContentRegion")을 설정

```ad-note
collapse: open
title: ModuleAModule.cs
~~~csharp
using Prism.Ioc;
using Prism.Modularity;
using Prism.Regions;

using ModuleA.Views;

namespace ModuleA;

public class ModuleAModule : IModule
{
	public void OnInitialized(IContainerProvider containerProvider)
	{
		var regionManager = containerProvider.Resolve<IRegionManager>();
		regionManager.RegisterViewWithRegion("ContentRegion", typeof(ViewA));
	}

	public void RegisterTypes(IContainerRegistry containerRegistry)
	{
	}
}
~~~
```

```ad-note
collapse: close
title: ViewA.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.ViewA"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             mc:Ignorable="d" 
             d:DesignHeight="300" d:DesignWidth="300">
    <Grid>
        <TextBlock Text="View A" FontSize="38" />
    </Grid>
</UserControl>
~~~
```

### Modules
---
##### [App.xaml.cs] ModuleCatalog에 호출할 ModuleA 등록
##### [MainWindow.xaml] RegionManager에 영역("ContentRegion") 설정

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="Modules.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Shell" Height="350" Width="525">
    <Grid>
        <ContentControl prism:RegionManager.RegionName="ContentRegion" />
    </Grid>
</Window>
~~~
```

```ad-note
collapse: open
title: App.xaml.cs
~~~csharp
using Prism.Modularity;
using Prism.Unity;
using Prism.Ioc;

using Modules.Views;

namespace Modules;

/// <summary>
/// Interaction logic for App.xaml
/// </summary>
public partial class App : PrismApplication
{
	protected override Window CreateShell()
	{
		return Container.Resolve<MainWindow>();
	}

	protected override void RegisterTypes(IContainerRegistry containerRegistry)
	{
	}

	protected override void ConfigureModuleCatalog(IModuleCatalog moduleCatalog)
	{
		moduleCatalog.AddModule<ModuleA.ModuleAModule>();
	}
}
~~~
```
