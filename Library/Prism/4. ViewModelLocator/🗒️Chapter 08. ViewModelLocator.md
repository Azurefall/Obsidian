# ✍️ How to use

### ViewModelLocator
---
※ ViewModel 명은 네이밍 규칙(xaml 명칭에 "ViewModel" Subfix)에 위배되지 않아야 적용 가능
##### [MainWindow.xaml] Window tag에 ViewModelLocator 설정 - 별도로 View(MainWindow)와 ViewModel(MainWindowViewModel)을 매핑하지 않아도 연동
##### [MainWindowViewModel.cs] BindableBase를 상속하여 프로퍼티의 변경점 대응

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
collapse: open
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
collapse: close
title: App.xaml.cs
~~~csharp
using Prism.Ioc;
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
}
~~~
```
