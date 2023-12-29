
```ad-info
title: IConfirmNavigationRequest
~~~csharp

namespace Prism.Region;

/// <summary>
/// Provides a way for objects involved in navigation to determine if a navigation request should continue.
/// </summary>
public interface IConfirmNavigationRequest : INavigationAware
{
	/// <summary>
	/// Determines whether this instance accepts being navigated away from.
	/// Implementors of this method do not need to invoke the callback before this method is completed, but they must ensure the callback is eventually invoked.
	/// </summary>
	/// <param name="navigationContext">The navigation context.</param>
	/// <param name="continuationCallback">The callback to indicate when navigation can proceed.</param>
	void ConfirmNavigationRequest(NavigationContext navigationContext, Action<bool> continuationCallback);
}
~~~
```
