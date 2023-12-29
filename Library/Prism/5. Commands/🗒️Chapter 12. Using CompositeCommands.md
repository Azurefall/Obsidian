##### 다중 View의 각 DelegateCommand를 동시에 컨트롤하는 Command Class
					 CompositeCommand.SaveCommand
								실행
		┏━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
	TabA.SaveDelegateCommand TabB.SaveDelegateCommand TabC.SaveDelegateCommand
		변경                       변경                        변경

# ✍️ How to use

### 🧩UsingCompositeCommands.Core
---
```ad-note
collapse: close
title: IApplicationCommands.cs

~~~csharp
public interface IApplicationCommands
{
	CompositeCommand SaveCommand { get; }
}
```

* CompositeCommand Wrapper
```ad-note
collapse: close
title: ApplicationCommands.cs

~~~csharp
public class ApplicationCommands : IApplicationCommands
{
	private CompositeCommand _saveCommand = new CompositeCommand();
	public CompositeCommand SaveCommand
	{
		get { return _saveCommand; }
	}
}
```


### 🧩UsingCompositeCommands
---
* Default (PrismApplication)
```ad-note
collapse: close
title: App.xaml
~~~csharp
<prism:PrismApplication x:Class="UsingCompositeCommands.App"
                        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                        xmlns:prism="http://prismlibrary.com/"
                        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Application.Resources>
         
    </Application.Resources>
</prism:PrismApplication>
~~~
```

* CreateShell - MainWindow
* ConfigureModuleCatalog - AddModule\<ModuleA.ModuleAModule\>
* 🚩RegisterTypes - RegisterSingleton\<IApplicationCommands, ApplicationCommands\>
```ad-note
collapse: close
title: App.xaml.cs
~~~csharp
/// <summary>
/// Interaction logic for App.xaml
/// </summary>
public partial class App : PrismApplication
{
	protected override Window CreateShell()
	{
		return Container.Resolve<MainWindow>();
	}

	protected override void ConfigureModuleCatalog(IModuleCatalog moduleCatalog)
	{
		moduleCatalog.AddModule<ModuleA.ModuleAModule>();
	}

	protected override void RegisterTypes(IContainerRegistry containerRegistry)
	{
		containerRegistry.RegisterSingleton<IApplicationCommands, ApplicationCommands>();
	}
}
~~~
```

* 🚩Button Command="{Binding ApplicationCommands.SaveCommand}"
* TabControl prism:RegionManager.RegionName="ContentRegion"
```ad-note
collapse: close
title: MainWindow.xaml
~~~csharp
<Window x:Class="UsingCompositeCommands.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:prism="http://prismlibrary.com/"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">

    <Window.Resources>
        <Style TargetType="TabItem">
            <Setter Property="Header" Value="{Binding DataContext.Title}" />
        </Style>
    </Window.Resources>
    
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <Button Content="Save" Margin="10" Command="{Binding ApplicationCommands.SaveCommand}"/>
        <TabControl Grid.Row="1" Margin="10" prism:RegionManager.RegionName="ContentRegion" />
    </Grid>
</Window>
~~~
```

* Default
```ad-note
collapse: close
title: MainWindow.xaml.cs
~~~csharp
namespace UsingCompositeCommands.Views;

/// <summary>
/// Interaction logic for MainWindow.xaml
/// </summary>
public partial class MainWindow : Window
{
	public MainWindow()
	{
		InitializeComponent();
	}
}
~~~
```

* 🚩ApplicationCommands
```ad-note
collapse: close
title: MainWindowViewModel.cs

~~~csharp
using Prism.Mvvm;

using UsingCompositeCommands.Core;

namespace UsingCompositeCommands.ViewModels;

public class MainWindowViewModel : BindableBase
{
	private string _title = "Prism Unity Application";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	private IApplicationCommands _applicationCommands;
	public IApplicationCommands ApplicationCommands
	{
		get { return _applicationCommands; }
		set { SetProperty(ref _applicationCommands, value); }
	}

	public MainWindowViewModel(IApplicationCommands applicationCommands)
	{
		ApplicationCommands = applicationCommands;
	}
}
~~~
```


### 🧩ModuleA
---
* 🚩IApplicationCommands.SaveCommand - RegisterCommand(UpdateCommand)
```ad-note
collapse: close
title: TabViewModel.cs
~~~csharp
using Prism.Commands;
using Prism.Mvvm;

using UsingCompositeCommands.Core;

namespace ModuleA.ViewModels;

public class TabViewModel : BindableBase
{
	IApplicationCommands _applicationCommands;

	private string _title;
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	private bool _canUpdate = true;
	public bool CanUpdate
	{
		get { return _canUpdate; }
		set { SetProperty(ref _canUpdate, value); }
	}

	private string _updatedText;
	public string UpdateText
	{
		get { return _updatedText; }
		set { SetProperty(ref _updatedText, value); }
	}

	public DelegateCommand UpdateCommand { get; private set; }

	public TabViewModel(IApplicationCommands applicationCommands)
	{
		_applicationCommands = applicationCommands;
		UpdateCommand = new DelegateCommand(Update).ObservesCanExecute(() => CanUpdate);
		_applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
	}

	private void Update()
	{
		UpdateText = $"Updated: {DateTime.Now}";
	}
}
```

* 🚩Button Command="{Binding UpdateCommand}"
```ad-note
collapse: close
title: TabView.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.TabView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True">
    <StackPanel VerticalAlignment="Center" HorizontalAlignment="Center">
        <TextBlock Text="{Binding Title}" FontSize="18" Margin="5"/>
        <CheckBox IsChecked="{Binding CanUpdate}" Content="Can Execute" Margin="5"/>
        <Button Command="{Binding UpdateCommand}" Content="Save" Margin="5"/>
        <TextBlock Text="{Binding UpdateText}"  Margin="5"/>
    </StackPanel>
</UserControl>
~~~
```

* Default
```ad-note
collapse: close
title: TabView.xaml.cs
~~~csharp
namespace ModuleA.Views;

/// <summary>
/// Interaction logic for TabView
/// </summary>
public partial class TabView : UserControl
{
	public TabView()
	{
		InitializeComponent();
	}
}
~~~
```

* RegionManager.Regions["ContentRegion"] - Add TabViews
```ad-note
collapse: close
title: ModuleAModule.cs
~~~csharp
using Prism.Ioc;
using Prism.Modularity;
using Prism.Regions;

using ModuleA.ViewModels;
using ModuleA.Views;

namespace ModuleA;

public class ModuleAModule : IModule
{
	public void OnInitialized(IContainerProvider containerProvider)
	{
		var regionManager = containerProvider.Resolve<IRegionManager>();
		IRegion region = regionManager.Regions["ContentRegion"];

		var tabA = containerProvider.Resolve<TabView>();
		SetTitle(tabA, "Tab A");
		region.Add(tabA);

		var tabB = containerProvider.Resolve<TabView>();
		SetTitle(tabB, "Tab B");
		region.Add(tabB);

		var tabC = containerProvider.Resolve<TabView>();
		SetTitle(tabC, "Tab C");
		region.Add(tabC);
	}

	void SetTitle(TabView tab, string title)
	{
		(tab.DataContext as TabViewModel).Title = title;
	}
}
```
