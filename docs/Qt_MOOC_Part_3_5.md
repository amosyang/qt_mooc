# <center>鼠标处理<center>

Qt可以处理不同输入设备的输入，如触摸屏，鼠标等。基本上有两种不同的方法来处理输入事件。一种是区域类型，它可以处理来自任意数量设备的输入和输入处理器。在本章中，我们将介绍鼠标处理类型`MouseArea`。下一章将讨论一般的输入处理器。

# 1. MouseArea

我们可以使用`MouseArea`来启用鼠标与元素的交互。它是一个不可见的矩形项，可以像下面这样嵌套到元素中来捕捉鼠标事件:

    Rectangle {
        width: 100; height: 100
        color: "green"
         
        MouseArea {
            anchors.fill: parent
            onClicked: parent.color = 'red'
        }
    }
    
现在鼠标处理的逻辑包含在`MouseArea`项中。这个区别是Qt Quick UI的一个重要方面，因为它将输入处理与可视化表示分离开来。这使得可视项可以是任意大小的，但是输入事件只能在输入元素中定义的约束范围内被接收。

如果有多个`MouseArea`重叠，则默认只有可视化层次结构中的顶层Item接收该事件。您可以设置`propagateComposedEvents`属性为`true`来允许事件沿着可视化堆叠顺序进一步传播。有关更多信息和示例，请参考[文档](http://doc.qt.io/qt-5/qml-qtquick-mousearea.html#propagateComposedEvents-prop "文档")。在下一章中，我们将讨论`Input Handler`，用它处理重叠区域比使用`MouseArea`更简单。

# 2. 使用鼠标左键的通用鼠标事件

默认情况下`MouseArea`对鼠标左键做出响应并发出`onClicked`信号。如果要设置`MouseArea`响应其他按钮，可以为属性`acceptedButtons`设置所需的[Qt::MouseButton](http://doc.qt.io/qt-5/qt.html#MouseButton-enum "Qt::MouseButton")标志。多个标志可以用`|`（或）运算符组合。要访问当前按下的按钮，可以将`&`（与）运算符与`pressedButtons`属性一起使用。

除了方便的`onClicked`处理器，还有`onPressed`，`onWheel`或者`onPositionChanged`等，使得我们能够处理更多特定的鼠标事件。

许多`MouseArea`信号在发出时都会传递一个`MouseEvent`类型的`mouse`参数，该参数包含了关于鼠标事件的信息，如位置、按钮类型和按键修饰符等。

下面这个例子中，我们同时启用了鼠标的左右按钮，并相应地改变了`Rectangle`的颜色:

    Rectangle {
        width: 100; height: 100
        color: "green"
         
        MouseArea {
            anchors.fill: parent
            acceptedButtons: Qt.LeftButton | Qt.RightButton
            onClicked: {
                if (mouse.button == Qt.RightButton) {
                    parent.color = 'blue';
                } else {
                    parent.color = 'red';
                }
            }
        }
    }
    

# 3. 鼠标悬停可视化

默认情况下，`MouseArea`在单击或按住按钮时会处理鼠标事件。其实当鼠标在当前区域内悬停时，它也可以处理事件。

要启用悬停事件处理，必须将`hoverEnabled`属性设置为`true`。这会影响`onEntered`，`onExited`以及`onPositionChanged`处理器。当鼠标进入或退出该区域时，会向`onEntered`和`onExited`处理器发送信号，可以用于高亮显示或激活项目。当鼠标在该区域内移动时，`onPositionChanged`处理器就会收到信号。

下面这个例子演示了当鼠标进入`MouseArea`时，Text位置会跟随着鼠标移动：

    Text {
        id: movingText
        text: "hover"
    }
         
    MouseArea {
        anchors.fill: parent
        hoverEnabled: true
        onPositionChanged: {
            movingText.x = mouse.x
            movingText.y = mouse.y
        } 
    }
    

# 4. 拖曳

在某些用户界面中，使一些元素可以拖动是非常有必要的，比如音量滑块或图像。我们可以通过`MouseArea`的`drag`属性来实现。

`drag`本身也具有用于指定如何完成拖曳的属性。

*   `drag.target`：指定要拖动的项目ID。
*   `drag.active`：指定当前是否拖动目标项目。
*   `drag.axis`：指定是否可以水平（`Drag.XAxis`），垂直（`Drag.YAxis`）或两个方向都可以（`Drag.XAndYAxis`）进行拖动。
*   `drag.minimum`和`drag.maximum`：可以沿指定轴将目标拖动多远。

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
