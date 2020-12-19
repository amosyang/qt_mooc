# <center>QML介绍<center>

我们在前面两章主要介绍了Qt基本模块，它几乎可以用于任何Qt应用程序中，无论该应用程序多么简单或复杂。我们讨论了应用程序逻辑，实现应用程序行为的引擎。然而除了逻辑之外，我们通常还需要创建一个用户界面来与引擎交互，最好是具有一个酷炫且现代化的用户界面。

在第三章，我们将讨论使用Qt Quick创建用户界面。我们将使用的编程语言是声明式脚本语言QML，它扩展了JavaScript。虽然可以用JavaScript实现应用程序逻辑，但首选的方法是使用QML和JavaScript来声明UI组件、布局和动画，使用C++来实现应用程序逻辑。这是因为使用JavaScript实现的逻辑的性能几乎总是比C++差。

在第五章，我们将继续介绍用C++编写应用程序逻辑与QML交互的主题。而在本章，我们只讨论QML语言。

Qt Creator中有一个图形化的UI设计工具Qt Quick Designer。课程中的UI文件可以通过直接编辑QML代码来实现，也可以使用Qt Quick Designer生成。Qt还提供了Qt Design Studio，主要提供给设计师使用。它允许UI设计师从Adobe Photoshop中导入图形资源，并进一步让软件工程师使用Qt Design Studio中生成的UI文件。然而，对于课程练习使用Qt Design Studio是有点太笨重了。

# 1. Qt Quick

Qt Quick是一个模块，包含了QML的所有基本类型和功能。它包括可视类型，交互类型，动画，模型和视图，粒子效果和着色器效果。QML应用程序开发人员可以通过导入相应的QML模块来访问这些功能。模块是插件，为QML引擎提供了类型。一个简单的模块可能只是一个由QML文本文件组成的文件夹。

Qt Quick QML库由Qt Quick模块提供。有关Qt Quick提供的各种QML类型和其他功能的更多信息，请参考[Qt Quick模块](http://doc.qt.io/qt-5/qtquick-index.html "Qt Quick模块")文档。

在使用Qt Quick模块时，您需要知道如何使用QML语言编写QML应用程序。

使用Qt Quick模块，设计人员和开发人员可以高效地处理同一产品，而无需长时间等待设计人员修改或等待开发人员实现某些UI特性。这会显著减少设计人员和开发人员之间的沟通时间，减少产品上市的周期。

# 2. QML

QML（Qt Meta-Object Language）即Qt元对象语言（又称为Qt建模语言），它是一种用户界面标记语言。

Qt QML模块提供了使用QML语言开发应用程序和库的框架。它定义并实现了语言和引擎基础架构，并提供了一些API，使应用程序开发人员能够使用自定义类型来扩展QML语言并将QML代码与JavaScript和C++集成在一起。Qt QML模块同时提供了QML API和C++ API。

您应该使用C++处理应用程序逻辑。在大多数情况下，它会比使用JavaScript实现逻辑具有更好的性能。

![](https://materiaalit.github.io/qt-mooc/img/part-3/qt_quick_workflow-7d7f1af7.png)

QML是一种声明性语言，它允许根据可视化组件以及它们之间如何交互和关联来描述用户界面。它是一种高度可读的语言，旨在使组件以动态方式互连，并且允许在用户界面中轻松地重用和定制组件。使用Qt Quick模块，设计人员和开发人员可以轻松地在QML中创建流畅并具有动画功能的用户界面，并可以选择将这些用户界面连接到任何后端C++库。

QML中的UI组件是用对象、对象属性和属性绑定声明的，可以设置它们来定义应用程序行为。我们可以像在Qt/C++中一样声明信号和槽的连接。应用程序的行为还可以通过编写JavaScript脚本来实现，JavaScript是该语言的一个子集。此外，QML里大量使用了Qt, Qt允许从QML应用程序直接访问它的类型和其他Qt特性。实际上，QML类型(也称为组件)中的每个JavaScript方法都是一个公共的槽，可以连接到任何一个信号。

# 3. .qml文件和QML语法

在QML中定义了多个对象，并使它们能够以一种语言描述它们的变化。与纯粹的命令式代码相比，QML的声明性语法将属性和行为的更改直接集成到单个对象的定义中，这种代码通过一系列逐步处理的语句来表示属性和行为的更改。在需要复杂的自定义应用程序行为的情况下，这些属性定义仍可以包括命令代码。

QML源代码通常由引擎通过QML文件（.qml）加载，这些文件是QML代码的独立文件。它们可以用来定义QML的对象类型，然后在整个应用程序中重用。定义新类型的QML文件也称为组件。

请注意，类型名称必须以大写字母开头，以便在QML文件中声明为QML对象类型。

## 3.1 在项目中使用QML

要在项目中使用QML，您需要完成以下三个步骤：

1.使用以下指令包含模块类的定义:

    #include <QtQml>
    

2\. Qt QML中的QML类型可以通过导入QtQML获得。要使用这些类型，请在.qml文件中添加以下导入语句:

    import QtQml 2.0
    

3.要链接到模块，请在qmake .pro文件中添加这一行:

    QT += qml
    

## 3.2 导入

在QML文件中可能存在一个或多个导入。导入可以是以下任一项：

*   已注册并具有版本号和命名空间的类型（如插件）
*   一个包含类型定义的相对目录
*   一个JavaScript文件

导入JavaScript文件时必须对其进行限定，以便可以访问它们提供的属性和方法。

各种导入的通用形式如下：

    import Namespace VersionMajor.VersionMinor
    import Namespace VersionMajor.VersionMinor as SingletonTypeIdentifier
    import "directory"
    import "file.js" as ScriptIdentifier
    

例子：

    import QtQuick 2.0
    import QtQuick.LocalStorage 2.0 as Database
    import "../privateComponents"
    import "somefile.js" as Script
    

# 4. 使用Quick UI工程和qmlscene进行原型设计

为了更快地创建用户界面原型，可以创建一个[Qt Quick UI](http://doc.qt.io/qtcreator/quick-projects.html "Qt Quick UI")项目。这个项目不包含任何C++代码，资源文件`.qrc`或部署代码（`qmake`文件）。这样设计人员就可以在不编译任何代码的情况下启动应用程序，开发人员也可以快速共享概念验证。例如，在Qt Design Studio中，可以只创建Qt Quick UI项目。QML场景也可以在移动设备或嵌入式设备中远程执行，UI编辑器中的任何更改都将立即反映在UI中。Qt Quick Designer没有类似的功能。

您还可以为原型应用程序[qmlscene](https://doc.qt.io/qt-5/qtquick-qmlscene.html "qmlscene")启用键盘快捷方式，该快捷方式可呈现任意`.qml`文件。`qmlscene`可以通过在Qt Creator中启用`Options -> Environment -> Keyboard -> search for qmlscene -> assign shortcut`。当一个`.qml`文件是在编辑或设计模式下打开，按快捷方式呈现该文件。

# 5. 国际化

Qt Quick具有广泛的国际化和本地化支持。

让我们来看看基本的流程:

    Text {
        id: txt1;
        text: qsTr("Back");
    }
    

使用函数qsTr()在UI中声明可翻译字符串。这使得“Back”成为翻译文件中的一个关键条目。

    Text {
        id: txt1;
        // This user interface string is used only here
        //: The back of the object, not the front
        //~ Context Not related to back-stepping
        text: qsTr("Back", "not front");
    }
    

您可以使用`//:`和可选的`//~`开头的注释为翻译器添加上下文。前者是译者的主要评论，后者是可选的额外信息。

有时候同一个词在不同的上下文中有不同的意思。通过向`qsTr()`添加第二个参数来区分它们，然后该参数在翻译文件中为单词提供惟一的id。

    Text {
        text: qsTr("File %1 of %2").arg(counter).arg(total)
    }
    

由于语言之间的句子结构各不相同，所以要避免连接单词和数据。相反，使用`%`将参数插入到字符串中。这样译者就可以改变参数在句子中的位置。

您可以在指定参数时包括`L`修饰符，例如`%L1`，以便根据当前的区域设置本地化数字。例如，把数字1337.42格式化为美国的“1,337.42”和德语的“1.337,42”。

有关国际化的更详细信息，请参阅[Qt Quick的国际化和本地化](https://doc.qt.io/qt-5/qtquick-internationalization.html "Qt Quick的国际化和本地化")和[Qt Linguist Manger](https://doc.qt.io/qt-5/linguist-manager.html "Qt Linguist Manger")

[Source](https://materiaalit.github.io/qt-mooc/part3/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
