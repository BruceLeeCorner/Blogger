# 枚举

```c#
public static class EnumHelper
{
    public static bool IsSame(this Enum @enum, Enum enum2) => @enum.GetType() == enum2.GetType() && @enum.ToInt32() == enum2.ToInt32();
    public static int ToInt32(this Enum @enum) => ((IConvertible)@enum).ToInt32(null);
}
```



## 位运算

一般情况下，一个枚举变量只能表示一种状态，但是C#的枚举支持一个枚举变量表示多种状态。枚举位运算的优点是能够仍旧以枚举类型实现集合才有的操作，如增、删、查、遍历。

枚举定义需要满足以下要求：

枚举成员的值必须是`2的整数次幂`，且额外添加无意义的初始状态`None=0`，尽量添加特性`[System.Flags]`.

```c#
[System.Flags]
public enum WeekDay
{
    None = 0,
    Monday = 1,     // 1 << 0  0000_0001
    Tuesday = 2,    // 1 << 1  0000_0010
    Wednesday = 4,  // 1 << 2  0000_0100
    Thursday = 8,   // 1 << 3  0000_1000
    Friday = 16,    // 1 << 4  0001_0000
    Saturday = 32,  // 1 << 5  0010_0000
    Sunday = 64     // 1 << 6  0100_0000
}
```

**① add    |    重复添加无影响**

```c#
WeekDay day = WeekDay.None;

day = day | WeekDay.Monday;
day |= WeekDay.Tuesday;

WeekDay day2 = WeekDay.Wednesday | WeekDay.Thursday;
day |= day2;

// 上述代码可以用下面的集合理解
List<WeekDay> days = new List<WeekDay>();
days.Add(WeekDay.Monday);
days.Add(WeekDay.Tuesday);
days.Add(WeekDay.Wednesday);
days.Add(WeekDay.Thursday);
```

以二进制位操作理解，0000_0001 | 0000_0010 | 0000_0100 | 0000_1000 = 0000_1111.

枚举变量`day`对应整数是`15`,==我们发现15并不是合法的枚举定义(WeekDay的没有任何成员映射到15)，WeekDay day = (WeekDay)15不能表示任意一个枚举变量。如果只是不满足位运算定义的普通枚举情况下，15是无意义的，通常表示我们的代码逻辑出现问题，程序会发生故障，但是针对特意满足位运算而定义的枚举，15是有意义的，它可以表示多个枚举变量！==

**② rmv   &~    有则删除，无则无影响**

```c#
day = day &(~WeekDays.Monday);
// 上述代码可以用下面的集合理解
List<WeekDay> days = new List<WeekDay>();
days.Remove(WeekDay.Monday);
```

以二进制位操作理解，0000_1111 & ~(0000_0001) = 0000_1110.

**③ toggle    ^    有则删除，无则加之**

```c#
day = day ^ WeekDays.Wednesday;
```

**④ contain    & ==    判断是否含有某个枚举状态**

```c#
var hasElem = (day & WeekDays.Saturday) == WeekDays.Saturday;
```



至于[System.Flags] 这个属性，对运算结果没什么影响，但是如果ToString() 的话，则会不同。没有该属性时，ToString()的会输出对应的数值，加上该属性，则输出 Thursday, Friday, Saturday, 因此更好理解。

```c#
//对于以上枚举，如果不带Flags特性
Console.WriteLine(WeekDay.Monday | WeekDay.Tuesday); //3
//对于以上枚举，如果带上Flags特性
Console.WriteLine(WeekDay.Monday | WeekDay.Tuesday); //Monday, Tuesday
※特性Flags一般和枚举组合一起使用，便于查看枚举中的枚举值的组合；
```

## 真实案例

Win32 API 注册全局快捷键，`ModifierKeys fsModifiers`接收修饰键，我们知道修饰键可能不止一个，但是API的参数类型并不是ModifierKeys[ ]，而仍旧是ModifierKeys。

```c#
[DllImport("user32.dll", SetLastError = true)]
public static extern bool RegisterHotKey(IntPtr hWnd, int id, ModifierKeys fsModifiers, int vk);

ModifierKeys modifierKeys = ModifierKeys.None;
modifierKeys = ModifierKeys.Control | ModifierKeys.Alt;
RegisterHotKey(_,_,modifierKeys,_);
```

