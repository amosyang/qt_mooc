# <center>创建新的QML类型<center>

到目前为止，我们已经讨论了如何将对象实例公开给QML上下文。有时我们还希望在QML中可以使用注册类本身。注册允许将类当作QML中的数据类型来使用。此外，注册还可以提供其他功能，比如允许在QML中将类用作可实例化的QML对象类型，或者允许在QML中导入和使用类的单例实例。

通常我们使用`Q_OBJECT`宏注册从`QObject`派生的类，也可以用`Q_GADGET`宏声明一个比`QObject`“更轻”的版本。在这些更轻的类中，我们可以访问它们的属性、枚举和可调用的方法，但不能使用信号槽系统，我们稍后会进行介绍。

# 1. 注册

## 1.1 注册实例化类型

要将`QObject`派生的类注册为可实例化的QML对象类型，请调用`qmlRegisterType()`将类作为QML类型注册到特定的类型名称空间中。然后，客户端可以导入该名称空间以使用该类型。

假设有一个带有`author`和`creationDate`属性的`Message`类：

    class Message : public QObject
    {
        Q_OBJECT
        Q_PROPERTY(QString author READ author WRITE setAuthor NOTIFY authorChanged)
        Q_PROPERTY(QDateTime creationDate READ creationDate WRITE setCreationDate NOTIFY creationDateChanged)
    public:
        // ...
    };

可以通过使用合适的类型名称空间和版本号调用`qmlRegisterType()`来注册此类型。比如使该类型在`com.mycompany.messaging`版本1.0的名称空间中可用：

    qmlRegisterType<Message>("com.mycompany.messaging", 1, 0, "Message");
    

然后在QML的对象声明中可以使用该类型，并且可以按以下示例读取和写入其属性：

    import com.mycompany.messaging 1.0
    
    Message {
        author: "Amelie"
        creationDate: new Date()
    }

## 1.2 注册非实例化类型

有时，可能需要向QML类型系统注册`QObject`派生类时不希望将其注册为可实例化类型。比如一个C++类：

*   是不应实例化的接口类型
*   是不需要暴露给QML的基类类型
*   声明一些能从QML访问的枚举，但自己不应该被实例化
*   一个应该通过单例实例提供给QML的类型，并且不应该从QML实例化

Qt QML模块提供了几种注册非实例化类型的方法：

*   `qmlRegisterType()`（不带参数）注册一个不可实例化且无法从QML引用的C++类型。这使引擎能够强制执行可从QML实例化的所有继承类型。
*   `qmlRegisterInterface()`使用特定的QML类型名称注册Qt的接口类型。类型不能从QML实例化，但可以通过其类型名引用。
*   `qmlRegisterUncreatableType()`注册一个C++类型，该类型不可实例化，但应可识别为QML类型系统的类型。如果类型的枚举或附加属性应该可以从QML访问，但是类型本身不应该是可实例化的，那么这很有用。
*   `qmlRegisterSingletonType()` 注册可以从QML导入的单例类型。

下面我们讨论一下单例类型。

`QObject`单例类型可以与任何其他`QObject`或实例化类型进行交互，但只有一个实例存在，而且必须通过类型名称而不是id进行引用。`QObject`单例类型的`Q_PROPERTY`可以用于属性绑定，`QObject`模块API的`Q_INVOKABLE`函数可以用于信号处理器表达式。这使得单例类型成为实现样式化或主题化的理想方式。

假设我们有一个用于主题的单例类型，它已注册到`MyThemeModule`1.0版本的名称空间中，其中`QObject`具有一个`QColor color`属性。然后我们可以简单地使用它：

    import MyThemeModule 1.0 as Theme
    
    Rectangle {
        color: Theme.color // binding.
    }

另一个例子是我们有一个只用于枚举的类。请注意，在这种情况下，我们可以使用前面提到的更轻量级的`Q_GADGET`宏（因为我们不需要信号和槽）：
    
    // modeclass.h
    #include <QObject>
    
    class ModeClass
    {
        Q_GADGET
    public:
        explicit ModeClass();
        enum Mode { Slow, Normal, Fast, UsainBolt };
        Q_ENUM(Mode)
    };    

注册：

    qmlRegisterUncreatableType<ModeClass>("com.mycompany.modes", 1, 0, "ModeClass", "Message");
    

# 2. 创建自定义QObject

让我们通过一个例子来看看如何创建一个自定义的`QObject`派生类然后在QML中使用：

    // cppperson.h
    class CppPerson : public QObject
    {
        Q_OBJECT
        Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)
        Q_PROPERTY(int shoeSize READ shoeSize WRITE setShoeSize NOTIFY shoeSizeChanged)
    public:
        CppPerson(QObject *parent = nullptr);
    
        QString name() const;
        void setName(const QString &name);
    
        int shoeSize() const;
        void setShoeSize(int size);
    
    signals:
        void nameChanged();
        void shoeSizeChanged();
    
    private:
        QString m_name;
        int m_shoeSize;
    };

<br/>    

    // cppperson.cpp
    #include "cppperson.h"
    
    CppPerson::CppPerson(QObject *parent) :
        QObject(parent), m_shoeSize(0)
    {
    }
    
    QString CppPerson::name() const
    {
        return m_name;
    }
    
    void CppPerson::setName(const QString &name)
    {
        if (m_name != name) {
            m_name = name;
            emit nameChanged();
        }
    }
    
    int CppPerson::shoeSize() const
    {
        return m_shoeSize;
    }
    
    void CppPerson::setShoeSize(int size)
    {
        if (m_shoeSize != size) {
            m_shoeSize = size;
            emit shoeSizeChanged();
        }
    }

<br/>    

    // main.cpp
    ...
    qmlRegisterType<CppPerson>("People", 1,0, "Person");
    ...

<br/>    

    // main.qml
    import QtQuick 2.9
    import QtQuick.Window 2.3
    import People 1.0
    
    Window {
        width: 640; height: 480
        visible: true
    
        Rectangle {
            anchors.fill: parent
    
            // Person is implemented in C++
            Person {
                id: person
                name: "Bob Jones"
                shoeSize: 12
                onNameChanged: {
                    console.log("New name: " + name)
                }
                onShoeSizeChanged: {
                    console.log("New shoe size: " + shoeSize)
                }
            }
    
            Column {
                anchors.fill:  parent
                spacing: 20
                Text {
                    font.bold: true
                    font.pixelSize: 26
                    text: "Person name: " + person.name
                }
                Text {
                    font.bold: true
                    font.pixelSize: 26
                    text: "Person shoe size: " + person.shoeSize
                }
            }
    
            MouseArea {
                anchors.fill: parent
                onClicked: {
                    person.name = "John Doe"
                    person.shoeSize = 9
                }
            }
        }
    }

C++端现在应该开始变得熟悉，我们使用`Q_PROPERTY`在头文件中声明了属性及其访问器和信号，并在`cppperson.cpp`文件中实现了getter和setter。（不要忘记在setter中发出`xxxChanged`信号！）

我们在QML里导入已注册的类型，然后就可以像使用其他QML类型一样使用它。属性绑定与通常一样是按名称进行的，信号的槽自动存在，遵循命名约定`onXxxChanged`。

# 3. QStandardItemModel

一个常见的需求是在C++中实现一个模型来存储数据，然后在QML中显示。更专业的用例通常要求我们编写自己的模型，例如在下一节中，我们将通过子类化`QAbstractTableModel`（继承自`QAbstractItemModel`）来实现我们自己的表格模型。Qt还提供了一个简单的通用模型来存储自定义数据，称为`QStandardItemModel`。
    
    // mymodel.h
    #include <QStandardItemModel>
    
    class MyModel : public QStandardItemModel
    {
        Q_OBJECT
    public:
        enum Roles {
            BrandRole = Qt::UserRole + 1,
            ModelRole
        };
    
        explicit MyModel(QObject *parent = nullptr);
    
        QHash<int, QByteArray> roleNames() const Q_DECL_OVERRIDE;
    
        Q_INVOKABLE void addCar(const QString &brand, const QString &model);
    };

<br/>    

    // mymodel.cpp
    #include "mymodel.h"
    
    MyModel::MyModel(QObject *parent) : QStandardItemModel(parent)
    {
    }
    
    QHash<int, QByteArray> MyModel::roleNames() const
    {
        QHash<int, QByteArray> mapping = QStandardItemModel::roleNames();
        mapping[BrandRole] = "brand";
        mapping[ModelRole] = "model";
        return mapping;
    }
    
    void MyModel::addCar(const QString &brand, const QString &model)
    {
        QStandardItem *item = new QStandardItem;
        item->setData(brand, BrandRole);
        item->setData(model, ModelRole);
        appendRow(item);
    }

这里要注意的最重要部分是重写的`roleNames()`函数，其中指定的角色被映射到数字(枚举)。这使得我们能够使用名称“brand”和“model”来引用QML中的数据。

另外，不要忘记用`Q_INVOKABLE`声明`addCar`方法，以允许它可以在QML中使用。

    // main.cpp
    qmlRegisterType<MyModel>("org.mooc.cars", 1, 0, "MyModel");
    
<br/>
    
    // main.qml
    import QtQuick 2.9
    import QtQuick.Window 2.2
    import org.mooc.cars 1.0
    
    Window {
        visible: true
        width: 640
        height: 480
        title: qsTr("Cars")
    
        MyModel {
            id: cars
        }
    
        ListView {
            anchors.fill: parent
            model: cars
            delegate: Text {
                text: "The brand is " + brand + " and the model is " + model
            }
        }
    
        // Some way to populate the model, cars.addCar("brand", "model")
        ...
    }

# 4. QML插件

导入自定义QML组件时，它会首先加载到内存中。为了缩短启动时间，您可以将组件改为插件，在这种情况下，一旦创建了对象，它就会被动态加载。

要编写QML扩展插件，我们需要：

*   子类化`QQmlExtensionPlugin`
*   为插件编写一个项目文件
*   创建一个`qmldir`文件描述此插件

假设我们有一个C++类`TimeModel`，需要将其作为新的QML类型使用。它提供当前时间`hour`和`minute`属性。

    class TimeModel : public QObject
    {
        Q_OBJECT
        Q_PROPERTY(int hour READ hour NOTIFY timeChanged)
        Q_PROPERTY(int minute READ minute NOTIFY timeChanged)
        ...

现在，我们创建一个名为`QExampleQmlPlugin`的类，它继承至`QmlExtensionPlugin`：

    class QExampleQmlPlugin : public QQmlExtensionPlugin
    {
        Q_OBJECT
        Q_PLUGIN_METADATA(IID QQmlExtensionInterface_iid)
    
    public:
        void registerTypes(const char *uri) override
        {
            Q_ASSERT(uri == QLatin1String("TimeExample"));
            qmlRegisterType<TimeModel>(uri, 1, 0, "Time");
        }
    };

*   我们使用`Q_PLUGIN_METADATA()`宏将插件注册到具有唯一标识符的元对象系统。
*   重写了`registerTypes()`方法，用`qmlRegisterType`注册`TimeModel`类型。
*   这里的`Q_ASSERT`不是必须的，但是我们可以使用它来确保使用此插件的任何QML组件都能正确导入类型名称空间。

接下来，我们需要一个`.pro`项目文件：

    TEMPLATE = lib
    CONFIG += qt plugin
    QT += qml
    
    DESTDIR = imports/TimeExample
    TARGET = qmlqtimeexampleplugin
    SOURCES += qexampleqmlplugin.cpp
    

它将项目定义为插件库，指定构建目录，并注册插件目标名称。

最后，我们需要一个`qmldir`文件来描述插件：

    module TimeExample
    plugin qmlqtimeexampleplugin

[Source](https://materiaalit.github.io/qt-mooc/part5/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
