##### 객체가 존재하는지 체크 interface

```ad-info
title: IRegionMemberLifetime Interface
collapse: close
~~~csharp
namespace Prism.Region;

/// <summary>
/// When implemented, allows an instance placed in a Prism.Regions.IRegion that uses a Prism.Regions.Behaviors.RegionMemberLifetimeBehavior to indicate in should be removed
/// when it transitions from an activated to deactivated state.
/// </summary>
public interface IRegionMemberLifetime
{
	/// <summary>
	/// Gets a value indicating whether this instance should be kept-alive upon deactivation.
	/// </summary>
	bool KeepAlive { get; }
}
~~~
```

# ✍️ How to use

### ModuleA
---
##### [ViewAViewModel.cs] IRegionMemberLifetime 구현
##### [ModuleAModule.cs] Navigation을 위한 컨테이너 등록 (ViewA, ViewB)

```ad-note
collapse: close
title: ViewA.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.ViewA"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True">
    <Grid>
        <TextBlock Text="ViewA" FontSize="48" HorizontalAlignment="Center" VerticalAlignment="Center" />
    </Grid>
</UserControl>
~~~
```

```ad-note
collapse: open
title: ViewAViewModel.cs
~~~csharp
using Prism.Mvvm;

namespace ModuleA.ViewModels;

public class ViewAViewModel : BindableBase, INavigationAware, IRegionMemberLifetime
{
	public bool KeepAlive
	{
		get
		{
			return false;
		}
	}
	
	public bool IsNavigationTarget(NavigationContext navigationContext)
	{
		return false;
	}
	
	public void OnNavigatedFrom(NavigationContext navigationContext)
	{
		
	}
	
	public void OnNavigatedTo(NavigationContext navigationContext)
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
    <Grid>
        <TextBlock Text="ViewB" FontSize="48" HorizontalAlignment="Center" VerticalAlignment="Center" />
    </Grid>
</UserControl>
~~~
```

```ad-note
collapse: close
title: ViewBViewModel.cs
~~~csharp
using Prism.Mvvm;

namespace ModuleA.ViewModels;

public class ViewBViewModel : BindableBase, INavigationAware
{
	public bool IsNavigationTarget(NavigationContext navigationContext)
	{
		return false;
	}

	public void OnNavigatedFrom(NavigationContext navigationContext)
	{
		
	}

	public void OnNavigatedTo(NavigationContext navigationContext)
	{
		
	}
}
~~~
```

```ad-note
collapse: open
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

### RegionMemberLifetime
---
##### [App.xaml.cs] ModuleCatalog에 호출할 ModuleA 등록
##### [MainWindow.xaml] Naviation 매핑된 동작(Command) 정의, Navigation이 표시될 Region 영역 정의, Views property 영역 추가
* 객체 ViewA는 Views collection에 존재하지 않을 경우 추가
* 객체 ViewB는 ViewA가 존재할 경우 ViewB로 변경, ViewB가 존재하지 않을 경우 추가
##### [MainWindowViewModel.cs] Navigation Command에 regionManager의 Navigation 응답 매핑, Collection 변경 이벤트 추가

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="RegionMemberLifetime.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
    <DockPanel LastChildFill="True">
        <StackPanel Orientation="Horizontal" DockPanel.Dock="Top" Margin="5" >
            <Button Command="{Binding NavigateCommand}" CommandParameter="ViewA" Margin="5">Navigate to View A</Button>
            <Button Command="{Binding NavigateCommand}" CommandParameter="ViewB" Margin="5">Navigate to View B</Button>
        </StackPanel>
        <ItemsControl ItemsSource="{Binding Views}" Margin="5">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <Border Background="LightBlue" Height="50" Width="50" Margin="2">
                        <TextBlock Text="{Binding}" FontWeight="Bold" HorizontalAlignment="Center" VerticalAlignment="Center" />
                    </Border>
                </DataTemplate>                
            </ItemsControl.ItemTemplate>
        </ItemsControl>
        <ContentControl prism:RegionManager.RegionName="ContentRegion" Margin="5"  />
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

namespace RegionMemberLifetime.ViewModels;

public class MainWindowViewModel : BindableBase
{
	private readonly IRegionManager _regionManager;

	private string _title = "Prism Unity Application";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	private ObservableCollection<object> _views = new ObservableCollection<object>();
	public ObservableCollection<obejct> Views
	{
		get { return _views; }
		set { SetProperty(ref _views, value); }
	}
	
	public DelegateCommand<string> NavigateCommand { get; private set; }

	public MainWindowViewModel(IRegionManager regionManager)
	{
		_regionManager = regionManager;
		_regionManager.Regions.CollectionChanged += Regions_CollectionChanged;

		NavigateCommand = new DelegateCommand<string>(Navigate);
	}

	private void Navigate(string navigatePath)
	{
		if (navigatePath != null)
			_regionManager.RequestNavigate("ContentRegion", navigatePath);
	}

	private void Regions_CollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
	{
		if (e.Action == NotifyCollectionChangedAction.Add)
		{
			var region = (IRegion)e.NewItems[0];
			region.Views.CollectionChanged += Views_CollectionChanged;
		}
	}

	private void Views_CollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
	{
		if (e.Action == NotifyCollectionChangedAction.Add)
		{
			Views.Add(e.NewItems[0].GetType().Name);
		}
		else if (e.Action == NotifyCollectionChangedAction.Remove)
		{
			Views.Remove(e.OldItems[0].GetType().Name);
		}
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

using RegionMemberLifetime.Views;

namespace RegionMemberLifetime;

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
