==FontWeight==

序列化：可以转换成整数。

```c#
textBlock.FontWeight.ToOpenTypeWeight();
```

反序列化：由整数转换成实例

```c#
textBlock.FontWeight = FontWeight.FromOpenTypeWeight(100);
```

WPF内置了如下比较典型的FontWeight，每个实例其实对应着一个整数。

```c#
public static class FontWeights
{
    public static FontWeight Black { get; }
    public static FontWeight Bold { get; }
    public static FontWeight DemiBold { get; }
    public static FontWeight ExtraBlack { get; }
    public static FontWeight ExtraBold { get; }
    public static FontWeight ExtraLight { get; }
    public static FontWeight Heavy { get; }
    public static FontWeight Light { get; }
    public static FontWeight Medium { get; }
    public static FontWeight Normal { get; }
    public static FontWeight Regular { get; }
    public static FontWeight SemiBold { get; }
    public static FontWeight Thin { get; }
    public static FontWeight UltraBlack { get; }
    public static FontWeight UltraBold { get; }
    public static FontWeight UltraLight { get; }
}
```



==FontStretch==

同上。

```c#
public static class FontStretches
{
    public static FontStretch Condensed { get; }
    public static FontStretch Expanded { get; }
    public static FontStretch ExtraCondensed { get; }
    public static FontStretch ExtraExpanded { get; }
    public static FontStretch Medium { get; }
    public static FontStretch Normal { get; }
    public static FontStretch SemiCondensed { get; }
    public static FontStretch SemiExpanded { get; }
    public static FontStretch UltraCondensed { get; }
    public static FontStretch UltraExpanded { get; }
}
```



==FontStyle==

序列化：

```c#
textBlock.FontStyle.ToString()
```

反序列化：提前最好字符串映射实例的表，然后查表。

```c#
public static class FontStyles
{
    public static FontStyle Italic { get; }
    public static FontStyle Normal { get; }
    public static FontStyle Oblique { get; }
}
```



==FontFamily==

序列化

```c#
textBlock.FontFamily.Source
```

反序列化

```c#
new FontFamily("微软雅黑")
```



**FontFamily，FontWeight，FontStyle，FontStretch 4者之间的关系**

先有Family确认体系，然后通过其FamilyTypefaces属性拿到其修饰风格。一个Family可以只提供少量的FontStretch，比如只提供Normal，并不提供Medium，Light等修饰风格。

Family相当于一个目录，拿到目录后，然后遍历整个目录，获取它有几种FontWeight，FontStyle，FontStretch。

```c#
var interfaces = family.FamilyTypefaces;

foreach (var item in interfaces)
{

    MessageBox.Show(item.Stretch.ToString());
    MessageBox.Show(item.Weight.ToString());
    MessageBox.Show(item.Style.ToString());
}
```





