**Radial: 辐射状的**

RadialGradientBrush，径向渐变画刷，它的颜色效果是从一个原点开始向外呈环形状向外“辐射”，可以想象成一颗石头落在水面上，向外辐射的波纹，石头的落点就是起始原点(GradientOrigin)。

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231026211148068-553667603.png" width="30%" height="30%" title=""/></div>

RadialGradientBrush的颜色分成2个部分，第一部分是渐变区，第二部分是纯色区。
渐变区在1个大椭圆之内，大椭圆由Center，RadiusX，RadiusY确定，Center是圆心，RadiusX是长轴，RadiusY是短轴，当RadiusX等于RadiusY，椭圆便变成圆。
Center,RadiusX,RadiusY的取值范围是[0,1]，表示RadialGradientBrush作用到的控件在X和Y方向上的比例。假设控件Rectangle的X=300，Y=300,
`<RadialGradientBrush Center="0.5,0.5" RadiusX="0.5" RadiusY="0.5">`,则圆心坐标(150,150),长轴长度是0.5 * 300 = 150，短轴长度是0.5 * 300 = 150。半径是150的圆形区域是渐变区，当SpreadMethod="Pad"时，圆形之外是纯色区，大椭圆的外围的面积可以想象成是无限大，颜色是纯色，颜色是大椭圆内部Offset最大的梯度颜色。

现在再把注意点放到渐变区，径向渐变具有起点和终点，起点是焦点(GradientOrigin)，并不是Center！渐变由焦点出发，终点是大椭圆的边(由Center，RadiusX，RadiusY确定)。GradientOrigin始终在大椭圆内，如果GradientOrigin和Center相同，椭圆区的环形带的粗细是均匀的，如果GradientOrigin和Center不相同，会导致一边的环形带比较细，渐变较快，颜色显得比较浓，另一边环形带比较粗，渐变较慢，颜色显得比较淡。

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231026213654366-1629311374.png" width="30%" height="30%" title=""/></div>

`GradientOrigin是一个点，假想此点与大椭圆的边上的所有点有无数个连接线，GradientStop的Offset的取值范围是[0,1]，定位方式是在连接线上的比例。GradientOrigin和Center如果不重叠，连接线较短的一侧渐变的很快(颜色较浓)，连接线较长的一侧渐变的较慢(颜色较淡)。`

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231025005257084-1417422538.png" width="30%" height="30%" title=""/></div>

---


RadialGradientBrush有个属性叫SpreadMethod，SpreadMethod=Pad时，大椭圆外的颜色是纯色，SpreadMethod=Repeat时，大椭圆外重复渐变效果，如下：

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231025010510132-1676483251.png" width="30%" height="30%" title=""/></div>

---
---
---

**GradientOrigin和Center取值不同时的展示案例图**

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231025010619433-749761526.png" width="30%" height="30%" title=""/></div>

<br>
<br>

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231025010704083-751791108.png" width="30%" height="30%" title=""/></div>


```xml
<Grid Margin="10">
    <Rectangle Width="300"
               Height="300"
               RadiusX="150"
               RadiusY="150">
        <Rectangle.Fill>
            <RadialGradientBrush Center="0.5,0.5" GradientOrigin="0.25,0.5" RadiusX="0.5" RadiusY="0.5">
                <RadialGradientBrush.GradientStops>
                    <GradientStop Offset="0.2" Color="#FFFFFFFF" />
                    <GradientStop Offset="1" Color="#FF444444" />
                </RadialGradientBrush.GradientStops>
            </RadialGradientBrush>
        </Rectangle.Fill>
    </Rectangle>
</Grid>
```

![image](https://img2023.cnblogs.com/blog/1037641/202310/1037641-20231025011257473-1350327923.png)