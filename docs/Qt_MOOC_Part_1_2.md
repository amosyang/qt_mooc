# <center>字符串处理和值类型<center>

在这一章中，我们将讨论字符串及其操作，并简要讨论Qt中的值与类型。如果您希望了解更具体的技术细节，我们鼓励您查看官方Qt文档。

# 1. QChar

`QChar`类提供了一个16位的Unicode字符。在Qt中，Unicode字符是没有任何标记或结构的16位实体，这个类表示这样的实体。它是轻量级的，所以可以在任何地方使用。大多数编译器将其视为无符号的short。该类具有许多您期望的方法，例如`isNull()`和`isNumber()`。如果您想了解更多的`QChar`信息，请参阅[官方文档](https://doc.qt.io/qt-5/qchar.html "官方文档")。

# 2. QString

`QString`类提供一个Unicode字符串。`QString`存储一串16位的`QChars`，每个`QChar`对应一个Unicode 4.0字符。（编码值大于65535的Unicode字符使用代理对来存储（即两个连续的`QChar`）。

`QString`底层使用隐式共享（写时复制）来减少内存使用并避免不必要的数据复制。这也有助于减少存储16位字符而不是8位字符的固有开销。

除了`QString`，Qt还提供了`QByteArray`用于存储原始字节和传统的8位以'\\ 0'结尾的字符串的类。大多数情况下您只需要使用`QString`类。它在整个Qt API中使用，并且对Unicode的支持可以确保您在某种程度上希望扩展应用程序的市场时，您的应用程序将很容易转换。`QByteArray`适用于两种主要的情况：需要存储原始二进制数据，以及对内存的保护至关重要时（例如在嵌入式系统中）。

# 2.1 初始化字符串

初始化`QString`的一种方法是简单地将`const char *`传递给它的构造函数。例如，下面的代码创建了一个大小为5的`QString`，其中包含数据"Hello":

    QString str = "Hello";
    
还有其他方法，例如以`QChar`数组的形式提供字符串。

另一种方法是使用`resize()`设置字符串的大小，并使用`[]`运算符来初始化每个字符，例如`str[2] = QChar('A')`。调用`resize()`函数后，新分配的字符具有未定义的值。要将字符串中的所有字符设置为特定值，请使用`fill()`函数。

对于只读访问，另一种语法是使用`at()`函数。该`at()`函数比`operator[]()`更快，因为它永远不会导致深拷贝的出现。

为了简化字符串的使用`QString`提供了几十种重载方法。例如，如果您想将QString与字符串文字进行比较，则可以编写如下代码，它将按预期工作：

    QString str;
    if (str == "auto" || str == "extern" || str == "static" || str == "register") {
        // ...
    }
    
您还可以通过调用`QString(const char *)`构造函数，将字符串文本传递给以`QString`作为参数的函数。

# 2.2 处理字符串数据

`QString`提供下列基本功能用于修改字符数据：`append()`，`prepend()`，`insert()`，`replace()`，和`remove()`。例如：

    QString str = "and";
    str.prepend("rock "); // str == "rock and"
    str.append(" roll"); // str == "rock and roll"
    str.replace(5, 3, "&"); // str == "rock & roll"
    

如果您逐步构建`QString`时知道它将包含多少个字符，则可以调用`reserve()`，它会为`QString`预先分配一定数量的内存。您也可以调用`capacity()`获取`QString`实际分配了多少内存。

一个常见的需求是从字符串中删除空白字符。如果希望从`QString`的两端删除空格，可以使用`trimmed()`函数。如果希望删除两端的空格，并将字符串中的多个连续空格替换为单个空格字符，请使用`simplified()`。

`indexOf()`和`lastIndexOf()`函数返回它们找到的字符或子字符串的第一个索引位置，前者是从前往后查找，而后者从后往前查找; 如果未找到，则返回-1。

字符串列表由`QStringList`类处理。您可以使用`split()`函数将字符串拆分为字符串列表，也可以使用分隔符将字符串列表合并为单个字符串`QStringList::join()`。您还可以使用`QStringList::join()`函数从包含特定子字符串或匹配特定`QRegExp`的字符串列表中获取字符串列表。

# 2.3 转换

`QString`提供了以下三个函数：`toUtf8()`，`toLatin1()`，和`toLocal8Bit()`，它们返回一个`const char *`版本的字符串即`QByteArray`。相应的，从这些编码转换是`fromLatin1()`，`fromUtf8()`和`fromLocal8Bit()`。另外，`QTextCodec`类支持其他编码。

# 2.4 QTextCodec

Qt使用Unicode来存储、绘制和操作字符串。在许多情况下，您可能希望处理使用不同编码的数据。例如，大多数日本文档仍然存储在Shift-JIS或ISO 2022-JP中，而俄罗斯用户通常将文档存储在KOI8-R或Windows-1251中。`QTextCodec`类可以提供文本编码之间的转换。我们不会在本课程中介绍不同的编码，但是如果您有兴趣学习它，请访问[QTextCodec文档](http://doc.qt.io/qt-5/qtextcodec.html "QTextCodec文档")。

# 2.5 null和empty字符串之间的区别

由于历史原因，`QString`会区分null字符串和empty字符串。null字符串是使用`QString`的默认构造函数或通过将`const char *`0传递给构造函数来初始化字符串。empty字符串是大小为0的字符串。因此，null字符串始终为空，但empty字符串不一定为空（例如`QString()`，既等于null又等于empty，`QString("")`等于empty，但不等于null）。

除了`isNull()`之外，所有函数都将empty字符串视为null字符串。例如，`toUtf8().constData()`为空字符串返回一个指向'\\ 0'字符的指针（不是空指针），`QString()`等于`QString("")`。我们建议您始终使用`isEmpty()`函数并避免使用`isNull()`。

# 2.6 高效的字符串构造函数

许多字符串在编译时是已知的。但是普通构造函数`QString("Hello")`将复制字符串的内容，并将内容作为Latin-1处理。为了避免这种情况，可以使用`QStringLiteral`宏在编译时直接创建所需的数据。从文本中构造`QString`在运行时不会造成任何开销。

另一种效率稍低的方法是使用`QLatin1String`。这个类包装了一个C语言的字符串文本，在编译时预先计算它的长度，然后使用它可以比普通的C语言字符串文本更快地与`QString`进行比较并转换成`QString`。

使用`QString`的'+'运算符，可以很容易地从多个子字符串构造一个复杂的字符串。您可能经常编写如下代码：

    QString foo;
    QString type = "long";
    
    foo->setText(QLatin1String("vector<") + type + QLatin1String(">::iterator"));
    
    if (foo.startsWith("(" + type + ") 0x"))
        ...
    

这两种字符串构造都没有错误，但是存在效率低下的问题。

首先，多次使用'+'运算符通常意味着要多次分配内存。当串联n个子字符串（其中n>2）时，最多会有n-1个对内存分配器的调用。

内部模板类`QStringBuilder`可以与一些辅助函数一起使用。此类被标记为内部类，因此未出现在文档中，因为您无需在代码中实例化该类。如下所述，它会被自动使用。如果要查看该类，可以在`src/corelib/tools/qstringbuilder.cpp`中找到。

`QStringBuilder`使用表达式模板并重新实现了'%'操作符，这样当您使用'%'进行字符串连接而不是'+'时，多个子字符串连接将被推迟，直到将最终结果分配给一个`QString`。此时，最终结果所需的内存量已经知道了。然后调用内存分配器以获得所需的空间，然后将子字符串一个接一个地复制到内存分配器中。

可以有两种方法访问这个改进的字符串构造方法。最直接的方法是在任何您想使用的地方包含`QStringBuilder`，并在连接字符串时使用'％'运算符而不是'+'：

    #include <QStringBuilder>
    
    QString hello("hello");
    QStringRef el(&hello, 2, 3);
    QLatin1String world("world");
    QString message =  hello % el % world % QChar('!');
    
一个更方便的是使用全局方法，但它不完全与源代码兼容，是在你的`.pro`文件中定义:

    DEFINES *= QT_USE_QSTRINGBUILDER

然后'+'将在所有地方自动作为`QStringBuilder`的'％'执行。

在本练习中，您将实现几个字符串处理的函数。您将在练习模板的`strings.cpp`文件中找到练习说明。

# 3. QByteArray

[`QByteArray`](http://doc.qt.io/qt-5/qbytearray.html "QByteArray")类提供了一个字节数组。

`QByteArray`用于存储原始字节和传统的8位以'`\0`'结尾的字符串。使用`QByteArray`通常比使用`const char *`方便的多。在底层它始终确保数据后面带有'`\0`'结束符，并使用隐式共享（写时复制）来减少内存使用并避免不必要的数据复制。

初始化`QByteArray`的一种方法是简单地将`const char *`传递给它的构造函数。例如，以下代码创建一个大小为5的字节数组，其中包含数据"Hello"：

    QByteArray ba("Hello");
    
尽管`size()`字节数为5，但字节数组最后还保留了一个额外的'`\0`'字符，因此，如果使用的函数要求指向原始数据（例如，对`data()`的调用），则保证所指向的数据为'`\0`'时结束。

`QByteArray`可以嵌入字节'`\0`'。`size()`函数始终返回整个数组的大小，包括嵌入的' `\0`'字节，但不包括`QByteArray`添加的结束符`\0`。如果您想获得到第一个`\0`字符数据的长度，可以使用`qstrlen()`。

与`QString`一样，您还可以使用`resize()`设置数组的大小，然后使用`operator[]()`初始化每个字节的数据。

要获取指向实际字符数据的指针，请调用`data()`或`constData()`。这些函数返回一个指向数据开头的指针。在`QByteArray`上调用非const函数之前，该指针保持有效。

`QByteArray`提供了与`QString`相同的用于修改和搜索字节数据的函数，如`append()`和`indexOf()`。

执行数值数据类型和字符串之间转换的函数在C语言环境中执行，与用户的语言环境设置无关。使用`QString`在数字和字符串之间执行可识别语言环境的转换。

在`QByteArray`中，大写和小写以及哪个字符大于或小于另一个字符的概念取决于语言环境。这将影响支持不区分大小写选项的函数、比较函数或其参数的小写或大写。如果两个字符串都仅包含ASCII字符，则不区分大小写的操作和比较将是准确的。（如果设置了$LC_CTYPE，则大多数Unix系统都会做“正确的事”。）这个问题不适用于`QString`，因为它们使用Unicode表示字符。

在本练习中，您需要对字节有所考虑。您可以像往常一样在模板的文件中找到练习的说明`bytes.cpp`。

# 4. 值类型vs标识类型

让我们简短地谈谈Qt中的值与标识类型。本章讨论的所有对象（例如`QString`）都是值类型，表示可以复制它们。在本课程的稍后部分，我们将介绍`Qt Objects`标识类型。这里的不同之处在于，在赋值和复制值的地方，克隆的是标识类型。克隆意味着创建一个新的标识类型，而不是旧标识类型的精确副本。如果您想进一步了解其背后的原因，[请参阅文档](https://doc.qt.io/qt-5/object.html "请参阅文档")。

# 4.1 QVariant

在本课程的稍后部分，我们将使用`QVariant`，因此我们将在这里简要讨论。

`QVariant`类充当最常见Qt数据类型(特别是值类型)的联合。因为C++禁止union包含具有非默认构造函数或析构函数的类型，所以大多数有趣的Qt类都不能在union中使用。`QVariant`解决了这个问题。

`QVariant`对象每次保存单个`type()`的单个值。（有些`type()`是多值的，例如一个字符串列表。）您可以找出该变量持有的类型`T`，使用`convert()`将该变量转换为其他类型，使用其中一个`toT()`函数（例如`toSize()`）获取其值，使用`canConvert()`检查是否可以使用将类型转换为特定类型。有关更多详细信息，请参见[QVariant文档](https://doc.qt.io/qt-5/qvariant.html "QVariant文档")。

[Source](https://materiaalit.github.io/qt-mooc/part1/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
