# <center>值对象与QObject<center>

# 1. 元对象系统

Qt的元对象系统为对象间通信、运行时类型信息(RTTI)和动态属性系统提供了信号和槽机制。信号和槽是Qt中最重要的概念之一，我们将在下一章进行讨论。

元对象系统通过三种机制实现：

*   `QObject` 是元对象系统中所有对象继承的基类。
*   `Q_OBJECT`在类定义中声明时，该宏用于启用元对象功能。
*   元对象编译器(`moc`)将读取带有已声明的`Q_OBJECT`宏的类定义，并生成元对象代码。

# 2. 值类型和标识类型

值类型可以被复制和赋值。许多Qt值类型，如`QString`和Qt容器，也使用隐式共享(写时复制)。隐式共享类在作为参数传递时既安全又高效，因为只有指向数据的指针才会被传递，而且只有当函数写入数据时数据才会被复制。自定义值类型可以通过使用`Q_DECLARE_METATYPE`宏让元对象系统知道，这使得它们可以存储在`QVariant`中。这在读取属性时非常有用。如何做到这一点，将在本章的后面讨论，并会讨论属性系统。

标识类型派生于`QObject`。它使用元对象系统扩展了C++的许多动态特性。`QObject`被设计成没有复制构造函数或赋值运算符。实际上是在私有部分中声明了`Q_DISABLE_COPY()`宏。实际上，所有从`QObject`派生的Qt类(直接或间接)都使用这个宏将它们的复制构造函数和赋值操作符声明为私有了。导致的结果是您应该使用指向`QObject`（或`QObject`子类）的指针，否则您可能会试图使用`QObject`子类作为值。例如，如果没有复制构造函数，就不能使用`QObject`的子类作为要存储在某个容器类中的值。您必须存储指针。

# 3. Qt对象模型和QObject类

`QObject`类是所有Qt对象的基类。它是Qt对象模型的核心。该模型的核心特性是一种非常强大的对象通信机制，称为信号和槽。信号和槽系统将在下一章详细讨论。

`QObject`将自己组织在对象树中。当您创建一个`QObject`以其他对象作为父对象的对象时，该对象会自动将其自身添加到父对象的`children()`列表中。父对象拥有对象的所有权，也就是说，它将在其析构函数中自动删除其子级。您可以使用`findChild()`或`findChildren()`根据名称和可选类型查找对象。父子关系将在后面讨论。

每个对象都有一个`objectName()`，它的类名可以通过对应的`metaObject()`找到(参考`QMetaObject::className()`)。通过使用`inherits()`函数，可以确定对象的类是否继承`QObject`继承层次结构中的另一个类。

当一个对象被删除时，它会发出一个`destroyed()`信号。您可以捕获这个信号，以避免对`QObject`的悬空引用。

最后重要的是，`QObject`在Qt中提供基本的定时器支持，有关计时器的高级支持，请参阅`QTimer`。

# 3.1 Q_OBJECT宏

`Q_OBJECT`宏必须出现在类定义的私有部分中，该类定义声明自己的信号和槽，或者使用Qt元对象系统提供的其他服务。

`moc`工具会读取C++头文件。如果它发现一个或多个包含`Q_OBJECT`宏的类声明，它将生成一个C++源文件，其中包含这些类的元对象代码。这个元对象代码实现了运行时特性所需的底层功能。当使用`qmake`时，你不需要手动运行`moc`工具，它会自动完成。

示例如下：

    #include <QObject>
    
    class Counter : public QObject
    {
        Q_OBJECT
    
    public:
        Counter() { m_value = 0; }
        int value() const { return m_value; }
    
    public slots:
        void setValue(int value);
    
    signals:
        void valueChanged(int newValue);
    
    private:
        int m_value;
    };
    

这个宏要求类是`QObject`的子类。您可以使用`Q_GADGET`而不是`Q_OBJECT`来启用元对象系统对非`QObject`子类中的枚举的支持。`Q_GADGET`宏是`Q_OBJECT`宏的简易版本，用于那些不从`QObject`继承但仍然希望使用`QMetaObject`提供的一些反射功能的类。就像`Q_OBJECT`宏一样，它必须出现在类定义的私有部分中。

注意，`Q_OBJECT`宏对于任何实现信号、槽或属性的对象都是必须的。我们强烈建议在`QObject`的所有子类中使用这个宏，不管它们是否实际使用信号、槽和属性，因为如果不这样做，可能会导致某些函数表现出奇怪的行为。

# 3.2 国际化（I18n）

所有`QObject`子类都支持Qt的翻译功能，从而可以将应用程序的用户界面翻译成不同的语言。

为了使用户可见的文本可翻译，必须将其包装在`tr()`函数中。[编写翻译的源代码](https://doc.qt.io/qt-5/i18n-source-translation.html "编写翻译的源代码")文档中对此进行了详细说明。我们不会在这门课上涉及国际化，但是有必要知道它的存在。

# 4. 属性系统

Qt提供了一个复杂的[属性系统](http://doc.qt.io/qt-5/properties.html "属性系统")，类似于一些编译器供应商提供的属性系统。但是，作为与编译器和平台无关的库，Qt不依赖于非标准的编译器功能，例如`__property`或`[property]`。Qt的解决方案适用于Qt支持的每个平台上的任何标准C++编译器。它是基于元对象系统的，也提供了信号和槽机制。

# 4.1 声明属性的要求

要声明属性，请在继承`QObject`的类中使用`Q_PROPERTY()`宏。下面的示例展示了如何使用`MEMBER`关键字将成员变量导出为Qt属性。注意，`NOTIFY`必须指定一个信号以允许QML属性绑定。我们将在第3部分中进一步讨论QML！

    Q_PROPERTY(QColor color MEMBER m_color NOTIFY colorChanged)
    Q_PROPERTY(qreal spacing MEMBER m_spacing NOTIFY spacingChanged)
    Q_PROPERTY(QString text MEMBER m_text NOTIFY textChanged)
    
    ...
    
    signals:
        void colorChanged();
        void spacingChanged();
        void textChanged(const QString &newText);
    
    private:
        QColor  m_color;
        qreal   m_spacing;
        QString m_text;
    

属性的行为类似于类的数据成员，但是它具有可以通过[元对象系统](http://doc.qt.io/qt-5/metaobjects.html "元对象系统")访问的附加特性。

属性声明语法有多个关键字来指定所声明属性的行为。以下是最相关的几个:

*   `READ`\-用于读取属性值。理想情况下，将const用于此函数，并且它必须返回属性的类型或对该类型的const引用。例如，`QWidget::focus`是具有READ函数的只读属性`QWidget::hasFocus()`。如果没有指定`MEMBER`变量，则该关键字必须存在。
*   `WRITE`\-用于设置属性值。它必须返回void，并且必须正好接受一个参数，该参数可以是属性的类型，也可以是对该类型的指针或引用。例如，`QWidget::enabled`具有`WRITE`函数`QWidget::setEnabled()`。只读属性不需要`WRITE`函数。例如，`QWidget::focus`没有`WRITE`函数。
*   `MEMBER`\-如果没有指定`READ`访问器函数，则该关键字必须存在。这使得给定的成员变量可以读和写，而不需要创建`READ`和`WRITE`访问器函数。如果需要控制变量访问，仍然可以在`MEMBER`变量关联之外使用`READ`或`WRITE`访问器函数(但不能同时使用)。
*   `RESET`\- 可选的。用于将属性重置为其上下文特定的默认值。例如，`QWidget::cursor`具有典型的`READ`和`WRITE`函数`QWidget::cursor()`和`QWidget::setCursor()`，并且还具有`RESET`函数`QWidget::unsetCursor()`，因为没有调用`QWidget::setCursor()`可能意味着重置上下文特定的游标。`RESET`函数必须返回`void`并且不接受任何参数。
*   `NOTIFY`\- 可选的。如果定义，则应在该类中指定一个现有信号，只要该属性的值发生更改，该信号就会发出。`MEMBER`变量的`NOTIFY`信号必须接收零个或一个参数，该参数必须与属性具有相同的类型。该参数将接受该属性的新值。例如，`NOTIFY`应该仅在属性确实发生更改时才发出信号，以避免不必要地在QML中重新绑定。Qt会在没有显式设置的`MEMBER`属性需要时自动发出该信号。
*   `USER`\-属性表明是否将该属性指定为类面向用户的属性或用户可编辑的属性。通常，每个类只有一个`USER`属性(默认为false)。例如，`QAbstractButton::checked`是（可选）按钮的用户可编辑属性。

属性类型可以是`QVariant`支持的任何类型，也可以是用户定义的类型。在此示例中，类`QDate`被认为是用户定义的类型。

    Q_PROPERTY(QDate date READ getDate WRITE setDate)
    

因为`QDate`是用户定义的，所以必须在属性声明中包含`<QDate>`头文件。

由于历史原因，`QMap`和`QList`作为属性类型与`QVariantMap`和`QVariantList`是等价的。

# 4.2 属性的读写

可以使用通用函数`QObject::property()`和`QObject::setProperty()`来读写属性，除了属性的名称外，不需要知道所属类的任何信息。在下面的代码片段中，对`QAbstractButton::setDown()`的调用和对`QObject::setProperty()`的调用都将属性设置为"down"。

    QPushButton *button = new QPushButton;
    QObject *object = button;
    button->setDown(true);
    object->setProperty("down", true);
    
通过属性的`WRITE`访问器访问属性是两种方法中较好的一种，因为它在编译时更快并提供更好的诊断，但是以这种方式设置属性要求您在编译时了解该类。通过名称访问属性可以访问编译时不知道的类。您可以在运行时通过查询类的`QObject`、`QMetaObject`和`QMetaProperties`来发现类的属性。

    QObject *object = ...
    const QMetaObject *metaobject = object->metaObject();
    int count = metaobject->propertyCount();
    for (int i = 0; i < count; ++i) {
        QMetaProperty metaproperty = metaobject->property(i);
        const char *name = metaproperty.name();
        QVariant value = object->property(name);
        ...
    }
    
在以上代码段中，`QMetaObject::property()`用于获取有关某个未知类中定义的每个属性的元数据。从元数据中获取属性名称，并将其传递给`QObject::property()`以获取当前对象中属性的值。

# 4.3 示例

假设我们有一个类`MyClass`，它派生自`QObject`，并在其私有部分中使用`Q_OBJECT`宏。我们想要在`MyClass`中声明一个属性以跟踪`priority`值。该属性的名称是`priority`，其类型为枚举类型`Priority`，该枚举类型在`MyClass`中定义。

我们在类的私有部分使用`Q_PROPERTY()`宏声明该属性。`READ`函数名为`priority`，`WRITE`函数名为`setPriority`。

枚举类型必须使用`Q_ENUM()`宏向元对象系统注册。该宏向元对象系统中注册一个枚举类型。这将启用有用的功能，例如，如果在`QVariant`中使用，您可以将它们转换为字符串。同样，将它们传递给`QDebug`将打印出其名称。它必须放在具有`Q_OBJECT`或`Q_GADGET`宏的类中的枚举声明之后。

注册枚举类型使枚举名称可以在`QObject::setProperty()`调用中使用。我们还必须为`READ`和`WRITE`函数提供我们自己的声明。

    class MyClass : public QObject
    {
        Q_OBJECT
        Q_PROPERTY(Priority priority READ priority WRITE setPriority NOTIFY priorityChanged)
    
    public:
        MyClass(QObject *parent = 0);
        ~MyClass();
    
        enum Priority { High, Low, VeryHigh, VeryLow };
        Q_ENUM(Priority)
    
        void setPriority(Priority priority);
        Priority priority() const;
    
        signals:
            void priorityChanged(Priority);
        
        private:
            Priority m_priority;
    };
    

给定一个指向`MyClass`实例的指针或指向一个`MyClass`实例的`QObject`的指针，我们有两种方法来设置它的`priority`属性：

    MyClass *myinstance = new MyClass;
    QObject *object = myinstance;
    
    myinstance->setPriority(MyClass::VeryHigh);
    object->setProperty("priority", "VeryHigh");
    

在本例中，作为属性类型的枚举类型在`MyClass`中声明，并使用`Q_ENUM()`宏向元对象系统注册。这使得枚举值可以作为字符串使用，就像在`setProperty()`调用中一样。如果在另一个类中声明了枚举类型，则将需要使用其完全限定的名称（即`OtherClass::Priority`），而其他类也必须继承`QObject`并使用`Q_ENUM()`宏在那里注册枚举类型。

还有一个类似的宏`Q_FLAG()`。与`Q_ENUM()`类似，它注册了一个枚举类型，但它将该类型标记为一组标志，即可以进行“或”运算。一个I/O类可能有枚举值`Read`和`Write`，然后`QObject::setProperty()`可以接收`Read|Write`。应该使用`Q_FLAG()`来注册此枚举类型。

# 4.4 动态属性

还可以使用`QObject::setProperty()`在运行时向类的实例添加新属性。当使用名称和值调用它时，如果`QObject`中存在具有给定名称的属性，并且给定值与属性的类型兼容，则将该值存储在属性中，并返回`true`。如果该值与属性的类型不兼容，则不会更改该属性，并返回`false`。但是，如果具有给定名称的属性在`QObject`中不存在(即如果它没有使用`Q_PROPERTY()`声明)，一个具有给定名称和值的新属性将自动添加到`QObject`中，但仍然返回`false`。这意味着不能使用`false`的返回值来确定是否实际设置了某个特定的属性，除非您预先知道该属性已经存在于`QObject`中。

请注意，动态属性是按实例添加的，也就是说它们被添加到`QObject`，而不是`QMetaObject`。通过将属性名和无效的`QVariant`值传递给`QObject::setProperty()`，可以从实例中删除属性。`QVariant`的默认构造函数构造了一个无效的`QVariant`。

可以使用`QObject::property()`查询动态属性，就像在编译时使用`Q_PROPERTY()`声明的属性一样。

# 4.5 属性和自定义类型

属性使用的自定义值类型需要使用`Q_DECLARE_METATYPE()`宏注册，以便它们的值可以存储在`QVariant`对象中。这使得它们既适用于在类定义中使用`Q_PROPERTY()`宏声明的静态属性，也适用于在运行时创建的动态属性。

通过将`Q_DECLARE_METATYPE()`宏放在自定义类的头文件中来完成声明。例如：

    #include <QMetaType>
    
    class YourCustomClass
    {
    public:
        YourCustomClass();
    };
    
    Q_DECLARE_METATYPE(YourCustomClass)
    

# 4.6 向类添加其他信息

连接到属性系统的是一个附加的宏`Q_CLASSINFO()`，它可以用来给类的元对象附加额外的键值对，例如:

`Q_CLASSINFO("Version", "3.0.0")`

像其他元数据一样，类信息可以在运行时通过元对象访问，详细信息请参见`QMetaObject::classInfo()`。

在本练习中，您将熟悉访问QObject的属性。您可以在`reflection.cpp`中找到相关说明。

提示：`QVariant`将很有用。

这个练习有点麻烦。您将创建一个自定义类`Student`，并实现`StudentRegistry`的功能。您可以在`studentregistry.cpp`中找到相关规定。

[Source](https://materiaalit.github.io/qt-mooc/part2/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
