##### 모든 활성화 View 반 Collection 

```ad-info
title: AllActiveRegion Class
collapse: close
~~~csharp
namespace Prism.Region;

/// <summary>
/// Region that keeps all the views in it as active. Deactivation of views is not allowed.
/// </summary>
public class AllActiveRegion : Region
{
	/// <summary>
	/// Gets a readonly view of the collection of all the active views in the region. These are all the added views.
	/// </summary>
	/// <value>An <see cref="IViewsCollection"/> of all the active views.</value>
	public override IViewsCollection ActiveViews => Views;

	/// <summary>
	/// Deactive is not valid in this Region. This method will always throw <see cref="InvalidOperationException"/>.
	/// </summary>
	/// <param name="view">The view to deactivate.</param>
	/// <exception cref="InvalidOperationException">Every time this method is called.</exception>
	public override void Deactivate(object view)
	{
		throw new InvalidOperationException(Resources.DeactiveNotPossibleException);
	}
}
~~~
```

# ✍️ How to use

### CustomRegions
---
##### App.xaml.cs에서 PrismApplication을 상속하여 App 실행 (RegionAdapter 사용)

```ad-note
collapse: open
title: StackPanelRegionAdapter.cs
~~~csharp
using Prism.Regions;

namespace Regions;

public class StackPanelRegionAdapter : RegionAdapterBase<StackPanel>
{
	/// <summary>
	/// Call from App.xaml.cs.ConfigureRegionAdapterMapping
	/// </summaray>
	public StackPanelRegionAdapter(IRegionBehaviorFactory regionBehaviorFactory)
		: base(regionBehaviorFactory)
	{

	}

	/// <summary>
	/// Call from RegionAdapterBase.Initialize (Second) 
	/// </summaray>
	protected override void Adapt(IRegion region, StackPanel regionTarget)
	{
		region.Views.CollectionChanged += (s, e) =>
		{
			if (e.Action == System.Collections.Specialized.NotifyCollectionChangedAction.Add)
			{
				foreach (FrameworkElement element in e.NewItems)
				{
					regionTarget.Children.Add(element);
				}
			}

			//handle remove
		};
	}

	/// <summary>
	/// Call from RegionAdapterBase.Initialize (First) 
	/// </summaray>
	protected override IRegion CreateRegion()
	{
		return new AllActiveRegion();
	}
}
~~~
```

```ad-note
collapse: close
title: MainWindow.xaml
~~~csharp
<Window x:Class="Regions.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:prism="http://prismlibrary.com/"
        Title="Shell" Height="350" Width="525">
    <Grid>
        <StackPanel prism:RegionManager.RegionName="ContentRegion" />
    </Grid>
</Window>
~~~
```

```ad-note
collapse: open
title: App.xaml.cs
~~~csharp
using Prism.Ioc;
using Prism.Regions;
using Prism.Unity;
using Regions.View;

namespace Regions;

/// <summary>
/// Interaction logic for App.xaml
/// </summary>
public partial class App : PrismApplication
{
	/// <summary>
	/// Call from PrismApplicationBase.Initialize (Third)
	/// </summaray>
	protected override Window CreateShell()
	{
		return Container.Resolve<MainWindow>();
	}

	/// <summary>
	/// Call from PrismApplicationBase.Initialize (First)
	/// </summaray>
	protected override void RegisterTypes(IContainerRegistry containerRegistry)
	{
		
	}

	/// <summary>
	/// Call from PrismApplicationBase.Initialize (Second)
	/// </summaray>
	protected override void ConfigureRegionAdapterMappings(RegionAdapterMappings regionAdapterMappings)
	{
		base.ConfigureRegionAdapterMappings(regionAdapterMappings);
		regionAdapterMappings.RegisterMapping(typeof(StackPanel), Container.Resolve<StackPanelRegionAdapter>());
	}
}
~~~
```
