# <center>容器<center>

# 1. Qt容器

Qt库提供了一组通用的基于模板的容器类。这些类可用于存储指定类型的项。例如，如果您需要一个可调整大小的QString数组可以使用`QVector<QString>`。

这些容器类的设计比[STL容器](http://www.cplusplus.com/reference/stl/ "STL容器")更轻，更安全，更易于使用。如果您不熟悉STL，或者更喜欢以“Qt方式”进行操作，则可以使用这些类而不是STL类。不过也可以使用标准容器。

Qt容器类是[隐式共享](https://doc.qt.io/qt-5/implicit-sharing.html "隐式共享")的，它们是[可重入](https://doc.qt.io/qt-5/threads-reentrancy.html "可重入")的，并且它们针对速度，低内存消耗和最小的内联代码扩展进行了优化，从而可以生成较小的可执行文件。此外，在所有用于访问它们的线程将它们用作只读容器的情况下，它们是线程安全的。

要遍历存储在容器中的项目，可以使用两种类型的迭代器：Java样式的迭代器和STL样式的迭代器。Java样式的迭代器更易于使用并提供高级功能，而STL样式的迭代器效率更高，可以与Qt和STL的通用算法一起使用。

Qt还提供了一个`foreach`关键字，使访问容器中存储的所有项目变得非常容易。

# 2. 容器类

Qt提供了以下顺序容器：`QList`，`QLinkedList`，`QVector`，`QStack`和`QQueue`。对于大多数应用程序，`QVector`是最佳的使用类型，除非您对其他类型有特定的需求。

Qt还提供了一些关联容器：`QMap`，`QMultiMap`，`QHash`，`QMultiHash`和`QSet`。"Multi"容器可以很方便地支持单个键关联多个值。"Hash"容器通过使用哈希函数而不是对排序集进行二进制搜索来提供更快的查找。

在特殊情况下，`QCache`和`QContiguousCache`类提供了对有限缓存存储中的对象的高效散列查找。

容器可以嵌套。例如，完全可以使用`QMap<QString, QList<int>>`，其中键类型是`QString`，值类型是`QList<int>`。

容器被定义在与容器同名的头文件中(例如`<QLinkedList>`)。为了方便起见，在`<QtContainerFwd>`中前置声明容器。

存储在各种容器中的值可以是任何可赋值的数据类型。为了具有此资格，类型必须提供默认构造函数、复制构造函数和赋值操作符。这涵盖了您可能想要存储在容器中的大多数数据类型，包括基本类型，如int和double、指针类型和Qt数据类型，如`QString`、`QDate`和`QTime`，但不包括`QObject`和`QObject`子类(`QWidget`、`QDialog`、`QTimer`等)。如果尝试实例化`QList<QWidget>`，编译器将会抱怨`QWidget`的复制构造函数和赋值操作符被禁用。如果要将此类对象存储在容器中，请将它们存储为指针，例如作为`QList<QWidget *>`。

下面是一个满足可赋值的数据类型要求的自定义数据类型示例：

    class Employee
    {
    public:
         Employee() {}
         Employee(const Employee &other);
    
         Employee &operator=(const Employee &other);
    
     private:
          QString myName;
          QDate myDateOfBirth;
    };
    

如果我们不提供复制构造函数或赋值操作符，C++会提供一个默认实现，执行成员的逐个复制。在上面的例子中，这就足够了。另外，如果您没有提供任何构造函数，C++提供了一个默认构造函数，它使用默认构造函数初始化它的成员。虽然它没有提供任何显式构造函数或赋值操作符，但以下数据类型可以存储在容器中：

    struct Movie
    {
        int id;
        QString title;
        QDate releaseDate;
    };
    

有些容器对它们可以存储的数据类型有额外的要求。例如，`QMap<Key, T>`的`Key`类型必须提供`operator<()`。这些特殊要求记录在类的详细描述中。在某些情况下，特定的功能有特殊的要求，这些是按每个函数描述的。如果没有满足要求，编译器会报错。

Qt的容器提供了`operator<<()`，`operator>>()`操作符，因此可以使用`QDataStream`轻松地对它们进行读写。这意味着存储在容器中的数据类型还必须支持`operator<<()`和`operator>>()`。提供这种支持很简单，下面我们如何对上面的Movie结构执行此操作：

    QDataStream &operator<<(QDataStream &out, const Movie &movie)
    {
        out << (quint32)movie.id << movie.title
            << movie.releaseDate;
        return out;
    }
    
    QDataStream &operator>>(QDataStream &in, Movie &movie)
    {
        quint32 id;
        QDate date;
    
        in >> id >> movie.title >> date;
        movie.id = (int)id;
        movie.releaseDate = date;
        return in;
    }
    

某些容器类函数的文档引用默认构造的值，例如`QVector`自动用默认构造的值初始化它的项，如果指定的key不在map中，`QMap::value()`返回一个默认构造的值。对于大多数值类型，这仅意味着使用默认构造函数创建一个值（例如的空字符串`QString`）。但是对于如`int`和`double`这些原始类型以及指针类型，C++语言未指定任何初始化，在这种情况下，Qt的容器会自动将该值初始化为0。

# 3. 正确且有效的使用迭代器

迭代器提供了一种统一的方法来访问容器中的项目。Qt的容器类提供了两种类型的迭代器：Java样式的迭代器和STL样式的迭代器。

对于不可变量的迭代可以通过一个简单的范围循环来完成`for (const &noteConstRefToMyItem : container)`。

Qt4中引入了Java风格的迭代器。在某些方面，它们比使用STL风格的迭代器更方便，但效率稍低。它们的API模仿Java的迭代器类。

一般来说，我们不建议使用普通Java风格的迭代器，但如果您愿意，也可以这样做。对于可变迭代，Java风格的可变迭代器可以说是最容易使用的：

`QMutableListIterator`的示例如下：

    QMutableListIterator<int> i(list);
    while (i.hasNext()) {
        if (i.next() % 2 != 0)
            i.remove();
    }
    

该代码从列表中删除所有奇数。

`next()`函数会在每次循环中调用。它会跳到列表中的下一项。该`remove()`函数删除了我们从列表中跳到的最后一项。调用`remove()`不会使迭代器无效，因此可以安全地继续使用它。

如果我们只想修改现有项目的值，则可以使用`setValue()`。在下面的代码中，我们将大于128的值都替换为128：

    QMutableListIterator<int> i(list);
    while (i.hasNext()) {
        if (i.next() > 128)
            i.setValue(128);
    }
    

关联容器的迭代器的工作方式略有不同，但思想是相同的。[官方文档](http://doc.qt.io/qt-5/containers.html#the-iterator-classes "官方文档")深入介绍了不同的迭代器，还包括多个示例。

在本练习中，您将熟悉`QVector`和`QMap`，练习说明可以在`containers.cpp`中找到。

# 4. 容器操作中的算法

如果您有兴趣了解Qt容器和数据类型的算法复杂性，请在[此处](http://doc.qt.io/qt-5/containers.html#algorithmic-complexity "此处")查看文档。在很多方面，理解为什么使用某些容器比其他容器更合适是很有用的，在某些应用中甚至关注最有效的增长策略，但在这里讨论这些已经超出了本课程的目标。


[Source](https://materiaalit.github.io/qt-mooc/part1/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
