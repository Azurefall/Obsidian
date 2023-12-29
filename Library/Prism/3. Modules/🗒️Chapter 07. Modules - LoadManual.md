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
##### [App.xaml.cs] 호출 모듈(ModuleA) 정보를 읽어오기 위해 ModuleCatalog 생성(설정)
##### [MainWindow.xaml] RegionManager에 영역("ContentRegion") 설정
##### [MainWindow.xaml.cs] 버튼 클릭 시 모듈 호출

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="Modules.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Shell" Height="350" Width="525">
    <DockPanel LastChildFill="True">
        <Button DockPanel.Dock="Top" Content="Load Module" Click="Button_Click" />
        <ContentControl prism:RegionManager.RegionName="ContentRegion" />
    </DockPanel>
</Window>
~~~
```

```ad-note
collapse: open
title: MainWindow.xaml.cs
~~~csharp
using Prism.Modularity;

namespace Modules.Views;

/// <summary>
/// Interaction logic for MainWindow.xaml
/// </summary>
public partial class MainWindow : Window
{
	IModuleManager _moduleManager;

	public MainWindow(IModuleManager moduleManager)
	{
		InitializeComponent();
		_moduleManager = moduleManager;
	}

	private void Button_Click(object sender, RoutedEventArgs e)
	{
		_moduleManager.LoadModule("ModuleAModule");
	}
}
~~~
```

```ad-note
collapse: open
title: App.xaml.cs
~~~csharp
using Prism.Ioc;
using Prism.Modularity;
using Prism.Unity;

using ModuleA;
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
		var moduleAType = typeof(ModuleAModule);
		moduleCatalog.AddModule(new ModuleInfo()
		{
			ModuleName = moduleAType.Name,
			ModuleType = moduleAType.AssemblyQualifiedName,
			InitializationMode = InitializationMode.OnDemand
		});
	}
}
~~~
```

### ※ 추가사항
---
##### Chapter 06의 내용을 차용하여 [MainWindow.xaml.cs]에서 생성자 호출 시 IContainerExtension을 추가하여 모듈을 호출하는 방법도 생각해 볼 수 있다.