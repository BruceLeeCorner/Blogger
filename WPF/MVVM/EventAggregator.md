# EventAggregator的目的和作用

- EventAggregator最大的作用是让类型之间解耦合，提高应用程序的模块分离性。
- 可以从2个角度看待EventAggregator机制：
  - Topic消息收发，类似MQTT和RabbitMQ ，MQTT、RabbitMQ的消息在应用程序间流动，CTMvvm的消息在对象实例间流动
  - 观察者模式，发送消息的实例是事件发布者，接收消息的实例是事件订阅者。
- CTMvvm的消息机制，能够实现任何实例之间的相互通信，实例通过消息的数据类型和消息的Token决定是否处理消息！
- 任何对象都可以是消息发送者，任何对象都可以是消息接收者，任何对象都可以是消息。
- EventAggregator可以轻松的实现非耦合的观察者模式：
  - .NET的事件模型，需要发布者和订阅者共存情况下，订阅者去做注册动作，EventAggregator不需要手动注册，天生订阅。
  - .NET的事件模型如果让发布者和订阅者诞生后就已订阅而不是手动订阅，一般需要将发布者作为订阅者的构造方法参数传入订阅者，在订阅者的构造方法完成订阅，这样做导致订阅者和发布者耦合，需要订阅者和发布者在同一个类库或者订阅者所在的类库需要引用发布者所在的类库。但EventAggregator的消息机制，让订阅者和发布者无需知道对方的存在的情况下，天生订阅。
- 在UI框架中Messenger的运行机制



<img src="https://raw.githubusercontent.com/BruceLeeCorner/img4md/master/2024/202501241404872.png" alt="img" style="zoom:67%;" />



CTMvvm引入消息机制的根本目的是降低View和ViewModel的耦合性。消息机制虽然可适用于任何.NET对象，但主要用于View和ViewModel，ViewModel和ViewModel之间通信，它能很好的解决了ViewModel需要操作UI元素，父子窗体之间相互交互需耦合不同窗体类型这两大场景。

MVVM设计模式中，View负责视图，ViewModel负责逻辑，ViewModel中不应当出现操作UI元素的代码！

## View和ViewModel之间通信

场景一：假设ViewModel中开启定时器，实时采集数据，当数据变化时，需要修改UI，当然可以把数据映射成ViewModel中的多个属性，这些属性与View绑定，达到修改刷新View的效果，

## ViewModel之间通信

在桌面开发中，经常会遇到这个场景：点击父窗体，弹出子窗体编辑界面。用户在子窗体进行编辑时，编辑的结果需要实时显示到父窗体上，又或者子窗体关闭后，要将一些子窗体产生的数据流入父窗体，让父窗体根据这些数据做一些事情。

针对上述场景，在Winfrom开发时代，一般让父窗体作为子窗体的构造方法的参数来实现这种交互效果，但这种解决方案耦合性太强了。使用消息机制，在子窗体想要驱动父窗体做某件事情时，只需要子ViewModel向父ViewModel发送一条消息就可以了。

场景：两个视图，一个视图是用户信息列表，一个视图是用户信息添加页面，添加信息之后，用户信息列表视图实时刷新。

<img src="https://raw.githubusercontent.com/BruceLeeCorner/img4md/master/2024/202501241342826.png" alt="image-20220818140951991" style="zoom:67%;" />

**主窗体**

```xaml
<StackPanel>
    <ListBox ItemsSource="{Binding Users}" DisplayMemberPath="Name" />
    <Button Content="添加用户" Command="{Binding AddUserCommand}" />
</StackPanel>
```



```c#
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        WeakReferenceMessenger.Default.Register<string>(this, (recipient, msg) =>
        {
            (new AddUserView()).ShowDialog();
        });
        this.DataContext = new MainViewModel();
    }
}
```



```c#
public class MainViewModel
{
    public RelayCommand AddUserCommand { get; }

    public ObservableCollection<User> Users { get; }

    public MainViewModel(IEventAggregator ea)
    {
        Users = new ObservableCollection<User>()
        {
            new User(){Name = "张三",Age = 20,Email = "zhangsan@gmail.com"},
            new User(){Name = "李四",Age = 21,Email = "lisi@qq.com"},
            new User(){Name = "王五",Age = 22,Email = "wangwu@qq.com"},
        };

        AddUserCommand = new RelayCommand(() =>
        {
            // 弹窗显示新增新用户编辑画面
        });

        ea.GetEvent<UserAddedEvent>().Subscribe(user=>Users.Add(msg));
    }
}
```



**编辑新用户子窗体**

```xaml
<StackPanel>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="姓名" />
        <TextBox Text="{Binding Path=User.Name}" Width="150" />
    </StackPanel>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="年龄" />
        <TextBox Text="{Binding Path=User.Age}" Width="150" />
    </StackPanel>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="邮箱" />
        <TextBox Text="{Binding Path=User.Email}" Width="150" />
    </StackPanel>
    <Button Content="提交" Command="{Binding SubmitCommand}" />
</StackPanel>
```



```c#
public partial class AddUserView : Window
{
    public AddUserView()
    {
        InitializeComponent();
        this.DataContext = new AddUserViewModel();
    }
}
```



```c#
public class AddUserViewModel
{
    public RelayCommand SubmitCommand { get; }
    
    public AddUserViewModel(IEventAggregator ea)
    {
        User = new User();
        SubmitCommand = new RelayCommand(() =>
        {
            ea.GetEvent<UserAddedEvent>().Publish(new User() { Name = User.Name, Age = User.Age, Email = User.Email });
        });
    }

    public User User { get; }
}
```



```c#
public class UserAddedEvent : PubSubEvent<User>
{
    
}
```



# EventAggregator

> `Execution Action`, `Filter Action`, `SubscriptionToken`共同组成`EventSubscription` 。每一次Subscribe都会将事件处理函数，过滤函数，即时生成的UUID封装成一个`EventSubscription`对象并存储进EventBase里 。EventBase的工作模型就像List集合，专门存储`EventSubscription` 对象。EventAggregator相当于一个字典，以EventBase的派生类的Type为Key，EventBase实例为Value。

## 快速入门

第①步：定义事件

```c#
class ItemAddedEvent : PubSubEvent<Student> { }
```

第②步：订阅事件

```c#
var itemAddedEvent = ea.GetEvent<ItemAddedEvent>();
itemAddedEvent.Subscript(AddNew);

private void AddNew(Student student)
{
    students.Add(student);
}
```

第③步：发布事件

```c#
itemAddedEvent.Publish(new Student());
```

第④步：取消订阅

```c#
itemAddedEvent.Unsubscribe(AddNew);
```

# Event Categoty

首先继承EventBase实现派生类DerivedEvent，EventAggregator维护一个字典，该字典以`typeof(EVENT_TYPE)`作Key，`EVENT_TYPE`的实例作Value。Subscribe和Publish时，都需要先`EventAggregator.GetEvent<DerivedEvent>`,如果字典中暂无此Key，会添加此Key并实例化DerivedEvent作Value，所以开发者不必操心“确保先在字典中添加DerivedEvent实例以避免注册事件处理函数时抛出NullReferenceException”。

```c#
// 定义事件AEvent
AEvent : PubSubEvent { }
// 定义继承了AEvent的事件BEvent
BEvent : AEvent { }
// 订阅事件
ea.GetEvent<AEvent>().Subscript(()=> Console.WriteLine("A");)
ea.GetEvent<BEvent>().Subscript(()=> Console.WriteLine("B");)
// 发布事件
ea.GetEvent<AEvent>().Publish(); // 这行代码只能在控制台输出"A"，并不会输出"B"
```

强调两点：① AEvent和BEvent是不同的信道  ② 注册到同一种Event映射的EventBase的函数会在publish时都被激发。

不要被事件类之间的继承关系所干扰！这种继承关系没有任何特殊之处，EventAggregator是基于typeof(EVENT_TYPE)而不是IsAssignedFrom工作的，只要不是同一个类型，事件注册和响应完全独立不相干，互不影响。

## Subscribe

Subscribe方法参数最多的重载：

`SubscriptionToken Subscribe(Action<TPayload> action, ThreadOption threadOption, bool keepSubscriberReferenceAlive, Predicate<TPayload> filter)`

filter：返回值true响应事件的发布，会执行action；false，则不执行action。可以检查TPayload的成员是否满足某个条件以决定是否响应事件。

keepSubscriberReferenceAlive：默认值false。true，注册中心保留对订阅者对象的强引用，false，弱引用。

threadOption：个人觉得这个封装有点鸡肋，我一般不会使用它，需要时自己写代码指定线程。

# Schema

## EventAggregator，EventBase，EventSubscription

EventBase相当于一个==集合==，里面存储许多EventSubscription，每个EventSubscription内有一个Delegate(MethodInfo + Instance)，EventAggregator内部有一个字典，typeof(EventBase的派生类)作Key，EventBase的派生类的单例做Value，每一次Subscribe都会在Value中新增一个EventSubscription实例。Publish会从字典中找到Value遍历并执行其包含的所有EventSubscription的Delegate。

> 继承EventBase定义一个DerivedEvent，相当于开辟一个新的信道。实际上，DerivedEvent在整个Application中是单例，仅生成一个被放在字典中的实例。

## EventSubscription，BackgroundEventSubscription，DispatcherEventSubscription

`EventAggregator`只负责定位到`EventBase`，接着由`EventBase`内含的`EventSubscription`实例负责Invoke被封装到其内部的事件处理函数，这牵扯到在哪个线程上(UI线程，线程池线程，当前线程)执行函数，根据策略的不同，所以又派生了2个类`BackgroundEventSubscription`和`DispatcherEventSubscription`，3者的不同点只是如何(在哪个线程上)调用事件处理函数。

- `EventSubscription`  在publisher线程
- `BackgroundEventSubscription` 在线程池线程
- `DispatcherEventSubscription` 在UI线程

在Subscribe时根据ThreadOption，选择相应类型的EventSubscription来实例化以封装事件处理函数，再放入到EventBase。在EventBase里面的EventSubscription类型可能不同，但它们对外只暴露一个Invoke方法，Invoke里面隐藏了在哪个线程执行的细节。

## EventBase、PubSubEvent、PubSubEvent\<T>

EventBase支持object[] args形式的，PubSubEvent继承了EventBase，是强制指定参数类型，这样在注册和发布时，可以让IDE智能提示，避免运行时参数数量或类型不匹配而抛出异常。

继承PubsubEvent,泛型是事件参数，订阅者的事件处理程序必须是泛型类型。

# Parameterless Handler

```c#
class ItemRemovedEvent : PubSunEvent { }
```

无参继承PubSubEvent即可。

## DelegateReference

EventSubscription内部存储事件处理函数，最容易想到的方式就是使用一个Delegate类型的字段，就像下面的代码片段

```c#
public class EventSubscription
{
    Delegate _delegate;
    EventSubscription(Delegate @delegate)
    {
        _delegate = @delegate;
    }
}
```

但实际上Prism并不是这么做的，而是像下面这样

```c#
public class EventSubscription
{
    IDelegateReference _reference;
    EventSubscription(IDelegateReference reference)
    {
        _reference = reference;
    }
}

public interface IDelegateReference
{
    Delegate Target { get; }
}
```

Prism不直接用Delegate作字段，而是封装进一个类，作为属性，这貌似是冗余的。支持背后字段和计算型。这是因为要处理弱引用问题。

Delegate本来就是两个成员，实例引用 + MethodInfo,所以，，，，，，

如果是字段，EventSubscription只要在EventBase那就一直在EventAggregator，这样Delegate指向的实例永远不会被垃圾回收。如果是计算型，弱引用会垃圾回收。

如果是调用Target的Dispose后，但是还在聚合器中，响应事件会异常。

如果Target已经

# PubSubEvent，SubscriptionToken，Filter,DelegateReference

可以把SubscriptionToken当作简单的GUID来理解！注册时，GUID & Execution Action &  Fileter Action共同组成EventSubscription存储到EventBase表示一次订阅，GUID的作用就是在EventBase集合中唯一标识其宿主EventSubscription，然后再取消订阅时，根据GUID查找到EventSubscription删除之。之所以不直接暴露EventSubscription作为Key用于查找删除，是因为避免直接拿到引用时因权限太大，导致开发者做出不期望的动作。



GetExecutionStrategy的目的是为了兼容有无事件参数的情况。

# publisher如何获取响应结果

```c#
public class FakeEvent : PubSubEvent<FakeEventArgs> { }

public class FakeEventArgs
{
    public object Respond { get; set; }
    public object Msg { get; set; }
}

// 响应者订阅事件
EventAggregator.GetEvent<FakeEvent>().Subscribe((e) =>
{
    // 响应者将响应结果返回给publisher
    e.Respond = "This is the return value that is designed to be used by publisher.";
});


var args = new FakeEventArgs();
EventAggregator.GetEvent<FakeEvent>().Publish(args);
// publisher使用responder的响应结果
MessageBox.Show(args.Respond.ToString());

```



# Thread Model

## ThreadOption

注册事件处理函数时，可以指定函数在哪个线程上执行，有如下3种选择

```c#
public enum ThreadOption
{
    // 在Publish线程执行，这是默认选项。
    PublisherThread,
    // 在UI主线程执行
    UIThread,
    // 在线程池线程执行
    BackgroundThread
}
```

`ThreadOption.BackgroundThread调用函数的方式`

```c#
Task.Run(() => action(argument));
```

`ThreadOption.UIThread调用函数的方式`

```c#
// syncContext关联的是UI线程，下面有更详细的讲解。注意，是异步，无需等待其在UI线程执行结束。
syncContext.Post((o) => action((TPayload)o), argument); 
```

`ThreadOption.PublisherThread调用函数的方式`

```c#
action(argument);
```

Publish( ) 意味着什么？Publish( ) 等价于在当前线程执行所有的订阅函数，这些函数全部执行结束，Publish()才会返回。

```c#
ea.GetEvent<StudentAddedEvent>().Publish();
// 等价于下面的伪代码
void Publish()
{
    foreach(Action action in all_actions)
	{
    	action();
    	// syncContext.Post((o) => action((TPayload)o), argument);
    	// Task.Run(() => action(argument));
	}
}

// 中途某一个事件处理函数抛出异常，根据所在线程造成的结果不同。
// 如果是在publisher线程，则导致后续订阅函数不会被运行。如果是ThreadPool,则不影响后续订阅函数继续执行。
// 如果是UI线程，将击穿UI线程，如果不妥善处理，会导致程序崩溃，如果捕获异常并标记成e.Handled=true,则不会影响后续订阅函数的执行。
```

**下面使用一个具体的案例，来理解Publish**

现有如下订阅，它指定在UI线程执行事件处理函数。

```c#
_ea.GetEvent<StudentAddedEvent>().Subscribe(item => 
    { 
        Console.WriteLine(item.ToString()); 
    }, threadOption: ThreadOption.UIThread);
```

以下发布事件代码片段

```c#
HttpClient hc = new HttpClient();
_ea.GetEvent<ItemAddedEvent>().Publish(2024);
hc.GetStringAsync("www.google.com");
```

等价于

```c#
HttpClient hc = new HttpClient();
// 注意，是BeginInvoke，不是Invoke
Application.Current.Dispatcher.BeginInvoke(() =>
{
    Console.WriteLine(2024.ToString());
});
// syncContext.Post((o) => action((int)o), 2024);
hc.GetStringAsync("www.google.com");
```

## EventAggregator instance must be built in the UI thread

`EventSubscription` : Delegate   Token    SynchronizationContext

`EventBase`: 里面含有很多EventSubscription，还有SynchronizationContext

`EventAggregator`: 里面含有很多EventBase，还有SynchronizationContext

EventSubscription的SyncContext来自EventBase，EventBase是在GetEvent时实例化的，其SyncContext是EventAggregator传递过来的，所以，所有在UI线程上执行的事件处理函数，都是通过EventAggregator的SyncContext抛到UI线程的，那么，我们必须保证EventAggregator的SyncContext关联的必须是UI线程，不然没办法使用ThreadOptions。

下面是EventAggregator的syncContext赋值的源码，是在其构造函数种执行的。

```c#
private readonly SynchronizationContext syncContext = SynchronizationContext.Current;
```

所以，务必要保证，EventAggregator是在UI线程中被实例化的。

==**现在探究EventAggregator是在哪里实例化的？**==

PrismApplicationBase已经将EventAggregator以Singleton模式注册到Container,源码如下：

```c#
internal static void RegisterRequiredTypes(this IContainerRegistry containerRegistry, IModuleCatalog moduleCatalog)
{
    containerRegistry.TryRegisterSingleton<IEventAggregator, EventAggregator>();
}
```

所以实例化EventAggregator是在第一次Resolve\<IEventAggregator>()处。

In General,  Prism.ViewModelLocator.AutoWire是在UI线程上定位并实例化ViewModel的，且往往第一个被寻找的ViewModel就是 MainWindowViewModel。如果MainWindowViewModel的构造函数注入了IEventAggregator,只要保证MainWindowViewModel在UI线程中实例化，那就保证了EventAggregator的SynchronizationContext关联的是UI线程。

## How to do if Multiple UI Threads in the application？

一个WPF应用程序可以有多个UI线程，但是`ThreadOption.UIThread`只能使用其中一个UI线程(实例化EventAggregator的UI线程)。开发者可以在Subscribe时一律采用默认选项ThreadOption.PublisherThread，但是在事件处理函数内部将代码抛到正确的UI线程上。

# Filter

```c#
ea.GetEvent<MsgSentEvent>().Subscribe(filter:(MSG msg)=> msg.DestAddr == 0x0001);
```



- if(filter() == true) Handler(); 效果相当于在Handler第一行加个检测条件，不满足直接return。
- MsgSentEvent关联的Subscripter都会响应Public，但是Subscribe时添加了filter的Subscriber会检测条件是否满足。类似Modbus协议，每个从机都能收到消息，但是发现不是给自己的，就丢弃。
- filter管理强弱引用和线程模型的方式与handler完全一致，二者在同一个线程上被执行，且同是弱引用管理或强引用管理。
- Filter的参数与handler相同，返回值类型是bool.

# Unsubscribe

如果能拿到事件处理函数方法句柄，可以像`_event.Unsubscribe(ShowNews)`这样取消订阅。

```c#
public class MainWindowViewModel
{
    TickerSymbolSelectedEvent _event;
    public MainWindowViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent<TickerSymbolSelectedEvent>();
        _event.Subscribe(ShowNews);  // 订阅
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(ShowNews); // 取消订阅
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

`Subscribe时，ShowNews被封装进一个Action实例放入EventBase内；取消订阅时，再以ShowNews封装成的Action在EventBase里查找先前注册的Action删除之。虽然注册和取消注册时使用的Action不是ManagedHeap上的同一个实例，但是Action的Equals已经被Override，只要Target和MethodInfo相等，则Action就相等。`

如果事件处理函数是通过lambda expression生成的，无法拿到函数句柄，可以利用SubscriptionToken取消订阅。

```c#
public class MainWindowViewModel
{
    TickerSymbolSelectedEvent _event;
    SubscriptionToken _token;
    public MainWindowViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent<TickerSymbolSelectedEvent>();
        _token = _event.Subscribe((companySymbol)=>{ /*implement logic*/ }); // 订阅
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(_token); // 取消订阅
    }
}
```

`Subscribe时，Action捆绑上一个GUID一起存入到EventBase，GUID唯一标识Action.取消订阅时，以GUID为引，找到关联的Action然后从EventBase中删除之。`

# keepSubscriberReferenceAlive

每个EventSubscription内部的方法Delegate都索引着对象，这些对象无法被GC，除非取消注册。





# 弱引用和内存泄漏



==IsActive==



# 如何在View中使用EventAggreagtor

```c#
// 定义事件
public class WeatherForecastEvent : PubSubEvent<String>
{
}
// 订阅事件
var token = Container.Resolve<IEventAggregator>().GetEvent<WeatherForecastEvent>().Subscribe(Do);
private void Do(string weather) { }

// 发布事件
Container.Resolve<IEventAggregator>().GetEvent<WeatherForecastEvent>().Publish("Rain");

// 取消订阅的第一种方式
Container.Resolve<IEventAggregator>().GetEvent<WeatherForecastEvent>().Unsubscribe(token);
// 取消订阅的第二种方式
Container.Resolve<IEventAggregator>().GetEvent<WeatherForecastEvent>().Unsubscribe(Do);
```











