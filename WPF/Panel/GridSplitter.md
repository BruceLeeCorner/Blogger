> GidSplitter放在Grid的某个单元格中才能实现分割。分割条所在单元格所处的整行或整列为楚河汉界，将Grid一分为二(GridSpliter所在行的上边所有行是一个分割区域，下面的所有行是另一个分割区域)。即使一行有多个单元格，而分割条只是放到该行的其中一个单元格，但仍旧是以整行为边界去分割Grid。所以建议设置GridSplitter的RowSpan或ColumnSpan为合适值，让其横跨整个Grid,以使宽度或高度与实际分割边界长度一样，这样更合理，而不是将GridSplitter限制在某个单元格中。如图，红色的GridSpliter的长度虽然只占Grid高度的三分之一，但是拖动时，可分割整个Grid。

![](https://raw.githubusercontent.com/BruceLeeCorner/img4md/master/2024202411252009336.gif)

---

> GridSplitter可竖可横，竖向分割条左右拖拽调整左右侧尺寸，横向分割条上下拖拽调整上下侧尺寸。
> VerticalAlignment=Stretch HorizontalAlignment=Center Width=x 是竖向分割
> VerticalAlignment=Center HorizontalAlignment=Stretch Height=x 是横向分割.
> `GridSpliter默认外观是小小的正方形，竖向Stretch相当于拉长，所以是竖向分割，横向Center,是让其宽度变成0，然后通过Width=XX显示指定宽度。`
---

> Grid应当特意多出一列或一行专门放GridSplitter，`且该行或列的高度或宽度设置成auto`.
---

> 如果希望GridSplitter以更大的幅度(如每次10个单位)进行移动，可以调整DragIncrement属性。如果希望控制区域的最大尺寸和最小尺寸，不让其挤压到消失，可以在ColumnDefinitions中为列的宽度设置合适的属性。GridSplitter默认样式是灰色柱状，可以设置其Background为丰富多彩的复杂画刷，让其更显眼更美观更活泼。也可以结合Style,Trigger，Behavior，BorderBrush，BorderThickness，Width制作鼠标经过GridSplitter的动画效果。

---

> 可以在被分割的区域内再放置一个含有Splitter的Grid，这样就实现了多个可独立调整尺寸的区域。
```XML
<Grid ShowGridLines="True">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="auto" />
        <ColumnDefinition Width="*" />
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition />
        <RowDefinition />
        <RowDefinition />
    </Grid.RowDefinitions>
    <GridSplitter
        Grid.Row="1"
        Grid.Column="1"
        Width="15"
        HorizontalAlignment="Center"
        VerticalAlignment="Stretch"
        Background="Red"
        BorderBrush="Orange"
        BorderThickness="5"
        ShowsPreview="False" />

    <Grid
        Grid.Row="0"
        Grid.Column="2"
        ShowGridLines="True">
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition Width="auto" />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <GridSplitter
            Grid.Column="1"
            Width="4"
            HorizontalAlignment="Center"
            VerticalAlignment="Stretch"
            Background="Lime" />
    </Grid>
</Grid>
```



![](https://raw.githubusercontent.com/BruceLeeCorner/img4md/master/2024202411252017690.gif)



