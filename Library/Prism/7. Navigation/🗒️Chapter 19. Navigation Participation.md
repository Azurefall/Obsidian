
# ✍️ How to use

### ModuleA
---
##### [ViewA.xaml] Binding Properties
##### [ViewB.xaml] Binding Properties
##### [ModuleAModule.cs] Navigation을 위한 컨테이너 등록 (ViewA, ViewB)

```ad-note
collapse: open
title: ViewA.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.ViewA"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True">
    <StackPanel>
        <TextBlock Text="{Binding Title}" FontSize="48" HorizontalAlignment="Center" VerticalAlignment="Center" />
        <TextBlock Text="{Binding PageViews}" FontSize="24" HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </StackPanel>
</UserControl>
~~~
```

```ad-note
collapse: open
title: ViewAViewModel.cs
~~~csharp
using Prism.Mvvm;
using Prism.Regions;

namespace ModuleA.ViewModels;

public class ViewAViewModel : BindableBase, INavigationAware
{
	private string _title = "ViewA";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	private int _pageViews;
	public int PageViews
	{
		get { return _pageViews; }
		set { SetProperty(ref _pageViews, value); }
	}

	public void OnNavigatedTo(NavigationContext navigationContext)
	{
		PageViews++;
	}

	public bool IsNavigationTarget(NavigationContext navigationContext)
	{
		return true;
	}

	public void OnNavigatedFrom(NavigationContext navigationContext)
	{
	}
}
~~~
```

```ad-note
collapse: close
title: ViewB.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.ViewB"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True">
    <StackPanel>
        <TextBlock Text="{Binding Title}" FontSize="48" HorizontalAlignment="Center" VerticalAlignment="Center" />
        <TextBlock Text="{Binding PageViews}" FontSize="24" HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </StackPanel>
</UserControl>
~~~
```

```ad-note
collapse: close
title: ViewBViewModel.cs
~~~csharp
using Prism.Mvvm;
using Prism.Regions;

namespace ModuleA.ViewModels;

public class ViewBViewModel : BindableBase, INavigationAware
{
	private string _title = "ViewB";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	private int _pageViews;
	public int PageViews
	{
		get { return _pageViews; }
		set { SetProperty(ref _pageViews, value); }
	}

	public void OnNavigatedTo(NavigationContext navigationContext)
	{
		PageViews++;
	}

	public bool IsNavigationTarget(NavigationContext navigationContext)
	{
		return true;
	}

	public void OnNavigatedFrom(NavigationContext navigationContext)
	{
	}
}
~~~
```

```ad-note
collapse: close
title: ModuleAModule.cs
~~~csharp
using Prism.Ioc;
using Prism.Modularity;

using ModuleA.Views;

namespace ModuleA;

public class ModuleAModule : IModule
{
	public void OnInitialized(IContainerProvider containerProvider)
	{
	}

	public void RegisterTypes(IContainerRegistry containerRegistry)
	{
		containerRegistry.RegisterForNavigation<ViewA>();
		containerRegistry.RegisterForNavigation<ViewB>();
	}
}
~~~
```

### NavigationParticipation
---
##### [App.xaml.cs] ModuleCatalog에 호출할 ModuleA 등록
##### [MainWindow.xaml] Naviation 매핑된 동작(Command) 정의, Navigation이 표시될 Region 영역 정의 (Tab control)
##### [MainWindowViewModel.cs] Navigation Command에 regionManager의 Navigation 응답 매핑

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="NavigationParticipation.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
    <Window.Resources>
        <Style TargetType="TabItem">
            <Setter Property="Header" Value="{Binding DataContext.Title}" />
        </Style>
    </Window.Resources>
    <DockPanel LastChildFill="True">
        <StackPanel Orientation="Horizontal" DockPanel.Dock="Top" Margin="5" >
            <Button Command="{Binding NavigateCommand}" CommandParameter="ViewA" Margin="5">Navigate to View A</Button>
            <Button Command="{Binding NavigateCommand}" CommandParameter="ViewB" Margin="5">Navigate to View B</Button>
        </StackPanel>
        <TabControl prism:RegionManager.RegionName="ContentRegion" Margin="5"  />
    </DockPanel>
</Window>
~~~
```

```ad-note
collapse: open
title: MainWindowViewModel.cs
~~~csharp
using Prism.Commands;
using Prism.Mvvm;
using Prism.Regions;

namespace NavigationParticipation.ViewModels;

public class MainWindowViewModel : BindableBase
{
	private readonly IRegionManager _regionManager;

	private string _title = "Prism Unity Application";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	public DelegateCommand<string> NavigateCommand { get; private set; }

	public MainWindowViewModel(IRegionManager regionManager)
	{
		_regionManager = regionManager;

		NavigateCommand = new DelegateCommand<string>(Navigate);
	}

	private void Navigate(string navigatePath)
	{
		if (navigatePath != null)
			_regionManager.RequestNavigate("ContentRegion", navigatePath);
	}
}
~~~
```

```ad-note
collapse: close
title: App.xaml.cs
~~~csharp
using Prism.Ioc;
using Prism.Modularity;
using Prism.Unity;

using NavigationParticipation.Views;

namespace NavigationParticipation;

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
