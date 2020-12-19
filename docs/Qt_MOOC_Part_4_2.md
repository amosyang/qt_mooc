# <center>QML模型<center>

除了`ListModel`之外，还有其他几种QML模型类型。如`XmlistModel`，使用它可以从XML数据源构造模型，
它提供了一种非常方便的方法来解析XML数据。`ObjectModel`附带了要在视图中使用的可视项，它不需要委托，因为模型本身已经包含了可视项。`DelegateModel`在使用`QAbstractItemModel`索引访问数据项的情况下非常有用，典型的用例是层次树模型，用户可以在子树上下导航。

您甚至可以把对象实例当作model。在这种情况下，对象属性就是角色名称，如以下示例所示。

    Text {
        id: myText
        text: "Hello"
        visible: false 
    }
    
    Component {
        id: myDelegate
        Text { text: model.text } 
    }
    
    ListView {
        anchors.fill: parent
        model: myText
        delegate: myDelegate
    }    

# 1. XmlListModel

[XmlListModel](https://doc.qt.io/qt-5/qml-qtquick-xmllistmodel-xmllistmodel.html "XmlListModel")用于从XML数据创建只读模型。这是一种从类似RSS Feeds读取数据并从相关XML元素创建数据模型的非常方便的方法。

一个简单的XML结构可能如下：

    <?xml version="1.0" encoding="utf-8"?>
    <rss version="2.0">
        <channel>
            <item>
                <title>A blog post</title>
                <pubDate>Sat, 07 Sep 2010 10:00:01 GMT</pubDate>
            </item>
            <item>
                <title>Another blog post</title>
                <pubDate>Sat, 07 Sep 2010 15:35:01 GMT</pubDate>
            </item>
        </channel>
    </rss>

这个XML文档包含了**标签**（`<rss>`）和**属性**（`version="2.0"`）。开始标签`<item>`和结束标签`</item>`形成了一个**元素**，元素还可以具有子元素。这意味着它是一棵节点树，需要遍历该树以提取数据。

以下示例显示了如何从XML数据创建模型。

    import QtQuick 2.0
    import QtQuick.XmlListModel 2.0
    
    XmlListModel {
        id: xmlModel
        source: "http://www.mysite.com/feed.xml"
        query: "/rss/channel/item"
    
        XmlRole { name: "title"; query: "title/string()" }
        XmlRole { name: "pubDate"; query: "pubDate/string()" }
    }

`source`属性用于定义XML文档的位置，可以是本地资源或远程资源。

`query`属性值应该是一个有效的[XPath (XML Path Language) ](https://en.wikipedia.org/wiki/XPath "XPath (XML Path Language) ")选择器。除此之外，XPath用于从XML文档树中选择与给定路径表达式匹配的节点。

通过查询`/rss/channel/item`，我们选择了所有的`<item>`，该`<item>`是`<channel>`的子元素，而`<channel>`是文档的根元素`<rss>`的子元素。有关更多详细用法，请参阅[XPath使用示例](https://en.wikipedia.org/wiki/XPath#Usage_examples "XPath使用示例")。

角色是使用[XmlRoles](https://doc.qt.io/qt-5/qml-qtquick-xmllistmodel-xmlrole.html "XmlRoles")来定义的。注意，这些`<item>`元素包含了其他所有元素，因此我们需要一个查询来绑定特定数据。在示例中，我们查询了`<item>`元素的`<title>`和`<pubDate>`，然后使用`string()`函数来获取它们的值，然后将该值绑定到对应命名的角色`title`和`pubDate`。有关更多示例，请参见[XmlRole::query](https://doc.qt.io/qt-5/qml-qtquick-xmllistmodel-xmlrole.html#query-prop "XmlRole::query")。

在`ListView`中使用上面生成的模型是完全正常的。我们可以把委托绑定到模型的角色：

    ListView {
        width: 180; height: 300
        model: xmlModel
        delegate: Text { text: title + ": " + pubDate }
    }

# 2. ObjectModel

[ObjectModel](http://doc.qt.io/qt-5/qml-qtqml-models-objectmodel.html "ObjectModel")可以用于定义一组可视项目，这样我们就不必使用委托。

    ObjectModel {
        id: itemModel
        Text { color: "red"; text: "Hello " }
        Text { color: "green"; text: "World " }
        Text { color: "blue"; text: "again" }
    }
    
    ListView {
        anchors.fill: parent
        model: itemModel
        orientation: Qt.Horizontal
    }

# 3. DelegateModel

在某种意义上，[DelegateModel](https://doc.qt.io/qt-5/qml-qtquick-xmllistmodel-xmllistmodel.html "DelegateModel")与`ObjectModel`类似，它也将模型和委托封装在一起。此外，它被用于层次模型。注意，其实所有QML模型类型实际上都是一个列表。您可以通过对`QAbstractItemModel`进行子类化来创建表、树和层次模型。`DelegateModel`有一个`rootIndex`属性，它是一种`QModelIndex`类型。使用模型索引可以在树结构的子项和父项之间进行导航。我们后面会介绍它。

`DelegateModel`对于在多个视图中共享委托数据项或对项进行排序和筛选也很有用。

## 3.1 共享数据项

`DelegateModel`使用`Package`类型可以将具有共享上下文的委托提供给多个视图。`Package`中的所有项都可以通过`Package.name`附加属性分配一个名称。类似下面的示例，它具有两个不同的`Package.name`项目。实际内容`Text`是基于对包含奇数索引的`Package`委托索引或对包含偶数索引的`Package`委托索引。模型`myModel`仅包含具有`display`角色的列表元素。

    DelegateModel {
        id: visualModel
        delegate: Package {
            Item { id: odd; height: childrenRect.height; Package.name: "oddIndex" }
            Item { id: even; height: childrenRect.height; Package.name: "evenIndex" }
            Text {
                parent: (index % 2) ? even : odd
                text: display
            }
        }
        model: myModel
    }

这样就可以在不同的视图中使用程序的包名称。`DelegateModel`的属性`parts`选择了一个`DelegateModel`从指定的部分创建委托。

    ListView {
        height: parent.height/2
        width: parent.width
        model: visualModel.parts.oddIndex
    }
    
    ListView {
        y: parent.height/2
        height: parent.height/2
        width: parent.width
        model: visualModel.parts.evenIndex
    }
    

## 3.2 对数据项进行排序和过滤

您可以使用`DelegateModel`中的组对委托进行排序和筛选。默认情况下，每个委托都有一个组。也可以使用`DelegateModelGroup`类型定义其他组。它提供了创建和删除组中的项目或将项目移动到组内其他索引位置的方法。

使用组的第一步是向`DelegateModel`添加一个或多个组。在下面示例中有两个组，默认情况下，所有委托项都添加到`group1`组中。请注意，项目也会添加到默认`items`组中，因此任何委托都可能同时属于几个组。`DelegateModel`的`filterOnGroup`属性确定了我们只对`group1`中的委托感兴趣。

    DelegateModel {
        id: visualModel
        model: myModel // contains ListElements with the display role 
        filterOnGroup: "group1"
    
        groups: [
            DelegateModelGroup { name: "group1"; includeByDefault: true },
            DelegateModelGroup { name: "group2" }
         ]

在`DelegateModel`中定义的组都会向每个委托项添加两个附加属性。其中`DelegateModel.inGroupName`用于表示该项目是否属于该组，`DelegateModel.groupNameIndex`则保存了该组中项的索引。

附加属性在检查和更改委托所属的组或更改项目在该组中的位置时非常有用。下面的示例展示了如何通过按下鼠标来更改组：

    delegate: Text {
        id: item
        text: display
        MouseArea {
            anchors.fill: parent
            onClicked: {
                if (item.DelegateModel.inGroup1) {
                    item.DelegateModel.inGroup1 = false;
                    item.DelegateModel.inGroup2 = true;
                }
                else {
                    item.DelegateModel.inGroup1 = true;
                    item.DelegateModel.inGroup2 = false;
                }
          }
       }
    }

您也可以将组筛选器赋值为要显示的组，这样一次只能显示一个组。

    ListView {
        anchors.fill: parent
        model: visualModel
        focus: true
        Keys.onReturnPressed: {
            visualModel.filterOnGroup = (visualModel.filterOnGroup === "group1") ? "group2" : "group1";
        }
    }

## 3.3 本节涉及的参考资料

https://doc.qt.io/qt-5/qtquick-modelviewsdata-modelview.html#xml-model
http://doc.qt.io/qt-5/qml-qtquick-xmllistmodel-xmllistmodel.html
https://qmlbook.github.io/en/ch06/index.html#a-model-from-xml
http://doc.qt.io/qt-5/xquery-introduction.html
http://doc.qt.io/qt-5/qml-qtqml-models-objectmodel.html
http://doc.qt.io/qt-5/qml-qtqml-models-delegatemodel.html

[Source](https://materiaalit.github.io/qt-mooc/part4/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
