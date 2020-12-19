# <center>锚定布局<center>

# 1. 锚定

除了传统的`Grid`，`Row`和`Column`布局，Qt Quick还提供了锚定布局。每个项目都有7条不可见的“锚线”：左，水平居中，右，顶部，垂直居中，基线和底部。

![](https://materiaalit.github.io/qt-mooc/img/part-3/QML_anchors-0a678f84.png)

基线（上图未显示）对应于文本所在的假想线。对于没有文本的项目，它与顶部相同。

Qt Quick锚定系统允许您定义不同项目的锚定线之间的关系。例如，您可以编写：

    Rectangle { id: rect1; ... }
    Rectangle { id: rect2; anchors.left: rect1.right; ... }
    
本例中，`rect2`的左边缘与`rect1`的右边缘绑定，产生如下结果:

![](https://materiaalit.github.io/qt-mooc/img/part-3/anchors_1-72f90073.png)

您也可以指定多个锚定。例如：

    Rectangle { id: rect1; ... }
    Rectangle { id: rect2; anchors.left: rect1.right; anchors.top: rect1.bottom; ... }

![](https://materiaalit.github.io/qt-mooc/img/part-3/anchors_2-3fad9d3e.png)

通过指定多个水平或垂直锚定，则可以控制项的大小。下面的例子中，`rect2`锚定在`rect1`的右边和`rect3`的左边。如果其中一个蓝色矩形被移动，`rect2`将根据需要自动拉伸和收缩：

    Rectangle { id: rect1; x: 0; ... }
    Rectangle { id: rect2; anchors.left: rect1.right; anchors.right: rect3.left; ... }
    Rectangle { id: rect3; x: 150; ... }
    

![](https://materiaalit.github.io/qt-mooc/img/part-3/anchors_3-889a8843.png)

还有一些方便的锚。如`anchors.fill`，它就像将左、右、顶和底锚定到目标项目的左、右、顶和底一样。`anchors.centerIn`是另一个方便的锚，它就像将`verticalCenter`和`horizontalCenter`锚定到目标项目的`verticalCenter`和`horizontalCenter`一样。

出于性能的原因，您只能将项目锚定到它的**兄弟项或直接父项**。例如，下面的锚是无效的，并会产生一个警告：
    
    Item {
        id: group1
        Rectangle { id: rect1; ... }
    }
    Item {
        id: group2
        Rectangle { id: rect2; anchors.left: rect1.right; ... } 
    }
    

另外，基于锚的布局**不能**与绝对定位混在一起使用。如果项目指定了其`x`坐标并同时设置了`anchors.left`或锚定了其左边缘和右边缘，但又同时设置了宽度，则**结果是不确定的**，因为我们不清楚该项目应该使用锚定还是绝对定位。下面的情况也是一样的，比如使用了`anchors.top`和`anchors.bottom`，然后又设置了`y`和高度。或者使用了`anchors.fill`，然后又设置了宽度和高度。这个规则同样适用于`Row`和`Grid`之类的定位器，因为这些定位器会设置项目的`x`和`y`属性。如果希望将基于锚的定位改为绝对定位，可以通过将锚值设置为undefined来取消锚定位。

# 2. 边距

锚定系统还允许为项目的锚布局指定边距和偏移量。边距代表要保留到项目锚点外部的空间，而偏移量允许使用中心锚线控制定位。我们可以通过`leftMargin`，`rightMargin`，`topMargin`和`bottomMargin`分别指定其锚定的边距，或使用`anchors.margins`指定项目四周具有相同值的边距。而偏移量可以通过`horizontalCenterOffset`，`verticalCenterOffset`或`baselineOffset`来指定。

![](https://materiaalit.github.io/qt-mooc/img/part-3/QML_margins-ec5df116.png)

下面的示例指定一个左边距：

    Rectangle { id: rect1; ... }
    Rectangle { id: rect2; anchors.left: rect1.right; anchors.leftMargin: 5; ... }

这样在`rect2`的左侧就保留了5个像素的边距，产生如下结果：

![](https://materiaalit.github.io/qt-mooc/img/part-3/margins_1-4d9c5352.png)

请注意：锚边距仅适用于锚布局，它不是给一个`Item`设置边距的通用方法。如果您为`Item`的边指定了边距，但并没有把这条边锚定到的任何`Item`，那么设置边距是无效的。

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
