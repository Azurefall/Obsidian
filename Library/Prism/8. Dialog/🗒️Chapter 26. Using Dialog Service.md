
# ✍️ How to use

### 🧩UsingDialogService
---
* Default (PrismApplication)
```ad-note
collapse: close
title: App.xaml
~~~csharp
<prism:PrismApplication x:Class="UsingDialogService.App"
                        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                        xmlns:prism="http://prismlibrary.com/"
                        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Application.Resources>
         
    </Application.Resources>
</prism:PrismApplication>
~~~
```

* CreateShell - MainWindow
* 🚩RegisterTypes - RegisterDialog\<NotificationDialog, NotificationDialogViewModel\>
```ad-note
collapse: close
title: App.xaml.cs
~~~csharp
using Prism.Modularity;
using Prism.Unity;
using Prism.Ioc;

using UsingDialogService.ViewModels;
using UsingDialogService.Views;

namespace UsingDialogService;

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
		containerRegistry.RegisterDialog<NotificationDialog, NotificationDialogViewModel>();
	}
}
~~~
```

* Button Command="{Binding Path=ShowDialogCommand}"
```ad-note
collapse: close
title: MainWindow.xaml
~~~csharp
<Window x:Class="UsingDialogService.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525" >
    <Button Command="{Binding Path=ShowDialogCommand}" Content="Show Dialog" HorizontalAlignment="Left" VerticalAlignment="Top" Margin="5" Width="75" Height="25" />
</Window>
~~~
```

* Default
```ad-note
collapse: close
title: MainWindow.xaml.cs
~~~csharp
namespace UsingDialogService.Views;

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

* 🚩IDialogService 를 통한 ShowDialog
* 🚩DialogParameters 전달
```ad-note
collapse: close
title: MainWindowViewModel.cs
~~~csharp
using Prism.Commands;
using Prism.Mvvm;
using Prism.Services.Dialogs;

namespace UsingDialogService.ViewModels;

public class MainWindowViewModel : BindableBase
{
	private DelegateCommand _showDialogCommand;
	public DelegateCommand ShowDialogCommand =>
		_showDialogCommand ?? (_showDialogCommand = new DelegateCommand(ShowDialog));

	private string _title = "Prism Application";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	private IDialogService _dialogService;

	public MainWindowViewModel(IDialogService dialogService)
	{
		_dialogService = dialogService;
	}

	private void ShowDialog()
	{
		var message = "This is a message that should be shown in the dialog.";
		//using the dialog service as-is
		_dialogService.ShowDialog("NotificationDialog", new DialogParameters($"message={message}"), r =>
		{
			if (r.Result == ButtonResult.None)
				Title = "Result is None";
			else if (r.Result == ButtonResult.OK)
				Title = "Result is OK";
			else if (r.Result == ButtonResult.Cancel)
				Title = "Result is Cancel";
			else
				Title = "I Don't know what you did!?";
		});
	}
}
~~~
```

* Button Command="{Binding CloseDialogCommand}" (OK / Cancel)
```ad-note
collapse: close
title: NotificationDialog.xaml
~~~csharp
<UserControl x:Class="UsingDialogService.Views.NotificationDialog"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True"
             Width="300" Height="150">
    <Grid x:Name="LayoutRoot" Margin="5">
        <Grid.RowDefinitions>
            <RowDefinition />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        <TextBlock Text="{Binding Message}" HorizontalAlignment="Stretch" VerticalAlignment="Stretch" Grid.Row="0" TextWrapping="Wrap" />
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right" Margin="0,10,0,0" Grid.Row="1" >
            <Button Command="{Binding CloseDialogCommand}" CommandParameter="true" Content="OK" Width="75" Height="25" IsDefault="True" />
            <Button Command="{Binding CloseDialogCommand}" CommandParameter="false" Content="Cancel" Width="75" Height="25" Margin="10,0,0,0" IsCancel="True" />
        </StackPanel>
    </Grid>
</UserControl>
~~~
```

* Default
```ad-note
collapse: close
title: NotificationDialog.xaml.cs
~~~csharp
namespace UsingDialogService.Views;

/// <summary>
/// Interaction logic for NotificationDialog.xaml
/// </summary>
public partial class NotificationDialog : UserControl
{
	public NotificationDialog()
	{
		InitializeComponent();
	}
}
~~~
```

* 🚩IDialogAware 상속
* 🚩CloseDialog → RaiseRequestClose → Invoke RequestClose = DialogService.ShowDialog()의 3번째 Action으로 callback → IDialogResult 응답
```ad-note
collapse: close
title: NotificationDialogViewModel.cs
~~~csharp
using Prism.Commands;
using Prism.Mvvm;
using Prism.Services.Dialogs;

namespace UsingDialogService.ViewModels;

public class NotificationDialogViewModel : BindableBase, IDialogAware
{
	private DelegateCommand<string> _closeDialogCommand;
	public DelegateCommand<string> CloseDialogCommand =>
		_closeDialogCommand ?? (_closeDialogCommand = new DelegateCommand<string>(CloseDialog));

	private string _message;
	public string Message
	{
		get { return _message; }
		set { SetProperty(ref _message, value); }
	}

	protected virtual void CloseDialog(string parameter)
	{
		ButtonResult result = ButtonResult.None;

		if (parameter?.ToLower() == "true")
			result = ButtonResult.OK;
		else if (parameter?.ToLower() == "false")
			result = ButtonResult.Cancel;

		RaiseRequestClose(new DialogResult(result));
	}

	public virtual void RaiseRequestClose(IDialogResult dialogResult)
	{
		RequestClose?.Invoke(dialogResult);
	}

	#region IDialogAware

	private string _title = "Notification";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	public event Action<IDialogResult> RequestClose;

	public virtual bool CanCloseDialog()
	{
		return true;
	}

	public virtual void OnDialogClosed()
	{

	}

	public virtual void OnDialogOpened(IDialogParameters parameters)
	{
		Message = parameters.GetValue<string>("message");
	}

	#endregion IDialogAware
}
~~~
```
