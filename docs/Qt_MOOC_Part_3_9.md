# <center>Qt Quick Controls<center>

Qt Quick Controls提供了现成的UI控件，它使得您可以更快地进行工作，因为您不必从头开始实现所有的控件。
例如`ApplicationWindow`提供了一个带有页眉、页脚、菜单栏和弹出窗口的`QQuickWindow`。其中的窗口可以包含视图、容器和控件的布局。

请注意，当我们在本课程中谈到Qt Quick Controls时，指的是Qt Quick Controls 2。如果您查看Qt文档，您可能还会发现关于Qt Quick Controls 1的信息。它们在将来会被弃用，您应该避免使用它们。

# 1. 准备开始

要使用Qt Quick Controls，您需要在.qml文件中导入`QtQuick.Controls`。例如：

    import QtQuick 2.6
    import QtQuick.Controls 2.4
    
    ApplicationWindow {
        title: "My Application"
        width: 640
        height: 480
        visible: true
    
        Button {
            text: "Push Me"
            anchors.centerIn: parent
        }
    }

您应该使用`ApplicationWindow`作为应用程序中的根项，并通过使用C++中的`QQmlApplicationEngine`来启动它。这确保您可以从QML控制顶级窗口的属性。例如：
    
    // main.cpp
    #include <QGuiApplication>
    #include <QQmlApplicationEngine>
    
    int main(int argc, char *argv[])
    {
        QGuiApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
        QGuiApplication app(argc, argv);
        QQmlApplicationEngine engine;
        engine.load(QUrl(QStringLiteral("qrc:/main.qml")));
        return app.exec();
    }
    
注意，如果您是从源码编译的Qt，请确保还编译了Qt Graphical Effects模块，因为Qt Quick Controls 2需要该模块。

# 2. ApplicationWindow

`ApplicationWindow`是一个可以方便地在窗口中添加菜单栏、页眉和页脚的`Window`。如前所述，通常您应该使用`ApplicationWindow`作为应用程序中的根项目。

    import QtQuick 2.11
    import QtQuick.Controls 2.4
    
    ApplicationWindow {
        visible: true
        width: 640
        height: 480
        title: qsTr("Application Window")
        minimumHeight: 300
        minimumWidth: 330
    
        menuBar: MenuBar {
            Menu {
                title: qsTr("File")
                MenuItem { text: qsTr("Open") }
                MenuItem { text: qsTr("Close") }
            }
        }
    
        header: Label {
            horizontalAlignment: Text.AlignHCenter
            height: 40
            font.pixelSize: 70
            minimumPixelSize: 8
            fontSizeMode: Text.Fit
            text: qsTr("Header")
            background: Rectangle {
                anchors.fill: parent
                border { width: 2; color: "black" }
                color: "lightgreen"
            }
        }
    
        footer:
            Label {
                horizontalAlignment: Text.AlignHCenter
                text: qsTr("Footer")
                font.pixelSize: 24
            }
    }
    

![](https://materiaalit.github.io/qt-mooc/img/part-3/application_window-2dc70ed4.png)

在这个简短的示例中，我们定义了菜单栏，页眉和页脚。如您所见，使用Qt Quick Controls可以很容易地做到这一点。我们建议您将示例复制到Qt Creator中的新QML项目中并运行它。尝试使用一些参数来看看会发生什么。

# 3. 视图

我们通过Qt Quick Controls可以轻松的实现流畅的UI。它提供了一些不同的视图，比如`ScrollView`，`SwipeView`和`StackView`。这里我们会详细的介绍`StackView`。您可以查看官方文档来使用其他视图并了解它们的行为。

`StackView`可以与一组相互链接的页面一起使用。前进时视图被推到堆栈上，向后时弹出视图。

下面是一个简单的例子，点击相关的按钮，mainView被推到堆栈上或从堆栈中弹出：

    import QtQuick 2.11
    import QtQuick.Controls 2.4
    
    ApplicationWindow {
        title: qsTr("StackView")
        width: 400
        height: 280
        visible: true
    
        StackView {
            id: stack
            initialItem: mainView
            anchors.fill: parent
        }
    
        Component {
            id: mainView
    
            Column {
                spacing: 10
    
                Button {
                    text: "Push"
                    onClicked: stack.push(mainView)
                }
                Button {
                    text: "Pop"
                    enabled: stack.depth > 1
                    onClicked: stack.pop()
    
                }
                Text {
                    font.pixelSize: 30
                    text: stack.depth
                }
            }
        }
    }

在应用程序中使用`StackView`非常简单，只需将它作为子元素添加到窗口中即可。它通常锚定在窗口的边缘，除了在顶部或底部，还可以锚定在状态栏或其他类似的UI组件。然后可以通过调用它的导航方法来使用。`StackView`中显示在最上层的是分配给`initialItem`的项，或者如果没有设置`initialItem`的话，则显示的是最上面的项。

`StackView`包括三个主要的导航操作：`push()`，`pop()`和`replace()`。这些对应于经典的堆栈操作，其中`push()`是将一个项目添加到堆栈的顶部，`pop()`是从堆栈中删除顶部的项目，`replace()`就像一个弹出然后紧跟着一个推的操作，即使用新项目替换掉最上面的项目。堆栈中最上面的项对应于屏幕上当前可见的项。

## 3.1 转场效果

对于每个push或pop操作，我们都可以将不同的转场动画应用于进入或退出项目。这些动画定义了当项目进入或退出时的效果。可以通过为`StackView`的`pushEnter`、`pushExit`、`popEnter`、`popExit`、`replaceEnter`和`replaceExit`属性分配不同的转场来定制动画。

请注意，转场之间的行为会受到转场动画的相互影响。为一个转场自定义一个动画而离开另一个转场时可能会产生意外的结果。

以下示例为推入和弹出操作定义了一个简单的淡入淡出的转场效果：

    StackView {
        id: stackview
        anchors.fill: parent
    
        pushEnter: Transition {
            PropertyAnimation {
                property: "opacity"
                from: 0
                to:1
                duration: 200
            }
        }
        pushExit: Transition {
            PropertyAnimation {
                property: "opacity"
                from: 1
                to:0
                duration: 200
            }
        }
        popEnter: Transition {
            PropertyAnimation {
                property: "opacity"
                from: 0
                to:1
                duration: 200
            }
        }
        popExit: Transition {
            PropertyAnimation {
                property: "opacity"
                from: 1
                to:0
                duration: 200
            }
        }
    }

注意：您不可以在添加到`StackView`的项目上使用锚定布局。因为如果使用了锚定之后，“push”、“pop”和“replace”再设置动画效果是不可能实现的。**不过**这只适用于`StackView`项本身，如果为它的子项使用了锚定布局则仍然可以正常工作。

# 4. 控件

Qt Quick Controls提供了大量的控件，可用于在Qt Quick中构建完整的界面。包括各种按钮控件`Button`和`Switch`，输入控件`TextArea`和`Slider`等。

像`Button`，`Slider`等这些简单的控件使用起来也非常简单，例如您可以定义一个从0到100的滑块：

    Slider {
        id: percentage
        from: 0
        to: 100
        value: 20
    }

![](https://materiaalit.github.io/qt-mooc/img/part-3/slider-ec3120cb.png)

稍微复杂一点的控件，如`Drawer`和`ComboBox`，使用起来也相当简单。我们在这里没有给出`Drawer`的例子，因为大多数人是在PC上做的课程无法滑动，但如果您对此感兴趣可以查看相关文档来学习它。对于`ComboBox`，这里有几个例子：

    ComboBox {
        textRole: "key"
        model: ListModel {
            ListElement { key: "First"; value: 123 }
            ListElement { key: "Second"; value: 456 }
            ListElement { key: "Third"; value: 789 }
        }
    }

![](https://materiaalit.github.io/qt-mooc/img/part-3/combobox-7a7242a0.png)

这里需要注意的是，当使用具有多个角色的model时，必须为`ComboBox`设置特定的`textRole`属性用于显示模型中的文本。

您也可以将`ComboBox`设置为可编辑状态，如下所示：

    ComboBox {
        editable: true
        model: ListModel {
            id: model
            ListElement { text: "Banana" }
            ListElement { text: "Apple" }
            ListElement { text: "Coconut" }
        }
        onAccepted: {
            if (find(editText) === -1)
                model.append({text: editText})
        }
    }
    

## 4.1 容器控件

`Container`类型支持添加，插入，移动和删除项目。您可以使用现有的可视化`Container`子类如`SwipeView`来实现。要实现自定义容器，API中最重要的部分是`contentModel`，它提供了包含的项，可以作为项视图和重复器的委托模型。例如：

    Container {
        id: container
    
        contentItem: ListView {
            model: container.contentModel
            snapMode: ListView.SnapOneItem
            orientation: ListView.Horizontal
        }
    
        Text {
            text: "Page 1"
            width: container.width
            height: container.height
        }
    
        Text {
            text: "Page 2"
            width: container.width
            height: container.height
        }
    }

我们将在第4章讨论用于可视化数据的数据模型。

# 5. 样式

Qt Quick Controls带有多种样式。除了默认样式外，还有Fusion，Material，Imagine和Universal样式。您可以在[文档](https://doc.qt.io/qt-5/qtquickcontrols2-styles.html "文档")中查看它们。

为了运行具有特定样式的应用程序，您可以在C++中使用`QQuickStyle`配置样式，或传递命令行参数，或设置环境变量，或在配置文件中指定首选样式和特定样式的属性。

这些方法的优先级按照上面列出的顺序从高到低。也就是说，`QQuickStyle`用于设置样式将始终优先使用命令行参数。例如：

*   在C++中，您可以使用`QQuickStyle::setStyle("Material");`设置`QQuickStyle`。
*   在命令行传递`-style`参数是测试不同样式的便捷方法：`./app -style material`。
*   设置环境变量`QT_QUICK_CONTROLS_STYLE`可用于设置系统范围的样式首选项。`QT_QUICK_CONTROLS_STYLE=universal ./app`
*   配置文件：Qt Quick Controls 2支持一个特殊的配置文件`:/qtquickcontrols2.conf`，该文件内置于应用程序的资源中。配置文件可以指定首选样式（可以由前面介绍的任何一种方法覆盖）和某些样式特定的属性。如下：

    [Controls]

    Style=Material

该配置文件也可以用来定义某些其他东西，请参阅[文档](https://doc.qt.io/qt-5/qtquickcontrols2-configuration.html "文档")了解更多细节。

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
