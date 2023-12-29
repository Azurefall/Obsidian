# ✍️ How to use

### ViewDiscovery
---
##### RegionManager에 정의된 위치("ContentRegion")에 View 표시

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="ViewDiscovery.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:prism="http://prismlibrary.com/"
        Title="Shell" Height="350" Width="525">
    <Grid>
        <ContentControl prism:RegionManager.RegionName="ContentRegion" />
    </Grid>
</Window>
~~~
```

```ad-note
collapse: open
title: MainWindow.xaml.cs
~~~csharp
using Prism.Regions

namespace ViewDiscovery.Views;

/// <summary>
/// Interaction logic for MainWindow.xaml
/// </summary>
public partial class MainWindow : Window
{
	public MainWindow(IRegionManager regionManager)
	{
		InitializeComponent();
		//view discovery
		regionManager.RegisterViewWithRegion("ContentRegion", typeof(ViewA));
	}
}
~~~
```

```ad-note
collapse: close
title: ViewA.xaml
~~~csharp
<UserControl x:Class="ViewDiscovery.Views.ViewA"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:ViewDiscovery.Views"
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

namespace ViewDirectory.Views

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
