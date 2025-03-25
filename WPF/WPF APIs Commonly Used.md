## 颜色转换

```c#
var color = (Color)System.Windows.Media.ColorConverter.ConvertFromString("#00AA68C2");
var hexColorString = $"#{color.A:X2}{color.R:X2}{color.G:X2}{color.B:X2}";
```

