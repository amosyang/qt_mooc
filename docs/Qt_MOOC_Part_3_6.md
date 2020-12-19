# <center>输入处理器<center>

Qt Quick有多种类型可以处理触摸事件，例如我们在上一节中使用的`MouseArea`，另外还有`PinchArea`、`MultiPointTouchArea`和`Flickable`。

上面这些项都存在一些问题。比如`MouseArea`，Qt只是假设有一个鼠标，对于`QMouseEvent`和`QTouchEvent`在Qt Quick中都被认为是的相同事件。这就导致您不能同时与两个`MouseArea`或`Flickable`进行交互。对于我们之前使用的`MouseArea`就意味着您不能同时按下两个按钮或同时拖动两个滑块。也意味着您不能同时使用`PinchArea`和`MouseArea`，因为当`PinchArea`处于活动状态时，它不会将事件传递给`MouseArea`。

为了解决这些问题，Qt引入了新的输入处理器类型。输入处理器可以在任何`Item`类型里声明，并且可以代表父对象处理来自任何指针设备的事件。它们是非常轻量级的类型，可以被声明为每个交互类型的处理器。每个`Item`都可以有无限的处理器类型，所以你不会担心它们不够用。

不管是触摸事件，还是鼠标按钮事件或者组合键在活动状态，您都可以将事件处理器对哪种交互进行响应进行约束。这是通过`acceptedButtons`和`acceptedDevices`属性来实现的，可以选择被接收的`Qt::MouseButtons`或被接收的指针设备（如`PointerDevice.Mouse`, `PointerDevice.Stylus`或者`PointerDevice.TouchScreen`）。

现在，让我们通过示例来了解一些输入处理器。

# 1. TapHandler

`TapHandler`与`MouseArea`有几个关键的区别：

    import Qt.labs.handlers 1.0
    
    Item {
        TapHandler {
            acceptedDevices: PointerDevice.Mouse | PointerDevice.Stylus
            onTapped: console.log("left clicked")
        }
        
        TapHandler {
            acceptedDevices: PointerDevice.TouchScreen
            onTapped: console.log("tapped")
        }
        
        TapHandler {
            acceptedButtons: Qt.RightButton
            onTapped: console.log("right clicked")
        }
    }

现在，我们可以接收来自特定设备的事件并做出相应的响应。也可以让多个`TapHandler`在同一个Item中活动，而不会出现多个`MouseArea`之类的问题。

# 2. DragHandler

`DragHandler`与`MouseArea`的`drag`属性类似，但更易于使用。它具有类似`MouseArea`中的`xAxis`和`yAxis` [属性组](http://doc-snapshots.qt.io/qt5-5.10/qml-qt-labs-handlers-draghandler.html#xAxis-prop "属性组")：

    import Qt.labs.handlers 1.0
    
    Rectangle {
        width: 50
        height: 50
        color: "green"
        
        DragHandler {
            yAxis.enabled: false
        }
    }
    
在本例中，我们可以拖动`Rectangle`，但只能沿着X轴方向，因为我们禁用了Y轴。

# 3. PointHandler

还有一个`PointHandler`可用于跟踪触摸点：

    import Qt.labs.handlers 1.0
    
    Item {
        id: root
    
        PointHandler {
            id: handler
            acceptedDevices: PointerDevice.Mouse | PointerDevice.Stylus
            target: Rectangle {
                parent: root
                color: "blue"
                visible: handler.active
                x: handler.point.position.x - width / 2
                y: handler.point.position.y - height / 2
                width: 20
                height: width
                radius: width / 2
            }
        }
    }

当一个press事件发生时，处理器将选择一个尚未绑定到其他处理器的点。它将检查给定的约束条件（`acceptedDevices`等）是否满足，是否符合该点的条件。然后它将跟踪`active`属性为true的点，直至释放。与其他处理器一样，它具有`target`属性，我们在其中放置了一个矩形并绑定了处理器属性。

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
