# <center>定位器<center>

定位器项是在声明性用户界面中管理项位置的容器项。使用定位器可以轻松的实现在一个规则的布局中排版多个项目。

Qt Quick Layouts也可以用于在用户界面中排列Qt Quick项。它们用于管理声明性用户界面上项的位置和大小，非常适合于可调整大小的用户界面。

# 1. Row, Column, Grid, Flow

## 1.1 Row

*   将它的子元素排成一行。

`Row`用于水平排列项目。下面的示例使用`Row`在外部彩色`Rectangle`定义的区域中排列三个带有圆角的`Rectangle`。`spacing`属性用于设置矩形之间的间距。

我们需要保证父`Rectangle`对象足够大，以便在水平居中的`Row`边缘周围留有一些空间。

    import QtQuick 2.0
    
    Rectangle {
        width: 320; height: 110
        color: "#c0c0c0"
    
        Row {
            anchors.horizontalCenter: parent.horizontalCenter
            anchors.verticalCenter: parent.verticalCenter
    
            spacing: 5
    
            Rectangle { width: 100; height: 100; radius: 20.0
                                 color: "#024c1c" }
            Rectangle { width: 100; height: 100; radius: 20.0
                                 color: "#42a51c" }
            Rectangle { width: 100; height: 100; radius: 20.0
                                 color: "white" }
        }
    }
    

![](https://materiaalit.github.io/qt-mooc/img/part-3/row-2d6f523b.png)

## 1.2 Column

*   将它的子元素排成一列。

`Column`用于垂直排列项目。下面的示例使用`Column`在由外部`Item`定义的区域中排列三个`Rectangle`。`spacing`属性用于设置矩形之间的间距。

    import QtQuick 2.0
    
    Item {
        width: 310; height: 170
        
        Column {
            anchors.horizontalCenter: parent.horizontalCenter
            anchors.verticalCenter: parent.verticalCenter
    
            spacing: 5
    
            Rectangle {
                color: "lightblue"; radius: 10.0
                width: 300; height: 50
                Text {
                    anchors.centerIn: parent
                    font.pointSize: 24; text: "Books"
                }
            }
            Rectangle {
                color: "gold"; radius: 10.0
                width: 300; height: 50
                Text {
                    anchors.centerIn: parent
                    font.pointSize: 24; text: "Music"
                }
            }
            Rectangle {
                color: "lightgreen"; radius: 10.0
                width: 300; height: 50
                Text {
                    anchors.centerIn: parent
                    font.pointSize: 24; text: "Movies"
                }
            }
        }
    }
    

![](https://materiaalit.github.io/qt-mooc/img/part-3/column-13ed3a77.png)

请注意，由于`Column`直接继承自`Item`，所以如果您需要给它设置一个背景色，那么必须将其添加到父项`Rectangle`上。

## 1.3 Grid

*   以网格的形式定位它的子元素。

`Grid`用于将项目以网格或表格的形式布局。下面的示例使用`Grid`将四个`Rectangle`布局在2×2的网格中。与其他定位器一样，我们可以使用`spacing`属性指定项目之间的间距。

    import QtQuick 2.0
    
    Rectangle {
        width: 112; height: 112
        color: "#303030"
    
        Grid {
            anchors.horizontalCenter: parent.horizontalCenter
            anchors.verticalCenter: parent.verticalCenter
            columns: 2
            spacing: 6
      
            Rectangle { color: "#aa6666"; width: 50; height: 50 }
            Rectangle { color: "#aaaa66"; width: 50; height: 50 }
            Rectangle { color: "#9999aa"; width: 50; height: 50 }
            Rectangle { color: "#6666aa"; width: 50; height: 50 }
        }
    }
    

![](https://materiaalit.github.io/qt-mooc/img/part-3/grid-4fbc24df.png)

在这些项目之间插入的水平和垂直间距是相等的，因此必须在项目本身内添加额外的空间。

`Grid`中的任何空单元格都必须在适当位置定义占位符项来创建。

## 1.4 Flow

*   并排放置它的子元素，根据需要进行换行。

`Flow`用于在页面上放置如单词之类的项目，其中行或列不会重叠。

`Flow`以类似于`Grid`的方式排列项目，它先是沿一个轴（短轴）排列成行，然后沿另一个轴（长轴）相邻放置。`Flow`的方向以及项目之间的间距由`flow`和`spacing`属性控制。

下面的示例显示了一个`Flow`包含着许多`Text`子项。它们的排列方式与屏幕截图中显示的类似。

    import QtQuick 2.0
    
    Rectangle {
        color: "lightblue"
        width: 300; height: 200
    
        Flow {
            anchors.fill: parent
            anchors.margins: 4
            spacing: 10
    
            Text { text: "Text"; font.pixelSize: 40 }
            Text { text: "items"; font.pixelSize: 40 }
            Text { text: "flowing"; font.pixelSize: 40 }
            Text { text: "inside"; font.pixelSize: 40 }
            Text { text: "a"; font.pixelSize: 40 }
            Text { text: "Flow"; font.pixelSize: 40 }
            Text { text: "item"; font.pixelSize: 40 }
        }
    }
    

![](https://materiaalit.github.io/qt-mooc/img/part-3/flow-3bbc3b28.png)

`Grid`和`Flow`定位器之间的主要区别是，如果`Flow`中的项在短轴上空间不足时将自动换行，并且如果项的大小不一致，则一行中的项可能不会与另一行中的项对齐。与`Grid`一样，它对项之间和项行之间的间距没有独立的控制。

# 2. childrenRect

此只读属性保存项的子元素的集合位置和大小。

如果需要访问项的子项的集合几何形状以正确调整项的大小，则此属性非常有用。

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
