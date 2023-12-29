
# ✍️ How to use

### 🧩ModuleA
---
##### [Person.cs] Data model class
##### [ModuleAModule.cs]
* "ContentRegion"에 PersonList 매핑
* 등록 container에 PersonList navigation 등록
* 등록 container에 PersionDetail navigation 등록
##### [PersonList / PersonDetail] Key point : IRegionNavigationJournal
| View              | Desc                  | Control              | Mapping property         | Location                 | Key point |
| ----------------- | --------------------- | -------------------- | ------------------------ | ------------------------ | --------- |
| PersonList.xaml   | Person 목록 표시      | ListBox              | People                   | PersonListViewModel.cs   |           |
|                   |                       | ListBox.EventTrigger | PersonSelectedCommand    |                          |           |
|                   |                       | Button               | GoForwardCommand         |                          | 🚩        |
| PersonDetail.xaml | Person 상세 내용 표시 | TextBlock            | SelectedPerson.FirstName | PersonDetailViewModel.cs |           |
|                   |                       | TextBlock            | SelectedPerson.LastName  |                          |           |
|                   |                       | TextBlock            | SelectedPerson.Age       |                          |           |
|                   |                       | Butoon               | GoBackCommand            |                          | 🚩        |

```ad-note
collapse: close
title: Person.cs
~~~csharp
namespace ModuleA.Models;

public class Person : INotifyPropertyChanged
{
	#region Properties

	private string _firstName;
	public string FirstName
	{
		get { return _firstName; }
		set
		{
			_firstName = value;
			OnPropertyChanged();
		}
	}

	private string _lastName;
	public string LastName
	{
		get { return _lastName; }
		set
		{
			_lastName = value;
			OnPropertyChanged();
		}
	}

	private int _age;
	public int Age
	{
		get { return _age; }
		set
		{
			_age = value;
			OnPropertyChanged();
		}
	}

	private DateTime? _lastUpdated;
	public DateTime? LastUpdated
	{
		get { return _lastUpdated; }
		set
		{
			_lastUpdated = value;
			OnPropertyChanged();
		}
	}

	#endregion //Properties

	#region INotifyPropertyChanged

	public event PropertyChangedEventHandler PropertyChanged;
	protected void OnPropertyChanged([CallerMemberName]string propertyname = null)
	{
		PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyname));
	}

	#endregion //INotifyPropertyChanged

	public override string ToString()
	{
		return String.Format("{0}, {1}", LastName, FirstName);
	}
}
~~~
```

```ad-note
collapse: close
title: PersonList.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.PersonList"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
             prism:ViewModelLocator.AutoWireViewModel="True">
    <Grid x:Name="LayoutRoot" Background="White" Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="100"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <ListBox x:Name="_listOfPeople" ItemsSource="{Binding People}">
            <i:Interaction.Triggers>
                <i:EventTrigger EventName="SelectionChanged">
                    <prism:InvokeCommandAction Command="{Binding PersonSelectedCommand}" CommandParameter="{Binding SelectedItem, ElementName=_listOfPeople}" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </ListBox>
        <Button Command="{Binding GoForwardCommand}" Grid.Row="1" Margin="10" Width="75">Go Forward</Button>
    </Grid>
</UserControl>
~~~
```

```ad-note
collapse: close
title: PersonListViewModel.cs
~~~csharp
using Prism.Mvvm;

using ModuleA.Models;

namespace ModuleA.ViewModels;

public class PersonListViewModel : BindableBase, INavigationAware
{
	IRegionManager _regionManager;
	IRegionNavigationJournal _journal;

	private ObservableCollection<Person> _people;
	public ObservableCollection<Person> People
	{
		get { return _people; }
		set { SetProperty(ref _people, value); }
	}

	public DelegateCommand<Person> PersonSelectedCommand { get; private set; }
	public DelegateCommand GoForwardCommand { get; set; }

	public PersonListViewModel(IRegionManager regionManager)
	{
		_regionManager = regionManager;

		PersonSelectedCommand = new DelegateCommand<Person>(PersonSelected);
		CreatePeople();

		GoForwardCommand = new DelegateCommand(GoForward, CanGoForward);
	}

	private void PersonSelected(Person person)
	{
		var parameters = new NavigationParameters();
		parameters.Add("person", person);

		if (person != null)
			_regionManager.RequestNavigate("PersonDetailsRegion", "PersonDetail", parameters);
	}

	private void CreatePeople()
	{
		var people = new ObservableCollection<Person>();
		for (int i = 0; i < 10; i++)
		{
			people.Add(new Person()
			{
				FirstName = String.Format("First {0}", i),
				LastName = String.Format("Last {0}", i),
				Age = i
			});
		}

		People = people;
	}

	public void OnNavigatedTo(NavigationContext navigationContext)
	{
		_journal = navigationContext.NavigationService.Journal;
		GoForwardCommand.RaiseCanExecuteChanged();
	}

	public bool IsNavigationTarget(NavigationContext navigationContext)
	{
		return true;
	}

	public void OnNavigatedFrom(NavigationContext navigationContext)
	{
		
	}

	private void GoForward()
	{
		_journal.GoForward();
	}

	private bool CanGoForward()
	{
		return _journal != null && _journal.CanGoForward;
	}
}
~~~
```

```ad-note
collapse: close
title: PersonDetail.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.PersonDetail"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True">
    <Grid x:Name="LayoutRoot" Background="White">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- First Name -->
        <TextBlock Text="First Name:" Margin="5" />
        <TextBlock Grid.Column="1" Margin="5" Text="{Binding SelectedPerson.FirstName}" />

        <!-- Last Name -->
        <TextBlock Grid.Row="1" Text="Last Name:" Margin="5" />
        <TextBlock Grid.Row="1" Grid.Column="1"  Margin="5" Text="{Binding SelectedPerson.LastName}" />

        <!-- Age -->
        <TextBlock Grid.Row="2" Text="Age:" Margin="5"/>
        <TextBlock Grid.Row="2" Grid.Column="1"  Margin="5" Text="{Binding SelectedPerson.Age}"/>

        <Button Command="{Binding GoBackCommand}" Grid.Row="3" Grid.ColumnSpan="2" HorizontalAlignment="Center" Margin="10" Width="75">Go Back</Button>
    </Grid>
</UserControl>
~~~
```

```ad-note
collapse: close
title: PersonDetailViewModel.cs
~~~csharp
using Prism.Commands;
using Prism.Mvvm;
using Prism.Regions;

using ModuleA.Models;

namespace ModuleA.ViewModels;

public class PersonDetailViewModel : BindableBase, INavigationAware
{
	private Person _selectedPerson;
	public Person SelectedPerson
	{
		get { return _selectedPerson; }
		set { SetProperty(ref _selectedPerson, value); }
	}

	IRegionNavigationJournal _journal;

	public DelegateCommand GoBackCommand { get; set; }

	public PersonDetailViewModel()
	{
		GoBackCommand = new DelegateCommand(GoBack);
	}

	public void OnNavigatedTo(NavigationContext navigationContext)
	{
		_journal = navigationContext.NavigationService.Journal;

		var person = navigationContext.Parameters["person"] as Person;
		if (person != null)
			SelectedPerson = person;
	}

	public bool IsNavigationTarget(NavigationContext navigationContext)
	{
		var person = navigationContext.Parameters["person"] as Person;
		if (person != null)
			return SelectedPerson != null && SelectedPerson.LastName == person.LastName;
		else
			return true;
	}

	public void OnNavigatedFrom(NavigationContext navigationContext)
	{
	}

	private void GoBack()
	{
		_journal.GoBack();
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
using Prism.Regions;

using ModuleA.Views;

namespace ModuleA;

public class ModuleAModule : IModule
{
	public void OnInitialized(IContainerProvider containerProvider)
	{
		var regionManager = containerProvider.Resolve<IRegionManager>();
		regionManager.RequestNavigate("ContentRegion", "PersonLIst");
	}

	public void RegisterTypes(IContainerRegistry containerRegistry)
	{
		containerRegistry.RegisterForNavigation<PersonList>();
		containerRegistry.RegisterForNavigation<PersonDetail>();
	}
}
~~~
```


### 🧩NavigationJournal
---
##### [App.xaml.cs]
* ModuleCatalog에 호출할 ModuleA 등록
##### [MainWindow.xaml]
* 모듈 표시 영역 영역("ContentRegion") 등록

```ad-note
collapse: close
title: MainWindow.xaml
~~~csharp
<Window x:Class="NavigationJournal.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
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
using Prism.Modularity;
using Prism.Unity;
using Prism.Ioc;

using RegionContext.Views;

namespace RegionContext;

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
