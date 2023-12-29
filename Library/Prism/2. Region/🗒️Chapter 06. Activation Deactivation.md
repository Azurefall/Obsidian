# ✍️ How to use

### ActivationDeactivate
---
##### 각 View에 대한 등록을 MainWindow Load 후 소스에서 등록
##### ViewA / ViewB를 RegionManager에 정의 위치("ContentRegion")에 활성/비활성 (표시/미표시)
##### DockPanel이 ContentControl을 포함하고 있기 때문에 IContainerExtension이 포함된 생성자가 호출

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="ActivationDeactivation.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:prism="http://prismlibrary.com/"
        Title="Shell" Height="350" Width="525">
    <DockPanel LastChildFill="True">
        <StackPanel>
            <Button Content="Activate ViewA" Click="Button_Click"/>
            <Button Content="Deactivate ViewA" Click="Button_Click_1"/>
            <Button Content="Activate ViewB" Click="Button_Click_2"/>
            <Button Content="Deactivate ViewB" Click="Button_Click_3"/>
        </StackPanel>
        <ContentControl prism:RegionManager.RegionName="ContentRegion" HorizontalAlignment="Center" VerticalAlignment="Center" />
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

namespace ActivationDeactivation.Views;

/// <summary>
/// Interaction logic for MainWindow.xaml
/// </summary>
public partial class MainWindow : Window
{
	IContainerExtension _container;
	IRegionManager _regionManager;
	IRegion _region;

	ViewA _viewA;
	ViewB _viewB;

	public MainWindow(IContainerExtension container, IRegionManager regionManager)
	{
		InitializeComponent();
		_container = container;
		_regionManager = regionManager;

		this.Loaded += MainWindow_Loaded;
	}

	private void MainWindow_Loaded(object sender, RoutedEventArgs e)
	{
		_viewA = _container.Resolve<ViewA>();
		_viewB = _container.Resolve<ViewB>();

		_region = _regionManager.Regions["ContentRegion"];

		_region.Add(_viewA);
		_region.Add(_viewB);
	}

	private void Button_Click(object sender, RoutedEventArgs e)
	{
		//activate view a
		_region.Activate(_viewA);
	}

	private void Button_Click_1(object sender, RoutedEventArgs e)
	{
		//deactivate view a
		_region.Deactivate(_viewA);
	}

	private void Button_Click_2(object sender, RoutedEventArgs e)
	{
		//activate view b
		_region.Activate(_viewB);
	}

	private void Button_Click_3(object sender, RoutedEventArgs e)
	{
		//deactivate view b
		_region.Deactivate(_viewB);
	}
}
~~~
```

```ad-note
collapse: close
title: ViewA.xaml
~~~csharp
<UserControl x:Class="ActivationDeactivation.Views.ViewA"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:ActivationDeactivation.Views"
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
title: ViewB.xaml
~~~csharp
<UserControl x:Class="ActivationDeactivation.Views.ViewB"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:ActivationDeactivation.Views"
             mc:Ignorable="d" 
             d:DesignHeight="300" d:DesignWidth="300">
    <Grid>
        <TextBlock Text="View B" FontSize="38" />
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
using ActivationDeactivation.Views;

namespace ActivationDeactivation;

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
