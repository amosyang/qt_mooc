# <center>创建和调试Qt项目<center>

您好！欢迎加入Qt MOOC！

本主题涵盖了您在学习本课程时所需的大部分基本知识。我们将从课程的角度，以练习的形式来讨论Qt项目。还会介绍一些重要概念，这些概念将帮助您了解Qt的基础知识。我们使用Qt的官方文档作为辅助材料，这会让您熟悉Qt的官方文档。另外，我们还将讨论构建，qmake和调试信息等。

如果所有这些对您来说都已经很熟悉了，你可以直接去完成练习，然后进入更具体的主题。如果您在完成Hello World练习时不知道发生了什么情况，请回来阅读基础知识，我们不会对您有任何偏见!

在开始繁重的工作之前，我们将介绍一些基本概念来帮助您开始。它们是Qt中的全局声明、模块和命名空间。

# 1. 全局声明和宏

让我们来讨论一下Qt中的全局声明，您可以在[Qt文档中](http://doc.qt.io/qt-5/qtglobal.html "Qt文档")找到一个完整的全局声明列表。

全局声明包括类型、函数和宏。在进入下一章之前，您绝对不需要记住这个列表，但重要的是，您要知道它是什么，可以用它来做什么，因为您可能会发现它在各种场合下都非常有用。在本主题的最后，我们将讨论在全局声明中找到的一些有用的调试函数。

类型定义部分是用于方便基本类型的定义(其中一些类型保证了Qt在所有支持的平台上的特定位大小)，部分是与Qt消息处理相关的类型。这些函数与生成消息、Qt版本处理以及比较和调整对象值有关。最后，一些声明的宏允许程序员将编译器或平台特定的代码添加到他们的应用程序中，而另一些宏则是用于更大操作的方便宏。

如果有必要，我们会在作业或学习主题中明确地向您介绍全局声明列表。

# 2. Qt模块和命名空间

接下来，让我们来讨论一下Qt中的模块。[这是Qt中的模块列表](http://doc.qt.io/qt-5/qtmodules.html "这是Qt中的模块列表")。

要将模块包含到项目中，您需要将它们包含在项目的`.pro`文件中。如果使用qmake，则默认情况下将包括Core和GUI模块。如果您没有在项目中使用GUI模块（无论如何，在开始时大部分时间我们都不会使用GUI模块），只需在您的项目`.pro`文件中写入`QT -= GUI`即可将其禁用。所有其他Qt模块都依赖于Qt Core模块。

接下来，我们看一下有关命名空间及其在Qt中的用法的一些基本知识。[这是Qt中主要命名空间的列表](https://doc.qt.io/qt-5.10/namespaces.html "这是Qt中主要命名空间的列表")。当然，该课程我们不会使用所有的命名空间，但是在开始之前，我们相信事先了解这个列表会对您有所帮助。

让我们举一个包含命名空间的示例！假设您想在代码中包含`QtConcurrent`命名空间。让我们仔细看看`QtConcurrent`的文档页面：

![](https://materiaalit.github.io/qt-mooc/img/part-1/QtConcurrentNamespace-72ddc814.png)

要包含`QtConcurrent`命名空间，您需要在两个地方进行具体说明：相关文件的头文件（`#include <QtConcurrent>`），

![](https://materiaalit.github.io/qt-mooc/img/part-1/QtConcurrentWorks-fefccb1c.png)

和项目`.pro`文件（`QT += concurrent`）。

![](https://materiaalit.github.io/qt-mooc/img/part-1/QtConcurrentInProFile-52079a2d.png)

# 3. Qt命名空间

在[Qt命名空间](http://doc.qt.io/qt-5/qt.html "Qt命名空间")中，您会发现Qt库中使用了各种各样的标识符。在进入本主题的下一章之前，我们希望您能记住这些列表。

只是开个玩笑！我们只是想让您知道，如果您碰巧需要使用命名空间，可以把它作为参考材料。

# 4. TMC练习的结构

讲完基础知识之后，该看一下TMC练习的结构组成了。

每个练习都位于每周主练习目录中自己的子目录中。该练习由根级别的`.pro`文件以及几个目录组成。

根级别的`.pro`文件是练习项目的项目文件。我们将在本章后面进一步讨论项目文件。但是现在，我们将重点放在项目结构上。

用于测试您的练习的本地测试位于`test_runner`目录中。

如果您在弄清楚为什么您的练习不能通过测试时遇到了问题，并且您已经用尽了所有其他方法，那么可能值得查看测试运行程序文件。弄清楚测试的内容可能会给您一些线索，让您知道代码中可能存在的问题。如果有助于您更好的理解问题的话，您甚至可以修改本地测试。在极少的情况下，问题可能是由于测试本身是错误的。在这种情况下，我们建议加入课程频道，看看其他人是否有同样的问题。我们已经做了我们所能做的一切来确保这种情况不会发生，但总有特殊情况。

请注意，本地测试只是为了告诉您，您的练习是否完成并准备好提交。您无法根据自己的偏好修改测试，并期望通过提交一份未完成的作业来通过练习!我们将在您获得分数之前在服务器上测试您的练习!有时候，服务器端可能会有隐藏的测试，以确保您不会根据测试的目的来尝试完成练习，而是完成实际的练习来支持您的学习。

`src`目录是神奇个地方，它是我们编程的位置。

它包括每个练习项目的相关源文件和头文件。在某些情况下，库可能包含额外的目录，在某些情况下，我们可能会要求您创建新的目录。没人知道!没有人知道什么时候!没有人知道!但我们稍后会算出来。

您是否已经注意到每个目录都有自己的`.pro`文件？这样做的原因是，我们包含在练习目录中的每一个子目录实际上都是实际练习项目中的子项目。让我们详细介绍一下根目录`.pro`文件：

    TEMPLATE = subdirs
    SUBDIRS += \
          src \
          test_runner
    
          test_runner.depends = src
    

它会告诉`qmake`（我们将在本章的下一步讨论它）该练习项目遵循一个名为`subdirs`的构建模板，并定义了包含在该项目中的子目录。使用子目录模板要求每个声明的子目录都是其自己的项目，并且在子目录中包含它自己的项目文件。最后，我们声明`test runner`子目录依赖于`src`目录。这是因为我们想在构建`test runner`之前构建您的练习，这样`test runner`就有东西要测试了。

# 5. qmake

[qmake](http://doc.qt.io/qt-5/qmake-manual.html "qmake")工具可以帮助您在各种平台开发项目时简化构建过程。它会自动生成Makefile，因此只需几行信息即可创建Makefile。您还可以将qmake用于其他软件项目，无论它是不是使用Qt编写的。

`qmake`根据项目文件中的信息生成一个`Makefile`。项目文件使用扩展名`.pro`，通常由Qt Creator中的项目模板创建。项目文件由开发人员处理，以指定他们`qmake`要用于其项目的指令集，并且通常很简单。不过您也可以为复杂的项目创建更复杂的项目文件。我们将在本章后面进一步讨论项目文件，但由于`qmake`使用的是存储在项目文件中的信息，所以在谈到`qmake`时，我们无法避免地会谈到它们的内容。

`qmake`的行为受项目构建过程中的变量声明的影响。其中一些声明资源对每个平台都是通用的，例如头文件和源文件。其他的用于定制特定平台上编译器和链接器的行为。

让我们再一次参考我们的练习结构来详细说明，源目录中的`.pro`文件如下：

    QT -= gui
    
    TARGET = main
    
    CONFIG += c++11 console
    CONFIG -= app_bundle
    
    win32 {
         CONFIG -= debug_and_release debug_and_release_target
    }
    
    DEFINES += QT_DEPRECATED_WARNINGS
    
    SOURCES += \
         main.cpp \
         hello.cpp
    
     HEADERS += \
          hello.h
    

您可以参考`01_HelloWorld`的目录结构和文件组成。也可以参照[qmake变量引用](http://doc.qt.io/qt-5/qmake-variable-reference.html "qmake变量引用")，并思考前面提到的`.pro`文件代码中的声明。您要知道我们要告诉`qmake`什么以及为什么。win32位可能不是完全不言自明的。

特定于平台的变量遵循它们的扩展或修改的变量的命名模式，但在其名称中包括相关平台的名称。例如，`LIBS`可用于指定项目需要链接的库列表。如下：

    unix:LIBS += -L/usr/local/lib -lmath
    win32:LIBS += c:/mylibs/math.lib
    
如果您想自己做一些探索，我们建议您查看[qmake变量的完整列表](http://doc.qt.io/qt-5/qmake-variable-reference.html "qmake变量的完整列表")，以便自行浏览。

# 6. .pro文件

在项目文件中，变量用于保存字符串列表。在最简单的项目中，这些变量会告知`qmake`要使用的配置选项，或者提供要在构建过程中使用的文件名和路径。

`qmake`会在每个项目文件中查找某些变量，并使用这些变量的内容来确定应该向`Makefile`写入什么。例如，`HEADERS`和`SOURCES`变量中的值列表用于声明`qmake`与项目文件位于同一目录中的头文件和源文件。

只要您使用`qmake`，默认情况下就会包括Core和GUI模块。

变量还可以在内部用于存储值的临时列表，现有的值列表可以用新值覆盖或扩展。

以下代码段说明了如何将值列表分配给变量：

    HEADERS = mainwindow.h paintwidget.h
    

变量中的值列表以以下方式扩展：

    SOURCES = main.cpp mainwindow.cpp \
              paintwidget.cpp
    CONFIG += console
    

注意：第一个赋值仅包含与`HEADERS`变量在同一行上指定的值。第二个赋值`SOURCES`使用反斜杠（`\`）将变量中的值分成几行。

# 7. Qt资源系统

[Qt资源系统](https://doc.qt.io/qt-5/resources.html "Qt资源系统")是一种独立于平台的方式，用于在应用程序的可执行文件中存储应用程序资源(图像、图标、翻译文件、数据等)。
资源在资源集合文件中`qrc`指定:

    <!DOCTYPE RCC><RCC version="1.0">
    <qresource>
        <file>images/icon.png</file>
        <file>data.txt</file>
    </qresource>
    </RCC>
    
然后将该文件包含在`.pro`文件中，如`RESOURCES = application.qrc`。指定的文件将被压缩并包装为一个C++静态数组。接着可以使用像[QFile](https://doc.qt.io/qt-5/qfile.html "QFile")或图像类型来访问它们，就像其他任何文件一样：

    QIcon icon(":/images/icon.png")
    QFile file(":/data.txt");
    
# 8. QDebug类

[QDebug](http://doc.qt.io/qt-5/qdebug.html "QDebug")类提供了一个用于调试信息的输出流。

当开发人员需要将调试或跟踪信息写入设备、文件、字符串或控制台时，就会使用[QDebug](http://doc.qt.io/qt-5/qdebug.html "QDebug")。

**基本用途**

在通常情况下，调用`qDebug()`函数以获得默认的QDebug对象以用于编写调试信息很有用。

    qDebug() << "Date:" << QDate::currentDate();
    qDebug() << "Types:" << QString("String") << QChar('x') << QRect(0, 10, 50, 40);
    qDebug() << "Custom coordinate type:" << coordinate;
    
`QDebug`的构造函数会接受`QtMsgType`值来构造一个`QtDebugMsg`对象。类似地，`qWarning()`，`qCritical()`和`qFatal()`也会返回相应的消息类型的QDebug对象。

这个类还为其他情况提供了一些构造函数，包括接收[QFile](https://doc.qt.io/qt-5/qfile.html "QFile")或用于将调试信息写入文件，套接字，其他进程等的任何其他[QIODevice](https://doc.qt.io/qt-5/qiodevice.html "QIO设备")子类的构造函数。接收QString的构造函数用于写入用于显示或序列化的字符串。

**格式化选项**

格式化QDebug的输出以便于阅读。它在参数之间会自动添加空格，并为`QString`，`QByteArray`，`QChar`的参数增加了引号。

您可以通过`space()`，`nospace()`，`quote()`，`noquote()`方法来调整这些选项。另外，可以将`QTextStream`通过管道传送到`QDebug`流中。

使用`QDebugStateSaver`会将格式更改限制为当前范围。`resetFormat()`将选项重置为默认选项。

**将自定义类型写入流**

可以将许多标准类型写入QDebug对象，而Qt提供了对大多数Qt值类型的支持。要添加对自定义类型的支持，您需要实现一个流操作符，如下面的示例所示:

    QDebug operator<<(QDebug debug, const Coordinate &c)
    {
        QDebugStateSaver saver(debug);
        debug.nospace() << '(' << c.x() << ", " << c.y() << ')';
    
        return debug;
    }
    

# 9. 输出调试信息

[qDebug](http://doc.qt.io/qt-5/qtglobal.html#qdebug "QDegug")，[qWarning](http://doc.qt.io/qt-5/qtglobal.html#qWarning "qWarning")，[qCritical](http://doc.qt.io/qt-5/qtglobal.html#qCritical "qCritical")和[qFatal](http://doc.qt.io/qt-5/qtglobal.html#qFatal "qFatal")是Qt全局声明中定义的函数。这些函数采用格式化字符串和参数列表，类似于C语言的`printf()`函数。格式应为Latin-1字符串。要禁止这些类型中的任何一种，您可以使用[qInstallMessageHandler](https://doc.qt.io/qt-5/qtglobal.html#qInstallMessageHandler "qInstallMessageHandler")安装自己的消息处理程序。函数使用定义的调试/严重/致命/警告消息调用消息处理程序。它们调用消息处理程序，如果没有消息处理程序，则将消息打印到`stderr`。在Windows上，大多数情况下，该消息会发送到调试器。

    qDebug(const char *message, ...)
    

在Windows下，如果消息是控制台应用程序，则将消息发送到控制台。否则，它将被发送到调试器。如果在编译过程中定义了`QT_NO_DEBUG_OUTPUT`，则此函数不执行任何操作。

例如：

    qDebug("Items in list: %d", myList.size());
    

如果导入`QtDebug`，还可以使用更方便的语法：

    qDebug() << "Brush:" << myQBrush << "Other value:" << i;
    

使用此语法，该函数返回一个QDebug对象，该对象配置为使用QtDebugMsg消息类型。它会自动在每个项目之间放置一个空格，并在末尾输出换行符。它支持许多C++和Qt类型。

导入`QtDebug`同样允许以使用`qInfo`，`qCritical`和`qWarning`等效语法。

    qCritical(const char *message, ...)
    

如果环境变量`QT_FATAL_CRITICALS`不为空，则退出。

例如：

    void load(const QString &fileName)
    {
         QFile file(fileName);
         if (!file.exists())
              qCritical("File '%s' does not exist!", qUtf8Printable(fileName));
    }
    

\-

    qWarning(const char *message, ...)
    
如果在编译期间定义`QT_NO_WARNING_OUTPUT`，则此函数不执行任何操作。如果在环境变量`QT_FATAL_WARNINGS`中的计数器对应的第n个警告时，它就会退出。也就是说，如果环境变量包含值1，它将在第一条消息上退出；如果它定义为10，它将在第10条消息上退出。任何非数字值都等于1。

例如：

    void f(int c)
    {
        if (c > 200)
            qWarning("f: bad argument, c == %d", c);
    }
    

    qFatal(const char *message, ...)
    

如果使用默认消息处理程序，则此函数将中止以创建核心转储。在Windows上，对于调试版本，此功能将报告`_CRT_ERROR`使您能够将调试器连接到应用程序。

例如：

    int divide(int a, int b)
    {
        if (b == 0)                                
             qFatal("divide: cannot divide by zero");
        return a / b;
    }
    

# 10. 影子构建

成功构建第一个Qt项目后，您可能会对构建的二进制文件感兴趣。默认情况下，Qt Creator将项目作为[影子构建](https://doc.qt.io/qtcreator/creator-glossary.html#glossary-shadow-build "影子构建")到一个单独的目录（\`build-ProjectName- \*\`）中，该目录与源目录分开。`qmake`也可以生成**源代码内部**版本，但是不建议这样做。影子构建的好处是保持该目录的清洁，这使得在Kit或构建配置之间进行更改的速度更快。

--------

> 嘿! 现在是我们第一次练习的时候了。这将是一个简单的过程，请放心。只需从TMC下载练习模板，打开项目，然后转到`hello.cpp`。 
在`world()`函数中，写上“Hello world”作为`qDebug`消息，并写上“Don't panic!” 作为`qWarning`消息。

--------

[Source](https://materiaalit.github.io/qt-mooc/part1/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
