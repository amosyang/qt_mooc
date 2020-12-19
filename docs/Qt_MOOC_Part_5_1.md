# <center>QML上下文属性和对象<center>

我们在第五章将学习如何集成C++和QML。大多数情况下，这意味着从QML访问C++，它是我们要介绍的重点。不过您也可以从C++访问QML对象，但通常这不是您想要做的，可能出于测试的目的除外，所以我们不在这里深入讨论。

C++和QML之间的所有通信都是通过Qt元对象系统进行的。

*   可以使用信号槽连接
*   可以访问和修改属性
*   QML可以调用C++中被`Q_INVOKABLE`标记的槽和方法
*   QML可以访问被`Q_ENUM`注册的枚举

# 1. QML上下文

C++对象和值可以使用上下文属性和上下文对象直接嵌入到加载的QML对象的上下文中。这是通过Qt QML模块提供的`QQmlContext`类实现的，它将数据公开给QML组件的上下文，允许将数据从C++注入到QML中。

每个`QQmlContext`都包含一组不同于其`QObject`属性的属性，这些属性允许根据名称显式地将数据绑定到上下文中。

上下文形成了一个层次结构，它的根是QML引擎的根上下文。根上下文由`QQmlEngine`自动创建。应该将引擎实例化的所有QML组件实例可用的数据放在根上下文中。把仅对组件实例的子集可用的其他数据添加到根上下文的父级子上下文中。子上下文继承父上下文的上下文属性。如果子上下文设置了父上下文中已经存在的上下文属性，则新的上下文属性将覆盖父上下文的属性。

## 1.1 上下文属性

您可以使用`setContextProperty`方法公开上下文属性。它可以同时公开`QVariant`支持的值类型和指向`QObject`的指针。前者仅公开您设置为可通过属性名称使用的值，而后者则公开对象本身。

例如：

    // main.cpp
    #include <QGuiApplication>
    #include <QQmlApplicationEngine>
    #include <QQmlContext>
    #include <QColor>
    
    int main(int argc, char *argv[])
    {
        QGuiApplication app(argc, argv);
        QQmlApplicationEngine engine;
    
        QQmlContext* context = engine.rootContext();
        context->setContextProperty("myBackgroundColor", QColor(Qt::yellow));
        context->setContextProperty("myText", "I'm text from C++!");
        
        context->setContextProperty("application", &app);
    
        engine.load(QUrl(QStringLiteral("qrc:/main.qml")));
        if (engine.rootObjects().isEmpty())
            return -1;
        
        return app.exec();
    }

<br/>

    // main.qml
    import QtQuick 2.9
    import QtQuick.Window 2.3
    
    Window {
        width: 640; height: 480; visible: true
    
        Rectangle {
            objectName: "rectangle"
            width: 200; height: 100
            color: myBackgroundColor
    
            Text {
                id: textField
                anchors.centerIn: parent
                font.pixelSize: 18
                text: myText
            }
    
            MouseArea {
                anchors.fill: parent
                onClicked: {
                    application.quit();
                }
            }
        }
    }

这里我们公开了名为“myBackgroundColor”和“myText”的值，然后在QML中通过属性名来获取这些值。

我们还向QML公开了“application”，从而允许我们退出该应用程序。请注意，这只是为了举例！通常我们会使用`Qt.quit()`来退出应用程序，这里只是演示如何从QML调用槽函数。在下一节，我们将讨论更多关于在QML中公开和使用C++对象的内容。

## 1.2 上下文对象

为了简化绑定和维护更大的数据集，可以使用`setContextObject`公开对象的所有属性。上下文对象的所有属性都可以在上下文中按名称使用，就像它们都是用`setContextProperty`单独设置的一样。

注意，这与使用`setContextProperty`公开对象不同。当您使用`setContextProperty`公开一个对象时，该对象本身在QML中是可用的，就像前面示例中的“application”一样。如前所述，使用`setContextObject`公开对象时，只有上下文的属性可用。

还需注意，所有用`setContextProperty`显式添加的属性优先于上下文对象的属性。

例如：

    // myclass.h
    #include <QObject>
    #include <QColor>
    class MyClass : public QObject
    {
        Q_OBJECT
        Q_PROPERTY(QColor myColor MEMBER m_color NOTIFY colorChanged)
        Q_PROPERTY(QString myText MEMBER m_text NOTIFY textChanged)
    
    public:
        explicit MyClass(QObject* parent = nullptr);
    
    signals:
        void colorChanged();
        void textChanged(const QString &newText);
    
    private:
        QColor m_color;
        QString m_text;
    };
    
<br/>

    // main.cpp
    context->setContextObject(&myClass);
    

[Source](https://materiaalit.github.io/qt-mooc/part5/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
