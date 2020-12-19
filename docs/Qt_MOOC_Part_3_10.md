# <center>自定义组件<center>

# 1. 关于属性的更多信息

在介绍本节的实际内容“自定义组件”之前，我们先对在第三章第二节中介绍的QML属性系统进行一些扩展。

## 1.1 默认关键字

在介绍属性时，您可能看到了这样的语法：

    [default] property <propertyType> <propertyName>
    
它用于在QML中定义属性。然而在第三章第二节中，我们并没有解释过可选关键字`default`的用途。现在我们来讨论一下这个话题。

声明对象时可以有一个唯一的`default`属性。如果一个对象是在另一个对象的定义中声明的，而没有将其声明为特定属性的值，则默认属性是为该对象赋值的属性。

假设有一个`MyLabel.qml`文件具有默认的属性`someText`：
    
    import QtQuick 2.0
    
    Text {
        default property var someText
    
        text: "Hello, " + someText.text
    }
    
该`someText`值可以在`MyLabel`对象定义中分配，像这样:

    MyLabel {
        someText: Text { text: "world!" }
    }

它与下面的效果完全相同：

    MyLabel {
        Text { text: "world!" }
    }

然而，由于someText属性已经被标记为默认属性，所以没有必要显式地将`Text`对象分配给这个属性。

您可能已经注意到，我们可以将子对象添加到任何`Item`上，而无需显式地将它们添加到children属性。这是因为`Item`的data属性是它的`default`属性，并且为某个`Item`添加到这个列表中的任何项都会自动添加到它的子列表中。

## 1.2 只读关键字

对象声明可以使用`readonly`关键字通过以下语法定义为只读属性：

    readonly property <propertyType> <propertyName> : <initialValue>

只读属性必须在初始化时赋值。在初始化只读属性之后，就不可能再给它赋值了，无论是通过命令式代码还是其他方式。

请注意，只读属性不能是默认属性。

## 1.3 自定义信号

我们在第三章第二节中讨论了QML中的信号处理器。那如果想定义自己的信号该怎么办呢？您可以在C++中去实现，也可以在与我们当前的主题相关的QML中实现。

在QML中声明信号的语法如下： `signal <signalName>[([<type> <parameter name>[, ...]])]`

尝试在同一类型中声明具有相同名称的两个信号或方法是错误的。但是新信号可以重用该类型上现有信号的名称。(此操作应谨慎进行，因为现有信号可能被隐藏而无法访问。)

这是信号声明的三个示例：

    import QtQuick 2.0
    
    Item {
        signal clicked
        signal hovered()
        signal actionPerformed(string action, var actionResult)
    }
    

如果信号没有参数，则括号“（）”可以省略。如果使用了参数，则必须声明参数类型，如上述`actionPerformed`信号的string和var参数一样。通常认为参数类型与属性中参数类型相同。

要发出信号，可以将其作为方法调用。在发出信号后，将调用所有相关的信号处理器，处理器可以使用定义的信号参数名来访问相应的参数。

## 1.4 连接类型

正如我们在第三章第二节中看到的，在QML中连接信号时，通常的方法是创建一个`on<Signal>`处理器，它会在接收到信号时做出响应。

然而在某些情况下，不可能以这种方式连接到信号，例如：

*   需要多次连接到同一信号
*   在信号发送方的范围之外创建连接
*   连接到QML中未定义的目标

当需要实现其中任何一种情况时，可以使用`Connections`类型。

例如，`Connections`对象可以是某个对象的子对象，而不是信号的发送方：

    MouseArea {
        id: area
    }
    
    Connections {
        target: area
        onClicked: foo(parameters)
    }

## 1.5 属性别名

属性别名是保存对另一个属性的引用的属性。与为属性分配新的唯一存储空间的普通属性定义不同，属性别名将新声明的属性（属性别名）连接起来，作为对现有属性的直接引用。

属性别名声明看起来就像普通的属性定义，只是它需要`alias`关键字而不是属性类型，并且属性声明的右侧必须是有效的别名引用：

    [default] property alias <name>: <alias reference>

与普通属性不同，别名具有以下限制：

*   它只能引用声明别名的类型范围内的对象或对象的属性。
*   不能包含任何JavaScript表达式。
*   不能引用在其类型范围之外声明的对象。
*   别名引用不是可选的，不像普通属性的可选默认值，它必须在第一次声明别名时提供。
*   不能引用分组属性，以下代码将不起作用：

    property alias color: rectangle.border.color

    Rectangle {
        id: rectangle
    }  

但是，值类型属性的别名确实有效：

    property alias rectX: object.rectProperty.x
    
    Item {
        id: object
        property rect rectProperty
    }
    

# 2. QML组件

我们之前已经在材料和练习中使用过组件，但没有介绍过组件通常是如何使用以及为什么使用的。QML里定义的组件是一个可实例化的QML类型。它通常包含在它自己的`.qml`文件中。例如，`Button`组件定义在`Button.qml`文件中。我们可以通过实例化这个`Button`组件来创建`Button`对象。组件也可以在`Component`项目内部定义。

定义`Button`组件时可以同时包含其他的组件，可以使用一个`Text`元素、一个`MouseArea`和其他元素来实现内部功能。将不同的组件组合成新的组件(以及有效的新接口)是良好的、可重用的UI组件的关键。

让我们看一个用于选择文本元素颜色的颜色选择器的示例。我们的选择器目前由四个不同颜色的单元格组成。一个非常幼稚的方法是在主应用程序中用`Rectangle`和`MouseArea`项编写多个`Item`。

    Rectangle {
        id: page
        width: 500
        height: 200
        color: "black"
    
        Text {
            id: helloText
            text: "Hello world!"
            y: 30
            anchors.horizontalCenter: page.horizontalCenter
        }
    
        Grid {
            id: colorPicker
            x: 4; anchors.bottom: page.bottom; anchors.bottomMargin: 4
            rows: 2; columns: 3; spacing: 3
    
            Item {
                width: 40
                height: 25
    
                Rectangle {
                    id: rectangle
                    color: "red"
                    anchors.fill: parent
                }
    
                MouseArea {
                    anchors.fill: parent
                    onClicked: helloText.color = "red"
                }
            }          
            // 想象一下，定义了更多具有不同颜色值的重复项目……
        }
    }

您可能注意到这里会有很多重复的代码！此外，要添加更多的颜色，我们需要定义更多只改变颜色值的元素。为了避免为每种颜色一次又一次地编写相同的代码，我们可以从重复的代码中提取一个`Cell`**组件**到`Cell.qml`里。

下面是我们提取的`Rectangle`和`MouseArea`的组件定义:

    Item {
        id: container
        width: 40; height: 25
    
        Rectangle {
            id: rectangle
            color: "red"
            anchors.fill: parent
        }
    
        MouseArea {
            anchors.fill: parent
            onClicked: console.log("clicked?")
        }
    }
    
通常，组件是用`Item`类型作为根项定义的。现在我们有了Cell组件，但目前它是一个没有外部影响的组件，具有一个硬编码的颜色值。它不是可重复使用的，我们如何使其可重用并与其他项交互呢？

我们无法访问`Text`项目的`text`属性，也无法访问`Cell`组件里的`rectangle.color`属性。

为此，我们需要公开一个**接口**供其他组件使用。可以使用属性关键字`alias`和`signal`来暴露此功能。

    Item {
        id: container
         
        property alias cellColor: rectangle.color
        signal clicked(color cellColor)
         
        width: 40; height: 25
    
        Rectangle {
            id: rectangle
            anchors.fill: parent
        }
    
        MouseArea {
            anchors.fill: parent
            onClicked: container.clicked(container.cellColor)
        }
    }    

现在，我们可以使用`cellColor`属性从组件外部设置`rectangle.color`，并且可以使用`clicked`属性为信号安装**信号处理器**`onClicked`：

    Cell {
        cellColor: "red"
        onClicked: helloText.color = cellColor
    }
    

这样我们可以很容易地添加更多的颜色，仅在单元格内定义:

    Rectangle {
        id: page
        width: 500
        height: 200
        color: "black"
    
        Text {
            id: helloText
            text: "Hello world!"
            y: 30
            anchors.horizontalCenter: page.horizontalCenter
        }
    
        Grid {
            id: colorPicker
            x: 4; anchors.bottom: page.bottom; anchors.bottomMargin: 4
            rows: 2; columns: 2; spacing: 3
    
            Cell { cellColor: "red"; onClicked: helloText.color = cellColor }
            Cell { cellColor: "green"; onClicked: helloText.color = cellColor }
            Cell { cellColor: "blue"; onClicked: helloText.color = cellColor }
            Cell { cellColor: "yellow"; onClicked: helloText.color = cellColor }
        }
    }
    

# 3. 可重用组件

到目前为止，我们一直在制作独立的组件，但是现在我们要看看如何将它们集成到具有导航、布局和样式的面向用户的应用程序中。

组件通常添加到根[窗口](https://doc.qt.io/qt-5/qml-qtquick-window-window.html "窗口")中，根窗口充当用户的顶层界面。但是，正如我们在上一章中看到的，还有一个扩展的[ApplicationWindow](https://doc.qt.io/qt-5/qml-qtquick-controls2-applicationwindow.html "ApplicationWindow")，它包含了向窗口添加常用UI项的方便方法。还提供了一个接口来从QML中控制窗口的属性、外观和布局。

正如前面在创建通用`Component`时所讨论的，最好不要引用应用程序根项目的id，因为这会对其他项目产生依赖关系。`ApplicationWindow`提供了这些通用属性来创建自己的接口，而不需要创建对某个窗口id的依赖。

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
