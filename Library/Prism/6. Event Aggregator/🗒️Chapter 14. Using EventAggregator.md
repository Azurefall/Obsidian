
# ✍️ How to use

### UsingEventAggregator.Core
---
##### PusSubEvent＜string＞ 상속
##### [MessageSentEvent.cs] 정의된 이벤트 Class를 Publish / Subscribe 한 객체간 공유(인터페이스)

```ad-note
collapse: open
title: MessageSentEvent.cs

~~~csharp
using Prism.Events;

namespace UsingEventAggregator.Core;

public class MessageSentEvent : PubSubEvent<string>
{
}
```


### UsingEventAggregator
---
##### [MainWindow.xaml] 좌/우 각각 Region 영역 ("LeftRegion"/"RightRegion") 생성
##### [App.xaml.cs] ModuleCatalog에 ModuleA/ModuleB 등록

```ad-note
collapse: close
title: MainWindowViewModel.cs

~~~csharp
using Prism.Mvvm;

namespace UsingEventAggregator.ViewModels

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
<Window x:Class="UsingEventAggregator.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:prism="http://prismlibrary.com/"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <ContentControl prism:RegionManager.RegionName="LeftRegion" />
        <ContentControl Grid.Column="1" prism:RegionManager.RegionName="RightRegion" />
    </Grid>
</Window>
~~~
```

```ad-note
collapse: close
title: App.xaml.cs
~~~csharp
using Prism.Ioc;
using Prism.Modularity;
using Prism.Unity;

using UsingEventAggregator.Views;

namespace UsingEventAggregator;

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
		moduleCatalog.AddModule<ModuleB.ModuleBModule>();
	}
}
~~~
```


### ModuleA
---
##### [ModuleAModule.cs] UsingEventAggregator에서 표시될 왼쪽 Region 영역("LeftRegion") 설정
##### [MessageViewModel.cs] 버튼 이벤트 배포

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
		regionManager.RegisterViewWithRegion("LeftRegion", typeof(MessageView));
	}

	public void RegisterTypes(IContainerRegistry containerRegistry)
	{
	}
}
```

```ad-note
collapse: close
title: MessageView.xaml
~~~csharp
<UserControl x:Class="ModuleA.Views.MessageView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True" Padding="25">
    <StackPanel>
        <TextBox Text="{Binding Message}" Margin="5"/>
        <Button Command="{Binding SendMessageCommand}" Content="Send Message" Margin="5"/>
    </StackPanel>
</UserControl>

~~~
```

```ad-note
collapse: open
title: MessageViewModel.cs
~~~csharp
using Prism.Commands;
using Prism.Events;
using Prism.Mvvm;

using UsingEventAggregator.Core;

namespace ModuleA.ViewModels;

public class MessageViewModel : BindableBase
{
	IEventAggregator _ea;

	private string _message = "Message to Send";
	public string Message
	{
		get { return _message; }
		set { SetProperty(ref _message, value); }
	}

	public DelegateCommand SendMessageCommand { get; private set; }

	public MessageViewModel(IEventAggregator ea)
	{
		_ea = ea;
		SendMessageCommand = new DelegateCommand(SendMessage);
	}

	private void SendMessage()
	{
		_ea.GetEvent<MessageSentEvent>().Publish(Message);
	}
}
```


### ModuleB
---
##### [ModuleAModule.cs] UsingEventAggregator에서 표시될 오른쪽 Region 영역("RightRegion") 설정
##### [MessageViewModel.cs] 버튼 이벤트 구독 (MessageReceived Action 등록) - 메시지 수신 시 Messages에 추가

```ad-note
collapse: close
title: ModuleBModule.cs
~~~csharp
using Prism.Ioc;
using Prism.Modularity;
using Prism.Regions;

using ModuleB.Views;

namespace ModuleB;

public class ModuleAModule : IModule
{
	public void OnInitialized(IContainerProvider containerProvider)
	{
		var regionManager = containerProvider.Resolve<IRegionManager>();
		regionManager.RegisterViewWithRegion("LeftRegion", typeof(MessageView));
	}

	public void RegisterTypes(IContainerRegistry containerRegistry)
	{
	}
}
```

```ad-note
collapse: close
title: MessageList.xaml
~~~csharp
<UserControl x:Class="ModuleB.Views.MessageList"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:prism="http://prismlibrary.com/"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             prism:ViewModelLocator.AutoWireViewModel="True" Padding="25">
    <Grid>
        <ListBox ItemsSource="{Binding Messages}" />
    </Grid>
</UserControl>
~~~
```

```ad-note
collapse: open
title: MessageListViewModel.cs
~~~csharp
using Prism.Events;
using Prism.Mvvm;

using UsingEventAggregator.Core;

namespace ModuleB.ViewModels;

public class MessageListViewModel : BindableBase
{
	IEventAggregator _ea;

	private ObservableCollection<string> _messages;
	public ObservableCollection<string> Messages
	{
		get { return _messages; }
		set { SetProperty(ref _messages, value); }
	}

	public MessageListViewModel(IEventAggregator ea)
	{
		_ea = ea;
		Messages = new ObservableCollection<string>();

		_ea.GetEvent<MessageSentEvent>().Subscribe(MessageReceived);
	}

	private void MessageReceived(string message)
	{
		Messages.Add(message);
	}
}
~~~
```
