# <center>视图<center>

QML提供了四种常用的动态视图类型：`ListView`，`GridView`，`TableView`和`PathView`。前三个都继承于`Flickable`类型，这样用户就可以在水平或垂直方向滑动内容。注意，Qt Quick Controls 1中也有一个`TableView`类型，我们不推荐使用该类型，这里也不做介绍。使用`PathView`可以以任意路径形状组织数据项。

每个视图都可以使用上一节中介绍的QML模型，然后在UI中设置委托项。QML中的所有模型都是列表。`TableView`是唯一可以直接显示C++表格模型中多个列的类型，比如使用`QAbstractItemModel`的子类做为模型。但这并不是一种限制，您仍然可以像其他视图使用委托显示模型列一样，映射到不同的角色。

在本节中，我们将集中讨论使用QML模型的用例。C++部分会在第五章中介绍。

# 1. ListView

[ListView](http://doc.qt.io/qt-5/qml-qtquick-listview.html "ListView")可以在水平或垂直方向的列表中显示委托数据。它提供了多种方法来修饰列表。为了更好的可视化，数据项可以分为多个部分。比如可以有一个特殊的页眉和页脚项，或者一个表示当前项的高亮显示组件。

在下面的示例中，`clip`属性设置为`true`。默认情况下QML父项不会剪裁其子项，因此子项可以在父项的边界框之外绘制，这在大多数情况下没有问题。但是，如果这个UI窗口中同时有一个工具栏和列表视图，我们很可能不希望将列表视图项绘制在工具栏的顶部。这时只要设置`clip`属性为`true`即可。

    ListView {
        anchors.fill: parent
        clip: true
        model: 50
        delegate: someNiceDelegate
    }    

## 1.1 附加属性

每个委托都可以访问该视图提供的许多附加属性。例如，`ListView`可以通过`ListView.view`属性来访问委托所绑定的对象，或者可以使用`ListView.isCurrentItem`来检查它是否是当前项目。

附加属性允许实现一个干净且可重复使用的委托，如委托并没有通过id直接绑定到视图。

附加属性可用于根委托项。子对象必须使用根委托的标识符来引用附加属性，如下例所示。我们有时候把一个新的自定义var属性绑定到附加属性id。但是请记住，每增加一个属性都会增加内存的消耗。

    ListView {
        width: 300; height: 200
        model: contactsModel
        delegate: contactsDelegate
    
        Component {
            id: contactsDelegate
            Rectangle {
                id: rootWrapper
                width: 80
                height: 80
                // Rectangle is the root item, we can refer to the attached property directly
                color: ListView.isCurrentItem ? "black" : "red"
                Text {
                    id: contactInfo
                    text: name + ": " + number
                    // Attached property is not accessible in the child
                    color: rootWrapper.ListView.isCurrentItem ? "red" : "black"
                }
            }
        }
    }

## 1.2 分组

将数据项分组到`section`可以提高列表视图的可读性。例如，按艺术家姓名分组的专辑或按部门分组的员工可以很容易地显示哪些项目是相关的。

使用`section`时，需要考虑两个属性。第一个是`section.property`，它用于定义使用哪个属性进行数据划分。值得注意的是，我们需要先对模型进行排序，这样可以保证`section`属性是连续的。如果`section`属性出现在模型的多个非连续位置中，则同一`section`可能在视图中会出现多次。第二个是`section.criteria`属性，它的默认值为`ViewSection.FullString`，这意味着将整个属性当作`section`。如果设置为`ViewSection.FirstCharacter`，则仅将属性中的第一个字符设置为`section`，比如根据姓氏的第一个字母将名称列表划分为`section`。

当使用了`section`进行分组之后，我们可以使用`ListView`的附加属性`ListView.section`，`ListView.previousSection`和`ListView.nextSection`来访问它们。

我们还可以通过在`ListView`中设置`section.delegate`属性来为每个`section`创建一个标题。

在下面的示例中，数据模型包含了一个艺术家及其专辑的列表，这些列表按艺术家分为几部分：

    ListView {
        anchors.fill: parent
        anchors.margins: 20
        clip: true
        model: albumsAndArtists
        delegate: albumDelegate
        section.property: "artist"
        section.delegate: sectionDelegate
    }
    
    Component {
        id: albumDelegate
        Item {
            width: ListView.view.width
            height: 20
            Text {
                anchors.left: parent.left
                anchors.verticalCenter: parent.verticalCenter
                anchors.leftMargin: 10
                font.pixelSize: 12
                text: album
            }
        }
    }
    
    Component {
        id: sectionDelegate
        Rectangle {
            width: ListView.view.width
            height: 20
            color: "lightblue"
            Text {
                anchors.left: parent.left
                anchors.verticalCenter: parent.verticalCenter
                anchors.leftMargin: 6
                font.pixelSize: 14
                text: section
            }
        }
    }
    
    ListModel {
        id: albumsAndArtists
        ListElement { album: "Crazy World"; artist: "Scorpions"; }
        ListElement { album: "Love at First Sting"; artist: "Scorpions"; }
        ListElement { album: "Agents of Fortune"; artist: "Blue Öyster Cult"; }
        ListElement { album: "Spectres"; artist: "Blue Öyster Cult"; }
        ListElement { album: "The Vale of Shadows"; artist: "Red Raven Down"; }
        ListElement { album: "Definitely Maybe"; artist: "Oasis"; }
    }

## 1.3 页眉和页脚

视图允许通过装饰属性（如`header`和`footer`）进行可视化定制。我们可以通过将一个对象（通常是另一个可视对象）绑定到这些属性来装饰视图。例如，可以使用一个`Rectangle`类型显示页脚，或者在列表的顶部页眉显示一个logo。需要注意的是，`ListView`中的`spacing`属性不对页眉和页脚产生影响，因此任何间距都必须是页眉和页脚本身的一部分。

    ListView {
        anchors.fill: parent
        anchors.margins: 20
        clip: true
        model: 4
        delegate: numberDelegate
        spacing: 2
        header: headerComponent
        footer: footerComponent
    }
    
    Component {
        id: headerComponent
        Rectangle {
            width: ListView.view.width
            height: 20
            color: "lightBlue"
            Text { text: 'Header'; anchors.centerIn: parent; }
        }
    }
    
    Component {
        id: footerComponent
        Rectangle {
            width: ListView.view.width
            height: 20
            color: "lightGreen"
            Text { text: 'Footer'; anchors.centerIn: parent; }
        }
    }
    
    Component {
        id: numberDelegate
        Rectangle {
            width: ListView.view.width
            height: 40
            border.color: "black"
            Text { text: 'Item ' + index; anchors.centerIn: parent; }
        }
    }    

## 1.4 键盘导航与高亮显示

当使用键盘在`ListView`中进行导航时，需要以某种形式的高亮显示来告诉用户当前选择了哪个项目。如果您想实现在`ListView`中使用键盘导航需要做两件事：第一，需要用`focus:true`属性为视图提供一个键盘焦点；第二，需要定义一个的高亮显示的委托。如下：

    ListView {
        id: view
        anchors.fill: parent
        anchors.margins: 20
        clip: true
        model: 20
        delegate: numberDelegate
        spacing: 5
        highlight: highlightComponent
        focus: true
    }
    
    Component {
        id: highlightComponent
        Rectangle {
            color: "lightblue"
            radius: 10
        }
    }
    
    Component {
        id: numberDelegate
        Item {
            width: ListView.view.width
            height: 40
            Text {
                anchors.centerIn: parent
                font.pixelSize: 10
                text: index
            }
        }
    }

高亮显示的委托被自动赋予了当前项的`x`，`y`和`height`属性。如果`width`未指定，则默认使用当前项的宽度。

# 2. GridView和TableView

[GridView](http://doc.qt.io/qt-5/qml-qtquick-gridview.html "GridView")与`ListView`非常相似，它的使用方式也几乎相同。主要的区别是，它不依赖于间距和委托的大小，而是在视图中定义了`cellWidth`与`cellHeight`。在`GridView`中同样可以使用页脚、页眉和高亮显示。您还可以通过`flow`属性来设置方向，如`GridView.LeftToRight`（默认）和`GridView.TopToBottom`。以下示例展示了`GridView`的用法：

    GridView {
        id: grid
        anchors.fill: parent
        cellWidth: 80;
        cellHeight: 80
        model: 100
        delegate: numberDelegate
        highlight: Rectangle { color: "lightsteelblue"; radius: 5 }
        focus: true
    }
    
    Component {
        id: numberDelegate
        Item {
            width: grid.cellWidth
            height: grid.cellHeight
            Text {
                text: index
                anchors.horizontalCenter: parent.horizontalCenter
                anchors.verticalCenter: parent.verticalCenter
            }
        }
    }

[TableView](http://doc-snapshots.qt.io/qt5-5.12/qml-qtquick-tableview.html# "TableView")可以用于显示多列属性，不过前提是底层模型要具有多列。由于QML没有具有多列的模型类型，因此它的模型必须继承于`QAbstractItemModel`。

# 3. PathView

[PathView](http://doc.qt.io/qt-5/qml-qtquick-pathview.html "PathView")是QML提供的动态视图中功能最强大并且可以自定义的类型。`PathView`用于在路径上显示模型数据，该路径可以用[Path](http://doc.qt.io/qt-5/qml-qtquick-path.html "Path")类型进行定义。您可以通过`PathView`各种各样的属性进行定制显示。比如`pathItemCount`属性，它用于控制可见项目的数量，以及`preferredHighlightBegin`和`preferredHighlightEnd`属性可以用于控制当前项目沿着路径显示的位置。这些属性值介于0和1之间，如果将它们都设置为0.5则会在路径的50％位置显示当前项。

`Path`是在滚动视图时所遵循的路径。它使用`startX`和`startY`属性定义路径，以及使用如[PathQuad](http://doc.qt.io/qt-5/qml-qtquick-pathquad.html "PathQuad")或[PathLine](http://doc.qt.io/qt-5/qml-qtquick-pathline.html "PathLine")定义路径段。所有路径段类型都可以在Qt文档中找到。Qt Quick Designer商业版甚至还有一个编辑器，用于创建和删除路径段并可以定义它们的形状。

以下示例显示了一个直线路径上的项目：

    PathView {
        id: view
        model: 20
        anchors.fill: parent
    
        path: Path {
            startX: 0
            startY: height
    
            PathCurve {
                x: view.width
                y: 0
            }
        }
        delegate: Text {
            text: "Index " + index
        }
    }

在定义了路径之后，我们可以使用`PathPercent`和`PathAttribute`类型对其进行调整。这些对象可以放在路径段之间，以提供对路径和委托的更细粒度的控制。`PathPercent`允许您在路径视图的路径上控制项目之间的间隔。您可以使用它将路径的一部分项目组合在一起，然后将它们分布到路径的其他部分上。`PathAttribute`对象允许为路径上的各个点指定由名称和值组成的属性。这些属性作为附加属性公开给委托，并可用于控制任何属性。沿着路径的任何特定点上的属性值都可以从`PathAttributes`属性中插入。

接下来，我们有一个更大的`PathView`示例，其中路径是用`PathQuad`定义的，项目的大小和不透明度是用`PathAttribute`更改的。我们还启用了键盘导航，默认情况下它是不可用的。通过将`focus`属性设置为true可以使键盘具有焦点，并定义了按下按键应该做什么（这里我们使用`Keys.onLeftPressed: decrementCurrentIndex()`和`Keys.onRightPressed: incrementCurrentIndex()`实现来回移动的功能）。

    // ContactModel.qml
    ListModel {
        ListElement { name: "Linus Torvalds"; icon: "pics/qtlogo.png"; }
        ListElement { name: "Alan Turing"; icon: "pics/qtlogo.png"; }
        ListElement { name: "Margaret Hamilton"; icon: "pics/qtlogo.png"; }
        ListElement { name: "Ada Lovelace"; icon: "pics/qtlogo.png"; }
        ListElement { name: "Tim Berners-Lee"; icon: "pics/qtlogo.png"; }
        ListElement { name: "Grace Hopper"; icon: "pics/qtlogo.png"; }
    }
    

    Rectangle {
        width: 250; height: 200
    
        Component {
            id: delegate
            Item {
                width: 80; height: 80
                scale: PathView.iconScale
                opacity: PathView.iconOpacity
                Column {
                    Image { anchors.horizontalCenter: nameText.horizontalCenter; width: 64; height: 64; source: icon }
                    Text { id: nameText; text: name; font.pointSize: 12 }
                }
            }
        }
    
        PathView {
            anchors.fill: parent
            model: ContactModel {}
            delegate: delegate
            focus: true
            Keys.onLeftPressed: decrementCurrentIndex()
            Keys.onRightPressed: incrementCurrentIndex()
            path: Path {
                startX: 120
                startY: 100
                PathAttribute { name: "iconScale"; value: 1.0 }
                PathAttribute { name: "iconOpacity"; value: 1.0 }
                PathQuad { x: 120; y: 25; controlX: 260; controlY: 75 }
                PathAttribute { name: "iconScale"; value: 0.3 }
                PathAttribute { name: "iconOpacity"; value: 0.5 }
                PathQuad { x: 120; y: 100; controlX: -20; controlY: 0 }
            }
        }
    }
    
当在`PathView`委托中使用`Image`类型时，将`smooth`属性绑定到`PathView.view.moving`是非常有用的。这样当视图处于运动状态时，可以把较少的处理能力用于平滑变换，而在静止状态下，图像再执行平滑变换。

## 3.1 本节涉及的参考资料

https://qmlbook.github.io/en/ch06/index.html  
https://doc.qt.io/qt-5/qml-qtquick-listview.html  
https://doc.qt.io/qt-5/qml-qtquick-gridview.html  
https://doc.qt.io/qt-5/qml-qtquick-pathview.html  
https://doc.qt.io/qt-5/qml-qtquick-tableview.html  

[Source](https://materiaalit.github.io/qt-mooc/part4/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
