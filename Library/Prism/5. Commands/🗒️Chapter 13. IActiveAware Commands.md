##### 다중 View의 각 DelegateCommand에 개별로 동기화(각각의 설정 저장)되는 공용 컨트롤 Interface
     Case 1)           CompositeCommand.SaveCommand (TabB 활성화)
                                                            실행
                      ┏━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
     TabA.SaveDelegateCommand   TabB.SaveDelegateCommand   TabC.SaveDelegateCommand
                   미변경                     변경                       미변경

     Case 2)           CompositeCommand.SaveCommand (TabA 활성화)
                                                 버튼활성 → 버튼비활성
                     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
     TabA.SaveDelegateCommand   TabB.SaveDelegateCommand   TabC.SaveDelegateCommand
               버튼비활성(실행)            버튼활성                     버튼활성
    ※ TabB or TabC로 이동 시 CompositeCommand 버튼은 활성화된 상태로 변경

# ✍️ How to use

### UsingCompositeCommands.Core
---
##### 컨트롤할 명령 정의
* CompositeCommand 생성 시 bool:true 선언 (Chap 12에서는 빈 생성자로 선언 - Chap 12 문서 내 CompositeCommand Class 참고)

```ad-note
collapse: close
title: IApplicationCommands.cs

~~~csharp
public interface IApplicationCommands
{
	CompositeCommand SaveCommand { get; }
}
```

```ad-note
collapse: open
title: ApplicationCommands.cs

~~~csharp
public class ApplicationCommands : IApplicationCommands
{
	private CompositeCommand _saveCommand = new CompositeCommand(true);
	public CompositeCommand SaveCommand
	{
		get { return _saveCommand; }
	}
}
```


### UsingCompositeCommands
---
##### 동시 컨트롤할 명령을 매핑
* App 실행 시  명령을 등록 (DI : App.xaml)
* MainWindowViewModel 생성 시 명령어 매핑

```ad-note
collapse: open
title: MainWindowViewModel.cs

~~~csharp
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
```

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


### ModuleA
---
##### 동시 컨트롤되는 각 명령을 매핑
* 모듈이 호출될 때 TabView x 3 생성 (ModuleAModule)
* IActiveAware Interface 상속
* TabView 생성 시 동시 컨트롤러에 각 DelegateCommand를 매핑

```ad-note
collapse: open
title: TabViewModel.cs
~~~csharp
public class TabViewModel : BindableBase, IActiveAware
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

	#region IActiveAware
	
	bool _isActive;
	public bool IsActive
	{
		get { return _isActive; }
		set
		{
			_isActive = value;
			OnIsActiveChanged();
		}
	}

	private void OnIsActiveChanged()
	{
		UpdateCommand.IsActive = IsActive;
		IsActiveChanged?.Invoke(this, new EventArgs());
	}

	public event EventHandler IsActiveChanged;

	#endregion IActiveAware
}
```

```ad-note
collapse: close
title: TabView.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.TabView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:prism="http://prismlibrary.com/"             
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

```ad-note
collapse: close
title: ModuleAModule.cs
~~~csharp
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


### ※ 추가사항
---
##### IActiveAware 적용 상태에서 CompositeCommand 생성 시 Chap 12와 같이 빈 생성자로 선언할 경우, 모든 Tab의 버튼이 활성화 된 경우에만 CompositeCommand도 활성화 된다.