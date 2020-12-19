# <center>对象通信：信号和槽<center>

几乎所有的UI工具包都有一种机制来检测用户操作，并对该操作做出响应。其中一些使用回调，另一些使用监听器，但基本上，所有这些都是受观察者模式的启发。

观察者模式用于观察对象想要通知其他观察者对象状态变化的情况。下面是一些具体的例子:

*   用户单击了一个按钮，应该会显示一个菜单。
*   一个Web页面刚刚加载完毕，一个进程应该从这个加载的页面中提取一些信息。
*   一个用户正在滚动一个项目列表(例如在一个app store)，并到达了终点，所以其他项目应该被加载。

观察者模式在GUI应用程序中随处可见，并且常常导致一些样板代码。创建Qt的初衷是删除这些样板代码，并提供一种漂亮而干净的语法，这就是信号和槽机制。

信号和槽是Qt和内部对象通信的关键。它们在某种意义上可以与回调相比较，但区别在于它们是类型安全的，而回调通常不是。我们在开始之前提到的一个好处是，信号和槽允许您构建多对多连接，通常情况下，如果使用多个虚函数，则虚方法是一对一或一对多的。

我们在上一章讨论了`Q_OBJECT`宏，它也会在本主题中再次出现。

# 1. 事件简介

在讲解信号和槽之前，让我们简要介绍一下事件。事件在事件循环中执行。这不是Qt特有的，可以证明，您使用的大多数应用程序都在等待输入事件，这花费了大部分时间，不管输入是来自用户、网络还是其他地方。可以有多个事件循环，例如每个线程都有一个。Qt支持使用[事件处理器](http://doc.qt.io/qt-5/eventsandfilters.html#event-handlers "事件处理器")，但是通常您会希望使用信号和槽系统。我们在这里引入事件的原因是，您应该了解事件循环的概念，因为它与信号和槽相关。本课程我们只介绍单线程的应用程序，但是当您[跨线程发送信号](http://doc.qt.io/qt-5/threads-qobject.html#signals-and-slots-across-threads "跨线程发送信号")时应该记住，该槽可能不会立即执行，而是可能被放置在接收线程的事件循环中，以等待控制权交给该线程。

# 2. 性能

与回调相比，信号和槽稍微慢一些，这是因为它们提供了更高的灵活性，但对于实际应用程序来说，这种差异是微不足道的。一般来说，发送一个信号连接到某些槽，比直接调用非虚函数要慢大约10倍。这是寻找连接对象、安全地遍历所有连接(即检查后续接收方在发射过程中没有被销毁)以及以通用方式排列参数所需的开销。

虽然相对于10个非虚函数的调用听起来很耗时，但是它比任何新增操作或删除操作的开销要小得多。一旦您在后台执行一个需要新建或删除的字符串、向量或列表操作，信号和槽开销只占整个函数调用开销的很小一部分。在槽中执行系统调用时也是如此，或间接调用超过十个函数。信号和槽机制的简单性和灵活性是值得的，用户甚至不会注意到这些开销。

# 3. 信号

当对象的内部状态以某种可能引起对象客户端或所有者兴趣的方式发生变化时，对象就会发出信号。信号是公有函数，可以从任何地方发出，但是我们建议只从定义信号的类及其子类发出信号。

要定义信号，请将其放在`signals:`类定义的块中：

    ...
    signals:
        void valueChanged(int newValue);
    ...
    

要发出信号，请使用`emit`关键字。这个关键字是纯语法的，但有助于将其与正常函数调用区分开来。

    void Counter::setValue(int value)
    {
        if (value != m_value) {
            m_value = value;
            emit valueChanged(value);
        }
    }
    

当一个信号发出时，连接到它的槽通常会立即执行，就像一个普通的函数调用一样。当发生这种情况时，信号和槽机制完全独立于任何GUI事件循环。`emit`语句之后的代码将在所有槽都返回之后执行。不过如果您使用排队连接，情况会略有不同。在这种情况下，`emit`关键字后面的代码将立即继续，槽将稍后执行。

下面是一些`QPushButton`类的信号示例:

*   clicked
*   pressed
*   released

如您所见，它们的名称非常明确。当用户点击(按下然后释放)、按下或释放按钮时，就会发送这些信号。

这些信号是由`moc`(元对象编译器)自动生成的，不能在`.cpp`文件中实现。它们永远不能有返回类型(只能使用void)。

开发经验表明，如果信号和槽不使用特殊类型，它们将具有更高的可重用性。如果`QScrollBar::valueChanged()`要使用一个特殊类型，比如假设的`QScrollBar::Range`，那么它只能连接到专门为QScrollBar设计的槽上。将不同的输入widgets连接在一起是不可能的。

# 4. 槽

当一个连接到槽的信号被发射时，该槽被调用。槽是普通的C++函数，可以正常调用。它们唯一的特点是信号可以与它们相连。

以下是来自不同类的一些槽:

*   `QApplication::quit`
*   `QWidget::setEnabled`
*   `QPushButton::setText`

如果将多个槽连接到同一个信号，则在发出信号时，将按照**连接的顺序**依次执行槽。

由于槽是普通的成员函数，所以在直接调用时它们也遵循普通的C++规则。但是，作为槽函数，任何组件都可以通过信号槽连接调用它们，而不管其访问级别如何。这意味着从任意类的实例发出的信号可能导致在不相关类的实例中调用私有槽。

# 5. 定义信号和槽

如前一章所述，所有使用信号和槽机制的类都需要在类定义的私有部分中定义`Q_OBJECT`宏。下面是实现信号和槽的类的头文件示例。

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
    

# 6. 连接信号和槽

要将信号连接到槽函数，请使用`QObject::connect()`。有几种连接信号和槽的方法。第一种是使用函数指针：

    connect(sender, &QObject::destroyed, this, &MyObject::objectDestroyed);
    

`QObject::connect()`与函数指针一起使用有几个优点。首先，它允许编译器检查信号的参数是否与槽的参数兼容。如果需要，还可以由编译器隐式转换参数。

您还可以连接到普通函数或C++11的lambda表达式：

    connect(sender, &QObject::destroyed, [=](){ this->m_objects.remove(sender); });
    

将信号连接到槽的传统方法是使用`QObject::connect()`和`SIGNAL()`/`SLOT()`宏。之所以在这里介绍它是因为它仍被广泛使用，但是通常，您应该使用前面介绍的一种较新的连接类型。如果参数具有默认值，则关于是否在`SIGNAL()`和`SLOT()`宏中包含参数的规则是，传递给`SIGNAL()`宏的签名不得少于传递给`SLOT()`宏的签名。

所以下面这些都会起作用：

    connect(sender, SIGNAL(destroyed(QObject*)), this, SLOT(objectDestroyed(Qbject*)));
    connect(sender, SIGNAL(destroyed(QObject*)), this, SLOT(objectDestroyed()));
    connect(sender, SIGNAL(destroyed()), this, SLOT(objectDestroyed()));
    

但这是行不通的：

    connect(sender, SIGNAL(destroyed()), this, SLOT(objectDestroyed(QObject*)));
    

...因为该槽函数期望的一个没有`QObject`的信号不会发送。该连接将报告运行时错误。请注意，使用此`QObject::connect()`重载时，编译器不会检查`signal`和`slot`参数。

在本练习中，您将创建两个类来练习定义信号和槽。您可以在`main.cpp`中找到相关说明。

# 7. 第三方库

您可能熟悉其他的信号槽机制，比如[Boost.Signals2库](https://www.boost.org/doc/libs/1_63_0/doc/html/signals2.html "Boost.Signals2库")。可以将Qt与第三方信号/槽机制一起使用，甚至可以在同一个项目中同时使用这两种机制。将以下定义添加到您的工程(`.pro`)文件中：

    CONFIG += no_keywords
    

它告诉Qt不要定义`moc`关键字`signals`、`slots`和`emit`，因为这些名称将被第三方库使用，例如`Boost`。然后，如果要继续在`no_keywords`环境中使用Qt信号和槽，只需将代码中的Qt`moc`关键字的替换为对应的Qt宏`Q_SIGNALS`(或`Q_SIGNAL`)、`Q_SLOTS`(或`Q_SLOT`)和`Q_EMIT`。

# 8. 对象通信中的事件

虽然在对象通信中通常首选使用信号和槽，但在某些情况下，所需的功能更容易用[事件](http://doc.qt.io/qt-5/eventsandfilters.html "事件")处理。例如，如果我们想发出多个信号，我们可以子类化相应的QObject并重新实现事件处理器。事件处理器可以在没有将信号连接到槽的情况下发出信号，槽随后会发出几个信号。

Qt是一个基于事件的系统。当调用`QCoreApplication::exec()`时，GUI线程进入事件循环。`QCoreApplication`可以处理GUI线程中的每个事件并将事件转发给`QObject`。接收者`QObject`可以处理或忽略相应的事件。

事件可以是自发的，也可以是合成的。自发事件在应用程序进程之外创建，例如由窗口管理器，并发送到应用程序。对于GUI事件，平台抽象插件(QPA)接收事件并将其转换为Qt事件类型。Qt事件是值类型，派生自`QEvent`，它为每个事件提供了枚举类型。如果我们想修改计时器事件处理，可以通过重新实现`QObject::event()`函数来完成，如下所示。

首先，先检查事件类型。如果事件类型是`QEvent::Timer`，我们就把`QEvent`转化为具体数据类型`QTimerEvent`。对于计时器，它有一个`timerId`。如果事件进一步传播到下一个接收对象（如果存在），则该函数返回一个布尔值以通知事件系统。所有未在此函数中处理的事件均由其基类处理。

    bool QObjectSubclass::event(QEvent *event)
    {
       
       if (event->type() == QEvent::Timer) {
           QTimerEvent *timerEvent = static_cast<QTimerEvent *>(event);
           if (timerEvent->timerId()) {
               
           }
       }
       
       return QObject::event(event);
    }
    
通常，我们不需要重新实现`event()`函数，而是需要使用一些特定于事件的处理器函数(请参阅:[事件处理器](http://doc.qt.io/qt-5/eventsandfilters.html#event-handlers "事件处理器"))。例如，可以在`void CustomObject::timerEvent(QTimerEvent *event)`中处理计时器事件。注意，这些函数不返回布尔值，但是它们通过`QEvent::accept()`和`QEvent::ignore()`来接受或忽略事件，以告知事件系统是否应该进一步传播该事件。。

    void QObjectSubclass::timerEvent(QTimerEvent *event)
    {
       
       if (event->timerId() == m_timerId) {
             
       }
       return QObject::timerEvent(event);
    }
    

# 8.1 事件过滤器

如果需要在几个不同的类中以相同的方式处理相同的事件，那么使用事件过滤器比子类化许多类型更容易。事件过滤器`QObject::eventFilter(QObject *watched, QEvent *event)`是`QObject`的成员函数，会在实际的事件处理函数之前调用。与`QObject::event()`函数类似，布尔返回值告诉我们事件是被过滤掉了(true)还是应该进一步传播(false)。实际上只有安装了`void QObject::installEventFilter(QObject *filterObject)`事件过滤器才会被调用。

事件过滤器有两种：应用程序范围的和对象本地的事件过滤器。唯一的区别是事件过滤器被安装到哪个对象。如果将它安装到`QCoreApplication`对象，则主线程中的所有事件都将通知事件过滤器。如果将其安装到其他`QObject`子类，则仅发送到该对象的事件过滤器。

应用程序范围的事件过滤器对于调试检查非常有用，例如，窗口管理器将预期的事件提供给Qt应用程序。通常，要避免应用程序范围的事件过滤器，因为为应用程序中的每个事件调用额外的函数可能会影响事件的处理性能。

大多数情况下，在事件处理程序中处理事件就足够了，但是正如上面所看到的，我们可以在传播的早期捕获事件。例如，对于触摸事件，没有特定于触摸的事件处理程序，因此它们必须在`QObject::event()`函数中处理。如果您希望对几种不同类型有类似的事件处理，事件过滤器很有用。

![](https://materiaalit.github.io/qt-mooc/img/part-2/event-propagation-c0352e5e.png)

# 8.2 自定义事件

有时我们没有合适的Qt事件类型，比如通知特定操作。在这些情况下，可以创建自定义事件。这可以很容易地从`QEvent`派生。每个自定义的事件都包含特定于事件的数据，因此需要添加成员数据并实现访问器函数来获取和设置数据。最后，事件必须被Qt事件系统识别，因此事件需要一个唯一的事件类型。您可以扩展现有的事件枚举，如下面的示例所示。

    const QEvent::Type customEventType = QEvent::Type(QEvent::User + 1);
    
    class CustomEvent : public QEvent
    {
    public:
       CustomEvent();
       int value() const;
       void setValue(int value);
    
    private:
       int m_m_value;
    };
    

# 8.3 同步和异步事件

在Qt中，事件可以同步或异步发送。异步事件使用`QCoreApplication::postEvent()`在事件队列中排队，该队列由`QAbstractEventDispatcher`的一个特定于平台的子类管理。同步事件`QCoreApplication::sendEvent()`无需排队。还要注意，异步事件是由事件系统管理的，这意味着它们必须在堆中分配，并且不能被开发人员使用代码删除。

异步事件是线程安全的。实际上，跨线程信号和槽是基于异步事件的。当一个线程中的一个对象向另一个线程中的一个对象发出信号时，假设连接类型是自动的或排队的，那么实际上在线程之间发送了一个事件。当事件在另一个线程中处理时，处理程序代码自动调用槽。

因为Qt中的任何线程都可以有自己的事件循环，所以不要直接从另一个线程调用slot或任何函数来中断事件处理，这一点很重要。使用排队连接或异步事件是安全的。只要接收线程返回到事件循环并开始处理新事件，事件就会进入队列。此外，如果您从工作线程通知GUI线程有新数据可用时，您应该始终异步地执行此操作。

### 练习
通过子类化`QObject`来实现`CustomObject`，并且:

*   在构造函数中启动一个计时器（例如3秒）。
*   重新实现`event()`函数。检查是否存在`CustomEvent`并打印到调试控制台："Custom event handled: Event data"
*   重新实现`timerEvent()`函数。计时器到期后，退出应用程序。

实现具有字符串成员的`CustomEvent`类。

使用`main.cpp`中提供的代码，程序应该向调试控制台打印两条消息，然后在计时器结束后退出应用程序。

# 9. 延伸阅读

有关信号和槽的更多详细信息，请参见官方[信号和槽](https://doc.qt.io/qt-5/signalsandslots.html "信号和槽")文档。

[Source](https://materiaalit.github.io/qt-mooc/part2/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
