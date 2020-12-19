# <center>模型视图框架<center>

许多应用程序需要向用户显示数据，甚至允许用户操作和创建新数据。Qt模型视图框架可以使开发人员轻松地创建这样的应用程序。模型与视图的分离，可以使多个视图共享同一个模型，或者动态地更改一个视图模型。

![](http://doc.qt.io/qt-5/images/modelview-overview.png)

*   Model是数据及其结构的适配器。实际数据可能存储在任何地方，例如数据库或云上的数据中心。在一般情况下，模型本身也可以包含数据。QML有几种用于创建模型的类型，但如果想获得更高的效率请使用`QAbstractItemModel`的子类。
*   View可以以任何可视结构显示数据，如列表、表格或路径。有用于堆栈项目的[StackView](http://doc.qt.io/qt-5/qml-qtquick-controls-stackview.html "StackView")或可以滚动大容量内容的[ScrollView](http://doc.qt.io/qt-5/qml-qtquick-controls-scrollview.html "ScrollView")视图。这些视图使用了内部模型并提供专门的API，因此不在此处介绍。开发人员还可以创建自定义的视图，以他们想要的任何方式构造UI。
*   Delegate规定了数据在视图中的显示方式。委托可以获取模型中的数据项并将其封装。可以通过委托访问模型数据。

# 1. 基本概念

动态视图（如[ListView](http://doc.qt.io/qt-5/qml-qtquick-listview.html "ListView")）具有一个模型，可以动态创建和销毁委托项以显示数据。您可以将任何变量类型赋值给视图的`model`属性。在下面的例子中，一个整数5被赋值给`model`，这样视图就会创建出来5个委托项目。由于模型可以是任何变量类型，所以可以赋值一个JavaScript、QList、QML模型类型或`QAbstractItemModel`的子类，这将在下一阶段中介绍。

    ListView {
        anchors.fill: parent
        model: 5
        delegate: Component {
            Text {
                text: index
            }
        }
    }

委托的类型是`Component`。我们这里直接把它定义在文件中，其实也可以定义为可重用的QML类型。委托的生命周期由视图管理。不同的视图使用不同的策略来管理生命周期，我们将在后面介绍它。

该视图为委托项公开了几个属性。其中`text`属性具有一个`index`值。`ìndex`根据上下文而来，这意味着每个委托项都具有不同的索引值。第一个索引为0，下一个为1，依此类推。这是识别委托对象的有用方法。

## 1.1 QML基本模型

一个简单且常用的QML模型类型是[ListModel](https://doc.qt.io/qt-5/qml-qtqml-models-listmodel.html "ListModel")，它是定义[ListElement](https://doc.qt.io/qt-5/qml-qtqml-models-listelement.html "ListElement")的容器。在[ListElement](https://doc.qt.io/qt-5/qml-qtqml-models-listelement.html "ListElement")中使用**角色名称**而不是属性来定义数据。请注意，角色名称必须与给定模型中的所有其他元素相同。

在下面的示例中，有两个角色，分别为name和teamColor。

    ListModel {
        id: teamModel
        ListElement { name: "Lars Knoll"; teamColor: "red" }
        ListElement { name: "Alan Kay"; teamColor: "lightBlue" }
        ListElement { name: "Trygve Reenskaug"; teamColor: "red" }
        ListElement { name: "Adele Goldberg"; teamColor: "yellow" }
        ListElement { name: "Ole-Johan Dahl"; teamColor: "red" }
        ListElement { name: "Kristen Nygaard"; teamColor: "lightBlue" }
    }

角色的类型在第一次使用角色时就固定下来。上例中的所有角色都是字符串类型。不过您也可以为动态角色分配不同的类型。若要启用动态角色，必须将`ListModel`的`dynamicRoles`属性设置为true。静态定义的数据（如前一个示例中所示）不能具有动态角色。具有动态角色的数据项必须使用JavaScript添加。

该视图将角色公开给委托项目，委托项目可以通过使用角色名称来引用角色，如以下示例所示。如果`Text`类型里也有`name`属性，则必须将模型限定符用作角色的前缀，以避免名称冲突。比如对于列表模型会向委托公开一个`modelData`属性。

    ListView {
        anchors.fill: parent
        model: teamModel
        delegate: Text {
            text: name
            // text: modelData.name // in case there is a clash between roles and properties
        }
    }

# 2. 模型操作

像我们在前面的示例中所做的那样，静态地创建和操作模型数据在实际应用程序中是不可扩展的。其实数据项还可以在JavaScript中进行操作，不过您更应该使用C++模型，我们以后会介绍它。`ListModel`有[append()](https://doc.qt.io/qt-5/qml-qtqml-models-listmodel.html#append-method "append()")，[insert()](https://doc.qt.io/qt-5/qml-qtqml-models-listmodel.html#insert-method "insert()")，[move()](https://doc.qt.io/qt-5/qml-qtqml-models-listmodel.html#move-method "move()")和[remove()](https://doc.qt.io/qt-5/qml-qtqml-models-listmodel.html#remove-method "remove()")等方法可以使用。

下面的例子演示了如何将数据`append`到一个`ListModel`：

    ListModel {
        id: teamModel
        ListElement {
            name: "Team C++"
        }
    }
    
    MouseArea {
        anchors.fill: parent
        onClicked: {
            teamModel.append({ name: "Team Quick" })
        }
    }
    

您可以使用JSON`{ "teamName": "Team Qt", "teamColor": "green" }`或JavaScript`{ teamName: "Team Qt", teamColor: "green" }`来创建新项目。应该注意的是，插入或声明到模型的第一项将决定该视图中可用的角色。换句话说，如果模型为空并添加了一个项，则仅绑定在该项目中定义的角色，并在后续添加的项目中使用。若重置角色，需要使用[ListModel::clear()](https://doc-snapshots.qt.io/qt5-dev/qml-qtqml-models-listmodel.html#clear-method "ListModel::clear()")方法清空模型。

## 2.1 在线程中处理模型

在某些情况下，修改模型的计算成本可能很高，而且由于是在UI线程中进行的，因此可能会阻塞一段时间。考虑在模型中创建1,000,000项的简单情况。需要大量计算的更新最好在后台线程中完成。例如，我们要实现一个Twitch聊天客户端，如果我们在主线程更新消息模型会很快遇到性能问题：

    TwitchChat {
        // signal messageReceived(string user, string message)
        onMessageReceived: {
            // Massage the received message data inside the signal handler ...
            // Remove old messages from backlog ...
            if (messageModel.count > 2147483647)
               messageModel.remove(0, 1337)
            // Append the message data to the model
            messageModel.append({"user": user, "message": message})
        }
    }    

为此，Qt提供了QML类型[WorkerScript](https://doc.qt.io/qt-5/qml-workerscript.html "WorkerScript")以从主线程中移除繁重的数据和模型操作。通过`sendMessage()`方法和`onMessage()`处理器在父线程和新子线程之间传递消息：

    TwitchChat {
        // signal messageReceived(string user, string message)
        onMessageReceived: {
            worker.sendMessage({"user": user, "message": message, "model": messageModel)
        }
    }
    
    WorkerScript {
        id: worker
        source: 'script.js'
    }

`onMessage`处理器在`script.js`文件中实现，在调用`sendMessage()`时触发：

    // script.js
    WorkerScript.onMessage = function(msg) {
        // Massage the received message data ...
        // Remove old messages from backlog ...
        if (msg.model.count > 2147483647)
            msg.model.remove(0, 1337)
        // Append the message data to the model
        msg.model.append({"user": user, "message": message});
        msg.model.sync();   // Needs to be called! 
    }

请注意，我们并没有导入`script.js`文件！这带来了此线程模型的局限性：`source`属性是在QML之外单独求值的。这意味着任何属性或方法以及所有数据都需要传递给`sendMessage()`。`sendMessage()`支持所有的JavaScript类型和对象。不支持`QObject`类型，只支持`ListModel`类型。另外，如果您希望您的修改在`ListModel`中生效，您必须在工作线程中调用`sync()`方法。

## 2.2 本节涉及的参考资料

https://doc.qt.io/qt-5/qtquick-modelviewsdata-modelview.html
https://qmlbook.github.io/en/ch06/index.html
http://doc.qt.io/qt-5/qml-qtqml-models-listmodel.html

[Source](https://materiaalit.github.io/qt-mooc/part4/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
