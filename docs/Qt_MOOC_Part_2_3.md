# <center>多任务处理<center>

在基于事件的系统中，保持GUI线程的响应并在工作线程中完成所有耗时的任务是很重要的。耗时的任务可能只需要几十毫秒，也可能会执行无限循环。在任何情况下，阻塞函数都不应该延迟GUI线程中的事件处理。

# 1. QThread

`QThread`是一个以平台无关的方式管理线程的类。线程本身是特定于平台的内核对象。`QThread`提供了一个API，通过一个平台无关的枚举来设置优先级——`setPriority(QThread::priority)`，启动线程——`start(QThread::priority)`，退出线程事件循环——`exit(int returnCode)`。调用`start()`函数会使得`QThread::run()`将在新线程中执行。该`run()`函数通过调用`QThread::exec()`来启动特定于线程的事件循环。信号`started()`和`finished()`分别在线程将要启动时（即`run()`函数尚未启动）和线程完成执行事件循环之后发出。

注意，还有一个终止线程的函数`terminate()`。我们不鼓励使用此函数，因为它可能会在数据处理中或还持有一个锁定的互斥对象的过程中立即终止线程。这可能会破坏数据的完整性或导致死锁。最好是从事件循环线程的`run()`函数中顺利返回。

如果不需要控制线程的执行，那么使用`QRunnable`对象会容易得多。使用`QRunnable`需要重新实现它的`run()`函数，`QRunnable`将在线程池提供的线程中执行，不需要手动创建和清除。

# 2. 线程关联性

线程关联定义`QObject`实例属于哪个线程。如果使用自动连接类型，则需要此信息来决定两个对象之间的信号是否应该排队。在实践中，线程关联就是`QThread *QObject::thread()`的返回值。如果值为0，则线程无法接收信号或发布的事件。

开发人员应该特别注意在他们的Qt程序中有正确的线程关联。尽管概念相当简单，但很容易产生令人讨厌的错误，这些错误似乎是随机发生的，并且对调试和测试构成了挑战。例如，在下面的类声明中，很容易出现计时器成员的线程关联错误。`MyThread`是在调用线程中实例化的，这意味着其`QObject`成员也将在调用线程中实例化。如果我们尝试启动或停止`run()`函数中的计时器，则会出现运行时错误：“Timers cannot be started from another thread”。

    class MyThread : public QThread
    {
    public:
       explicit MyThread();
    
    protected:
       void run() override;
    
    private:
       QTimer m_timer;
    };
    

若要解决此问题，您需要先修改计时器线程关联，然后才能在`run()`函数中使用它。可以使用`QObject::moveToThread(QThread *)`修改对象的线程关联。效果与在该`run()`函数中创建计时器相同。请注意`run()`函数中启动线程事件循环的基本调用。

    MyThread::MyThread()
    {
       m_timer.moveToThread(this);
    
       connect(&m_timer, &QTimer::timeout, [] () {
           qDebug() << "Timer expired";
       });
    }
    
    void MyThread::run()
    {
       m_timer.start(1000);
       QThread::run();
    }
    

通常，根本不需要子类化`QThread`。建议您创建一个worker `QObject`，然后修改该对象的线程关联。这样可以减少代码的错误。

# 3.3 后台任务

让我们看看，如何在不子类化`QThread`的情况下使用worker对象创建后台任务。一个普通的worker对象应该有一个在线程中执行的函数。在以下示例中，它是`run()`函数。在计时器超时后，worker就完成了，这将导致线程得到一个通知，它可以退出事件循环。注意，计时器是一个指针成员，在worker的构造函数中，计时器的父节点被设置为worker本身。这很方便，因为当我们修改worker线程关联时，它的所有子线程关联也将修改。父线程及其子线程不能具有不同的线程关联。

    class WorkerObject : public QObject
    {
       Q_OBJECT
    public:
       explicit WorkerObject(QObject *parent = nullptr);
       virtual void run();
    
    Q_SIGNALS:
       void finished();
    
    private:
       QTimer *m_timer;
    };
    
    WorkerObject::WorkerObject(QObject *parent)
       : QObject(parent)
       , m_timer(new QTimer(this))
    {
    }
    

启动工作程序所需的代码变得非常简单。我们创建worker线程和线程对象，并将worker线程关联到新线程。在线程运行之前，我们不能启动worker程序。否则，worker程序由主线程执行。worker程序完成后，它将通知主线程和新线程退出事件循环。线程完成事件循环后，堆内存将被清理。

    int main(int argc, char *argv[])
    {
       QCoreApplication a(argc, argv);
    
       auto *thread = new QThread;
       auto *worker = new WorkerObject;
       worker->moveToThread(thread);
       
       QObject::connect(thread, &QThread::started, worker, &WorkerObject::run);
       
       QObject::connect(worker, &WorkerObject::finished, thread, &QThread::quit);
       QObject::connect(worker, &WorkerObject::finished, &a, &QCoreApplication::quit);
       
       QObject::connect(thread, &QThread::finished, thread, &QThread::deleteLater);
       QObject::connect(thread, &QThread::finished, worker, &WorkerObject::deleteLater);
    
       thread->start();
    
       return a.exec();
    }
    

# 4. 优雅的销毁

如果worker对象正在运行一个繁忙的循环，那么我们应该如何很好地结束线程。请记住，我们应该避免使用`QThread::terminate()`。`QThread`提供了一个友好的函数`requestInterruption()`来请求线程中断，并可以使用`isInterruptionRequested()`函数来检查请求是否已经中断。这些函数甚至可以在未运行事件循环的线程中使用。当计时器超时后，我们的worker将停止计时器工作并通知线程终止。任何事件或信号都可以用来中断线程。

    void WorkerObject::run()
    {
       m_timer->start(1000);
       connect(m_timer, &QTimer::timeout, [this] () {
           m_timer->stop();
           thread()->requestInterruption();
       });
    
       while (!thread()->isInterruptionRequested()) {
           qDebug() << "Still running";
           QThread::currentThread()->eventDispatcher()->processEvents(QEventLoop::AllEvents);
       }
    
       qDebug() << "Thread finished";
    
       Q_EMIT finished();
    }
    

只要计时器超时，我们的worker对象就会运行一个繁忙循环。在繁忙循环中，线程不处理任何事件，因此我们需要偶尔检查是否有任何事件要处理。事件分配器函数`processEvents()`使我们能够处理任何未决事件。如果没有此功能，我们的繁忙循环将永远运行。

我们已经大量使用信号和槽在线程之间进行通信。它们是线程安全的，并且使线程之间的线程间通信相当简单。除信号和槽外，您还可以使用`QMetaObbject::invokeMethod()`。这对于通知（例如，从worker线程到GUI线程的状态修改）很有用。当从worker线程通知GUI线程时，永远不要使用直接连接或直接函数调用。

# 5. 线程同步

Qt为互斥和线程同步提供了几种锁类型。`QMutex`提供了一个递归互斥锁。在大多数是只读的共享数据访问的情况下，还提供了`QReadLocker`和`QReadWriteLocker`。`QSemaphore`提供单个进程中的`QSystemSemaphore`线程之间以及多个进程中的线程之间的计数信号量。`QWaitCondition`可用于同步线程，等待条件变为真。请从[此处](http://doc.qt.io/qt-5/threads-synchronizing.html)阅读更多信息。

实现一个WorkerObject，它在与GUI线程不同的线程中执行。

*   worker对象应该计算斐波纳契数（直到请求的数字为止）（例如，如果请求5个数字，则应该得到0、1、2、3）
*   worker对象应该有一个计时器来中断计算。
*   辅助对象线程应调用FibonacciApplication中的槽以显示从0到请求数字的斐波那契值。
*   如果计算出了直到请求数量的所有斐波那契值，或者计时器到期，则worker终止。在这两种情况下，您都应该退出应用程序。

通过子类化QCoreApplication来创建FibonacciApplication并添加一个槽，该插槽将在调试控制台中打印出一个数字。

[Source](https://materiaalit.github.io/qt-mooc/part2/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
