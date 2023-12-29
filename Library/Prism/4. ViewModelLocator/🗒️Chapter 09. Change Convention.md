# ✍️ How to use

### ViewModelLocator
---
##### [MainWindow.xaml] Window tag에 ViewModelLocator 설정 - 별도로 View(MainWindow)와 ViewModel(MainWindowViewModel)을 매핑하지 않아도 연동
* ViewModel 명은 네이밍 규칙(xaml 명칭에 "ViewModel" Subfix)에 위배되지 않아야 적용 가능
##### [MainWindowViewModel.cs] BindableBase를 상속하여 프로퍼티의 변경점 대응
##### [App.xaml.cs] ViewModelLocator에 View/ViewModel 설정값을 매핑

```ad-note
collapse: close
title: MainWindowViewModel.cs
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

		ViewModelLocationProvider.SetDefaultViewTypeToViewModelTypeResolver((viewType) =>
		{
			//var viewName = viewType.FullName; // when View & ViewModel is same location(folder)
			var viewName = viewType.FullName.Replace("Views", "ViewModels"); // change location(folder)
			
			var viewAssemblyName = viewType.GetTypeInfo().Assembly.FullName;
			var viewModelName = $"{viewName}ViewModel, {viewAssemblyName}";
			return Type.GetType(viewModelName);
		});
	}
}
~~~
```
