## ICommand--实现解耦式注册事件处理程序

<table>
    <td bgcolor="#6e7b99">点击按钮，如果当前是管理员身份且文本框有内容，则保存文本框输入的内容到磁盘。</td>
</table>
***View***

```xml
<StackPanel>
    <TextBox Name="TxtInfo" />
    <Button Name="BtnSave">
        <i:Interaction.Triggers>
            <i:EventTrigger EventName="Click">
                <prism:InvokeCommandAction Command="{Binding SaveCommand}" CommandParameter="{Binding Path=Text, ElementName=TxtInfo}" />
            </i:EventTrigger>
        </i:Interaction.Triggers>
    </Button>
</StackPanel>
```

***ViewModel***

```c#
public class ViewModel : ObservableObject
{
    private bool _isAdmin;

    public ViewModel()
    {
        SaveCommand = new DelegateCommand<string>(Save, CanSave);

        PropertyChanged += (sender, args) =>
        {
            if (args.PropertyName == nameof(IsAdmin))
            {
                SaveCommand.RaiseCanExecuteChanged();
            }
        };
    }

    private bool CanSave(string parameter)
    {
        return IsAdmin && !string.IsNullOrWhiteSpace(parameter);
    }

    private void Save(string parameter)
    {
        File.WriteAllText("D:\\info.txt", parameter);
    }

    public DelegateCommand<string> SaveCommand { get; }

    public bool IsAdmin
    {
        get => _isAdmin;
        set => SetProperty(ref _isAdmin, value);
    }
}
```

上面的代码的功能：监控Button的的Click事件，当事件发生时，执行`prism:InvokeCommandAction's Command`的`Execute(object? parameter)`方法，方法实参是`prism:InvokeCommandAction's CommandParameter`。

首先分析上述代码结构的第一个优点：`完成注册Click事件，且表示业务逻辑的事件处理程序位于ViewModel.cs，而非像Winform那样混杂在View.cs,即，解耦式注册事件处理程序。`

如果纯粹只为了完成解耦式注册事件，只需要Command实例实现约定的Execute(object? parameter)，但事实上`prism:InvokeCommandAction's Command`还有通过接口`IComamnd`进行约定的其他2个成员，这是为什么呢? 下面详细阐述这3个成员的作用。

## ICommand成员介绍

<table>
    <td bgcolor="#6e7b99">ICommand接口成员</td>
</table>

```c#
void Execute(object? parameter);
bool CanExecute(object? parameter);
event EventHandler? CanExecuteChanged;
```


### void Execute(object? parameter)

事件发生时，会立刻执行此方法。方法参数parameter来自CommandParameter。

### bool CanExecute(object? parameter)

CanExecute表示能否执行Execute,返回True可以False不可以.方法参数parameter来自CommandParameter。

CanExecute需要结合CanExecuteChanged使用，具体作用放到CanExecuteChanged节介绍。

### CanExecuteChanged

#### 事件作用

`CanExecuteChanged?.Invoke(this,EventArgs.Empty)`对外发布事件: *"Command的CanExecute的返回值已发生变化"*。此事件的订阅者是作为事件源的Control，事件处理程序是`EventSource.IsEnabled=IComamnd.CanExecute(parameter)`.

如果还想在View层防患于未然，在Execute不可执行时及时禁用事件源，就需要当`CanExecute`的返回值发生变化时，立刻发布 `CanExecuteChanged?.Invoke(this, EventArgs.Empty)`，事件源响应此事件会在UI线程执行一次`CanExecute()`刷新自己的IsEnabled,这样就杜绝了用户误触发事件。另外，事件源的禁用启用状态也可以让用户实时观测到当前事件处理程序Execute能否执行。

#### 事件发布契机

我们可以随时调用 `CanExecuteChanged?.Invoke(this, EventArgs.Empty)`来更新事件源Button的IsEnabled状态。但基本原则是：

① CanExecute返回值变化时，必须发布CanExecuteChanged，否则事件源的IsEnabled不会被更新。 

② 最好只当影响CanExecute的因子(变量)发生变化时，才发布事件，否则不必要的执行一次CanExecute导致浪费宝贵的UI线程。

③ 决定CanExecute返回值的因子有其参数parameter(即CommandParameter)和方法体内参与逻辑的变量，典型情景如上述案例代码片段，由CommandParameter和ViewModel属性决定，所以当IsAdmin的值变化后，应当及时发布CanExecuteChanged事件。针对如何监控ViewModel属性值变化以及时调用CanExecuteChanged,往往采用PropertyChnaged事件.
> *直接属性*
>
> ```c#
>PropertyChanged += (object? sender, PropertyChangedEventArgs e)=> 
> {
> if(e.Name=="Person")
>  CanExecuteChanged?.Invoke(this, EventArgs.Empty);
>   };
>    ```
> 
> *子属性*
>
> ```c#
>Person.PropertyChanged += (object? sender, PropertyChangedEventArgs e)=> 
> {
> if(e.Name=="Email")
>  CanExecuteChanged?.Invoke(this, EventArgs.Empty);
>   };
>    ```

④CommandParameter如果是绑定到具有通知能力的属性，其值变化时，WPF会自动执行EventSource.IsEnabled=CanExecute(CommandParameter); 程序员不必操心.

⑤ 如果影响CanExecute的因子(变量)在代码中难以分析出什么时候变化，为了避免遗漏疏忽，这种情况下建议用定时器每100ms发布一次事件，虽然这样会浪费UI线程。注意：View不可见时，要及时关闭定时器，以减轻UI线程的负担。


#### 事件发布线程

==注意== `CanExecuteChanged?.Invoke(this, EventArgs.Empty)`必须要在UI线程(实例化事件源控件的线程)上执行 ! 因为事件处理程序`EventSource.IsEnabled=IComamnd.CanExecute(parameter)`需要访问事件源控件,WPF不允许跨线程访问控件.所以,还有一个结论:`CanExecute`总是在UI线程上执行的.

==事件处理程序在发布事件代码行所在的上下文环境执行的. 事件处理程序执行结束,发布事件的函数才会返回.==

### CommandParameter与CanExecute

CommandParameter必须绑定的是依赖属性或实现INotifyPropertyChanged接口的属性，否则，当BaseProperty的值变化后，CommandParameter不会被更新导致向ViewModel传递错误的参数，同时，也不会自动提取ICommand.CanExecute刷新命令源的IsEnabled状态。

### CommandParameter传递多个参数

事件发生时，通过object类型的CommandParameter向ViewModel传递事件参数。这似乎有极大的限制：如果需要传递多个参数怎么办？

可以通过MultiBinding将CommandParameter绑定到多个依赖属性上，通过IValueConverter转换成`某个对象的多个属性`，CommandParameter指向的就是该对象。这需要定义新类来封装多个属性，如果想省事可以利用匿名类，dynamic，ValueTuple，ArrayList，视情况而定。)

**Code Sample: 点击提交按钮，保存用户信息(姓名，年龄，职业)**

```xml
<Window.Resources>
    <local:CommandParameterConvert x:Key="cpc" />
</Window.Resources>
<StackPanel VerticalAlignment="Center" HorizontalAlignment="Center">
    <StackPanel Orientation="Horizontal" Margin="3">
        <TextBlock Text="姓名" />
        <TextBox Width="100"  Name="txtName" />
    </StackPanel>
    <StackPanel Orientation="Horizontal" Margin="3">
        <TextBlock Text="年龄" />
        <TextBox Width="100" Name="txtAge" />
    </StackPanel>
    <StackPanel Orientation="Horizontal" Margin="3">
        <TextBlock Text="职业" />
        <TextBox Width="100" Name="txtJob" />
    </StackPanel>
    <Button Content="提交" Command="{Binding SubmitCommand}" Margin="5" Width="100" HorizontalAlignment="Left" Background="Orange">
        <Button.CommandParameter>
            <MultiBinding Converter="{StaticResource cpc}">
                <Binding ElementName="txtName" Path="Text" />
                <Binding ElementName="txtAge" Path="Text" />
                <Binding ElementName="txtJob" Path="Text" />
            </MultiBinding>
        </Button.CommandParameter>
    </Button>
</StackPanel>
```
```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        this.DataContext = new MainViewModel();
    }
}

class CommandParameterConvert : IMultiValueConverter
{
    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }

    object IMultiValueConverter.Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        string name = values[0].ToString();
        string age = values[1].ToString();
        string job = values[2].ToString();
        return new { Name = name, Age = age, Job = job };
    }
}
```
```csharp
public class MainViewModel
{
    public MainViewModel()
    {
        SubmitCommand = new RelayCommand<object>((obj) =>
        {
            dynamic d = obj;
            MessageBox.Show(d.Name + d.Age + d.Job);
        });
    }

    public RelayCommand<object> SubmitCommand { get; }
}
```
![image](https://img2022.cnblogs.com/blog/1037641/202208/1037641-20220817122012664-93292087.png)



### 小结

Execute是事件处理程序,CanExecuteChanged以及CommandParameter会监控CanExecute返回值及时禁用事件源,在Execute不可执行时避免用户触发事件，这种内置的防呆机制可以被关闭，由开发者自己去实现。



## ICommand是实现MVVM的基石

将ViewModel中的ICommand Implementation Class实例，赋值到控件的ICommand成员，便完成了Winform中的事件模型，但是事件处理程序并不在View文件中，而是在ViewModel文件中，从而实现了业务逻辑与前端分离解耦。

`赋值是通过数据绑定完成的。`

```c#
<Button Command="{Binding RelativeSource={RelativeSource AncestorType=UserControl}, Path=DataContext.MyCommand}" />

<Button Command="{Binding ElementName=TopLevel, Path=DataContext.MyCommand}" />
    
<Button Command="{Binding MyCommand}" />
```

正因为有了ICommand，就可以拿着ViewModel的Command赋值给它，就实现了解耦。

## 从事件模型到任务模型

ICommand是比事件更为高级的概念，它把应用程序的功能划分为任务，每个任务的具体内容以及任务当前是否可执行都封装在ICommand的具体类中，而这些任务可以由多种途径（菜单、工具栏和快捷键等）来触发，并复用相同的代码，同时可以随时发布可执行状态以供控件观测调整自己的启禁用状态。ICommand是`任务`模型的抽象，主要针对①任务内容 ②执行任务前先检查任务的可执行条件 ③ 任务可执行状态可被观测。



在UI-APP开发中，最主要且最频繁的场景就是响应用户操作去执行任务，本质是控件发布事件然后执行相应的事件处理程序，这种交互的关键点是：

1. 事件处理程序具体是什么。
2. 执行事件处理程序前，要先检查决定其是否可执行的条件，不成立便放弃执行。
3. 控件的IsEnabled要实时跟踪事件处理程序的可执行条件成立情况，这样既可在View层就提前杜绝事件被触发，又能让用户随时观察到任务可执行与否情况。
4. 不同控件的不同事件，触发同一个事件处理程序时(如点击button或MenuItem或Ctrl+C都Copy文本框的内容)，如何最大程序的复用代码。

WPF围绕上述4点，为这一开发场景提供了具有指导性的约束型开发模型---Trigger & ICommand,通过Trigger监控控件的某一事件，当控件发生事件时，提取ICommand成员执行之。



## ICommand的缺点以及如何克服

### IsEnabled无法共享问题

监控单个控件的多个事件，某个事件不可执行时禁用了事件源，导致另一个事件无法被触发。比如，Button有2个事件，左键单击事件和右键事件。左键单击事件的CanExecute返回false时导致Button的IsEnabled被设置成false，Button被禁用，但是此时右键事件是允许被触发的，但是由于Button被禁用而无法触发。

Command的CanExecute设计的初衷是方便开发者在Execute不可执行时，轻松禁用控件。但是针对多事件场景，这种内置机制无法很好的工作，这时候需要开发者关闭内置机制，自己实现防呆机制。

**禁止**

AutoEnable = false

防呆

在Execute方法头部检测CanExecute，返回值是false时，弹窗提示不可执行并立刻结束。

可观察

将一个可视化元素的依赖属性绑定到CommandParameter和ViewModel属性，并在Converter中嵌入CanExecute逻辑，比如true时展示绿色，false展示红色。

用控件的其他属性，如背景色，前景色，修饰控件，绑定CommandParameter，ViewModel属性，转换器就是CanExecute,来显示命令是否可以执行。



### 正在执行中,整个界面都不能操作.

Execute正在执行中，需要禁用View的多个区域，而不单单是禁用事件源控件。IsExecuting。









## Attached时发生了什么

1. 初始化Button的启用状态：Button.IsEnabled = SaveCommand.CanExecute()
2. 事件订阅：SaveCommand.CanExecuteChanged += Button.IsEnabled = SaveCommand.CanExecute()
3. 事件订阅：Button.Click += (button,eventArgs)=> { if SaveCommand.CanExecute==True,SaveCommand.Execute() }

## DelegateCommand

ICommad:约定2个方法1个成员给命令机制使用。绑定时，会抽取这3个方法。

```c#
typeof(ICommand).GetMethod("IComamnd.Execute").Invoke(DataContext,CommandParameter);
```


## 命令与线程

CanExecute始终在UI线程上执行，所以不能有耗时操作！也不能抛出异常，否则会导致UI线程崩溃。

CanExecute执行完毕后，才能执行Execute，所以不能有耗时操作，否则导致任务触发延迟很久。

CanExecuteChanged也必须要在UI线程上调用。

## How to Monitor Event

Event2Command

如果像Winform那样注册事件处理程序，事件处理程序只能写在View文件中，这样做不到前后端分离。在实际开发中，会把事件转换成命令来达到响应用户操作处理任务的目的。

#### 引入Nuget和附加属性

下载Nuget包`Prism.Wpf`和`Microsoft.Xaml.Behaviors.Wpf`，在xaml添加命名空间`xmlns:i="http://schemas.microsoft.com/xaml/behaviors"` 和 `xmlns:prism="http://prismlibrary.com/"`，为WPF元素`添加附加属性Interaction.Triggers`实现路由事件转命令。

- 方式1：直接在事件源控件上添加附加属性i:Interaction.Triggers

```xml
<StackPanel>
    <TextBox
        Name="TextBox"
        Width="150"
        Height="25"
        Text="{Binding Info, Mode=OneWayToSource}" />
    <Button>
        <i:Interaction.Triggers>
            <i:EventTrigger EventName="Click">
                <prism:InvokeCommandAction Command="{Binding SaveCommand}" CommandParameter="{Binding Path=Text,ElementName=TextBox}" Content="Save" />
            </i:EventTrigger>
        </i:Interaction.Triggers>
    </Button>
</StackPanel>
```

- 方式2：也可以添加到非事件源控件上，但并不是被添加i:Interaction.Triggers的控件触发的命令。

```xml
<StackPanel>
    <TextBox
        Name="TextBox"
        Width="150"
        Height="25"
        Text="{Binding Info, Mode=OneWayToSource}" />
    <Button Name="Button" Content="Save"></Button>
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="Click" SourceObject="{Binding ElementName=Button}">
            <prism:InvokeCommandAction Command="{Binding SaveCommand}"  CommandParameter="{Binding ElementName=TextBox,Path=Text}"/>
        </i:EventTrigger>
    </i:Interaction.Triggers>
</StackPanel>
```

`实战中，第一种直接在目标控件身上添加附加属性的方式最为常用，这样绑定SourceObject和CommandParameter会很方便。因为在VisualTree索引上层控件很容易，但索引下层控件很困难。`

#### prism:InvokeCommandAction

```xml
<ListBox ItemsSource="{Binding Items}" SelectionMode="Single">
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="SelectionChanged">
            <prism:InvokeCommandAction Command="{Binding SelectedCommand}" CommandParameter="{Binding MyParameter}" />
        </i:EventTrigger>
    </i:Interaction.Triggers>
</ListBox>
```

- <font>Command</font> Command与ViewModel中的ICommand属性绑定(如DelegateCommand,AsyncDelegateCommand,RelayCommand ,etc)。

- <font>CommandParameter</font> CommandParameter是传递到Execute和CanExecute的实参。没有设置CommandParameter则传递到Execute和CanExecute的实参是路由事件的RoutedEventArgs。

- <font>TriggerParameterPath</font> 这个属性只有向DelegateCommand传递的是RoutedEventArgs时才有用。TriggerParameterPath指向原始路由事件参数的某个属性。示例如下，传递SelectionChangedEventArgs的AddedItems属性。

    ```xml
      <Grid>
          <ListBox ItemsSource="{Binding Items}" SelectionMode="Single">
              <i:Interaction.Triggers>
                  <i:EventTrigger EventName="SelectionChanged">
                      <prism:InvokeCommandAction Command="{Binding SelectedCommand}"
                                                 TriggerParameterPath="AddedItems" />
                  </i:EventTrigger>
              </i:Interaction.Triggers>
          </ListBox>
      </Grid>
    ```

    这个功能让程序更符合MVVM，传递到ViewModel只是`纯粹的数据`,不会传过去控件和冗余数据，这样即便更换View，ViewModel完全不用更改，符合前后端分离和开放关闭原则。像Microsoft.Xaml.Behaviors.Wpf的InvokeCommandAction还提供了EventArgsConverter属性，可以对传入到ViewModel的RoutedEventArgs进行一次转换，这更牛逼，期望Prism也为自己的InvokeCommandAction加上这个属性！

- <font>AutoEnable</font> 默认值是True。如果false，命令源不会实时更新启禁用状态,即，既不会在命令绑定语句被执行时设置命令源控件初始的启禁用状态，也不会在ViewModel调用Command.CanExecuteChanged()时刷新启禁用状态！`命令源一直可用！` **记住，AutoEnable关闭的只是命令源启禁用状态更新功能，但并不会也不能关闭命令启禁用功能，触发ViewModel ICommand时，一定会先检查IComamd.CanExecute()来决定是否执行IComamnd.Execute().**

    `ViewModel中ICommand被触发后，一定是先调用ICommand.CanExecute()，返回True才会继续调用ICommand.Execute().`

> Prism的AutoEnable很棒，它解决了以下的场景难题：
>
> 1. 同一个控件的多个事件都转成命令，那么控件的启禁用状态跟随哪个命令呢？可以灵活关闭某些Command的启禁用功能。
> 2. 命令执行时，不仅仅要禁用命令源，还要一起禁用其他控件，这时候关闭命令源的启禁用功能，单独设计启禁用区域和效果。
> 3. ViewModel的命令任务不可执行时，UI的视觉效果是设置命令源的IsEnabled属性为False，这种视觉效果有时候并不是我们想要的，如果想对命令不可执行设计其他的视觉效果和交互效果，这时候就需要关闭自带的启禁用功能，不要自动设置IsEnabled为False。

#### 传递原始事件参数







#### i:EventTrigger的局限

> i:EventTrigger的意义是： 像winform那样，为`某一个控件`注册事件处理程序，但事件处理程序和该控件位于不同的cs源文件中，达到前后端分离解耦的目的。
>
> 【_textBox.TextChanged += xxx   ====> when _textBox.TextChanged to invoke viewmodel's command's xxx】

Microsoft.Xaml.Behaviors中的i:EventTrigger,用法如下面的代码片段。

```xml
<Button Name="Button">
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="Click" SourceObject="{Binding ElementName=Button}">
            <prism:InvokeCommandAction />
        </i:EventTrigger>
    </i:Interaction.Triggers>
</Button>
```

意思是，当SourceObject发布EventName指定的路由事件时，触发命令。

**关键信息点**

1. 有且仅有一个发布者，仅当SourceObject发布路由事件ClickEvent才会触发命令，其他对象发布路由事件ClickEvent无效。

2. EventName的字符串值必须是路由事件的ClrEventAccessor名称，不能是路由事件名称。如，是Click，不是ClickEvent。

3. EventName指定的事件必须是SourceObject的普通.NET事件成员，否则运行时抛出异常。原因请看下面Microsoft.Xaml.Behaviors.Wpf的源码，它通过反射查看SourceObject有无EventName。

    ```c#
    EventInfo @event = obj.GetType().GetEvent(eventName);
    if (@event == null)
    {
    	if (SourceObject != null)
    	{
    		throw new ArgumentException(
    		string.Format(CultureInfo.CurrentCulture, ExceptionStringTable.EventTriggerCannotFindEventNameExceptionMessage, 
    		eventName, obj.GetType().Name));
    	}
    }
    ```

    

    ==Execute不可执行时,禁用命令源,防患于未然,有时为了增加可靠性,会把CanExecute强制放到Execute头部.==

    
    
    // 如下面代码，可以正常触发命令。原因是MouseDown在UIElement，而Button继承自UIElement.
    
    ```c#
    <i:EventTrigger EventName="MouseDown" SourceObject="{Binding ElementName=Button}">
          <prism:InvokeCommandAction />
    </i:EventTrigger>
    ```
    
    
    
    ```c#
    class UIElement
    {
        public static readonly RoutedEvent MouseDownEvent;
        public event MouseButtonEventHandler MouseDown;
    }
    
    class Button : UIElemet { }
    ```

    
    
    // 下面代码无法触发命令，因为Button没有Checked事件。
    
    ```c#
    <i:EventTrigger EventName="Checked" SourceObject="{Binding ElementName=Button}">
          <prism:InvokeCommandAction />
    </i:EventTrigger>
    ```

    

    所以，
    
    ① `附加事件不能转换命令`。
    
    ② 如果只定义了静态路由事件字段，但`未提供CLR事件封装器，也不能转换成命令`。比如假设ButtonBase只定义了ClickEvent字段，未定义Click事件，那么Click无法事件转命令。

#### RoutedEvent2Command

布局控件StackPanel内部有多个Button，在StackPanel上订阅ButtonBase.ClickEvent，当点击任意一个按钮都会执行事件处理程序。这种场景i:EventTrigger做不到，原因就是它只能监控`某一个单一控件`发布在该控件内定义的路由事件。

如果要监控任意控件发布的冒泡、隧道、直接路由事件，当事件被发布时，触发命令，请使用下面的==RoutedEventTrigger==。

```c#
public class RoutedEventTrigger : EventTriggerBase<DependencyObject>
{
    public RoutedEvent RoutedEvent { get; set; }

    protected override void OnAttached()
    {
        Behavior behavior = base.AssociatedObject as Behavior;
        FrameworkElement associatedElement = base.AssociatedObject as FrameworkElement;
        if (behavior != null)
        {
            associatedElement = ((IAttachedObject)behavior).AssociatedObject as FrameworkElement;
        }
        if (associatedElement == null)
        {
            throw new ArgumentException("Routed Event trigger can only be associated to framework elements");
        }

        associatedElement.AddHandler(RoutedEvent, new RoutedEventHandler(this.OnRoutedEvent));
    }

    private void OnRoutedEvent(object sender, RoutedEventArgs args)
    {
        base.OnEvent(args);
    }

    protected override string GetEventName()
    {
        return RoutedEvent.Name;
    }
}
```



**using sample**

<img src="https://raw.githubusercontent.com/bruceleecorner/img4md/master/2024/202409111656257.png" alt="image-20240911165606135" style="zoom:50%;" />

```xml
<StackPanel x:Name="StackPanel" Margin="0,10,0,0">
    <i:Interaction.Triggers>
        <xp:RoutedEventTrigger RoutedEvent="{x:Static ButtonBase.ClickEvent}">
            <prism:InvokeCommandAction Command="{Binding GoCommand}" TriggerParameterPath="OriginalSource.Tag" />
        </xp:RoutedEventTrigger>
    </i:Interaction.Triggers>
    <Button
        MinWidth="150"
        Margin="10"
        Padding="5"
        HorizontalAlignment="Center"
        Content="1" />
    <Button
        MinWidth="150"
        Margin="10"
        Padding="5"
        HorizontalAlignment="Center"
        Content="2" />
    <Button
        MinWidth="150"
        Margin="10"
        Padding="5"
        HorizontalAlignment="Center"
        Content="3" />
</StackPanel>
```

也支持附加路由事件

```xml
<xp:RoutedEventTrigger RoutedEvent="{x:Static Validation.Error }">
    <prism:InvokeCommandAction />
</xp:RoutedEventTrigger>
```



#### Interaction.Triggers的使用规则

- SourceObject可以被省略，默认值是被添加i:Interaction.Triggers的控件。直接在事件源控件上添加附加属性时，往往会省略SourceObject。

- EventName指定的事件，必须是SourceObject控件拥有的。比如SourceObject是Button，EventName就不能是Checked。

    下面的代码`不能触发`ClickCommand，因为默认的TargetObject是StackPanel，但是StackPanel并无Click事件，无效。

    ```xml
    <StackPanel>
    	<i:Interaction.Triggers>
    		<i:EventTrigger EventName="Click">
    			<prism:InvokeCommandAction Command="{Binding ClickCommand}" />
    		</i:EventTrigger>
    	</i:Interaction.Triggers>
    	<Button />
    </StackPanel>
    ```

- 一个EventTrigger只能对某一个控件的某一个路由事件转换。

    下面的代码，点击button2并不能触发ClickCommand，点击button1才可以触发。

    ```xml
    <StackPanel>
    	<i:Interaction.Triggers>
    		<i:EventTrigger EventName="Click" SourceObject="{Binding ElementName=button1}">
    			<prism:InvokeCommandAction Command="{Binding ClickCommand}" />
    		</i:EventTrigger>
    	</i:Interaction.Triggers>
    	<Button Name="button1" />
    	<Button Name="button2" />
    </StackPanel>
    ```







> WPF控件有3种暴露ICommand接口的方法：
>
> 1. WPF的部分控件内置Command和CommandParameter，一般是对最常用的鼠标事件的封装。
> 2. 批量Command可以用InputBindings，一般是用于鼠标和快捷键。
> 3. 也可以使用第三方Nuget，自行进行事件转命令。




### ② InputBindings

```xml
<Window>
    <TextBlock FontSize="16" Text="{Binding Name}">
        <TextBlock.InputBindings>
            <MouseBinding
                Command="{Binding DataContext.EditFriendCommand, RelativeSource={RelativeSource Mode=FindAncestor, AncestorType=Window}}"
                CommandParameter="{Binding DataContext.SelectedFriend, RelativeSource={RelativeSource Mode=FindAncestor, AncestorType=Window}}"
            MouseAction="LeftDoubleClick" />
        </TextBlock.InputBindings>
    </TextBlock>
</Window>
```

 继承UIElement的WPF元素都有一个集合性质的InputBindings属性，可以存储更多ICommand,结构类似{ Command & CommandParameter & 其他新增属性 },在实际开发中，常用于注册鼠标和快捷键事件。`必须当控件获取焦点时，快捷键才会激发命令。`





## Prism's Command

### 命令分类

继承ICommand实现具体命令类，每种类表示一种任务，如CopyCommand，CutCommand，PasteCommand。命令类之间的差异就是表示任务详情的Execute和CanExecute，可以仅实现一种类，类似于命令模板，在其构造函数传入不同的委托供I Command.Execute和ICommand.CanExecute调用就行了，这样就不需要定义很多种类了。这也是为什么叫DelegateCommand和RelayCommand的原因，Just Delegate、Just Relay、Just Proxy、Just Lead。

`void Execute(object? parameter)`可以Invoke同步，异步，有参，无参方法。故Prism wrap了4种Command：

① DelegateComamnd  (Invoke 无参同步方法)

② DelegateCommand\<T> (Invoke 有参同步方法)

③ AsyncDelegateComamnd (Invoke 无参异步方法)

④ AsyncDelegateCommand\<T> (Invoke 有参异步方法)



实际上可以仅实现有参命令，如果想调用无参方法，只需要`new DelegateCommand<object>((parameter)=>{MethodWithoutParameter();})`,parameter是CommandParameter，它的值是什么没任何影响，因为ICommand.Execute并未使用它。这样虽然可以正常工作，但是造成开发者不得不强行把无参方法转换成有参方法后再传入DelegateCommand，It makes code ugly。综上，所以要封装一个无参命令，这样直接在构造函数中传入无参的任务函数。 

分同步和异步2种命令，是因为异步方法相对于同步方法，多出来很多要注意的问题（后面详细分析这些问题），可以在AsyncCommand中预处理这些问题，以便开发者开箱即用，少出BUG。

### DelegateCommand

控件ICommand成员的Execute和CanExecute调用的方法是无参方法时，可使用此命令。

下面是通过DelegateCommand执行`保存`任务。Save( )相当于事件处理程序。

```xml
<StackPanel>
    <StackPanel>
        <TextBox
            Name="TextBox"
            Width="150"
            Height="25"
            Text="{Binding Info, Mode=OneWayToSource}" />
        <Button Command="{Binding SaveCommand}" Content="Save" />
    </StackPanel>
</StackPanel>
```



```c#
public class MainWindowViewModel : ObservableObject
{
    private string _info;

    public MainWindowViewModel()
    {
        AssignCommands();

        this.PropertyChanged += (sender, args) =>
        {
            if (args.PropertyName == nameof(Info))
            {
                SaveCommand!.RaiseCanExecuteChanged();
            }
        };
    }

    public string Info
    {
        get => _info;
        set => SetProperty(ref _info, value);
    }

    public DelegateCommand SaveCommand { get; private set; }

    private void AssignCommands()
    {
        SaveCommand = new DelegateCommand(SaveExecute, SaveCanExecute);
    }

    private bool SaveCanExecute() => Convert.ToString(Info) != null;

    private void SaveExecute()
    {
        try
        {
            File.WriteAllText("D:\\info.txt", Convert.ToString(Info));
        }
        catch (Exception e)
        {
        }
    }
}
```

命令参数始终都会被传递到RelayCommand(控件不显示提供CommandParameter时默认是NULL)，然后调用ICommand.Execute(object? parameter)和ICommand.CanExecute(object? parameter)，只是二者内部并未使用parameter，即使控件显示指定了CommandParameter，也是被忽略。它们只是执行构造函数传递过来的委托。具体可查看RelayCommand源码。

#### CanExecute

<table><td bgcolor=#6e7b99>CanExecute</td></table>

```c#
public bool CanExecute()
{
    try
    {
        return _canExecuteMethod();
    }
    catch (Exception ex)
    {
        if (!ExceptionHandler.CanHandle(ex))
            throw;

        ExceptionHandler.Handle(ex, null);

        return false;
    }
}

protected override bool CanExecute(object? parameter)
{
    return CanExecute();
}
```

构造函数传入的canExecuteMethod,不出现异常，其返回值就是ICommand.CanExecute的返回值。

如果canExecuteMethod出现异常，先检查此异常能否被`统一异常处理中心`处理，如能，返回false，否则抛出此异常。

DelegateCommand的ICommand.CanExecute在运行时仍旧可能出现异常导致UI线程崩溃！

ICommand.CanExecute最终的执行逻辑由构造函数传入的canExecuteMethod以及`Catch(Action<Exception> @catch)`共同组成。

<table><td bgcolor=#6e7b99>如何调用实际的ICommand.CanExecute(parameter)</td></table>

```c#
DelegateCommand command = new DelegateCommand();
// 方式①
command.CanExecute();
// 方式②
((ICommand)command).CanExecute(null);

==把CanExecute弄成方法放在外面，弹窗的时候拿过来使用。==
```

<table><td bgcolor=#6e7b99>CanExecute永不是NULL</td></table>

构造方法不需要传入canExecuteMethod的DelegateCommand，默认永远返回true。即，每次更新命令源都是将其刷新成可用状态IsEnabled=True.

```c#
public DelegateCommand(Action executeMethod)
    : this(executeMethod, () => true) { }
```

#### Execute

```c#
public void Execute()
{
    try
    {
        _executeMethod();
    }
    catch (Exception ex)
    {
        if (!ExceptionHandler.CanHandle(ex))
            throw;

        ExceptionHandler.Handle(ex, null);
    }
}

protected override void Execute(object? parameter)
{
    Execute();
}
```

如果executeMethod出现异常，先检查此异常能否被`统一异常处理中心`处理，如不能抛出此异常。

DelegateCommand的ICommand.Execute在运行时仍旧可能出现异常导致UI线程崩溃！

ICommand.CanExecute最终的执行逻辑由构造函数传入的executeMethod以及`Catch(Action<Exception> @catch)`共同组成。

<table><td bgcolor=gray>如何调用实际的ICommand.Execute</td></table>

```c#
DelegateCommand command = new DelegateCommand();
// 方式①
command.Execute();
// 方式②
((ICommand)command).Execute(null);
```



#### RaiseCanExecuteChanged

`RaiseCanExecuteChanged`以public封装了`CanExecuteChanged?.Invoke(this, EventArgs.Empty)`,有3个优点。

1. Event无法在对象外部调用，public封装后就可以了。这样就可以在ViewModel中让DelegateCommand发布CanExecuteChanged事件了。

2. 简化事件发布代码。

   CanExecuteChanged必须要在UI线程上Invoke，开发者不得不

   ```c#
   Application.Current.Dispatcher.Invoke(()=>CanExecuteChanged?.Invoke(...));
   ```

   RaiseCanExecuteChanged保证在UI线程执行，开发者只需要`RaiseCanExecuteChanged()`,十分简洁。

   ```c#
   protected DelegateCommandBase()
   {
       // View定位ViewModel时，在UI线程实例化ViewModel，所以命令会用_synchronizationContext引存UI线程
       _synchronizationContext = SynchronizationContext.Current;
   }
   
   protected virtual void OnCanExecuteChanged()
   {
       var handler = CanExecuteChanged;
       if (handler != null)
       {
           // 判断RaiseCanExecuteChanged的当前线程是不是UI线程
           if (_synchronizationContext != null && _synchronizationContext != SynchronizationContext.Current)
               _synchronizationContext.Post((o) => handler.Invoke(this, EventArgs.Empty), null);// 推送到UI线程
           else
               handler.Invoke(this, EventArgs.Empty);// 当前是UI线程，直接执行就行了。
       }
   }
   ```

#### ObservesProperty

如果ViewModel的某个属性是影响CanExecute()返回值的因子，如上面代码中的`Info`。当`Info`值变化时，需要调用SaveCommand.RaiseCanExecuteChanged()来刷新Button的启禁用状态。上面的代码实现是

```c#
this.PropertyChanged += (sender, args) =>
{
    if (args.PropertyName == nameof(Info))
    {
        SaveCommand!.RaiseCanExecuteChanged();
    }
};
```

可以替代成

```c#
SaveCommand = new DelegateCommand(SaveExecute).ObservesProperty(()=>Info);
```

它表示当Info的值变化时，同步调用`SaveCommand.RaiseCanExecuteChanged()`.这比注册PropertyChanged事件更简洁。

<font>ObservesProperty不仅可以监控ViewModel的属性，还可以监控ViewModel的属性的属性。</font>

下面的代码，Person的值改变或Person的Name的值改变，会刷新SaveCommand的命令源控件的启禁用状态。

```c#
public class ViewModel : ObservableObject
{
    public DelegateCommand SaveCommand { get; private set; }
    public Person Person { get; }

    public ViewModel()
    {
        SaveCommand = new DelegateCommand(_,()=>!string.IsNullOrEmpty(Person.Name));
        
        // 影响CanExecute的返回值的因子不仅仅只有Name! Person被重新赋值，也会导致CanExecute的返回值变化！所以要同时监控Name和Person.
        
        this.Person.PropertyChanged += (sender, args) =>
        {
            if (args.PropertyName == nameof(Person.Name))
            {
                SaveCommand.RaiseCanExecuteChanged();
            }
        };
        
        this.PropertyChanged += (sender, args) =>
        {
            if (args.PropertyName == nameof(Person))
            {
                SaveCommand.RaiseCanExecuteChanged();
            }
        };
    }
}
```

```c#
SaveCommand = new DelegateCommand(() => { })
    .ObservesProperty(() => Person)
    .ObservesProperty(() => Person.Name);
```

> ObservesProperty()的表达式必须是返回ViewModel中的实现通知机制的直接属性或子属性。
> 

### DelegateCommand\<T>

#### 参数类型必须是引用类型or可空值类型

下面是泛型DelegateCommand的构造函数，它会检查委托参数是不是可为NULL，否则就抛出InvalidCastException异常。

```c#
public DelegateCommand(Action<T> executeMethod, Func<T, bool> canExecuteMethod)
    : base()
{
    TypeInfo genericTypeInfo = typeof(T).GetTypeInfo();
    // DelegateCommand allows object or Nullable<>.  
    // note: Nullable<> is a struct so we cannot use a class constraint.
    if (genericTypeInfo.IsValueType)
    {
        if ((!genericTypeInfo.IsGenericType) || (!typeof(Nullable<>).GetTypeInfo().IsAssignableFrom(genericTypeInfo.GetGenericTypeDefinition().GetTypeInfo())))
        {
            throw new InvalidCastException(Resources.DelegateCommandInvalidGenericPayloadType);
        }
    }
}
```

为什么这样设计呢？

Because `ICommand` takes an `object`, having a value type for `T` would cause unexpected behavior when `CanExecute(null)` is called during XAML initialization for command bindings.

Prism官方建议：

 If you wish to use a value type as a parameter, you must make it nullable by using `DelegateCommand<Nullable<int>>` or the shorthand `?` syntax (`DelegateCommand<int?>`).

> 如果CommandParameter是int、double、bool之类的值类型，ViewModel中命令也要定义成DelegateCommand<int?>,DelegateCommand<double?>,DelegateCommand<bool?>,这并不会产生任何副作用，兼容。
>
> int-->object CommandParameter-->Object parameter-->(nullable<int>)parameter.

### AsyncDelegateCommand

#### CanExecute

```c#
public bool CanExecute()
{
    try
    {
        if (!_enableParallelExecution && IsExecuting)
            return false;

        return _canExecuteMethod?.Invoke() ?? true;
    }
    catch (Exception ex)
    {
        if (!ExceptionHandler.CanHandle(ex))
            throw;

        ExceptionHandler.Handle(ex, null);

        return false;
    }
}
```

ICommand.CanExecute最终的执行逻辑由构造函数传入的`canExecuteMethod`，`IsExecuting`，`Catch(Action<Exception> @catch)`共同组成。`EnableParallelExecution`可以去掉`IsExecuting`对ICommand.CanExecute的影响，允许同时多次响应事件。

<table><td bgcolor=#6e7b99>如何调用实际的ICommand.CanExecute(parameter)</td></table>

```c#
AsyncDelegateCommand command = new AsyncDelegateCommand();
// 方式①
command.CanExecute();
// 方式②
((ICommand)command).CanExecute(null);
```

<table><td bgcolor=#6e7b99>如何调用原生的canExecuteMethod</td></table>

```c#
// 不要用Lambda表达式生成构造函数的额canExecuteMethod就可以了。
// 原生canExecuteMethod，不掺杂异常处理和IsExecuting，不是最终的ICommand.CanExecute(parameter)
private CanExecuteMethod() {} 
new AsyncDelegateCommand(_,CanExecuteMethod);
```

#### Execute

<table>
    <td bgcolor="#6e7b99">Execute wrapper chain</td>
</table>
`void ICommand.Execute`

`async void Execute`

`Task Execute(Token _getCancellationToken)`

`_executeMethod(token)`



异步命令用于调用异步方法，异步方法的签名形式，也就是参数由CommandParameter和CancellationToken组成，可能2者都具有，也可能一个都没。无论哪一种，都会在构造函数中被wrap成都有的形式赋到字段`private readonly Func<CancellationToken, Task> _executeMethod`。

如果构造函数传入的executeMethod本身已经有CancellationToken，则无恙，如果没有，则每次事件发生去执行Execute时，都会运行AsyncDelegateCommand的，该委托在构造命令时通过CancellationTokenSourceFactory(Func\<CancellationToken> factory)指定。


##### Parallel Execution

```c#
new AsyncDelegateCommand(async () => Task.CompletedTask)
    .EnableParallelExecution()
```

`EnableParallelExecution()` 允许`ICommand.Execute`并行，这时候能否执行完全取决于构造函数传入的canExecuteMethod了。

默认情况下，不允许多个`ICommand.Execute`并行，检查到`IsExecutinga`是True后`ICommand.CanExecute`立刻返回false。最终的`ICommand.CanExecute`逻辑由构造函数传入的`canExecuteMethod`+`Catch`+`bool enableParallelExecution`(通过`EnableParallelExecution()`改变其值)共同构成。

##### IsExecuting

反馈ICommand.Execute(parameter)是否正在执行中。在ICommand.Execute(parameter)函数的第一行标志位被置成True，在函数返回前的最后一步置成false。

IsExecuting可用于禁用View的指定区域。比如一个查询指定日期区间的日志查询界面，点击查询按钮，数据库查询尚未结束前，禁用View上的筛选，日期选择，导出为表格等操作按钮。

##### CancellationTokenSourceFactory(Func\<CancellationToken> factory)

实际上每次发生事件时，Invoke的异步方法的签名都带有CancellationToken参数，这个实参就是factory返回的！

==理论上，每次执行Execute，都会执行一次factory生成一个CancellationToken，只是如果AsyncDelegateCommand的构造函数传入的委托异步方法的签名不带CancellationToken(不支持取消)，不使用factory生成的Token而已。如果未CancellationTokenSourceFactory或CancelAfter，则factory为null，默认生成CancellationToken.None==

##### CancelAfter

```c#
new AsyncDelegateCommand(Save, CanSave).CancelAfter(TimeSpan.FromSeconds(5));
```

这个方法相当于指定了一个带超时自动取消的factory。

```c#
public AsyncDelegateCommand CancelAfter(TimeSpan timeout) =>
    CancellationTokenSourceFactory(() => new CancellationTokenSource(timeout).Token);
```









#### Error Handling


catch ex,object,object是CommandParameter。



DelegateCommand和AsyncDelegateCommand的构造函数需要传入的两个方法`executeMethod`和`canExecuteMethod`。在开发中，要捕获二者异常，否则异常会导致UI线程崩溃进而进程退出。所以往往会出现下面的代码片段：

```c#
private bool SaveCanExecute()
{
    try { Convert.ToString(Info) != null; }
    catch { return false; }
}

private void SaveExecute()
{
    try { File.WriteAllText("C:\\info.txt", Convert.ToString(Info)); }
    catch (Exception e) { }
}
```

try catch 让代码变成老太婆的裹脚布--又臭又长。Prism提供简化语法。

```c#
SaveCommand = new DelegateCommand(SaveExecute, SaveCanExecute).Catch<NullReferenceException>(
        nullReferenceException =>
        {
            // Provide specific handling for the specified Exception Type
        })
    .Catch(exception =>
    {
        // Handle any exception thrown
    });
```

它会捕获CanExecute和Execute的异常。



所以，总是为这两个方法提供Try Catch是很有必要的。Prism提供简化语法。

```c#
DownloadCommand = new AsyncDelegateCommand(Download, DownloadCanExecute).Catch<NullReferenceException>(
        nullReferenceException =>
        {
            // Provide specific handling for the specified Exception Type
        })
    .Catch(exception =>
    {
        // Handle any exception thrown
    });
```



##### CanExecute和Execute的异常一定会导致程序崩溃吗

并不一定。

二者的异常都会被抛到UI线程上，如果注册了全局UI线程异常处理并标记成`args.Handled = true`,UI线程不会崩溃。

```c#
Application.Current.Dispatcher.UnhandledException += (sender, args) =>
{
    MessageBox.Show(args.Exception.ToString());
    args.Handled = true;
};
```



##### 为什么AsyncDelegateCommand.Execute的异常会定向到UI线程

`canExecuteMethod`只能是同步方法(Func\<bool>),且一定是在UI线程执行的。所以其异常必在UI线程上。

`executeMethod`，如果是同步方法，全程在UI线程上执行，如果抛出异常，同上。

我们知道返回值是void的异步方法一旦出现异常，调用者的try catch并不能捕获，必然导致程序崩溃闪退。如下代码结构：

```c#
async void XXX()
{
    await executeMethodAsync();
}

try
{
    XXX();
}
catch
{
    // 此处代码不会被执行
}
```

命令源触发的是`void ICommand.Execute(object? parameter)`,其内部调用异步方法executeMethod,这样似乎一旦出现异常，程序就会崩溃。但实际上不是这样，看下AsyncDelegateCommand的源码就知道为什么了。

`executeMethod先被Task Execute()封装一下，其内部捕获异常，这样，Task Execute()就是一个一定不会抛出异常的异步方法，它直接被void ICommand.Execute()调用。`

`await methodAsync().ConfigureAwait(false),如果methodAsync()无异常，后面的代码在哪个线程上执行是不确定的，但是如果出现异常，后面的代码会在上文线程上执行，等同于ConfigureAwait(true),所以，下图中的catch块与Cancellation arg = cancellationToken ?? _getCancellationToken();在相同的线程上执行。`

<img src="https://raw.githubusercontent.com/bruceleecorner/img4md/master/2024/202409031631930.png" alt="image-20240903163133883" style="zoom:67%;" />

<img src="https://raw.githubusercontent.com/bruceleecorner/img4md/master/2024/202409031603657.png" alt="image-20240903160322576" style="zoom:67%;" />

#### AsyncDelegateCommand

AsyncDelegateCommand与DeleagteCommand并没有本质上的差别，只因为它调用的是长时间且不阻塞UI线程异步方法，用户在任务执行中有可乘之机操作UI，进而产生一些额外的开发负担，所以会为ICommand接口提供一些额外的成员。

① 观测异步方法是不是还在执行中。

 同步方法不需要该功能，因为同步方法耗时极短，谈不上有*Executing*这一状态，且结束前一直占用UI线程，使UI处于假死状态不刷新，没办法在UI效果上展示*Executing*。

② 是否允许多个Execute并行。同步方法不需要该功能，因为UI单线程，串行执行抛过来的代码片段，上一次的Execute没结束，不会开始下一次。

③ 超时或手动取消Execute，同步方法不需要该功能，因为必然是短时间方法，不管成功失败，都是极快时间内结束，中途取消没有必要。

异步命令还应当考虑如下几个问题：

- 异步方法的最终结果（成功，失败，超时，取消）要反馈给用户
- 异步方法正在执行中状态甚至进度要反馈给用户
- 异步方法在长时间执行过程中，UI是活着的，但要保证部分View区域不能被操作。

AsyncDelegateCommand继承ICommand接口成员的同时，又额外提供其他接口，以便开发者更容易的实现上述几个需求场景。



<table>
    <td bgcolor="#6e7b99">CanExecute wrapper chain</td>
</table>
==AsyncDelegateCommand==

构造函数传入无参方法`Func<bool> canExecute`,被委托字段`_canExecute`存储。

AsyncDelegateCommand新增`public bool CanExecute()`,它综合AsyncDelegateCommand的两个成员标志IsExecuting(命令是否正在执行，Execute方法尚未返回)和_enableParallelExecution(是否允许多个Execute并行)，以及`_canExecute`构成实际的ICommand.CanExecute(parameter)。

```c#
public bool CanExecute()
{
    try
    {
        if (!_enableParallelExecution && IsExecuting)
            return false;

        return _canExecuteMethod?.Invoke() ?? true;
    }
    catch (Exception ex)
    {
        if (!ExceptionHandler.CanHandle(ex))
            throw;

        ExceptionHandler.Handle(ex, null);

        return false;
    }
}
```



# RelayCommand和RelayCommand\<T>的构造方法
RelayCommand的构造方法如下：
`public RelayCommand(Action execute, Func<bool> canExecute = null)`
RelayCommand\<T>的构造方法如下：
`public RelayCommand(Action<T> execute, Predicate<T> canExecute)`

命令源触发命令后，其绑定的ViewModel中的ICommand属性会执行自己的bool CanExecute(object? parameter)和void Execute(object? parameter)。显然，ViewModel的构造函数必须为其包含的所有ICommand属性的CanExecute和Execute赋予逻辑，这个逻辑就是通过RelayCommand和RelayCommand\<T>的构造方法的参数传入的。但仔细观察RelayCommand和RelayCommand\<T>的构造方法，会发现委托参数要么不带输入参数，要么输入参数的类型不是object而是泛型，这是为什么呢？
RelayCommand和RelayCommand\<T>内部有两个私有委托字段，接受构造函数传入的委托参数。重写后的void Execute(object? parameter)和bool CanExecute(object? parameter)调用委托字段，也就是实际调用的是构造方法传入的委托类型参数。
RelayCommand实现代码如下：

```csharp
public void Execute(object? parameter)
{
    this.execute();
}
```
RelayCommand\<T>实现代码如下：
```csharp
public void Execute(object? parameter)
{
    if (!TryGetCommandArgument(parameter, out T? result))
    {
        ThrowArgumentExceptionForInvalidCommandArgument(parameter);
    }

    this.execute(result);
}
```
可见，RelayCommand会忽略命令参数，即使命令源设置了命令参数。RelayCommand\<T>会使用命令参数且自动的把object类型的命令参数转换成T。所以，<font color=red>**RelayCommand用于与UI元素未设置命令参数的Command绑定，RelayCommand\<T>用于与UI元素的设置了命令参数的Command绑定。**</font>多说一点，如果T是引用类型或可空值类型，命令参数的值也可以是null；如果T是非空值类型如int，double等，命令参数的值是null的话会导致类型转换失败抛出异常。在使用RelayCommand\<T>时，构造函数传入的泛型委托处理程序根据实际情况看是不是要特意处理下null实参。


# 多命令源绑定同一RelayCommand
假设有多个按钮的Command与ViewModel的同一个RelayCommand绑定，可以设置CommandParameter，以供RelayCommand区分是哪一个命令源触发的命令。

# AsyncRelayCommand和AsyncRelayCommand\<T>
我们知道，RelayCommand的Execute()是在UI线程上执行的。如果Execute()是耗时操作，则会卡UI.为了避免这种情况，我们通过构造函数`public RelayCommand(Action execute, Func<bool> canExecute = null)`传入RelayCommand的execute委托方法的内部，会开一个Task执行耗时操作，然后异步等待Task执行结束，并获取结果，如果Task是Task\<T>的话。示例代码如下：
```csharp
RelayCommand = new RelayCommand(async () =>
{
    Task<int> t = Task<int>.Run(() =>
    {
        return LongTimeJob();
    });

    // 等待耗时操作完成
    int ret = await t;
    // 使用结果
    Console.WriteLine(ret);
});

// 耗时操作
int LongTimeJob()
{
    Thread.Sleep(1000);
    return 2022;
}
```
上述写法虽然不再会卡UI，但存在以下不足：
1. 无法在UI显示命令的执行结果 
2. 无法在UI得知命令执行是否成功 
3. 假设我们想在触发命令后，本次命令任务未执行完成，命令源要处于禁用状态，命令执行结束后命令源恢复启用状态。如果Execute()是短时操作，由于时间非常短，用户没有感觉，不禁用命令源也行.如果要禁用命令源，需要在在ViewModel中添加一个私有变量标志位，使用标志位实现CanExecute()，在Execute()的开头和结尾各调用一次NotifyCanExecuteChanged()。代码示例如下：
```csharp
private bool _disableCommand = true;

RelayCommand = new RelayCommand(() =>
{
    _disableCommand = false;
    Application.Current.Dispatcher.Invoke(() =>
    {
          RelayCommand.NotifyCanExecuteChanged();
     });
    // 指定命令任务 await Task.Run(()=>{// 耗时操作})
    _disableCommand = true;
    Application.Current.Dispatcher.Invoke(() =>
    {
          RelayCommand.NotifyCanExecuteChanged();
     });
},()=>_disableCommand);
```
效果还可以，但是还是感觉写了好多代码。。。

CommuityToolkit.Mvvm内置了异步命令`AsyncRelayCommand和AsyncRelayCommand<T>`，前者是无命令参数的异步命令，对应同步命令RelayCommand，后者是有命令参数的异步命令，对应RelayCommand\<T>。这两个异步命令包含了一些属性，能够让我们轻松的在View层绑定这些属性，以监控命令的执行结果，执行是否失败，取消任务执行，命令执行未完成时命令源处于禁用状态。下面以`AsyncRelayCommand<T>`阐述演示异步命令的用法。

`AsyncRelayCommand<T>`签名最全的构造方法是`public AsyncRelayCommand(Func<T?, CancellationToken, Task> cancelableExecute, Predicate<T?> canExecute, AsyncRelayCommandOptions options)`.

`Func<T?, CancellationToken, Task> cancelableExecute` T:命令参数; CancellationToken:取消任务; Task: 如果cancelableExecute是同步方法，表示其内部开的Task的引用，如果cancelableExecute是异步方法，表示Task或Task\<T>类型的返回值。这个Task引用是未来检验异步方法执行结果的“允诺书”，会赋值到AsyncRelayCommand\<T>的Task? ExecutionTask属性，这个属性可以用来与View的依赖属性绑定，用于监测异步任务的执行结果和执行进度。

`Predicate<T?> canExecute`: 命令是否可执行。

`AsyncRelayCommandOptions options` 枚举类型。FlowExceptionsToTaskScheduler: cancelableExecute委托方法抛出的异常被当作线程池异常，忽略之，不会让应用程序崩溃。AllowConcurrentExecutions: 触发命令源后，不禁用命令源，允许上一次命令任务还未完成的情况下，再次触发命令源。但该操作会自动取消上次未执行结束的异步任务，命令的所有属性更换成最后一次触发的异步任务的特征；另外需要注意，取消上一次未完成的异步任务会触发CancellationToken.ThrowIfCancellationRequested()扔出异常，如果options未使用FlowExceptionsToTaskScheduler，同样会导致程序崩溃。 None，命令任务未完成，命令源处于禁用状态，直到当前命令任务执行结束，才会恢复启用状态；同时，如果cancelableExecute委托方法抛出异常会终止应用程序。

`public Task? ExecutionTask`: 当触发命令，命令被取消，命令完成时ViewModel通知UI目标。

`public bool IsRunning`: 等于 !ExecutionTask.IsCompleted. (Task.IsCompleted等于ture表示任务已结束，不管以何种状态结束(任务的状态可能是RanToCompletion、Faulted、Cancelled)。

**案例：使用异步命令监控异步任务的执行状况，并可随时取消命令的执行**
![image](https://img2022.cnblogs.com/blog/1037641/202208/1037641-20220820124508389-1824494237.gif)

![image](uploading...)
```xml
    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
        <StackPanel.Resources>
            <local:TaskResultConverter x:Key="taskConverter" />
            <local:StringToIntConverter x:Key="intConverter" />
            <local:BoolToProgressBarValue x:Key="bpvConverter" />
        </StackPanel.Resources>
        <StackPanel Orientation="Horizontal" Margin="5">
            <TextBlock Text="URL:  " />
            <TextBox MinWidth="200" Text="{Binding Url}" />
        </StackPanel>
        <StackPanel Orientation="Horizontal" Margin="5">
            <TextBlock Text="次数:  " />
            <TextBox Name="txtCount" Width="200" />
        </StackPanel>
        <Button Content="执 行 命 令" Margin="5" Command="{Binding DownloadCommand }"
                CommandParameter="{Binding ElementName=txtCount,Path=Text, Converter={StaticResource intConverter}}" />
        <Button Content="取 消 命 令" Margin="5" Command="{Binding CancelCommand}" />
        <StackPanel Orientation="Horizontal" Margin="5">
            <TextBlock Text="异步命令的执行结果:  " />
            <TextBlock
                Text="{Binding Mode=OneWay,Path= DownloadCommand.ExecutionTask , Converter={StaticResource taskConverter}}"
                Width="500" TextWrapping="Wrap" Background="CadetBlue" />
        </StackPanel>
        <StackPanel Orientation="Horizontal" Margin="5">
            <TextBlock Text="异步命令的线程状态:  " />
            <TextBlock Text="{Binding DownloadCommand.ExecutionTask.Status ,Mode=OneWay}" Width="200"
                       Background="AntiqueWhite" />
        </StackPanel>
        <StackPanel Orientation="Horizontal" Margin="5">
            <TextBlock Text="异步命令的启动状态:  " />
            <ProgressBar Minimum="0" Maximum="100"
                         IsIndeterminate="{Binding Mode=OneWay, Path=DownloadCommand.IsRunning}"
                         Value="{Binding Mode=OneWay, Path=DownloadCommand.IsRunning ,Converter={StaticResource bpvConverter}}"
                         Width="200" />
        </StackPanel>
    </StackPanel>
```

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        this.DataContext = new MainViewModel();
    }
}

public class TaskResultConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is Task<string> task)
        {
            var ret = CommunityToolkit.Common.TaskExtensions.GetResultOrDefault<string>(task);
            return string.IsNullOrEmpty(ret) ? string.Empty : ret.Substring(0, 100);
        }

        return string.Empty;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}

public class StringToIntConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        string cmdParam = value as string;
        return string.IsNullOrEmpty(cmdParam) ? 0 : System.Convert.ToInt32(value);
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}

public class BoolToProgressBarValue : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        bool isRunning = System.Convert.ToBoolean(value);
        return isRunning ? 0 : 100;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```

```csharp
public class MainViewModel
{
    public string Url { get; set; } =
        "https://docs.microsoft.com/en-us/windows/communitytoolkit/mvvm/asyncrelaycommand";

    public IAsyncRelayCommand<int> DownloadCommand { get; }

    public RelayCommand CancelCommand { get; }

    public MainViewModel()
    {
        DownloadCommand = new AsyncRelayCommand<int>(Excecute, AsyncRelayCommandOptions.AllowConcurrentExecutions | AsyncRelayCommandOptions.FlowExceptionsToTaskScheduler);

        CancelCommand = new RelayCommand(() =>
        {
            DownloadCommand.Cancel();
        });
    }

    /// <summary> Execute(object
    /// parameter)调用的异步方法。返回的Task<string>承诺暴露到AsyncRelayCommand<T>的ExecutionTask属性，用于与数据绑定
    /// </summary> <param name="count">命令参数</param> <param name="token">取消令牌</param> <returns></returns>
    private async Task<string> Excecute(int count, CancellationToken token)
    {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < count; i++)
        {
            string content;
            HttpClient client = new HttpClient();
            content = await client.GetStringAsync(Url, token);
            sb.Append(content.Replace(Environment.NewLine, ""));
            await Task.Delay(1000);
            token.ThrowIfCancellationRequested();
        }
        return sb.ToString();
    }
}
```

View绑定异步任务的Result时，必须使用值转换器，在转换器中使用CommunityToolkit.Common Nuget包中的 `CommunityToolkit.Common.TaskExtensions.GetResultOrDefault<string>(task)`，该方法在task未结束时，立刻返会null,并不阻塞UI线程。

注意区分线程的状态。检测线程是否取消，有两种方式，一种是token.IsCancellationRequested，被取消后，线程的状态是RunToCompletion；另一种是token.ThrowIfCancellationRequested()，被取消后，线程抛出OperationCancelledException，线程的状态时Cancelled；线程抛出其他异常，线程的状态是Faulted。

