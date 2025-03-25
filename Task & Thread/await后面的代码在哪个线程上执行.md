await methodAsync().ConfigureAwait(false),如果methodAsync()无异常，后面的代码在哪个线程上执行是不确定的，但是如果出现异常，后面的代码会在上文线程上执行，等同于ConfigureAwait(true)。



```c#
public async Task TestAsync()
{
    M1();
    try
    {
        await DownloadAsync().ConfigureAwait(false);
    }
    catch
    {
        M3();
    }
    M2();
}
```

M2 在哪个线程上执行是不确定的。

M3与M1在同一个线程上执行。