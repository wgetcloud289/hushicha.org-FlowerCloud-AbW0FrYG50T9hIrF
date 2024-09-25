

# 需求




  1\.使用osg三维引擎进行动力学模型仿真性能测试；  2\.打开动力学仿真模型文件，.k后缀的模型文件，测试加载解析过程；  3\.解决第三方company的opengl制作的三维引擎，绘制面较多与弹丸路径模拟较卡顿的问题；  4\.测试时，使用的模型为公开模型，基础面数量达到160多万个；  5\.测试时，模拟动画使用的时100万条弹丸路径平行飞射出去；



 

# Demo V1\.1\.0




  1\.新增打开双模型，第一个模型在原来的位置，第二个模型在偏移后的位置  2\.优化打开关闭重新打开模型的过程  3\.对于100万线动画射击，用于测试性能  4\.当前模型为160万个面，双模型的时候为320多万个元素基础面  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202409/1971530-20240924205416014-968966724.gif)  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202409/1971530-20240924205415947-31156245.gif)




  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202409/1971530-20240924205416199-1697742744.gif)




  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202409/1971530-20240924205416835-976777142.gif)



 

# Demo v1\.0\.0




  测试交互流畅性，交互无延迟！！！  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202409/1971530-20240924205416335-253736316.gif)



 

# 模块化部署




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202409/1971530-20240924205415798-1114723335.png)



 

# :[milou加速器](https://xinminxuehui.org)关键源码




## OsgWidget.h





```
#ifndef OSGWIDGET_H
#define OSGWIDGET_H

#include 
#include "OsgViewerWidget.h"
#include "MyManipulator.h"
#include "osg/PolygonMode"

class AnimationPathCameraMainpulator;

namespace Ui {
class OsgWidget;
}

class OsgWidget : public QWidget
{
    Q_OBJECT
public:
    // 模型结构体
    struct Element_Shell    // *ELEMENT_SHELL
    {
        Element_Shell() {
        }
        qint64 eid;         // 单元id
        qint64 pid;         // 材料id
        qint64 n1;          // 节点1，定义几何形状
        qint64 n2;          // 节点2，定义几何形状
        qint64 n3;          // 节点3，定义几何形状
        qint64 n4;          // 节点4，定义几何形状
        qint64 n5;          // 厚度，额外的节点在标准的LS-DYNA四边形壳单元定义中是没有意义的。
        qint64 n6;          // 积分点数，额外的节点在标准的LS-DYNA四边形壳单元定义中是没有意义的。
        qint64 n7;          // 额外的节点在标准的LS-DYNA四边形壳单元定义中是没有意义的。
        qint64 n8;          // 额外的节点在标准的LS-DYNA四边形壳单元定义中是没有意义的。
    };
    struct Part             // *PART
    {
        Part() {
        }
        qint64 pid;         // 部件的id号，唯一
        qint64 secid;       // 有*section关键字定义的section的id号
        QList listElementShell;  // 部件片元
        qint64 mid;         // 部件的材料号
        qint64 eosid;       // 部件所属材料涉及的状态方程号，由*EOS关键字定义
        qint64 hgid;        // 沙漏或体积粘性参数编号，由*HOURGLASS关键字定义，取0表示将采用默认的数值：
        qint64 grav;        // 仅对实体单元有效，取0表示对所有PART进行重力初始化，取1表示仅对当前材料初始化
        qint64 adpopt;      // 标识该部件是否采用自适应网格划分，取0表示不采用
        qint64 tmid;        // 标识该部件是否采用自适应网格划分，取0表示不采用
    };
    struct Node {
        Node() {
        }
        qint64 nid;         // 结点号，唯一
        double x;           // 三维x坐标（全局）
        double y;           // 三维y坐标（全局）
        double z;           // 三维z坐标（全局）
        int tc;             // 平动自由度受约束状态，枚举值：0-无平动约束,1-X方向平动约束,2-Y方向平动约束
        int rc;             // 转动自由度收约束状态，枚举值：0-无转动约束,1-X方向转动约束,2-Y方向转动约束
    };
    struct K_Mode
    {
        K_Mode() {}
        QList listPart;
        QList listNode;
        QHash<int, Node> hashNid2Node;
    };

    // 添加模型
    K_Mode kMode;

public:
    explicit OsgWidget(QWidget *parent = 0);
    ~OsgWidget();

public:
    bool getFixXAxis() const;               // 获取X轴固定状态
    bool getFixYAxis() const;               // 获取Y轴固定状态
    bool getFixZAxis() const;               // 获取Z轴固定状态
    void getCenter(double &x, double &y, double &z);
                                            // 获取引擎中心点坐标
    void getPersonPoint(double &x, double &y, double &z);
                                            // 获取初始化人眼的角度（看向引擎中心点）

public:
    void setFixXAxis(bool fixXAxis);        // 设置固定X轴
    void setFixYAxis(bool fixYAxis);        // 设置固定Y轴
    void setFixZAxis(bool fixZAxis);        // 设置固定Z轴
    void setCenter(double x, double y, double z);
                                            // 设置引擎中心点坐标
    void setPersonPoint(double x, double y, double z);
    void setEnablePolygonMode(bool enable);

    void startAnimation();
    void pauseAnimation();
    void stopAnimation();

public:
    bool loadKFile(QString filePath);
    bool loadK2File(QString filePath, int num, int x, int y, int z);

    void clear();
    void resetCoordinate();

protected:
    void initOsg();                 // osg初始化
    void loadNode(osg::ref_ptr pNode);
                                    // 加载场景根节点

protected:
    osg::ref_ptr createScene();          // 创建总场景
    osg::ref_ptr createAnimation();      // 创建动画

protected:
    void resizeEvent(QResizeEvent *event);
    void keyPressEvent(QKeyEvent* event);
    void keyReleaseEvent(QKeyEvent* event);
    void mousePressEvent(QMouseEvent* event);
    void mouseReleaseEvent(QMouseEvent* event);
    void mouseDoubleClickEvent(QMouseEvent* event);
    void mouseMoveEvent(QMouseEvent* event);
    void wheelEvent(QWheelEvent* event);
    void timerEvent(QTimerEvent *event);

private:
    Ui::OsgWidget *ui;

private:
    OsgViewerWidget *_pViewer;                  // osg场景嵌入Qt核心类
    osg::ref_ptr _pRoot;  // osg场景根节点

private:
    float _xDistance;                           // x轴单个tick间距
    int _xTickNumber;                           // x轴tick数(例如：5的时候，是6个，0~5)

    float _yDistance;                           // y轴单个tick间距
    int _yTickNumber;                           // y轴tick数(例如：5的时候，是6个，0~5)

    float _zDistance;                           // z轴单个tick间距
    int _zTickNumber;                           // z轴tick数(例如：5的时候，是6个，0~5)

    QString _zUnit;                             // z轴单位
    float _zTickLabelOffset;                    // z轴坐标偏移
    QString _yUnit;                             // y轴单位
    float _zTickUnitLabelOffset;                // z轴坐标偏移

    QColor _gridColor;                          // 轴颜色
    QColor _labelColor;                         // 轴tickLabel的颜色
    osg::ref_ptr _pNode;             // 模型
    osg::ref_ptr _pNode2;             // 子弹

    osg::ref_ptr _pManipulator;  // 自定义漫游器

    osg::Vec3d _eyeVect3D;                      // 原始坐标，用于复位原始视角
    osg::Vec3d _centerVect3D;                   // 原始坐标，用于复位原始视角
    osg::Vec3d _upVect3D;                       // 原始坐标，用于复位原始视角

    K_Mode _kMode;

    int _timerId;

    osg::ref_ptr _pStateSet;
    osg::ref_ptr _pPolygonMode;

    osg::ref_ptr _pVec3Array;   // 炮弹

    bool _animationPausing;

};

#endif // OSGWIDGET_H

```



## OsgWidget.cpp





```
bool OsgWidget::loadK2File(QString filePath, int num, int x, int y, int z)
{
    if(!QFile::exists(filePath))
    {
        LOG << "Not exist file:" << filePath;
        QMessageBox::information(0, "错误", QString("Not exist file: %1").arg(filePath));
        return false;
    }
    QFile file(filePath);
    if(!file.open(QIODevice::ReadOnly))
    {
        LOG << "Failed to open file:" << filePath;
        QMessageBox::information(0, "错误", QString("Failed to open file: %1").arg(filePath));
        return false;
    }

    kMode = K_Mode();

    QTextStream textStream(&file);
    QString context;
    qint64 rowIndex = -1;
    context = textStream.readLine();
    rowIndex++;
    LOG;
    ...
    file.close();


    LOG;
    osg::ref_ptr pGroup = new osg::Group;

    for(int index = 0; index < num; index++)
    {
        LOG << index;
        // 绘图
        {
            for(int partIndex = 0; partIndex < kMode.listPart.size(); partIndex++)
            {
                // 创建一个用户保存几何信息的对象
                osg::ref_ptr pGeometry = new osg::Geometry;
                // 创建四个顶点的数组
                osg::ref_ptr pVec3Array = new osg::Vec3Array;
                // 添加四个顶点
                pGeometry->setVertexArray(pVec3Array.get());

                // 创建四种颜色的数据
                osg::ref_ptr pVec4Array = new osg::Vec4Array;
                // 添加四种颜色
                pGeometry->setColorArray(pVec4Array.get());
                // 绑定颜色
                pGeometry->setColorBinding(osg::Geometry::BIND_PER_VERTEX);

                double r, g, b;
                r = qrand() % 100 * 1.0f / 100;
                g = qrand() % 100 * 1.0f / 100;
                b = qrand() % 100 * 1.0f / 100;
                for(int elementShellIndex = 0; elementShellIndex < kMode.listPart.at(partIndex).listElementShell.size(); elementShellIndex++)
                {
                    //                               x     y     z
                    pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).x + index * x,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).y + index * y,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).z + index * z));
                    pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).x + index * x,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).y + index * y,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).z + index * z));
                    pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).x + index * x,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).y + index * y,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).z + index * z));
                    pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).x + index * x,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).y + index * y,
                                                    kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).z + index * z));


                    //                               r    g    b    a(a设置无效，估计需要其他属性配合)
                    pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
                    pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
                    pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
                    pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));

                }
                // 注意：此处若不绑定画笔，则表示使用之前绑定的画笔
                // 为唯一的法线创建一个数组    法线: normal
                osg::ref_ptr pVec3ArrayNormal = new osg::Vec3Array;
                pGeometry->setNormalArray(pVec3ArrayNormal.get());
                pGeometry->setNormalBinding(osg::Geometry::BIND_OVERALL);
                pVec3ArrayNormal->push_back(osg::Vec3(0.0, -1.0, 0.0));

                // 由保存的数据绘制四个顶点的多边形
                pGeometry->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::QUADS, 0, kMode.listPart.at(partIndex).listElementShell.size() * 4));
        //            pGeometry->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::QUADS, 0, 4));

                // 向Geode类添加几何体(Drawable)
                osg::ref_ptr pGeode = new osg::Geode;
                pGeode->addDrawable(pGeometry.get());
#if 0
                {
                    _pStateSet = pGeometry->getOrCreateStateSet();
    //                _pPolygonMode = new osg::PolygonMode(osg::PolygonMode::FRONT_AND_BACK, osg::PolygonMode::LINE);
                    _pPolygonMode = new osg::PolygonMode(osg::PolygonMode::FRONT_AND_BACK, osg::PolygonMode::FILL);
                    _pStateSet->setAttribute(_pPolygonMode, osg::StateAttribute::ON);
                }
#endif
                pGroup->addChild(pGeode.get());
            }
        }
    }
    // 始终是灰色，这里需要设置关闭光照：OFF，同时旋转都能看到了（光照关闭，法向量不起作用）
    {
        osg::StateSet *pStateSet = pGroup->getOrCreateStateSet();
    //      pStateSet->setMode(GL_LIGHTING, osg::StateAttribute::ON);
        pStateSet->setMode(GL_LIGHTING, osg::StateAttribute::OFF);
    }


    _pNode = pGroup.get();

    if(_pNode.get() == 0)
    {
        return false;
    }
    _pRoot->addChild(_pNode);

    return true;
}


```


 

# 工程模板v1\.1\.0




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202409/1971530-20240924205415999-1613250480.png)



