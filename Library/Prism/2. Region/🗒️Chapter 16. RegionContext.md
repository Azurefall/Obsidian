# ✍️ How to use

### ModuleA
---
##### [Person.cs] Data model class
##### [PersonList.xaml] 현 모듈 내 상세정보 표시 영역인 Region("PersonDetailRegion")을 설정, Binding 정의
##### [PersonDetail.xaml] "PersonDetailRegion" 에 표시되는 형식
##### [PersonDetail.xaml.cs] "PersonDetailRegion" 에 Binding 전달하는 ResionContext 설정 - PropertyChanged Event 매핑
##### [ModuleAModule.cs] 적용될 표시 영역인 Region("ContentRegion")와 Region("PersonDetailRegion")을 등록

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
             prism:ViewModelLocator.AutoWireViewModel="True">
    <Grid x:Name="LayoutRoot" Background="White" Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="100"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <ListBox x:Name="_listOfPeople" ItemsSource="{Binding People}"/>
        <ContentControl Grid.Row="1" Margin="10"
                        prism:RegionManager.RegionName="PersonDetailsRegion"
                        prism:RegionManager.RegionContext="{Binding SelectedItem, ElementName=_listOfPeople}"/>
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

public class PersonListViewModel : BindableBase
{
	private ObservableCollection<Person> _people;
	public ObservableCollection<Person> People
	{
		get { return _people; }
		set { SetProperty(ref _people, value); }
	}

	public PersonListViewModel()
	{
		CreatePeople();
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
    </Grid>
</UserControl>
~~~
```

```ad-note
collapse: open
title: PersonDetail.xaml.cs
~~~csharp
using Prism.Common;
using Prism.Regions;

using ModuleA.Models;
using ModuleA.ViewModels;

namespace ModuleA.Views;

/// <summary>
/// Interaction logic for PersonDetail
/// </summary>
public partial class PersonDetail : UserControl
{
	public PersonDetail()
	{
		InitializeComponent();
		RegionContext.GetObservableContext(this).PropertyChanged += PersonDetail_PropertyChanged;
	}

	private void PersonDetail_PropertyChanged(object sender, System.ComponentModel.PropertyChangedEventArgs e)
	{
		var context = (ObservableObject<object>)sender;
		var selectedPerson = (Person)context.Value;
		(DataContext as PersonDetailViewModel).SelectedPerson = selectedPerson;
	}
}
~~~
```

```ad-note
collapse: close
title: PersonDetailViewModel.cs
~~~csharp
using Prism.Mvvm;

using ModuleA.Models;

namespace ModuleA.ViewModels;

public class PersonDetailViewModel : BindableBase
{
	private Person _selectedPerson;
	public Person SelectedPerson
	{
		get { return _selectedPerson; }
		set { SetProperty(ref _selectedPerson, value); }
	}

	public PersonDetailViewModel()
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
using Prism.Regions;

using ModuleA.Views;

namespace ModuleA;

public class ModuleAModule : IModule
{
	public void OnInitialized(IContainerProvider containerProvider)
	{
		var regionManager = containerProvider.Resolve<IRegionManager>();
		regionManager.RegisterViewWithRegion("ContentRegion", typeof(PersonList));
        regionManager.RegisterViewWithRegion("PersonDetailsRegion", typeof(PersonDetail));
	}

	public void RegisterTypes(IContainerRegistry containerRegistry)
	{
	}
}
~~~
```

### RegionContext
---
##### [App.xaml.cs] ModuleCatalog에 호출할 ModuleA 등록
##### [MainWindow.xaml] Window tag 에 ViewModelLocator와 Title Binding
* RegionManager에 영역("ContentRegion") 설정

```ad-note
collapse: open
title: MainWindow.xaml
~~~csharp
<Window x:Class="Modules.Views.MainWindow"
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
collapse: open
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
