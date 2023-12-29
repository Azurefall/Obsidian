
# ✍️ How to use

### 🧩UsingDelegateCommands
---
* Default (PrismApplication)
```ad-note
collapse: close
title: App.xaml
~~~csharp
<prism:PrismApplication x:Class="UsingDelegateCommands.App"
                        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                        xmlns:prism="http://prismlibrary.com/"
                        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Application.Resources>
         
    </Application.Resources>
</prism:PrismApplication>
~~~
```

* CreateShell - MainWindow
```ad-note
collapse: close
title: App.xaml.cs
~~~csharp
/// <summary>
/// Interaction logic for App.xaml
/// </summary>
public partial class App
{
	protected override Window CreateShell()
	{
		return Container.Resolve<MainWindow>();
	}
}
~~~
```

* 🚩Button Command="{Binding ExecuteDelegateCommand}"
* 🚩Button Command="{Binding DelegateCommandObservesProperty}"
* 🚩Button Command="{Binding DelegateCommandObservesCanExecute}"
* 🚩Button Command="{Binding ExecuteGenericDelegateCommand}"
```ad-note
collapse: close
title: MainWindow.xaml
~~~csharp
<Window x:Class="UsingDelegateCommands.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="Using DelegateCommand" Width="350" Height="275">
    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
        <CheckBox IsChecked="{Binding IsEnabled}" Content="Can Execute Command" Margin="10"/>
        <Button Command="{Binding ExecuteDelegateCommand}" Content="DelegateCommand" Margin="10"/>
        <Button Command="{Binding DelegateCommandObservesProperty}" Content="DelegateCommand ObservesProperty" Margin="10"/>
        <Button Command="{Binding DelegateCommandObservesCanExecute}" Content="DelegateCommand ObservesCanExecute" Margin="10"/>
        <Button Command="{Binding ExecuteGenericDelegateCommand}" CommandParameter="Passed Parameter" Content="DelegateCommand Generic" Margin="10"/>
        <TextBlock Text="{Binding UpdateText}" Margin="10" FontSize="22"/>
    </StackPanel>
</Window>
~~~
```

* Default
```ad-note
collapse: close
title: MainWindow.xaml.cs
~~~csharp
namespace UsingDelegateCommands.Views;

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

* 🚩ExecuteDelegateCommand - Execute, CanExecute
* 🚩DelegateCommandObservesProperty - Execute, CanExecute - ObservesProperty
* 🚩DelegateCommandObservesCanExecute - Execute - ObservesCanExecute
* 🚩ExecuteGenericDelegateCommand - ExecuteGeneric - ObservesCanExecute
```ad-note
collapse: close
title: MainWindowViewModel.cs
~~~csharp
public class MainWindowViewModel : BindableBase
{
	private bool _isEnabled;
	public bool IsEnabled
	{
		get { return _isEnabled; }
		set
		{
			SetProperty(ref _isEnabled, value);
			ExecuteDelegateCommand.RaiseCanExecuteChanged();
		}
	}

	private string _updateText;
	public string UpdateText
	{
		get { return _updateText; }
		set { SetProperty(ref _updateText, value); }
	}

	public DelegateCommand ExecuteDelegateCommand { get; private set; }
	public DelegateCommand<string> ExecuteGenericDelegateCommand { get; private set; }        
	public DelegateCommand DelegateCommandObservesProperty { get; private set; }
	public DelegateCommand DelegateCommandObservesCanExecute { get; private set; }

	public MainWindowViewModel()
	{
		ExecuteDelegateCommand = new DelegateCommand(Execute, CanExecute);
		DelegateCommandObservesProperty = new DelegateCommand(Execute, CanExecute).ObservesProperty(() => IsEnabled);
		DelegateCommandObservesCanExecute = new DelegateCommand(Execute).ObservesCanExecute(() => IsEnabled);
		ExecuteGenericDelegateCommand = new DelegateCommand<string>(ExecuteGeneric).ObservesCanExecute(() => IsEnabled);
	}

	private void Execute()
	{
		UpdateText = $"Updated: {DateTime.Now}";
	}

	private void ExecuteGeneric(string parameter)
	{
		UpdateText = parameter;
	}

	private bool CanExecute()
	{
		return IsEnabled;
	}
}
~~~
```
