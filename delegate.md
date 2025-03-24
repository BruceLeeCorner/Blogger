# Equals

```c#
public class Car
{
    public Car()
    {
        Action a = new Action(Run);
        Action b = new Action(Run);

        ReferenceEquals(a, b); // false
        a.Equals(b);           // true
    }

    public void Run() { }
}
```

`Delegate`存储方法的机制就是持有函数的`MethodInfo`和实例`Target`相等，则相等。

```c#
public bool Equals(Action action)
{
    return ReferenceEquals(this.Target,action.target) && this.MethodInfo.Equals(action.MethodInfo);
}
```



如果Run是静态方法，那么Target始终是NULL，所以a.Equals(b);一定是True。



如果是Lambda表达式，则一定不相等了。因为lambda被编译成匿名类和实例方法，类名和方法名一定不同

然后再实例化，所以一定不相等。

```c#
Action a = new Action(()=>{});
Action b = new Action(()=>{});
a.Equals(b); // false
```



# Delegate和Type互转

# delegate and memory leak

`NotificationManager`和`Account`是两个用于讲解的辅助类。

```c#
class NotificationManager
{
    private Mail _mail;
    public void Remind()
    {
        _mail.Send();
    }
}
```

```c#
class Account
{
    public event TwitterPostedEvent;
    public void PostTwitter() { }
}
```



delegate本质是类，它有两个字段 _target 和 _methodInfo.

```c#
class delegate
{
    private object _target;
    private MethodInfo _method; 
}
```

下面的代码

```c#
Account muskAccount;
NotificationManager notificationManager;
muskAccount.TwitterPostedEvent += notificationManager.Remind;
```

等价于

```c#
muskAccount.TwitterPostedEvent._target = notificationManager;
muskAccount.TwitterPostedEvent._method = notificationManager.GetType().GetMethod("Remind");
```

如果使用lambda注册，如

```c#
class NotificationManager
{
    private Mail _mail;
    public NotificationManager()
    {
        muskAccount.TwitterPostedEvent += ()=>
        {
            _mail.Send();
        } 
    }
}
```

每一个lambda编译后都会生成一个匿名类，lambda就是该类的方法，如果lambda引用了上下文字段（如_mail），匿名类会特意生成一个相应的字段指向它，像下面这样

```c#
class AnonymousClass
{
    private Mail _mail = notificationManager._mail;
    public LambdaMthod()
    {
        _mail.Send();
    }
}
```

```c#
muskAccount.TwitterPostedEvent._target = anonymousClass;
muskAccount.TwitterPostedEvent._method = anonymousClass.GetType().GetMethod("LambdaMthod");
```



使用公开方法的引用图

==muskAccount->TwitterPostedEvent->_target->notificationManager==

使用Lambda的引用图

==muskAccount->TwitterPostedEvent->_target->anonymousClass->_mail->notificationManager==

这样即使订阅者不再被使用，甚至已经调用了dispose方法，但是被观察者始终指向订阅者，导致订阅者永远不会被垃圾回收，除非被观察者能够被垃圾回收，这样就出现了内存泄漏。

为了避免上述情况，我们必须不能忘记取消订阅。

有一种兜底方案，即使忘记取消订阅，也不影响垃圾回收。

我们需要改造delegate，delegate的target改成weakreference就可以了。



但是这可能
