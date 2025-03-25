# Trigger Action Behavior区别和联系





# Trigger

## PropertyChangedTrigger

> `PropertyChangedTrigger` 监控某个属性的变化，当属性的值发生变化时，会执行其内部的Action。

`Binding`指明监控哪个对象的哪个属性。`Binding`缺省的Source是`Interaction.Triggers`附加到的对象，也可以根据VisualTree层级结构使用RelativeSource，或ElementName来指定被监控对象。

如下代码片段，当`Button`的DataContext或根元素`Window`的IsActive变化时，会执行ChangePropertyAction。

```xml
<Button>
    <i:Interaction.Triggers>
        <i:PropertyChangedTrigger Binding="{Binding}">
            <i:ChangePropertyAction
                Increment="True"
                PropertyName="Width"
                Value="20" />
        </i:PropertyChangedTrigger>
        <i:PropertyChangedTrigger Binding="{Binding RelativeSource={RelativeSource AncestorType=Window}, Path=IsActive}">
            <i:ChangePropertyAction
                Increment="True"
                PropertyName="Width"
                Value="20" />
        </i:PropertyChangedTrigger>
    </i:Interaction.Triggers>
</Button>
```

## DataTrigger

> 监控某个属性的值满足某个条件时，执行其内部的Action。

`DataTrigger`继承自PropertyChangedTrigger,新增了`Value`和`Comparison`两个依赖属性。

`Binding` 指定被监控的属性

`Value` & `Comparison` 当`Binding`指定的属性与`Value`的关系满足`Comparison`条件时，执行Action。

`Comparison`是枚举类型，有6种条件关系，如下：

```c#
public enum ComparisonConditionType
{
    Equal,
    NotEqual,
    LessThan,
    LessThanOrEqual,
    GreaterThan,
    GreaterThanOrEqual
}
```

如下代码片段，当`CheckBox`的`IsChecked`属性等于True或`IntegerUpDown`的`Value`属性`不小于`5000时，会执行ChangePropertyAction。

```xml
<i:Interaction.Triggers>
    <i:DataTrigger
        Binding="{Binding ElementName=CheckBox, Path=IsChecked}"
        Comparison="Equal"
        Value="True">
        <i:ChangePropertyAction
            Increment="True"
            PropertyName="Width"
            Value="20" />
    </i:DataTrigger>
    <i:DataTrigger
        Binding="{Binding ElementName=IntegerUpDown, Path=Value}"
        Comparison="GreaterThanOrEqual"
        Value="5000">
        <i:ChangePropertyAction
            Increment="True"
            PropertyName="Width"
            Value="20" />
    </i:DataTrigger>
</i:Interaction.Triggers>
```

## EventTrigger

> `EventTrigger` 监控事件，当发生事件时，会执行其内部的Action。

`EventName` 指定要监控的事件名称。

`SourceName` 指定要监控哪个控件的事件。字符串类型，接受控件的Name值。

`SourceObject` 也是用于指定要监控哪个控件的事件，比`SourceName`更强大，可以通过Binding指向Ancestor Element，Templated Element，Itself等。

显然，在使用`EventTrigger`应当只显示指定`SourceName`和`SourceObject`之一，如果二者都不指定，缺省情况下，SourceObject是`Interaction.Triggers`的宿主元素。

`EventName`必须是`SourceObject`具有的CLR普通事件成员，也就是必须是路由事件的封装器标识符。所以，`EventName="ClickEvent"` ， `EventName="ButtonBase.ClickEvent"`，`EventName="(Grid.Column)"`这3种形式都是错误的，不被允许的。



## TimerTrigger

> `TimerTrigger`继承自`EventTrigger`，与`EventTrigger`不同是，它可以

`TotalTicks`

`MillisecondsPerTick`

## KeyTrigger

监控键盘事件。当按键被按下时，会执行Action。

`Key` 

`ModifierKeys`  Control  Alt  Shift  Windows

`ActiveOnFocus` True, Interaction.Triggers附加到的控件获得焦点时，按下按键才会触发执行Action；False，即使焦点不在触发器宿主控件上，只要宿主控件所在窗体处于激活状态，无论当前焦点在哪个控件上，按下按键也会执行Action。

`FiredOn` KeyUp 松开按键后执行Action; KeyDown 按键被按下后立刻执行Action。

```xml
<i:KeyTrigger Key="D" Modifiers="Ctrl" ActiveOnFocus="False" FiredOn="KeyUp" >
    <i:ChangePropertyAction />
</i:KeyTrigger>
```

## StoryboardCompletedTrigger

当StoryBoard结束后，立刻调用Action。

在Trigger被Attach到WPF元素的Trigger集合时，会把InvokeActions()注册进StoryBoard的Completed事件。

```xml
<i:StoryboardCompletedTrigger Storyboard="{StaticResource sb}">
    <i:CallMethodAction MethodName="Show"  />
</i:StoryboardCompletedTrigger>
```



# Action

## ChangePropertyAction

ChangePropertyAction represents an action that will change a specified property to a specified value when invoked.

Using this behavior causes the property with the given `PropertyName` on the `TargetObject` to take on the given `Value`.

If `Increment` is set to ture, the behavior will first attempt to increment by the given `Value`. If the property can not be incremented, it will be set to the value instead.

If a `Duration` is set, the transition from the old value to the new value will be animated.

默认的`TargetObject`和`SourceObject`都是`Trigger`的宿主控件，如下代码片段，若不显示指定，默认的是Button。

```xml
<Button x:Name="Button">
  <i:Interaction.Triggers>
    <i:EventTrigger EventName="Click" SourceObject="{Binding ElementName=Button}">
      <i:ChangePropertyAction TargetObject="{Binding ElementName=DataTriggerRectangle}" PropertyName="Fill" Value="LightYellow"/>
    </i:EventTrigger>
  </i:Interaction.Triggers>
</Button>
```

## CallMethodAction

`TargetObject` 指定调用哪个对象的方法。

`MethodName` 指定调用方法的名称。

> 首先声明，MethodName方法的签名形式必须是 public void MethodName( ) 或 public void MethodName(object sender, EventArgs e) (注:第2个参数e的类型也可以是EventArgs的派生类)。原因请看如下源码片段，


```c#
private static bool AreMethodParamsValid(ParameterInfo[] methodParams)
{
    if (methodParams.Length == 2)
    {
        if (methodParams[0].ParameterType != typeof(object))
            return false;
        if (!typeof(EventArgs).IsAssignableFrom(methodParams[1].ParameterType))
            return false;
    }
    else if (methodParams.Length != 0)
        return false;
    return true;
}

private bool IsMethodValid(MethodInfo method)
{
    if (!string.Equals(method.Name, this.MethodName, StringComparison.Ordinal))
        return false;
    if (method.ReturnType != typeof(void))
        return false;
    return true;
}
```

> Trigger满足条件时会调用MethodName,那么WPF向sender和e传入的实参是什么？
>
> sender是Action所属的Trigger的宿主控件，也就是Interaction.Trigger所附加到的控件，并不是Trigger的SourceObject。e是Trigger在InvokeActions(object parameter)时传入的。不同类型的Trigger传入的数据不同(`但必须都要继承了EventArgs`)，EventTrigger传送的parameter是事件参数RoutedEventArgs的派生类，如KeyEventArgs，MouseEventArgs。原因请看如下源码片段，

```c#
if (parameters.Length == 0)
{
    methodDescriptor.MethodInfo.Invoke(this.Target, null);
}
else if (parameters.Length == 2 && this.AssociatedObject != null && parameter != null)
{
    if (parameters[0].ParameterType.IsAssignableFrom(this.AssociatedObject.GetType())
        && parameters[1].ParameterType.IsAssignableFrom(parameter.GetType()))
    {
        methodDescriptor.MethodInfo.Invoke(this.Target, new object[] { this.AssociatedObject, parameter });
    }
}
```



**确定方法的过程**

1. 如果在TargetObject中找到==public无返回值且无参==的MethodName方法，则调用该方法，过程结束。

2. 如果步骤①未找到，会继续寻找有2个参数的方法`public void MethodName(object sender, EventArgs e)`，如果未找到，会抛出异常或无反应。

3. 未指明TargetObject时，默认就是Interaction.Triggers的宿主控件，这点可见如下源码：

    

    ```c#
    private object Target
    {
        get => this.TargetObject ?? this.AssociatedObject;
    }
    ```

    

**代码案例**

用户控件被双击后，执行Delete方法。

```xml
<UserControl x:Name="uercontrol">
    <Border BorderThickness="3">
        <Button
            x:Name="button">
            <i:Interaction.Triggers>
                <i:EventTrigger EventName="MouseDoubleClick" SourceObject="{Binding ElementName=uercontrol}">
                    <i:CallMethodAction MethodName="Delete" TargetObject="{Binding}" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </Button>   
    </Border>
</UserControl>
```

UserControl的DataContext对应的ViewModel的Delete方法：

```c#
public void Delete(object sender, EventArgs e)
{
    ;
}
```

1. sender 实参是`Button`，并不是`UserControl`

2. e 的实参是`MouseButtonEventArgs`

3. `EventName="MouseDoubleClick"`, 相当于`Uercontrol.AddHandler(MouseDoubleClickRoutedEvent) {InvokeActions();} `.  所以即使双击`Border`同样也能触发CallMethodAction，因为顶级元素UserControl订阅了冒泡路由事件。 我们知道，路由事件处理函数的第一个参数是订阅者，而非发布者，为了保持与此概念相同，所以我建议，尽量把Interaction.Triggers直接写在被监控的控件上，然后缺省SourceObject，这样CallMethodAction调用的方法的第一个参数既是Trigger宿主，同时也是被监控的控件(即路由事件的订阅者)。

    


**补充 :** 在上述代码样例中，如果程序员提供了重载方法如下：

```c#
public void Delete(object sender, EventArgs e) { }
public void Delete( ) { }
```

那么，最终调用第2个无参方法，而不是第1个。

## InvokeCommandAction

有缺陷，不建议使用，推荐Prism的InvokeCommandAction。

## ControlStoryboardAction 

可以控制动画，如播放，暂停，恢复等操作。

ControlStoryboardAction represents an action that will change the state of the specified Storyboard when executed.

Using this behavior enables the triggering and controlling of a specified `Storyboard` using the `ControlStoryboard` property.

**ControlStoryboardOption enumeration**

| **Member Name** | **Description**                                              |
| --------------- | ------------------------------------------------------------ |
| Play            | Starts the storyboard.                                       |
| Stop            | Stops the storyboard.                                        |
| TogglePlayPause | Toggles the storyboard between playing and pausing.          |
| Pause           | Pauses the storyboard.                                       |
| Resume          | Resumes the storyboard after being paused.                   |
| SkipToFill      | Advances the storyboard's current time to the end of its active period. |

**Sample Code**

```xml
<UserControl.Resources>
  <Storyboard x:Key="StoryboardSample" AutoReverse="True" RepeatBehavior="Forever">
    <DoubleAnimation Duration="0:0:2.2" To="0.35"
      Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleX)"
      Storyboard.TargetName="StoryboardRectangle" />
    <DoubleAnimation Duration="0:0:2.2" To="0.35"
      Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleY)"
      Storyboard.TargetName="StoryboardRectangle" />
  </Storyboard>
 </UserControl.Resources>

<Rectangle x:Name="StoryboardRectangle" />
<Button x:Name="button">
  <i:Interaction.Triggers>
    <i:EventTrigger EventName="Click">
      <i:ControlStoryboardAction Storyboard="{StaticResource StoryboardSample}" ControlStoryboardOption="TogglePlayPause"/>
    </i:EventTrigger>
  </i:Interaction.Triggers>
</Button>
```

## LaunchUriOrFileAction

LaunchUriOrFileAction opens a file or website using the default program for that file type.

This behavior opens the file or uri at the given `Path` when triggered unless `IsEnabled` is set to false.

```xml
<Button x:Name="button">
  <i:Interaction.Triggers>
    <i:EventTrigger EventName="Click" SourceObject="{Binding ElementName=button}">
      <i:LaunchUriOrFileAction Path="https://www.visualstudio.com" />
    </i:EventTrigger>
  </i:Interaction.Triggers>
</Button>
```

该Action不支持启动进程时传入参数，可以继承LaunchUriOrFileAction重写Invoke方法实现，重点类是`ProcessStartInfo`.

# Behavior

# 如何自定义Behavior

# 如何自定义Action

