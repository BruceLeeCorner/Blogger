## Geometry

Geometry的位置数据是以其外层的控件为参考的，外层控件的最左上角坐标是(0,0)。可以在Geometry外层套一个任意尺寸的Canvas，Geometry相对Canvas绝对定位，Canvas可以足够大以便细腻的作图。使用Canvas图案时，将其放置在ViewBox中，就可以等比缩放图案了，实际图案的大小就是ViewBox的尺寸。如下代码展示的图案尺寸是60×60,而不是600×600.

```xml
<ViewBox Width="60" Height="60">
    <Canvas Width="600" Height="600">
        <Path>
            <Path.Data>
                <LineGeometry StartPoint="100,100" EndPoint="500,500">
                </LineGeometry>
            </Path.Data>
        </Path>
    </Canvas>
</ViewBox>
```

# 2D Graphics & Transform

## 如何确定一个Shape或Geometry的外矩

- 内置的常见图形，如Ellipse，Rectangle，Line等，找到图形上下左右4个方向最远的点，水平穿过上下两点，垂直穿过左右两点，4条直线相交截出的矩形就是此不规则图形的外矩。如下图，矩形ABCD是不规则四边形的外矩，图形的原点坐标(0,0)是点A。

<img src="https://gitee.com/li6v/img/raw/master/202407082229461.png" alt="image-20240708222929345" style="zoom:67%;" />

## 如何确定图形的位置

- 

## 图形内部坐标的含义

`LineGeometry`,`EllipseGeometry`,`PathGeometry`,`Polyline`等Geometry，内部的坐标是以图形本身左上角为(0,0)点为参考系的。快速的定位方法就是想象图形有width和height，左上角坐标是(0,0).

```xml
<Canvas Width="200" Height="200">
    <Polyline
        Canvas.Left="75"
        Canvas.Top="50"
        Points="25,25 0,50 25,75 50,50 25,25 25,0"
        Stroke="Blue"
        StrokeThickness="2" />
</Canvas>
```

<img src="https://gitee.com/li6v/img/raw/master/202407082151431.png" alt="image-20240708215105343" style="zoom:67%;" />

```xml
<Grid
    Width="200"
    Height="200"
    Background="AliceBlue">
    <Polyline
        HorizontalAlignment="Center"
        VerticalAlignment="Center"
        Points="25,75 0,150"
        Stroke="Blue"
        StrokeThickness="2" />
</Grid>
```

假设出图形的最左上角(0,0)，然后画出Points的所有点，再画出4条最外围的水平垂直线(注意：假设出的(0,0)点便是最上和最左的点！)，得到图形的矩形面积，最后平移矩形至矩形中心与Grid中心重合，便是图形的最终位置和形状。

<img src="https://gitee.com/li6v/img/raw/master/202407082236044.png" alt="image-20240708223648988" style="zoom:67%;" />

## 旋转

`<RotateTransform Angle="45" CenterX="25" CenterY="50" />` 

- CenterX和CenterY是旋转中心，CenterX和CenterY是以被旋转图形最左上角(0,0)为参考系的绝对坐标，Angle是旋转角度。

```xml
<Canvas
    Width="200"
    Height="200"
    Background="Azure">
    <Polyline
        Canvas.Left="75"
        Canvas.Top="50"
        Points="25,25 0,50 25,75 50,50 25,25 25,0"
        RenderTransformOrigin="0,0"
        Stroke="Blue"
        StrokeThickness="10">
        <Polyline.RenderTransform>
            <RotateTransform Angle="45" CenterX="25" CenterY="50" />
        </Polyline.RenderTransform>
    </Polyline>
</Canvas>
```

<img src="https://gitee.com/li6v/img/raw/master/202407092131009.png" alt="image-20240709213123940" style="zoom:67%;" />

旋转前

<img src="https://gitee.com/li6v/img/raw/master/202407092134210.png" alt="image-20240709213400165" style="zoom:67%;" />

<img src="https://gitee.com/li6v/img/raw/master/202407092138904.png" alt="image-20240709213803851" style="zoom:67%;" />



## 如何确定旋转后的图形位置

- 画出图形外矩，找到旋转中心
- 找到若干图形的特征点，连接点和旋转中心作为半径，旋转中心作为圆心，根据Angle画弧，弧的终点就是特征点旋转后的位置
- 将若干特征点旋转后的位置连接起来，便得到旋转后的位置。

