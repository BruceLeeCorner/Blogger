```c#
protected DelegateCommandBase()
{
    _synchronizationContext = SynchronizationContext.Current;
}

public virtual event EventHandler? CanExecuteChanged;

protected virtual void OnCanExecuteChanged()
{
    var handler = CanExecuteChanged;
    if (handler != null)
    {
        if (_synchronizationContext != null && _synchronizationContext != SynchronizationContext.Current)
            _synchronizationContext.Post((o) => handler.Invoke(this, EventArgs.Empty), null);
        else
            handler.Invoke(this, EventArgs.Empty);
    }
}

```

`SynchronizationContext`用于标记线程，然后后续可以将指定的代码发送到标记线程上执行。