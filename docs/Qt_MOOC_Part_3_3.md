# <center>QML基本类型<center>

# 1. Qt对象

`Qt`作为QML的类型为其提供了一个全局对象，用于引用Qt中的枚举和函数。如果要使用它，可以直接使用`Qt`对象调用全局的成员。例如:

    import QtQuick 2.0
    
    Text {
        color: Qt.rgba(1, 0, 0, 1)
        text: Qt.md5("hello, world")
    }
    

请参阅有关[Qt QML类型](http://doc.qt.io/qt-5/qml-qtqml-qt.html "Qt QML类型")的文档，以及其提供的所有内容。

# 2. Rectangle

[Rectangle](http://doc.qt.io/qt-5/qml-qtquick-rectangle.html "Rectangle")类型用于用纯色或渐变色填充区域或提供矩形边框。

`Rectangle`要么使用实线填充颜色(使用`color`属性指定)，要么使用`Gradient`类型定义并使用`gradient`属性设置渐变。如果同时指定了颜色和渐变，则使用渐变。

您可以通过设置`border.color`和`border.width`属性，为具有自己的颜色和厚度的矩形添加可选边框。将颜色设置为`transparent`以绘制没有填充颜色的边框。

您也可以使用`radius`属性来创建圆角矩形。因为这会给矩形的四角引入曲线边，所以设置`Item::antialiasing`属性以改善其外观可能是合适的。

示例：

    import QtQuick 2.0
    
    Rectangle {
        width: 100
        height: 100
        color: "red"
        border.color: "black"
        border.width: 5
        radius: 10
    }
   
如果你需要的是不规则的形状而不是矩形，请查看`Shape`类型。我们将在第3.11章中讨论`Shape`。 

# 3. Image

[Image](https://doc.qt.io/qt-5/qml-qtquick-image.html "Image")类型用于显示来自`source`属性中指定的URL的图像。它可以处理Qt支持的URL和图像类型（PNG，JPEG，SVG等）。

图像源可以小于或大于`Image`的尺寸。使用它的`fillMode`属性，可以设置在项目内部绘制图像时使用的策略。

默认情况下，从网络资源加载图像是通过单独的线程实现异步的。这样，如果网络缓慢将不会阻塞UI。而对于本地文件默认则是同步加载的，不过您也可以设置[asynchronous](https://doc.qt.io/qt-5/qml-qtquick-image.html#asynchronous-prop "asynchronous")属性为`true`来实现异步。如果有不需要立即在UI中显示的大文件，则异步加载会很有用。

要跟踪图像的状态，可以使用`Image`的[progress](https://doc.qt.io/qt-5/qml-qtquick-image.html#progress-prop "progress")和[status](https://doc.qt.io/qt-5/qml-qtquick-image.html#status-prop "status")属性。比如把它们绑定到UI中进行可视化。

在QML中使用图像通常是UI中最耗内存的。为了节省内存的使用，它们的大小应与[sourceSize](https://doc.qt.io/qt-5/qml-qtquick-image.html#sourceSize-prop "sourceSize")属性绑定。`sourceSize`代表所加载图像的实际宽度和高度，而`width`和`height`属性则包含图像将被缩放到的尺寸。

    import QtQuick 2.0
    
    Image {
        id: bigImage
        anchors.fill: parent
        asynchronous: true
        onStatusChanged: {
            if (bigImage.status == Image.Ready) console.log("Image loaded")
        }
        source: "bigImage.jpg"
        sourceSize.width: 1024
        sourceSize.height: 1024
    }

图像数据在内部进行缓存并共享，因此当使用相同`source`的对象时，它们将使用相同的数据。

# 4. BorderImage

[BorderImage](https://doc.qt.io/qt-5/qml-qtquick-borderimage.html "BorderImage")用于通过缩放或平铺从图像的各个部分形成边界。

`BorderImage` 将源图像分成9个区域，像这样： 

![](https://materiaalit.github.io/qt-mooc/img/part-3/borderimage2-4c97cf4b.png)

用`border`**属性组**来定义区域。将源图像形成的区域按以下方式进行缩放或平铺以创建显示的边界图像:

*   角（区域1、3、7和9）不做缩放处理。
*   区域2和8根据`horizontalTileMode`进行缩放。
*   区域4和6根据`verticalTileMode`进行缩放。
*   区域5作为中间区域根据`horizontalTileMode`和`verticalTileMode`缩放。

如果TileMode设置为`Stretch`，则如有需要，图像的各个部分将垂直/水平拉伸。如果将其设置为`Repeat`，则重复该部分。当边界区域2、4、6、8的宽度/高度不能以目标宽度/高度的精确倍数重复时，可以设置tilemode为[BorderImage.Round](https://doc.qt.io/qt-5/qml-qtquick-borderimage.html#horizontalTileMode-prop "BorderImage.Round")缩放区域以适合目标。

用法示例：

    BorderImage {
        width: 180; height: 180
        border { left: 30; top: 30; right: 30; bottom: 30 }
        horizontalTileMode: BorderImage.Repeat
        verticalTileMode: BorderImage.Repeat
        source: "pics/borderframe.png"
    }
    

# 5. Text

[Text](https://doc.qt.io/qt-5/qml-qtquick-text.html "Text")类型可以显示纯文本或富文本（使用[HTML标记](https://doc.qt.io/qt-5/richtext-html-subset.html "HTML标记")）。要使用户可以编辑文本，可以使用[TextEdit](https://doc.qt.io/qt-5/qml-qtquick-textedit.html "TextEdit")，它与`Text`非常相似。

    import QtQuick 2.0
    
    Text {
        text: "Hello <b>World</b>!"
        font.family: "Helvetica"
        font.pointSize: 24
        color: "red"
    }
    
如果没有显式地设置`width`和`height`，`Text`默认尺寸为文本所占的尺寸。通常显式设置大小不是最优的，因为它会导致布局重新计算，所以只在需要时设置它。

要启用文本换行，需要设置[wrapMode](https://doc.qt.io/qt-5/qml-qtquick-text.html#wrapMode-prop "wrapMode")属性，否则文本为单行模式。

要自定义字体，可以修改`font`组属性:

*   [font.family](https://doc.qt.io/qt-5/qml-qtquick-textedit.html#font.family-prop "font.family") 选择字体族
*   [font.pointSize](http://doc.qt.io/qt-5/qml-qtquick-text.html#font.pointSize-prop "font.pointSize") 设置与设备独立的字体大小
*   [font.pixelSize](http://doc.qt.io/qt-5/qml-qtquick-text.html#font.pixelSize-prop "font.pixelSize") 设置字体的像素大小

若要设置文本的样式，可使用属性[style](http://doc.qt.io/qt-5/qml-qtquick-text.html#style-prop "style")将样式更改为凸起（`Raised`）、轮廓（`Outline`）或凹形（`Sunken`）。使用[styleColor](http://doc.qt.io/qt-5/qml-qtquick-text.html#styleColor-prop "styleColor")更改添加样式的颜色。

    Row {
        Text { font.pointSize: 24; text: "Normal" }
        Text { font.pointSize: 24; text: "Raised"; style: Text.Raised; styleColor: "#AAAAAA" }
        Text { font.pointSize: 24; text: "Outline";style: Text.Outline; styleColor: "green" }
        Text { font.pointSize: 24; text: "Sunken"; style: Text.Sunken; styleColor: "#AAAAAA" }
    }
    

要缩放文字大小，您可以:

*   将[`font.pixelSize`](http://doc.qt.io/qt-5/qml-qtquick-text.html#font.pixelSize-prop "font.pixelSize")属性绑定到`Text`。
*   设置[`fontSizeMode`](http://doc.qt.io/qt-5/qml-qtquick-text.html#fontSizeMode-prop "fontSizeMode")缩放文字大小。


        Rectangle { 
            width: 400
            height: 400
            color: "lightblue" 
            
            Text { 
                x: parent.width * 0.25
                y: parent.height * 0.25 
                
                text: "Qt Quick"
                font {
                    family: "Sans"
                    pixelSize: parent.width * 0.1
                } 
            } 
        }
    

您还可以使用[FontMetrics](https://doc.qt.io/qt-5/qml-qtquick-fontmetrics.html "FontMetrics")来使用特定的字体：

    Rectangle {
    
        FontMetrics {
            id: metrics
            font.pointSize: 20
            font.family: "Courier"
        }
    
        width: 200
        height: metrics.height * 10
        
    }

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
