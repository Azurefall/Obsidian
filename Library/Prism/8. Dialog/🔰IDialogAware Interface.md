
```ad-info
title: IDialogAware
~~~csharp

namespace Prism.Services.Dialogs;

/// <summary>
/// Interface that provides dialog functions and events to ViewModels.
/// </summary>
public interface IDialogAware
{
    /// <summary>
    /// The title of the dialog that will show in the window title bar.
    /// </summary>
    string Title { get; }

    /// <summary>
    /// Instructs the Prism.Services.Dialogs.IDialogWindow to close the dialog.
    /// </summary>
    event Action<IDialogResult> RequestClose;

    /// <summary>
    /// Determines if the dialog can be closed.
    /// </summary>
    /// <results>If true the dialog can be closed. If false the dialog will not close.</results>
    bool CanCloseDialog();

    /// <summary>
    /// Called when the dialog is closed.
    /// </summary>
    void OnDialogClosed();

    /// <summary>
    /// Called when the dialog is opened.
    /// </summary>
    /// <param name="parameters">The parameters passed to the dialog.</param>
    void OnDialogOpened(IDialogParameters parameters);
}
~~~
```
