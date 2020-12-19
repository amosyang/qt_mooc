# <center>场景图项目<center>

Qt Quick 2使用专用场景图进行渲染。使用场景图而不是传统的命令式绘图系统(`QPainter`或类似的系统)进行绘图，意味着要渲染的场景可以在帧之间进行保留，并且在开始渲染之前就已经了知道要渲染的完整原语集。也意味着开启了很多的优化空间，比如批量渲染以最小化状态更改和丢弃模糊的原语。

假设一个用户界面是包含10个项目的列表，其中每个项目都有一个背景色、一个图标和一个文本。如果使用传统的绘图技术，就需要30次绘制调用和类似数量的状态更改。而如果使用场景图重新组织原语，可以在一次调用中绘制所有背景，然后是所有图标，最后是所有文本，从而将绘制调用的总数减少到只有3次。这样的批处理和状态更改减少可以极大地提高某些硬件的性能。

场景图与Qt Quick 2.0紧密相连，不能单独使用。场景图是由`QQuickWindow`类管理和渲染的，可以通过调用`QQuickItem::updatePaintNode()`自定义`Item`将它们的图形原语添加到场景图中。

# 1. 场景图结构

场景图由许多预定义的节点类型组成，每种类型都有特定的用途。尽管我们将其称为场景图，但更精确的定义是节点树。树是从QML场景中的`QQuickItem`类型构建的，然后在场景内部由绘制场景的渲染器处理。节点本身不包含任何活动的绘图代码或虚函数`paint()`。

## 1.1 节点

对用户而言，最重要的节点是`QSGGeometryNode`。它通过定义图形的几何形状和材质来定义自定义图形。使用`QSGGeometry`定义几何形状，并描述图形原语的形状或网格。它可以是一条线，一个矩形，一个多边形，许多断开的矩形，或复杂的3D网格。材质定义了这个形状中像素的填充方式。

一个节点可以有任意数量的子节点，几何节点将按子级顺序显示，父节点位于子节点之后。

## 1.2 材质

材质用于描述`QSGGeometryNode`中几何图形的内部是如何填充的。虽然大多数Qt Quick项目本身只使用了非常基本的材质，如纯色和纹理填充。但是材质还可以封装一个OpenGL着色器程序，这就提供了更多的灵活性。

可用的材质类别如下：

*   `QSGFlatColorMaterial` - 在场景图中绘制实体彩色几何图形的便捷方法
*   `QSGMaterial` - 封装着色器程序的渲染状态
*   `QSGMaterialShader` - 表示渲染器中的OpenGL着色器程序
*   `QSGMaterialType` - 与`QSGMaterial`结合使用时用作标记类型的唯一
*   `QSGOpaqueTextureMaterial` - 在场景图中渲染纹理几何的便捷方法
*   `QSGSimpleMaterial` - 模板生成的类，用来存储`QSGSimpleMateralShader`使用的状态
*   `QSGSimpleMaterialShader` - 为场景图构建基于OpenGL的自定义材质的便捷方法
*   `QSGTextureMaterial` - 在场景图中渲染纹理几何的便捷方法
*   `QSGVertexColorMaterial` - 在场景图中渲染每个顶点着色几何的便捷方法

# 2. 示例

现在，让我们看一个简单的例子，我们使用场景图项目绘制一个三角形。

    // triangleitem.h
    class TriangleItem : public QQuickItem
    {
        Q_OBJECT
    
    public:
        TriangleItem(QQuickItem *parent = nullptr);
    
    protected:
        QSGNode updatePaintNode(QSGNode node, UpdatePaintNodeData *data);
    
    private:
        QSGGeometry m_geometry;
        QSGFlatColorMaterial m_material;
    };
    
<br/>

    // triangleitem.cpp
    #include "triangleitem.h"
    #include <QSGGeometryNode>
    
    TriangleItem::TriangleItem(QQuickItem *parent)
        : QQuickItem(parent),
          m_geometry(QSGGeometry::defaultAttributes_Point2D(), 3)
    {
        setFlag(ItemHasContents);
        m_material.setColor(Qt::red);
    }
    
    QSGNode TriangleItem::updatePaintNode(QSGNode n, UpdatePaintNodeData *)
    {
        QSGGeometryNode node = dynamic_cast<QSGGeometryNode >(n);
        if (!node) {
            node = new QSGGeometryNode();
        }
    
        QSGGeometry::Point2D *v = m_geometry.vertexDataAsPoint2D();
        const QRectF rect = boundingRect();
        v[0].x = rect.left();
        v[0].y = rect.bottom();
    
        v[1].x = rect.left() + rect.width()/2;
        v[1].y = rect.top();
    
        v[2].x = rect.right();
        v[2].y = rect.bottom();
    
        node->setGeometry(&m_geometry);
        node->setMaterial(&m_material);
    
        return node;
    }
    
<br/>

    // main.cpp
    ...
    qmlRegisterType<TriangleItem>("ShapeObjects", 1, 0, "Triangle");
    ...

<br/>

    import QtQuick 2.9
    import QtQuick.Window 2.3
    import ShapeObjects 1.0
    
    Window {
        width: 640; height: 480
        visible: true
    
        Item {
            width: 300; height: 200
    
            Triangle {
                x: 50; y: 50
                width: 200; height: 100
            }
        }
    }
    

![](https://materiaalit.github.io/qt-mooc/img/part-5/scenegraph-dbfca0d7.png)

在这个例子中有几个需要注意的地方：

*   我们需要子类化`QQuickItem`，它是Qt Quick中所有可视项的基类。
*   使用`QSGGeometryNode`作为节点，`QSGFlatColorMaterial`作为材质。
*   在构造函数中设置`setFlag(ItemHasContents)`标志，以表示项目应该由场景图渲染。
*   实现了`updatePaintNode`函数来进行实际绘制(注意：OpenGL操作与场景图的交互只发生在渲染线程上，主要是在`updatePaintNode()`调用期间，这一点很重要。所以我们应该只在`QQuickItem::updatePaintNode()`函数中使用带有QSG前缀的类。）

[Source](https://materiaalit.github.io/qt-mooc/part5/)

# 更多资料，请关注本人公众号：**程序员练兵场**
![在这里插入图片描述](img/公众号.png)
