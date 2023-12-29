# ✍️ How to use

### ViewInjection
---
##### Button 이벤트 발생 시 RegionManager에 정의된 위치("ContentRegion")에 View 표시
##### DockPanel이 ContentControl을 포함하고 있기 때문에 IContainerExtension이 포함된 생성자가 호출

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="ViewInjection.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:prism="http://prismlibrary.com/"
        Title="Shell" Height="350" Width="525">
    <DockPanel LastChildFill="True">
        <Button DockPanel.Dock="Top" Click="Button_Click">Add View</Button>
        <ContentControl prism:RegionManager.RegionName="ContentRegion" />
    </DockPanel>
</Window>
~~~
```

```ad-note
collapse: open
title: MainWindow.xaml.cs
~~~csharp
using Prism.Ioc;
using Prism.Regions;

namespace ViewInjection.Views;

/// <summary>
/// Interaction logic for MainWindow.xaml
/// </summary>
public partial class MainWindow : Window
{
	IContainerExtension _container;
	IRegionManager _regionManager;

	public MainWindow(IContainerExtension container, IRegionManager regionManager)
	{
		InitializeComponent();
		_container = container;
		_regionManager = regionManager;
	}

	private void Button_Click(object sender, RoutedEventArgs e)
	{
		var view = _container.Resolve<ViewA>();
		IRegion region = _regionManager.Regions["ContentRegion"];
		region.Add(view);
	}
}
~~~
```

```ad-note
collapse: close
title: ViewA.xaml
~~~csharp
<UserControl x:Class="ViewInjection.Views.ViewA"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:ViewInjection.Views"
             mc:Ignorable="d" 
             d:DesignHeight="300" d:DesignWidth="300">
    <Grid>
        <TextBlock Text="View A" FontSize="38" />
    </Grid>
</UserControl>
~~~
```

```ad-note
collapse: close
title: App.xaml.cs
~~~csharp
using Prism.Ioc;
using Prism.Unity;
using ViewInjection.Views;

namespace ViewInjection;

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
