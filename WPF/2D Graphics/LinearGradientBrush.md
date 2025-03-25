StartPoint和EndPoint的坐标的取值范围是[0,1]，单位不是尺寸，而是在LinearGradientBrush作用的控件的X和Y上的占比。

假设我们将Grid的Background的值设置成LinearGradientBrush，Grid的尺寸是X=200,Y=500；StartPoint = "0.5,0.5",意味着StartPoint的坐标是(100,250)。

StartPoint与EndPoint构成一条直线，颜色沿着这条直线渐变。Offset的值的取值范围是[0,1]，Offset指在这条直线的长度占比。过Offset点画一条垂直于Start-End线段的直线，该直线是纯色。

## 示例一

```xml
<LinearGradientBrush StartPoint="0,0.5" EndPoint="1,0.5">
    <GradientStop Offset="0.25" Color="Black"/>
    <GradientStop Offset="0.5" Color="Red"/>
    <GradientStop Offset="0.75" Color="Blue"/>
    <GradientStop Offset="1" Color="Green"/>
</LinearGradientBrush>
```

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231011143341127-702503312.png" width="30%" height="30%" title=""/></div>


## 示例二

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231011143324241-124104151.png" width="30%" height="30%" title=""/></div>