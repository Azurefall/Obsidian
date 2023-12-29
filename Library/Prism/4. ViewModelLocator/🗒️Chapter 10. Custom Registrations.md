# ✍️ How to use

### ViewModelLocator
---
##### [App.xaml.cs] ViewModelLocationProvider에 ViewModel Generic View-generic ViewModel 매핑
##### ※ type-type / type-factory / generic-factory 매칭도 가능

```ad-note
collapse: close
title: CustomViewModel.cs
~~~csharp
using Prism.Mvvm;

namespace ViewModelLocator.ViewModels;

public class MainWindowViewModel : BindableBase
{
	private string _title = "Prism Unity Application";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	public MainWindowViewModel()
	{
	}
}
~~~
```

```ad-note
collapse: close
title: MainWindow.xaml
~~~csharp
<Window x:Class="ViewModelLocator.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
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
using Prism.Ioc;
using Prism.Mvvm;
using Prism.Unity;

using ViewModelLocator.ViewModels;
using ViewModelLocator.Views;

namespace ViewModelLocator;

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

	protected override void ConfigureViewModelLocator()
	{
		base.ConfigureViewModelLocator();

		// type / type
		//ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), typeof(CustomViewModel));

		// type / factory
		//ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), () => Container.Resolve<CustomViewModel>());

		// generic factory
		//ViewModelLocationProvider.Register<MainWindow>(() => Container.Resolve<CustomViewModel>());

		// generic type
		ViewModelLocationProvider.Register<MainWindow, CustomViewModel>();
	}
}
~~~
```
