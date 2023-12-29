
# ✍️ How to use

### 🧩UsingInvokeCommandAction
---
* Default (PrismApplication)
```ad-note
collapse: close
title: App.xaml
~~~csharp
<prism:PrismApplication x:Class="UsingInvokeCommandAction.App"
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
using Prism.Unity;
using Prism.Ioc;

using UsingInvokeCommandAction.Views;

namespace UsingInvokeCommandAction;

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

* 🚩ListBox.Interaction.Triggers - InvokeCommandAction Command="{Binding SelectedCommand}"
```ad-note
collapse: close
title: MainWindow.xaml
~~~csharp
<Window x:Class="UsingInvokeCommandAction.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        <StackPanel Grid.Row="0">
            <TextBlock Margin="5" FontSize="24" Foreground="DarkRed" TextWrapping="Wrap">InvokeCommandAction</TextBlock>
            <TextBlock Margin="5" TextWrapping="Wrap">The <Bold>InvokeCommandAction</Bold> is useful when you need to invoke a command in response to an event raised by a control in the view.</TextBlock>
            <TextBlock Margin="5" TextWrapping="Wrap">
                In the following example there is a list of items and we want to execute a command in the view model when an item is selected. 
                The view model will then change the "Selected Item" shown below.
            </TextBlock>
        </StackPanel>
        <ListBox Grid.Row="1" Margin="5" ItemsSource="{Binding Items}" SelectionMode="Single">
            <i:Interaction.Triggers>
                <!-- This event trigger will execute the action when the corresponding event is raised by the ListBox. -->
                <i:EventTrigger EventName="SelectionChanged">
                    <!-- This action will invoke the selected command in the view model and pass the parameters of the event to it. -->
                    <prism:InvokeCommandAction Command="{Binding SelectedCommand}" TriggerParameterPath="AddedItems" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </ListBox>
        <StackPanel Grid.Row="2" Margin="5" Orientation="Horizontal">
            <TextBlock Foreground="DarkRed" FontWeight="Bold">Selected Item:</TextBlock>
            <TextBlock AutomationProperties.AutomationId="SelectedItemTextBlock" Foreground="DarkRed" FontWeight="Bold" Margin="5,0" Text="{Binding SelectedItemText}" />
        </StackPanel>
    </Grid>
</Window>
~~~
```

* Default
```ad-note
collapse: close
title: MainWindow.xaml.cs
~~~csharp
namespace UsingInvokeCommandAction.Views;

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

* 🚩SelectedCommand - OnItemSelected
```ad-note
collapse: close
title: MainWindowViewModel.cs
~~~csharp
using Prism.Commands;
using Prism.Mvvm;

namespace UsingInvokeCommandAction.ViewModels;

public class MainWindowViewModel : BindableBase
{
	private string _title = "Prism Unity Application";
	public string Title
	{
		get { return _title; }
		set { SetProperty(ref _title, value); }
	}

	private string _selectedItemText;
	public string SelectedItemText
	{
		get { return _selectedItemText; }
		private set { SetProperty(ref _selectedItemText, value); }
	}

	public IList<string> Items { get; private set; }

	public DelegateCommand<object[]> SelectedCommand { get; private set; }

	public MainWindowViewModel()
	{
		Items = new List<string>();

		Items.Add("Item1");
		Items.Add("Item2");
		Items.Add("Item3");
		Items.Add("Item4");
		Items.Add("Item5");

		// This command will be executed when the selection of the ListBox in the view changes.
		SelectedCommand = new DelegateCommand<object[]>(OnItemSelected);
	}

	private void OnItemSelected(object[] selectedItems)
	{
		if (selectedItems != null && selectedItems.Count() > 0)
		{
			SelectedItemText = selectedItems.FirstOrDefault().ToString();
		}
	}
}
~~~
```
