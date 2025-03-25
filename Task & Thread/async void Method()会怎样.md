```c#
async void MethodAsync()
{
    // ①
    await ...
}
```

这种方式要严重杜绝，因为即使Try Catch也捕获不住。

```c#
try
{
    MethodAsync()
}
catch
{
    
}
```

仍旧会抛出异常，这个异常并不能被外部的Try Catch捕获，会异常沿着①代码所在的线程穿透抛出。所以这种代码一旦抛出异常，就会导致整个应用程序异常退出。有一种例外情况，如果①代码在UI线程上，注册了UI异常处理并把e.Handled=True，则async void 不会退出。