
```ad-info
title: ThreadOption
~~~csharp

namespace Prism.Events;

/// <summary>
/// Specifies on which thread a Prism.Events.PubSubEvent`1 subscriber will be called.
/// </summary>
public enum ThreadOption
{
	/// <summary>
	/// The call is done on the same thread on which the Prism.Events.PubSubEvent`1 was published.
	/// </summary>
	PublisherThread,

	/// <summary>
	/// The call is done on the UI thread.
	/// </summary>
	UIThread,

	/// <summary>
	/// The call is done asynchronously on a background thread.
	/// </summary>
	BackgroundThread
}
~~~
```