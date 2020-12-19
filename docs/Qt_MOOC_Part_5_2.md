# <center>从QML访问C++<center>

我们已经在上一节看到了使用上下文属性将C++对象暴露给QML的简单示例。在本节中，我们将通过在QML中使用C++对象的各种方式来讨论更多的细节，以及在C++端有哪些要求。

当使用`setContextProperty`公开`QObject`实例时，可以从QML访问对象的属性、信号、槽、用`Q_INVOKABLE`标记的方法和用`Q_ENUM`注册的枚举。让我们看一个例子。
    
    // simpleclass.h
    #include <QObject>
    #include <QColor>
    
    class SimpleClass : public QObject
    {
        Q_OBJECT
        Q_PROPERTY(QColor backgroundColor MEMBER m_backgroundColor NOTIFY backgroundColorChanged)
    
    public:
        explicit SimpleClass(QObject *parent = nullptr);
        Q_INVOKABLE void usefulMethod();
    
    signals:
        void backgroundColorChanged();
    
    public slots:
        void reactToColorChange();
    
    private:
        QColor m_backgroundColor;
    };
    
<br/>

    // simpleclass.cpp
    #include "simpleclass.h"
    
    SimpleClass::SimpleClass(QObject *parent) :
        QObject(parent),
        m_backgroundColor(QColor(Qt::blue))
    {
    }
    
    void SimpleClass::usefulMethod()
    {
        qDebug("I'm being useful :)");
    }
    
    void SimpleClass::reactToColorChange()
    {
        qDebug("Color changed!");
    }
    
<br/>
    
    // main.cpp
    #include "simpleclass.h"
    #include <QGuiApplication>
    #include <QQmlApplicationEngine>
    #include <QQmlContext>
    
    int main(int argc, char *argv[])
    {
        QGuiApplication app(argc, argv);
    
        QQmlApplicationEngine engine;
        QQmlContext* context(engine.rootContext());
        context->setContextProperty("simpleClass", new SimpleClass(&engine));
    
        engine.load(QUrl(QStringLiteral("qrc:/main.qml")));
    
        return app.exec();
    }

<br/>

    // main.qml
    import QtQuick 2.6
    import QtQuick.Window 2.2
    
    Window {
        visible: true
        width: 640
        height: 480
        title: qsTr("Exposed QObject")
    
        Rectangle {
            id: rectangle
            anchors.fill: parent
            focus: true
            color: simpleClass.backgroundColor
            Text {
                anchors.centerIn: parent
                font.pixelSize: 18
                text: "Hello World!"
            }
            MouseArea {
                anchors.fill: parent
                onClicked: {
                    simpleClass.backgroundColor = Qt.rgba(1, 0, 0, 1)
                }
            }
            Keys.onSpacePressed: simpleClass.usefulMethod()
        }
    
        Connections {
            target: simpleClass
            onBackgroundColorChanged: simpleClass.reactToColorChange()
        }
    }

这在上一节中已经很熟悉了。正如您所看到的，在使用`setContextProperty`公开对象之后，我们可以通过名称来访问`backgroundColor`属性，还可以访问`usefulMethod()`方法，因为我们使用`Q_INVOKABLE`宏声明了该方法。我们还可以从QML调用公共槽函数`reactToColorChange()`。

上面的示例没有枚举，不过它用法很简单。您可以像通常那样声明枚举，然后使用`Q_ENUM`宏在元对象系统中注册它。

    #include <QObject>
    
    class MyClass : public QObject
    {
        Q_OBJECT
    
    public:
        enum Status { Loading, Ready };
        Q_ENUM(Status)
        ...
    }

# 1. 数据模型

通常我们希望把来自C++的数据显示在QML视图中。我们已经在第四章中介绍过视图，但是还没有深入到C++端。在接下来的章节中，我们将学习如何创建我们自己的模型。不过这里先让我们看看如何用C++数据填充一个简单的`QStringList`，并通过将其暴露给QML上下文在QML中使用它。

    QStringList myList;
    myList.append("Dog");
    myList.append("Cat");
    myList.append("Mouse");
    myList.append("Dolphin");
    
    QQmlApplicationEngine engine;
    QQmlContext* context(engine.rootContext());
    context->setContextProperty("myModel", QVariant::fromValue(myList));

<br/>    

    ListView {
        width: 200; height: 200
        anchors.fill: parent
    
        model: myModel
        delegate: Rectangle {
            height: 30; width: 200
            Text {
                text: model.modelData
            }
        }
    }

[Source](https://materiaalit.github.io/qt-mooc/part5/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
