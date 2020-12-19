# <center>父子关系<center>

# 1. 对象树

`QObject`将自己组织在对象树中。当您以一个对象作为父对象创建`QObject`时，该对象将被添加到父对象的`children()`列表中，并在父对象被删除时删除。事实证明，这种方法非常适合GUI编程。例如，`QShortcut`（键盘快捷方式）是相关窗口的子对象，因此当用户关闭该窗口时，快捷方式也会被删除。

`QQuickItem`是Qt Quick模块的基本可视化元素，我们将在本课程的后面部分讨论它，它继承于`QObject`，但有一个可视化父元素的概念，它与`QObject`父元素不同。一个可视化元素不一定与它的父对象元素相同。有关更多详细信息，请参见[Concepts - Visual Parent in Qt Quick ](https://doc.qt.io/qt-5/qtquick-visualcanvas-visualparent.html "Concepts - Visual Parent in Qt Quick ")。

您还可以自己删除子对象，它们将从父对象中删除自己。例如，当用户删除一个工具栏时，需要应用程序删除它的`QToolBar`对象，在这种情况下，工具栏的父对象`QMainWindow`会检测到变化并重现刷新界面。

# 2. 对象和指针的持久性

当`QObject`在堆上创建时(即用`new`创建)，则可以按照任何顺序构造对象树，也可以按照任何顺序销毁树中的对象。当对象树中的`QObject`被删除时，如果该对象有父对象，析构函数将自动从其父对象中删除该对象。如果该对象有子对象，则析构函数会自动删除每个子对象。无论销毁顺序如何，`QObject`都不会被删除两次。

当`QObject`在栈上创建时，表现的行为与上述类似。正常情况下，销毁的顺序仍然不成问题。考虑以下代码段：

    int main()
    {
        QWidget window;
        QPushButton quit("Quit", &window);
        ...
    }
    

父对象(window)和子对象(quit)都是`QObject`，因为`QPushButton`继承`QWidget`, `QWidget`继承`QObject`。上面代码是正确的: quit的析构函数不会被调用两次，因为C++语言标准(ISO/IEC 14882:2003)规定对象的析构函数调用顺序与其构造函数顺序相反。因此，子对象quit的析构函数首先被调用，它在window的析构函数调用之前从父对象window中删除了自己。

但是现在考虑一下，如果我们更改了构造的顺序会发生什么，如下面的第二个代码段所示：

    int main()
    {
        QPushButton quit("Quit");
        QWidget window;
    
        quit.setParent(&window);
        ...
    }
    
在这种情况下，析构的顺序会导致问题。父类的析构函数被首先调用，因为它是最后创建的。然后调用其子对象quit的析构函数，这是不正确的，因为quit是一个局部变量。当quit随后超出作用域时，它的析构函数将被再次调用，这次调用是正确的，但是损害已经造成。

总结一下：

*   对象树可以以任意的顺序构造
*   对象树可以以任意的顺序销毁
*   如果对象拥有父对象则该对象首先从父对象中移除
*   如果对象拥有子对象则首先删除所有子对象
*   对象不会被删除两次

请注意，父子关系与继承不是一回事。

# 3. 悬浮指针问题

对象树不能解决悬浮指针问题，但是[QPointer](http://doc.qt.io/qt-5/qpointer.html "QPointer")为`QObject`提供了一个受保护的指针。当使用的对象被销毁时，指针将被设置为0。很容易将受保护的指针和普通指针混合使用。受保护的指针将自动转换为指针类型。

Qt对象也可以在销毁之前通知观察者。

[Source](https://materiaalit.github.io/qt-mooc/part2/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
