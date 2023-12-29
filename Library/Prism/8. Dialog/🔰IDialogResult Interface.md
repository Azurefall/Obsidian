
```ad-info
title: IDialogResult
~~~csharp

namespace Prism.Services.Dialogs;

/// <summary>
/// Contains Prism.Services.Dialogs.IDialogParameters from the dialog and the Prism.Services.Dialogs.ButtonResult of the dialog.
/// </summary>
public interface IDialogResult
{
	/// <summary>
	/// The parameters from the dialog.
	/// </summary>
	IDialogParameters Parameters { get; }

	/// <summary>
	/// The result of the dialog.
	/// </summary>
	ButtonResult Result { get; }
}
~~~
```
