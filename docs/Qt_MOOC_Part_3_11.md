# <center>视觉效果<center>

Qt Quick和QML为开发人员提供了多种方式来为用户界面添加视觉效果。Qt Quick Shapes允许您创建各种形状并填充图案。QML有大量的动画类型，这使得使用甚至非线性时间轴来设置任何对象的动画都很容易。OpenGL程序可以很容易地创建渐变或渐变阴影。

让我们来看看如何在应用程序中添加视觉效果。

# 1. 形状

`Shape`类型，用于渲染任意形状（与我们已经看到的`Rectangle`类型相反）。`Shape`通过`QPainterPath`手动三角剖分生成几何形状然后使用GPU来进行渲染。

这种方法不同于通过`QQuickPaintedItem`或2D Canvas渲染形状，因为在软件中路径不会被栅格化。因此，`Shape`适合于创建扩展到屏幕更大区域的形状，避免因纹理上传或framebuffer blits而造成性能损失。此外，声明式API允许操作、绑定到路径元素属性，甚至使其具有动画效果，比如开始和结束位置、控制点等等。

它支持各种不同的路径，例如直线，椭圆形，弧形，二次曲线等。有关所有可用路径元素的列表，请参阅[Path](https://doc.qt.io/qt-5.11/qml-qtquick-path.html "Path")文档。

和`Item`一样，`Shape`也允许将任何可视或非可视对象声明为子对象。`ShapePath`对象是专门处理的。这很有用，因为它允许添加可视项（如`Rectangle`或`Image`）和非可视对象（如`Timer`）作为`Shape`的子级。

在使用`Shape`时，注意潜在的性能影响是很重要的(你不需要记忆甚至完全理解这个列表，但是最好知道它)：

*   当应用程序使用基于三角剖分的通用`Shape`实现运行时，几何图形的生成完全在CPU上进行，这可能很耗时。改变`Shape`元素的集合，或者改变这些元素的属性，或者改变形状本身的某些属性，都会导致在每次更改时对受影响的路径进行重新处理。因此，将动画应用于这些属性可能会影响功能较弱的系统的性能。
*   然而，`Shape`API的数据驱动、声明性质通常意味着底层CPU和GPU资源有更好的可缓存性。一个`ShapePath`中的属性更改只会导致对受影响的`ShapePath`进行重新处理，而不改变`Shape`的其他部分。因此，与命令式绘制方法(例如`QPainter`)相比，频繁更改属性导致的总体系统负载相对较低。
*   如果必须设置除笔划和填充颜色以外的属性动画，则建议将提供`GL_NV_path_rendering`的系统作为目标，其中属性更改的成本较小。
*   同时，必须注意`Shape`场景中元素的数量，尤其是当使用`GL_NV_path_rendering`这种特殊的加速方法时。这种`Shape`项目在场景图中的表示方式不同于普通的基于几何的项目，并且在涉及OpenGL状态更改时会产生一定的成本。
*   通常，在并非绝对必要时，场景应避免使用单独的`Shape`项目。比起使用多个`Shape`项，最好使用一个带有多个`ShapePath`元素的`Shape`项。如果不能避免使用大量个别`Shape`的场景应考虑设置`Shape.vendorExtensionsEnabled`为false。

现在，让我们看一个例子：

    import QtQuick 2.9
    import QtQuick.Window 2.2
    import QtQuick.Shapes 1.0
    
    Window {
        visible: true
        width: 640
        height: 480
        title: qsTr("Shapes")
    
        Shape {
            id: shape
            anchors.fill: parent
            ShapePath {
                strokeWidth: 10
                strokeColor: "black"
                fillGradient: RadialGradient {
                    centerX: shape.width / 2; centerY: shape.height / 2
                    centerRadius: shape.width * 0.1
                    focalX: centerX; focalY: centerY
                    spread: ShapeGradient.RepeatSpread
                    GradientStop { position: 0; color: "yellow" }
                    GradientStop { position: 1; color: "blue" }
                }
                strokeStyle: ShapePath.SolidLine
                startX: shape.width * 0.3; startY: shape.height * 0.1
                PathLine { x: shape.width * 0.7; y: shape.height * 0.1 }
                PathLine { x: shape.width * 0.9; y: shape.height * 0.3 }
                PathLine { x: shape.width * 0.9; y: shape.height * 0.7 }
                PathLine { x: shape.width * 0.7; y: shape.height * 0.9 }
                PathLine { x: shape.width * 0.3; y: shape.height * 0.9 }
                PathLine { x: shape.width * 0.1; y: shape.height * 0.7 }
                PathLine { x: shape.width * 0.1; y: shape.height * 0.3 }
                PathLine { x: shape.width * 0.3; y: shape.height * 0.1 }
            }
        }
    }
    

![](https://materiaalit.github.io/qt-mooc/img/part-3/shape-aa350e45.png)

在`ShapePath`中，我们用直线定义了形状，并用重复的`RadialGradient`去填充它。如果您在执行代码时遇到困难（即使您没有做到），请继续尝试一下！试着注释掉一些`PathLine`，看看会发生什么。将`RadialGradient`的`spread`属性更改为`ShapeGradient.ReflectSpread`看看会变成什么形状。

# 2. 动画

Qt Quick旨在创建具有流畅用户体验的动态用户界面。显然，也可以创建传统的，更静态的用户界面。实现流畅的用户体验的一种方法是动画。开发人员可以从各种QML类型中选择动画类型，例如`PropertyAnimation`，`SequentialAnimation`，`SpringAnimation`，`Behavior`，`AnimatedSprite`等（参见完整的[文档](http://doc.qt.io/qt-5/qtquick-statesanimations-animations.html)列表）。`Animation`扩展出了大量的动画类型，它提供了控制动画状态的方法：`start()`，`stop()`，`pause()`。此外，还有一些属性可以设置动画的执行次数（loops）或检查动画是否正在运行（running）。

动画会插入属性值，这些属性值在类型之间可能相差很大。`AnimatedSprite`显示图像帧，`PathAnimation`沿着定义的路径对项目的位置和旋转进行动画处理。默认情况下，大多数动画都是线性的，但是Qt为非线性动画提供了很多缓冲曲线（缓冲曲线的完整列表可以在[此处](http://doc.qt.io/qt-5/qeasingcurve.html#Type-enum)找到）。您可以右键单击Qt Creator中的动画对象，然后使用上下文菜单显示“Qt Quick 工具栏”。工具栏是研究不同缓冲曲线在不同特性设置下如何工作的一种简单方法，因为它为所有更改设置了动画。

最基本的动画可以通过属性动画来实现。下面的代码片段使用`OutBounce`缓冲曲线将红色矩形移至右侧。可以使用属性`target`或`targets`定义对象的属性。在此示例中，target是父级，因此我们无需定义它。类似地，可以使用一个或多个属性来定义哪个或哪些属性使用动画。该示例使用另一个漂亮的语法`PropertyAnimation on x`，它定义了该矩形的`x`属性是具有动画效果的。

    Rectangle {
        width: 50; height: 50
        color: "red"
    
        PropertyAnimation on x {
            from: 50; to: 150
            duration: 1000;
            easing.type: Easing.OutBounce;
            loops: Animation.Infinite
        }
    }    

上面的动画运行了无数次。可以通过将`running`属性设置为false或通过调用`stop()`来停止动画。

    Rectangle {
        width: 50; height: 50
        color: "red"
    
        PropertyAnimation on x {
            id: animation
            running: false
            from: 50; to: 150
            duration: 1000;
            easing.type: Easing.OutBounce;
            loops: Animation.Infinite
        }
    
        TapHandler {
            onTapped: animation.running ? animation.stop() : animation.start();
        }
    }    

`Behavior`类型是定义动画的非常优雅的方式。每当属性值更改时，`Behavior`就对更改进行动画处理。修改前面的示例使移动到鼠标单击位置的矩形具有动画效果，可以像在下面示例中这样实现：

    Rectangle {
        id: rect
        width: 50; height: 50
        color: "red"
    
        Behavior on x { 
            PropertyAnimation { 
                easing.amplitude: 2.9; easing.type: Easing.InElastic; duration: 500 
            } 
        }
        Behavior on y { 
            PropertyAnimation { duration: 500 } 
        }
    }
    
    MouseArea {
        anchors.fill: parent
        onClicked: { rect.x = mouse.x; rect.y = mouse.y }
    }

请注意，不必在动画中定义所有的属性。例如，一个非常简单的动画对象可以是`NumberAnimation { }`。这将在250毫秒（默认值）内使用一条线性曲线使所定义的属性产生动画效果。

Qt提供了许多便利的QML类型，以使动画的使用非常简单。例如，`PathAnimation`会沿着预定义的路径设置动画。路径由一种`Path`类型定义，可以由任意数量的直线，圆弧，Svg或二次曲线以及三次方贝塞尔曲线组成。（请参阅[路径](http://doc.qt.io/qt-5/qml-qtquick-path.html)文档）。

下面的例子显示了相当随机的路径，由两个三次贝塞尔曲线组成。火箭是一个图像，会沿着路径运动。

    Path {
        startX: rocket.width/2; startY: rocket.height/2
    
        PathCubic {
            x: window.width - rocket.width/2
            y: window.height - rocket.height/2
    
            control1X: x; control1Y: rocket.height/2
            control2X: rocket.width/2; control2Y: y
        }
        PathCubic {
            x: window.width - rocket.width/2
            y: window.height - rocket.height/2
    
            control1X: x; control1Y: rocket.height/2
            control2X: rocket.width/2; control2Y: y
        }
    } 

定义完路径后，可以在其他类型中使用相同的路径。如果要绘制路径，可以使用`Shape`。

    Shape {
        anchors.fill: parent
        ShapePath {
            strokeWidth: 4
            strokeColor: "white"
            fillColor: "transparent"
    
            // The path definition goes here
        }
    }

最后，我们可以使用动画项目`PathAnimation`来移动项目。

    Item {
        id: rocket
        x: 0; y: 0
        width: 128; height: 96
    
        Image {
            source: "qrc:/images/rocket.png"
            anchors.centerIn: parent
            rotation: 90
        }
    
        MouseArea {
            anchors.fill: parent
            onClicked: pathAnim.running ? pathAnim.stop() : pathAnim.start()
        }
    }
    
    PathAnimation {
        id: pathAnim
        duration: 2000
        easing.type: Easing.InOutQuad
    
        target: rocket
        orientation: PathAnimation.RightFirst
        anchorPoint: Qt.point(rocket.width/2, rocket.height/2)
    
        path: Path {
            // The path definition goes here
            }
        }
    }

![](https://materiaalit.github.io/qt-mooc/img/part-3/qt-rocket-5c4d69e9.gif)

除了`PropertyAnimation`及其子类型外，还有一些动画类型，它们是扩展了`Item`而不是`Animation`类型。`AnimatedSprite`和`AnimatedImage`使显示动图变得容易，包括几个帧或gif动画。`AnimatedSprite`允许帧水平或垂直组织，如下图所示。图像由51个水平排列的雪花帧组成。

唯一需要做的就是确定在帧序列中可见帧的时间。

    AnimatedSprite {
        id: animSprite
        width: parent.width * 0.25
        height: parent.height * 0.25
        running: true
        source: "qrc:/images/snowflake.png"
        frameCount: 51  // Frame width automatically calculated
        frameDuration: 10
    }

## 2.1 状态与动画状态转场

动画通常与状态和转场一起使用。每个Qt Quick项都有一个`state`属性，它定义了该项的状态。也可以在`states`属性中定义一个状态列表。每个状态都由一个字符串名称标识。状态名称默认是一个空字符串`“”`。

下面的例子中有两个矩形。在默认状态下，我们只想显示红色矩形view1。蓝色矩形view2在红色矩形的右边并且大小与父对象一致。所以，如果父对象设置了clip属性，那么它是不可见的。

    Rectangle {
        id: view1
        color: "red"
        width: parent.width
        height: parent.height
    }
    
    Rectangle {
        id: view2
        color: "blue"
        width: parent.width
        height: parent.height
        x: parent.width
    }

如果想添加更多状态，可以使用`PropertyChanges`类型，它用于定义在该状态下哪些属性更改为哪些值。请注意，我们不必定义多个状态，因为默认状态在开始时定义了矩形位置。当状态变为“inView2”时，红色矩形的位置在父元素的左边，蓝色矩形填充父元素。当状态发生时，如果没有显式地为项的`state`属性指定新的状态名，那么可选的`when`属性将很有用。

    states: [
        State {
            name: "inView2"
            when: tapHandler.pressed
            PropertyChanges {
                target: view1
                x: -root.width
            }
            PropertyChanges {
                target: view2
                x: 0
            }
        }
    ]

请注意，示例中还没有动画。动画在转场列表中定义。当状态发生变化时，可能会有一个状态转换，该转换会为用更改的属性设置动画`PropertyChanges`。

    transitions: [
        Transition {
            from: ""
            to: "inView2"
            NumberAnimation { properties: "x"; duration: 1000 }
        },
        Transition {
            from: "inView2"
            to: ""
            NumberAnimation { properties: "x"; duration: 1000 }
        }
    ]
    

下面是我们的例子：（看上去卡顿是由.gif图片导致的，与Qt无关）

![](https://materiaalit.github.io/qt-mooc/img/part-3/transition-example-d7323c79.gif)

在我们的示例中，从默认状态到“inView2”状态的转场和从“inView2”状态到默认状态是相似的。这种情况下只需要一个转场，我们可以通过将`reversible`属性设置为true来实现。

    Transition {
        from: ""
        to: "inView2"
        reversible: true
        NumberAnimation { properties: "x"; duration: 1000 }
    }

效果与前面一样，但是代码更干净，也更容易阅读。

# 3. 图形效果

图形效果是将效果应用于QML项目的视觉项目。每个效果都有其自己的特定效果属性。效果是使用OpenGL顶点着色器和片元着色器来修改顶点位置或片段、像素、颜色来获得效果。在具有图形处理器的硬件平台上，效果是非常有效的。但是在使用软件渲染的嵌入式平台中，通常应该避免效果的使用，以便为更重要的任务节省CPU资源。

使用`Rectangle`类型及其`radius`属性在Qt中创建矩形甚至椭圆非常简单。矩形可以用渐变色绘制，但是我们如何创建非线性渐变呢？一种解决方案是使用`RadialGradient`。

在下面的示例中，我们在窗口中创建了一个径向渐变。它使用了两种颜色，先从黄色变为黑色，在从黑色变为黄色。

    import QtQuick 2.12
    import QtGraphicalEffects 1.12
    import QtQuick.Window 2.12
    
    Window {
        visible: true
        width: 640
        height: 480
        title: qsTr("Radial gradient")
    
        RadialGradient {
            anchors.fill: parent
            gradient: Gradient {
                GradientStop { position: 0.0; color: "yellow" }
                GradientStop { position: 0.5; color: "black" }
                GradientStop { position: 1.0; color: "yellow" }
    
            }
        }
    }

其他渐变也同样易于使用。

| 渐变 | 示例 |
| --- | --- |
| [线性渐变](http://doc.qt.io/qt-5/qml-qtgraphicaleffects-lineargradient.html) | ![](https://materiaalit.github.io/qt-mooc/img/part-3/lineargradient-31c326dd.png) |
| [锥形渐变](http://doc.qt.io/qt-5/qml-qtgraphicaleffects-conicalgradient.html) | ![](https://materiaalit.github.io/qt-mooc/img/part-3/conicalgradient-e9f098fa.png) |
| 径向渐变 | ![](https://materiaalit.github.io/qt-mooc/img/part-3/radialgradient-79dc7ec1.png) |

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
