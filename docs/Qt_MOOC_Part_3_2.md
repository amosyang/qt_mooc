# <center>QML类型和属性<center>

在本章的开始部分，我们先对QML类型和代码的结构进行简单的介绍。之后，我们将讨论QML对象属性和QObject属性。

# 1. QML类型和结构

QML类型是标记语言中的结构，它们表示可视化和非可视化部分。非可视QML类型包括状态（states）、转换（transitions）、模型（models）、路径（paths）、渐变（gradients）和计时器（timers）等。

Qt Quick中的所有可视项都继承自`Item`，但它本身并不是可见的。使用`Item`作为顶层QML对象（作为组件的根项目）不会产生视觉效果。如果希望顶级QML对象产生视觉效果，则可以使用其他类型，例如`Rectangle`或`Image`。

请注意，尽管`Item`可以很方便作为您的组件的根对象，但在应用程序的`main.qml`中，您通常希望使用`Window`或它的一个子类型作为根。当使用Qt Creator向导创建新的QML项目时，会自动为`main.qml`创建一个`Window`对象。

尽管`Item`对象本身不可见，但它定义了可见项目之间共有的所有属性，例如x和y坐标，宽度和高度，锚定和键盘焦点处理。它支持分层，通常用于对可视类型进行分组。使用`Item`可以创建不透明度效果，例如创建一个不可见的容器来容纳其他组件。

`Item`可以用于将多个items分组到单个根可视项目下。例如:

    import QtQuick 2.0
    
    Item {
        id: root
        Text {
            text: "Hello There!"
            color: black
        }
        Image {
            id: imageOne
            x: 80
            width: 100
            height: 100
            source: "tile.png"
        }
        Image {
            id: imageTwo
            x: 190
            width: 100
            height: 100
            fillMode: Image.Tile
            source: "tile.png"
        }
    }
    

请注意，为了简单起见，我们在示例中使用了硬编码。通常，您应该尝试将x、y、宽度和高度绑定到父项或其他项，比如组件的根项。稍后将解释如何做到这一点。

## 1.1 QML对象树

从语法上讲，QML代码块定义了要创建的QML对象树。对象是使用对象声明定义的，对象声明描述了要创建的对象类型以及要赋予对象的属性。任何对象声明都可以通过嵌套对象声明来定义子对象。通过这种方式，任何对象声明都会隐式地声明一个对象树，该对象树可以包含任意数量的子对象。

例如，下面的`Rectangle`对象声明包含一个`Gradient`对象，而`Gradient`对象又声明包含了两个`GradientStop`对象：

    import QtQuick 2.0
    
    Rectangle {
        width: 100
        height: 100
    
        gradient: Gradient {
            GradientStop { position: 0.0; color: "yellow" }
            GradientStop { position: 1.0; color: "green" }
        }
    }
    
但是请注意，在QML对象树的上下文中，这是父子关系，而不是在可视场景的上下文中。可视场景中的父子关系的概念是由QtQuick模块中的`Item`类型提供的，它是大多数QML类型的基本类型，因为大多数QML对象都希望以可视化方式呈现。下面我们将讨论可视化的父子关系。

## 1.2 父子组件的可视化

在第二章我们讨论了`QObject`父子关系。现在来看看父子组件的可视化，因为理解这个概念很重要。一个Item的可视父元素不一定与它的对象父元素相同。Qt Quick中的可视父元素的概念与`QObject`父层次结构中对象的父对象的概念是不同的，但又是相关的。

所有QML对象都有一个对象父对象，该对象父对象由声明该对象的对象层次结构确定。使用`QtQuick`模块时，`Item`类型是此模块提供的所有可视项的基本类型，并且它提供了附加的可视父Item的概念，这由Item的parent属性定义。每个Item都有一个视觉上的父Item，如果Item的父属性值为`null`，则该Item将不会在场景中渲染。

为了方便内存管理，被赋值到`Item`的`data`属性的任何对象都将成为该Item在其`QObject`层次结构中的子对象。另外，如果添加到`data`属性的对象是`Item`类型，那么它也会被赋给`Item::children`属性，并在可视场景层次结构中成为该项目的子对象。(大多数Qt Quick层次采集算法，特别是渲染算法，只考虑可视化的父层次结构。)

为了方便起见，`Item`的`data`属性是其默认属性。这意味着，在`Item`对象内声明的任何未分配给特定属性的子项都会自动分配给`data`属性，并成为该`Item`的子`Item`，如上所述。因此，下面的两个代码块将产生相同的结果，并且您几乎总是会看到以下所示的形式：

    import QtQuick 2.0
    
    Item {
        width: 100; height: 100
        
        Rectangle { 
            width: 50;
            height: 50;
            color: "red"
        }
    }
    

而不是显式的为`data`赋值：

    import QtQuick 2.0
    
    Item {
        width: 100; height: 100
    
        data: [
            Rectangle {
                width: 50;
                height: 50;
                color: "red"
            }
        ]
    }
    

一个Item的可视父属性可以在任何时候通过设置它的父属性来更改。因此，一个Item的可视父Item不一定与它的对象父项相同。

当某个Item成为另一Item的子元素时：

*   子元素的父元素引用其父元素。
*   父元素的child和childrenRect属性将代表该子元素。

声明一个Item为另一个Item的子项并不自动意味着该子项将被适当地定位或调整大小以适应其父Item。一些QML类型可能具有影响子Item位置的内置行为，例如，`Row`对象自动将其子Item重新定位为水平形式——但这些行为是由类型自己的特定实现强制执行的。此外，父Item不会自动剪辑它的子Item，以在父Item的可视范围内包含它们，除非它的`clip`属性被设置为true。

# 2. QML对象属性和QObject属性

每个QML对象类型都有一组已定义的属性。可以用该对象类型定义的属性集来创建对象类型的实例。

属性是对象的一个属性，它可以被赋值一个静态值，或者绑定到一个动态JavaScript表达式，甚至是一个代码块。属性的值可以由其他对象根据属性可见性范围规则读取。通常，另一个对象也可以修改它，除非特定的QML类型明确地禁止对特定属性进行这样的修改。

在C++中，可以通过注册类的`Q_PROPERTY`来为类型定义属性，然后将该类注册到QML类型系统。我们会在最后一章中讨论这些`QObject`属性。或者，对象类型的自定义或动态属性可以用以下语法在QML文件或组件的对象声明中定义:

    [default] property <propertyType> <propertyName>
    
通过这种方式，对象声明可以向外部对象公开特定值，或者更容易地维护某些内部状态。

## 2.1 分组属性

在某些情况下，属性包含一组逻辑的子属性。可以使用点符号或组符号将这些子属性分配给它们。

例如，`Text`类型具有字体组属性。下面，第一个`Text`对象使用点表示法初始化其字体值，而第二个对象使用组表示法：

    Text {
        
        font.pixelSize: 12
        font.bold: true
    }
    
    Text {
        
        font { pixelSize: 12; bold: true }
    }
    

## 2.2 id属性

`id`属性是用于标识QML对象的特殊属性。它用于表示该对象。对象的`id`属性允许其他对象在以下方面引用它:

*   相对调整和定位
*   使用其属性
*   更改其属性（例如，用于动画）
*   重复使用常见类型（例如渐变，图像）。

例如：

    Item { 
        width: 300; height: 115 
        Text { 
            id: title 
            x: 50; y: 25 
            text: "Qt Quick" 
            font { family: "Helvetica"; pointSize: parent.width * 0.1 } 
        } 
        
        Rectangle { 
            x: title.x; y: title.y + title.height - height; height: 5 
            width: title.width 
            color: "green" 
        }
    } 
    

## 2.3 作用域

每个QML组件都定义了一个作用域。每个文件至少具有一个根组件，但可以具有其他内联子组件。组件作用域是组件内的对象ID与组件的根对象的属性的并集。

    Item {
        property string title
    
        Text {
            id: titletype
            text: "<b>" + title + "</b>"
            font.pixelSize: 22
            anchors.top: parent.top
        }
    
        Text {
            text: titletype.text
            font.pixelSize: 18
            anchors.bottom: parent.bottom
        }
    }
    

上面的示例显示了一个简单的QML组件，该组件在顶部显示一个富文本标题字符串，在底部显示一个较小的相同文本副本。第一种`Text`类型`title`在形成要显示的文本时直接访问组件的属性。这使得在整个组件中分发数据变得容易。第二种`Text`类型使用ID直接访问第一种文本。

组件实例将其组件作用域连接在一起以形成作用域层次结构。组件实例可以直接访问其祖先的组件作用域。这种动态作用域界定使我们可以执行以下操作：

    // TitlePage.qml
    import QtQuick 2.0
    Item {
        property string title
    
        TitleText {
            size: 22
            anchors.top: parent.top
        }
    
        TitleText {
            size: 18
            anchors.bottom: parent.bottom
        }
    }
    
    // TitleText.qml
    import QtQuick 2.0
    Text {
        property int size
        text: "<b>" + title + "</b>"
        font.pixelSize: size
    }
    
当从`TitlePage`中使用时，即使它们存在于不同的文件中，`TitleText`也可以访问`TitlePage`的`title`属性。如果在其他地方使用，`title`属性可能会以不同的方式解析。

在考虑附加属性的作用域时，需要特别注意。我们后面会讲到附加属性。

## 2.4 信号处理器

许多QML类型都提供了可以捕获的信号，例如`MouseArea`有`onPressed`和`onReleased`等信号。

QML类型还提供了内置的属性更改信号，只要属性值更改，该信号就会发出。这些信号采用以下形式`on<Property>Changed`，其中<Property>是首字母大写的属性名称。这些类型的文档通常没有列出这些信号，因为在类型具有属性的情况下它们是隐式可用的。

在第三章的稍后部分，我们将讨论在QML中实现自己的自定义信号。

信号处理器是一种特殊的方法属性，每当发出相关信号时，QML引擎就会调用该方法的实现。例如：

    TextInput {
        text: "Change this!"
        onTextChanged: console.log("Text has changed to:" text)
    }
    

## 2.5 附加属性和附加信号处理器

附加属性和附加信号处理器是使对象能够使用其他属性或信号处理器进行注释的机制，而附加属性或信号处理器对于该对象是不可用的。特别是，它们允许对象访问与单个对象特别相关的属性或信号。

对附加属性和处理器的引用采用以下语法形式：

    <AttachingType>.<propertyName>
    <AttachingType>.on<SignalName>
    

例如，`ListView`类型具有一个附加属性`ListView.isCurrentItem`，该属性可用于`ListView`中的每个委托对象。每个单独的委托对象都可以使用它来确定它是否是视图中当前选中的Item：

    import QtQuick 2.0
    
    ListView {
        width: 240; height: 320
        model: 3
        delegate: Rectangle {
            width: 100; height: 30
            color: ListView.isCurrentItem ? "red" : "yellow"
        }
    }
    

在这种情况下，附加类型的名称为`ListView`，相关属性为`isCurrentItem`，因此`ListView.isCurrentItem`就是附加属性。

附加的信号处理器以相同的方式引用。例如`Component.onCompleted`，当组件创建过程完成时，附加的信号处理器通常用于执行一些JavaScript代码。在以下示例中，一旦`ListModel`创建完成，其`Component.onCompleted`信号处理器将自动被调用以填充模型：

    import QtQuick 2.0
    
    ListView {
        width: 240; height: 320
        model: ListModel {
            id: listModel
            Component.onCompleted: {
                for (var i = 0; i < 10; i++)
                    listModel.append({"Name": "Item " + i})
            }
        }
        delegate: Text { text: index }
    }

由于附加类型的名称为`Component`并且该类型具有完整的信号，因此附加信号处理器称为`Component.onCompleted`。

## 2.6 关于作用域的注意事项

一个**常见的错误**是假定附加的属性和信号处理器可以从附加这些属性的对象的**子对象直接访问**。事实并非如此。附加类型的实例只附加到特定对象，而不是附加到对象及其所有子对象。

例如，下面是前面示例的修改版本，涉及附加属性。这次`Item`是它的委托，而`Rectangle`是该`Item`的子组件：

    import QtQuick 2.0
    
    ListView {
        width: 240; height: 320
        model: 3
        delegate: Item {
            width: 100; height: 30
    
            Rectangle {
                width: 100; height: 30
                color: ListView.isCurrentItem ? "red" : "yellow" 
            }
        }
    }
    

以上代码不能正常的运行，原因是`ListView.isCurrentItem`仅附加到根委托对象，而不附加到其子对象。由于`Rectangle`是委托的子组件，而不是委托本身，因此不能用`isCurrentItem`方式访问附加的属性`ListView.isCurrentItem`。`Rectangle`里的`isCurrentItem`应该通过根委托进行访问：

    ListView {
        
        delegate: Item {
            id: delegateItem
            width: 100; height: 30
    
            Rectangle {
                width: 100; height: 30
                color: delegateItem.ListView.isCurrentItem ? "red" : "yellow"   
            }
        }
    }
    

现在，`delegateItem.ListView.isCurrentItem`正确引用了委托的附加属性`isCurrentItem`。

## 2.7 属性绑定

正如我们刚才提到的，对象的属性可以被分配一个静态值，该值在显式地分配一个新值之前保持不变。然而，为了充分利用QML及其对动态对象行为的内置支持，大多数QML对象使用属性绑定。属性绑定是QML的核心特性，它允许开发人员指定不同对象属性之间的关系。当属性的依赖项值发生更改时，该属性将根据指定的关系自动更新。

在后台，QML引擎监视属性的依赖关系（即绑定表达式中的变量）。检测到更改时，QML引擎将重新计算绑定表达式并将新结果应用于属性。

如果要创建属性绑定，需要为属性分配一个计算结果为所需值的JavaScript表达式。最简单地说，绑定可以是对另一个属性的引用。以下面的例子为例，蓝色矩形的高度被绑定到其父矩形的高度:

    Rectangle {
        width: 200; height: 200
    
        Rectangle {
            width: 100
            height: parent.height
            color: "blue"
        }
    }
    

只要父矩形的高度发生变化，子矩形的高度就会自动更新为相同的值。

绑定可以包含任何有效的JavaScript表达式或语句。绑定可以访问对象属性，调用方法，并使用内置的JavaScript对象（例如`Date`或）`Math`。稍微复杂一点的示例将一个`color`对象绑定到另一个对象中文本的长度。

    color: myTextInput.text.length <= 10 ? "red" : "blue"
    

当另一个对象（具有id `myTextInput`）的文本超过10个字符时，该对象的颜色从蓝色变为红色。

值得注意的是，虽然绑定属性的值会自动更新，但如果JavaScript语句稍后为其分配静态值，则绑定将被删除。这是一个常见的问题，特别是对初学者！例如:

    import QtQuick 2.0
    
    Rectangle {
        width: 100
        height: width * 2
    
        focus: true
        Keys.onSpacePressed: {
            height = width * 3
        }
    }
    

在这里，矩形最初确保其高度总是其宽度的两倍。但是，当按下空格键时，width*3的当前值将被指定为height的静态值。在此之后，即使宽度改变，高度也将保持在此值不变。静态值的赋值删除了绑定。

在许多情况下，这是可取的行为。如果目的是给矩形一个固定的高度并停止自动更新，那么代码就可以做到这一点。但是，如果想要在宽度和高度之间建立一种新的关系，那么新的绑定表达式必须包装在`Qt.binding()`函数中：

    import QtQuick 2.0
    
    Rectangle {
        width: 100
        height: width * 2
    
        focus: true
        Keys.onSpacePressed: {
            height = Qt.binding(function() { return width * 3 })
        }
    }
    

现在，按下空格键后，矩形的高度将继续自动更新，始终是其宽度的三倍。

这一章是理论性很强的，因为我们现在还不能做太多，但这对以后理解属性系统很有帮助。

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
