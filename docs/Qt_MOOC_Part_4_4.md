# <center>委托<center>

委托用于充当实例化视图中的可视项模板。模型提供的数据角色绑定到可视项属性，例如`Text.text`或`Image.source`属性。只要为角色分配新值，委托就会更新绑定到角色的数据。模型则会通知所有视图数据值的更改。

在下面的例子中，我们有一个`TeamDelegate`，它会在单击时更新绑定的数据：

    ListModel {
        id: teamModel
        ListElement {
            teamName: "Team C++"
            teamColor: "blue"
        }
    }
    
    ListView {
        anchors.fill: parent
        model: teamModel
        delegate: TeamDelegate {}
    }
    
    // TeamDelegate.qml
    Component {
        Text {
            text: teamName
            color: teamColor
    
            MouseArea {
                anchors.fill: parent
                onClicked: {
                        model.teamName = "Team Quick"
                        model.teamColor = "red"
                    }
                }
            }
        }
    }

在委托中，可以将[数据角色](http://doc.qt.io/qt-5/qtquick-modelviewsdata-modelview.html#qml-data-models "数据角色")作为内部属性进行访问。如果委托项的属性名称和模型角色有名称冲突，还需使用`model`限定符来解决。

# 1. 委托的大小

到目前为止，在大多数示例中，我们使用了`Text`或`Image`类型作为委托。这些类型具有隐式的大小，因此不需要使用`Item::width`和`Item::height`属性来显式定义。实际上，在`Text`中同时使用隐式大小和显式大小会导致性能下降，因为必须对它进行两次布局。

在实际应用中，委托很少只包含文本或图像组件。更多的情况是委托由一些其他QML类型组成。由于委托也是组件，因此它必须包含一个根项目。通常我们使用`Item`作为根项目，因为它适合用来从其他项目组合委托。但是`Item`并没有明确的大小，所以我们需要显式的声明它。

一种方法是将根项目的大小绑定到视图大小。在以下示例中，假设模型包含了`n`项目，我们就将委托的高度绑定到项目的可用高度。为了保持文本的可读性，我们使用`FontMetrics`来计算最小高度。

    ListView {
        anchors.fill: parent
        anchors.margins: 20
        clip: true
        model: 50
        delegate: numberDelegate
    }
    
    FontMetrics {
        id: fontMetrics
        font { family: "Arial"; pixelSize: 14 }
    }
    
    Component {
        id: numberDelegate
        Item {
            width: ListView.view.width
            height: Math.max(ListView.view.height / ListView.view.count, fontMetrics.height)
    
            Rectangle {
                anchors.fill: parent
                border.color: "black"
                Text {
                    font { family: "Arial"; pixelSize: 14}
                    text: 'Item ' + index;
                    anchors.centerIn: parent;
                }
            }
        }
    }

另一种方法是让委托子元素确定其大小。如果子元素具有隐式大小(如下面示例中的`Text`)，则此方法很有用。

    Component {
        id: numberDelegate
        Item {
            id: rootItem
            width: ListView.view.width
            height: childrenRect.height
    
            Rectangle {
                width: rootItem.width
                height: childrenRect.height + 2
                border.color: "black"
                Text {
                    font { family: "Arial"; pixelSize: 14}
                    text: 'Item ' + index;
                }
            }
        }
    }

您需要确保您的委托尽可能简单，并且大小易于计算，避免复杂的大小计算和委托对象之间的关系依赖。

# 1.1 裁剪

视图的`clip`属性可以确保视图之外的其他视图项都不可见。如果将其设置为`false`，则项目可能会超出该视图。但是您应该避免在委托中使用`clip`。因为如果在委托内部启用了`clip`，那么每个委托都将进行处理。QML渲染器为所有可见项创建了一个场景图，它试图通过分批绘制来最大程度地减少OpenGL状态更改的次数。一个批处理只包含不需要OpenGL状态更改的操作，较少的批次可以得到更好的渲染性能。

裁剪会导致将场景图中的节点及其完整的子树放入一个批处理中。如果在视图中使用了`clip`属性，而不是在委托中使用，那么视图和委托的整个树就可以放入一个批处理中。但是一旦对委托使用了`clip`属性，就会为每个委托创建一个新的场景图子树。这将增加批处理的数量，并对性能产生负面影响。

[Scenegraph renderer](http://doc.qt.io/qt-5/qtquick-visualcanvas-scenegraph-renderer.html "Scenegraph renderer")里更详细地介绍了批处理。

# 2. 内存管理

动态视图的委托可以被动态地创建和销毁，但`TableView`是个例外，它能重用内存池中的现有委托。您可以使用`TableView`的`reuseItems`属性控制是否重用。

`ListView`，`GridView`和`PathView`都会创建尽可能多的委托项，只要视图可以在其区域中显示。这使得Qt开发人员可以使用包含数百万个项目的庞大项目模型，因为您一次只能看到和创建一小部分项目。当用户滑动视图时，如果可见项目超出了视图范围就会被销毁，而如果它在视图中变为可见状态则会被自动创建出来。

项目的动态创建意味着您永远不能将状态信息存储到委托中。在项目被销毁之前，它始终将状态信息存储到模型中。

    Component.onDestruction: {
        someBooleanRole = (state === "false") ? false : true;
    }

当用户滑动视图时，视图可以提供缓存以提高性能。如`ListView`和`GridView`使用了`cacheBuffer`，`PathView`使用了`cacheItemCount`。`PathView::pathItemCount`是确定要创建多少个可见项目，而`PathView::cacheItemCount`确定了在缓存中创建了多少个其他项目。缓存委托项可以提高性能，但会增加内存消耗。

`cacheBuffer`属性的值是一个整数，用于确定要缓存多少个委托项。如果在列表视图中`cacheBuffer`的值为100，并且委托项的高度为20，则在当前可见项目的前面会缓存5个项目，后面再缓存5个项目。

`GridView`的缓存原理与`ListView`相似。如果在垂直视图中委托的高度是20个像素，并且具有3列，而且把`cacheBuffer`设置为40，则可以创建或保留可见区域上方的最多6个委托项目和可见区域下方的6个委托项目。

`TableView`也可以重用委托项目。当一个项目被弹出时，就会被移至重用池，该池是存储未使用项目的内部缓存，然后发出`TableView::pooled`信号来通知该项目。同理，当项目从池中移回时，将发出`TableView::reused`信号。

当项目被重用时，来自模型的项目的所有属性都会被更新，包括索引、行、列和模型的角色。

您应该避免把状态存储在委托中。如果一定要这样做，请在收到`TableView::reused`信号后手动将其重置。

如果某个项目具有计时器或动画，请考虑在接收到`TableView::pooled`信号时暂停它们。这样，您可以避免将CPU资源用于不可见的项目。同样，如果某个项目具有无法重复使用的资源，则可以释放它们。

下面的示例显示了一个委托，它为旋转的矩形设置动画。当它被移至重用池时，动画将暂停。

    Component {
        id: tableViewDelegate
        Rectangle {
            implicitWidth: 100
            implicitHeight: 50
    
            TableView.onPooled: rotationAnimation.pause()
            TableView.onReused: rotationAnimation.resume()
    
            Rectangle {
                id: rect
                anchors.centerIn: parent
                width: 40
                height: 5
                color: "green"
    
                RotationAnimation {
                    id: rotationAnimation
                    target: rect
                    duration: (Math.random() * 2000) + 200
                    from: 0
                    to: 359
                    running: true
                    loops: Animation.Infinite
                }
            }
        }
    }

# 3. 复杂委托

您应该让您的委托尽可能简单。虽然有很多方法可以管理复杂的委托，但是从性能和内存消耗角度来看，最好的解决方案还是声明一个简单的委托。

那么为什么要确保委托简单呢？比如当委托出于某些奇怪的原因而需要包含自动播放视频的[Video](https://doc.qt.io/qt-5/qml-qtmultimedia-video.html "Video")项目时，就会出现问题。

    Item {
        id: videoDelegate
    
        Video {
            id: video
            width: 800
            height: 600
            source: model.videosrc
            autoPlay: true
        }
    }

如果目标平台是个低端设备，那么在启动时加载所有这些项目将花费大量的时间，甚至在有些设备上可能无法同时播放多个视频源。

如果您确实需要在委托中加载资源密集型的项目，则可以使用动态创建对象，如[动态创建对象](https://doc.qt.io/qt-5/qtqml-javascript-dynamicobjectcreation.html "动态创建对象")文档中所述。

QML里有一个[Loader](https://doc.qt.io/qt-5/qml-qtquick-loader.html "Loader")类型，可用于加载UI的不同部分（如复杂的委托）以提高性能。下面的示例中有一个委托，该委托具有一个`Component`类型，里面含有一个视频组件。然后我们再向委托添加一个`Loader`和一个`MouseArea`，当单击鼠标时通过`sourceComponent`属性设置`videoComponent`组件，这样就可以用`Loader`加载`videoComponent`。

    Item {
        id: lazyVideoDelegate
    
        width: 200
        height: 200
    
        Component {
            id: videoComponent
    
            Video {
                id: video
                width: 800
                height: 600
                source: model.videosrc
                autoPlay: true
            }
        }
    
        Loader {
            id: videoLoader
        }
    
        MouseArea {
            anchors.fill: parent
            onClicked: videoLoader.sourceComponent = videoComponent
        }
    }

请注意，虽然声明的`Video`项目可以自动显示，但上述情况并非如此。因为这里是在`Component`内部定义的它。

这样就可以大大减少加载的时间，因为委托最初只包含`Loader`和`MouseArea`对象。但是请注意，我们创建了一个`Loader`对象。虽然它是轻量级的，但无论如何还是会消耗一些内存。更好的方法是尝试避免在委托中使用`Loader`。比如使用`Image`来显示视频的缩略图或类似内容的图像。

## 3.1 本节涉及的参考资料

http://doc.qt.io/qt-5/model-view-programming.html
https://qmlbook.github.io/en/ch06/index.html#delegate
https://www.quora.com/What-are-delegates-in-Qt
https://doc.qt.io/archives/qq/qq24-delegates.html
http://doc.qt.io/qt-5/qitemdelegate.html
https://www.ics.com/designpatterns/book/delegates.html
http://doc.qt.io/qt-5/qitemdelegate.html#details

[Source](https://materiaalit.github.io/qt-mooc/part4/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
