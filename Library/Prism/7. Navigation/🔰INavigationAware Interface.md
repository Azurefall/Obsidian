
```ad-info
title: INavigationAware
~~~csharp

namespace Prism;

/// <summary>
/// Provides a way for objects involved in navigation to be notified of navigation activities.
/// </summary>
public interface INavigationAware
{
	/// <summary>
	/// Called when the implementer has been navigated to.
	/// </summary>
	/// <param name="navigationContext">The navigation context.</param>
	void OnNavigatedTo(NavigationContext navigationContext);

	/// <summary>
	/// Called to determine if this instance can handle the navigation request.
	/// </summary>
	/// <param name="navigationContext">The navigation context.</param>
	/// <returns>true if this instance accepts the navigation request; otherwise, false.</returns>
	bool IsNavigationTarget(NavigationContext navigationContext);
	
	/// <summary>
	/// Called when the implementer is being navigated away from.
	/// </summary>
	/// <param name="navigationContext">The navigation context.</param>
	void OnNavigatedFrom(NavigationContext navigationContext);
}
~~~
```
