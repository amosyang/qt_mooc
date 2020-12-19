# <center>文本输入与按键处理<center>

# 1. 焦点

为了让对象接收键盘事件，它需要获取焦点。本节我们关注的是文本输入，但任何项目都可以有焦点，从而响应输入，无论是通过键盘还是其他地方。

最简单的，如果需要让一个项目获取焦点，可以通过设置`focus`属性为`true`来实现。例如:

    Rectangle {
        color: "lightsteelblue"; width: 240; height: 25
        Text { id: myText }
        Item {
            id: keyHandler
            focus: true
            Keys.onPressed: {
                if (event.key == Qt.Key_A)
                    myText.text = 'Key A was pressed'
                else
                    myText.text = 'Key other than A was pressed'
            }
        }
    }
    

但是通过设置`focus`属性并不能保证对象就具有活动焦点，而只是保证对象能够接收焦点。随着应用程序不断开发，肯定会有多个对象需要活动焦点，这时仅仅设置焦点属性已经不够了。如果有多个对象请求焦点，那么最后一个创建的对象将获取焦点。

有几种方法可以解决这个问题。例如，`TextInput`组件的`activeFocusOnPress`属性默认为true，这会让它在单击时接收活动的焦点。（请注意，如果`TextInput`的内容为空，那么我们无法单击它，除非它具有宽度或使用锚布局。）

另一种获得焦点的方法是在对象之间显式地传递焦点。在本例中，`nameField`项定义了`KeyNavigation.tab`属性，这样就可以在按下Tab按键时将焦点传递给`addressField`项。

    TextInput { 
        id: nameField
        focus: true 
        KeyNavigation.tab: addressField 
    }
    
然后这个`addressField`项定义了`KeyNavigation.backtab`属性，这样就可以在按下Shift + Tab时将焦点转移到`nameField`项目。

    TextInput { 
        id: addressField
        KeyNavigation.backtab: nameField 
    }
    

第三种方法是使用特殊类型`FocusScope`。

# 1.1 FocusScope

在构建可重用的QML组件时，`FocusScope`有助于按键的焦点处理。

在`FocusScope`里，我们可以将对象的`Item::focus`属性设置为true。如果有多个项的焦点属性被设置，则最后一个设置焦点的类型将拥有焦点，其他的项将被取消焦点设置，这与没有`FocusScope`的情况类似。当一个`FocusScope`接收到活动焦点时，如果它里面嵌套了一个类型并且该类型被设置为具有焦点，那么这个类型也会获得活动焦点。

让我们来看一个例子。该代码创建两个MyClickableWidget实例：

    Rectangle {
        id: window
    
        color: "white"; width: 240; height: 150
    
        Column {
            anchors.centerIn: parent; spacing: 15
    
            MyClickableWidget {
                focus: true  // Initial focus here
                color: "lightblue"
            }
            MyClickableWidget {
                color: "palegreen"
            }
        }
    }

初始焦点设置给第一个创建的`MyClickableWidget`。如果`MyClickableObject`里**没有**`FocusScope`，则上面赋予初始焦点不会成功，因为在`MyClickableWidget`内部的`Rectangle`已经将自己的焦点设置为true，那么最终焦点将转到创建的最后一个对象，这样会导致第二个`MyClickableWidget`实例获取到了焦点。

    // MyClickableWidget
    FocusScope {
    
        id: scope
    
        property alias color: rectangle.color
        x: rectangle.x; y: rectangle.y
        width: rectangle.width; height: rectangle.height
    
        Rectangle {
            id: rectangle
            anchors.centerIn: parent
            color: "lightsteelblue"; width: 175; height: 25; radius: 10; antialiasing: true
            Text { id: label; anchors.centerIn: parent }
            focus: true
            Keys.onPressed: {
                if (event.key == Qt.Key_A)
                    label.text = 'Key A was pressed'
                else if (event.key == Qt.Key_B)
                    label.text = 'Key B was pressed'
                else if (event.key == Qt.Key_C)
                    label.text = 'Key C was pressed'
            }
        }
        MouseArea { anchors.fill: parent; onClicked: { scope.focus = true } }
    }
    
这个矩形是在`FocusScope`中创建的，并且创建了一个`MouseArea`(下一章将讨论)使最后一个被点击的矩形获取焦点。

请注意，由于`FocusScope`类型不是可视类型，因此它的子属性需要暴露给`FocusScope`的父项。布局和定位类型将使用这些视觉和样式属性来创建布局。在我们的示例中，该`Column`类型无法正常显示两个小部件，因为`FocusScope`自身缺少可视属性。`MyClickableWidget`组件直接绑定到矩形属性，以允许该`Column`类型创建`FocusScope`子元素的布局。

# 2. 文本输入

在QML中，有两种类型显示文本输入`TextEdit`和`TextInput`。`TextEdit`显示多行输入，`TextInput`则显示单行文本输入。

此外，Qt Quick Controls提供了类似功能的`TextField`和`TextArea`，因此我们无需从头开始创建每个组件。

# 2.1 TextInput

`TextInput`类型显示单行可编辑的纯文本。

`TextInput`用于接收单行文本的输入。可以通过设置`TextInput`的属性(例如，validator或inputMask)来约束输入，还可以通过设置`echoMode`属性使`TextInput`用于密码输入。

例如：

    TextInput {
        id: hexNumber
        validator: RegExpValidator { regExp: /[0-9A-F]+/ }
    }
    

正则表达式校验器设置为仅允许输入十六进制（0-9，AF）。

    TextInput {
        id: ipAddress
        inputMask: "000.000.000.000"
    }

IPv4地址的输入可以通过设置inputMask实现。

# 2.2 TextEdit

`TextEdit`用来显示一块可编辑的格式化文本。

它可以显示纯文本和富文本。例如：

    TextEdit {
        width: 240
        text: "<b>Hello</b> <i>World!</i>"
        font.family: "Helvetica"
        font.pointSize: 20
        color: "blue"
        focus: true
    }

请注意，`TextEdit`没有实现滚动、跟随光标或特定于外观的其他行为。例如，添加跟随光标的可滑动滚动条：

    Flickable {
        id: flick
    
        width: 300; height: 200;
        contentWidth: edit.paintedWidth
        contentHeight: edit.paintedHeight
        clip: true
        
        function ensureVisible(r) {
            if (contentX >= r.x)
                contentX = r.x;
            else if (contentX+width <= r.x+r.width)
                contentX = r.x+r.width-width;
            if (contentY >= r.y)
                contentY = r.y;
            else if (contentY+height <= r.y+r.height)
                contentY = r.y+r.height-height;
        }
    
        TextEdit {
            id: edit
            width: flick.width
            focus: true
            wrapMode: TextEdit.Wrap
            onCursorRectangleChanged: flick.ensureVisible(cursorRectangle)
        }
    }
    
可以使用如`SmoothedAnimation`来实现具有平滑滚动效果的特定外观，可以有一个可见的滚动条，或者一个渐变显示位置的滚动条，等等。

剪贴板的支持是由`cut()`，`copy()`和`paste()`方法提供的。通过设置`selectByMouse`属性可以实现用传统的鼠标机制来处理选择功能，或者通过调用`selectionStart`和`selectionEnd`或使用`selectAll()`或`selectWord()`完全在QML里进行处理。

您可以使用`positionAt()`和`positionToRectangle()`在光标位置(文档的起始字符)和像素点之间进行转换。

# 3. 按键

所有可视组件都是通过`Keys`附加属性来支持按键处理。可以通过`onPressed`和`onReleased`信号属性来处理按键。

信号属性有一个`KeyEvent`类型的`event`参数，它包含了事件的详细信息。比如您可以通过`event.key`来获取访问的是哪个按键，通过`event.modifiers`来获取组合键。

可以将`event.accepted`设置为`true`来防止将事件传播到层次结构中其他项。

可以将附加属性`Keys`设置在它附加到的项之前或之后来处理按键事件。这样就可以截获事件以便重写项的默认行为，或者不由该项处理而做为按键的储备。

在下面的示例中，我们使用常规的`onPressed`处理程序来测试是否通过Ctrl修饰符按下了“Y”键：

    Item {
        anchors.fill: parent
        focus: true
        Keys.onPressed: {
            if ((event.key == Qt.Key_Y) && (event.modifiers &Qt.ControlModifier)) {
                console.log("Y pressed with Ctrl");
                event.accepted = true;
            }
        }
    }

有些按键可以通过特定的信号属性来处理，例如`onLeftPressed`。这些处理程序会自动设置`event.accepted`为`true`。

    Item {
        anchors.fill: parent
        focus: true
        Keys.onLeftPressed: console.log("move left")
    }
    

# 4. 按键处理优先级

默认优先级为`Keys.BeforeItem`，其中按键事件处理的顺序为：

*   设置为`forwardTo`的项 
*   特定的按键处理程序，如`onReturnPressed`
*   `onPressed`，`onReleased`处理程序
*   特殊项的按键处理，如`TextInput`按键处理
*   父项

如果优先级是`Keys.AfterItem`，按键事件处理的顺序为：

*   特殊项的按键处理，如`TextInput`按键处理
*   设置为`forwardTo`的项 
*   特定的按键处理程序，如`onReturnPressed`
*   `onPressed`，`onReleased`处理程序
*   父项

如果在上述任何一个步骤中接收了事件，则按键将停止传播。

# 5. 按键处理概述

当用户按下或释放按键时，将发生以下情况：

*   Qt接收按键操作并生成一个按键事件。
*   如果`QQuickWindow`是活动窗口，则将按键事件传递给它。
*   按键事件由场景传递给具有活动焦点的`Item`。如果没有`Item`具有活动焦点，则忽略该按键事件。
*   如果`QQuickItem`具有活动焦点则接收按键事件，然后停止传播。否则，事件将发送到该项的父项，直到接收该事件或到达根项为止。

在某些情况下，您甚至可能希望在将这些事件传递到QML引擎之前在Qt中捕获这些事件。但是，在本课程的这一部分中，我们暂时不会介绍C++和QML的集成。

以下示例中，如果`Rectangle`的类型具有活动焦点并且按下了A键，则该事件将不会进一步传播。如果按下B键，事件将传播到根项目，因此将被忽略。

    Rectangle {
        width: 100; height: 100
        focus: true
        Keys.onPressed: {
            if (event.key == Qt.Key_A) {
                console.log('Key A was pressed');
                event.accepted = true;
            }
        }
    }

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
