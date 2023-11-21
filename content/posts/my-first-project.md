---
title: 'Dynamic Identification Method and Application of Oiltanks in Ports Around the Bohai Sea Based on Deep Learning | 基于深度学习的环渤海港口储油罐动态识别方法与应用'
date: 2023-11-15T12:00:20+08:00
draft: false
cover:
    image: /img/my-first-project/proj1-img5-compressed.jpg
    alt: 'This is a post-image (alt)'
    caption: 'Dynamic Identification Method and Application of Oil Tanks in Ports Around the Bohai Sea Based on Deep Learning'

tags: ["project"]
categories: ["project"]
showtoc: true
---
> As the group leader, I conducted the Innovation Training Program for College Students in Jilin University, which is ***Dynamic Identification Method and Application of Oil Tanks in Ports Around the Bohai Sea Based on Deep Learning***. The project got a **Provincial Excellent Conclusion** in May 2023. Here is the detailed description of this project.
>
> Currently, this post is mainly written in Chinese.
>
>作为项目负责人，完成吉林大学大学生创新创业训练计划省级课题《基于深度学习的环渤海港口储油罐动态识别方法与应用》。该项目获得省级优秀结题，为本年级唯一优秀结题项目。下面是该项目的介绍。

建立了一个环渤海地区港口储油罐动态识别与展示系统。其以高分二号遥感影像和ESRI瓦片地图作为数据源，自动执行基于GDAL的滑动窗口切片、基于YOLO v5模型的目标检测、冗余结果去除、基于OpenCV的霍夫变换圆检测、油罐高度信息提取、地理/图像坐标转换等程序，实现油罐要素服务的自动更新与油罐地理和属性信息的动态展示。

!["img1 alt"](/img/my-first-project/proj1-img1-compressed.jpg "图1 系统架构图")

系统可通过二维平面和三维场景两种方式展示油罐信息。在三维场景中，储油罐要素将被渲染成三维的圆柱体，视觉效果较好。总体而言，系统自动化程度高，油罐识别精度高（96.97%），地理数据与属性信息展示效果较好。

!["img2 alt"](/img/my-first-project/proj1-img2.png "图 2 环渤海港口储油罐识别与展示系统截图（2D）")

!["alt"](/img/my-first-project/proj1-img3.png "图 3 环渤海港口储油罐识别与展示系统截图（3D）")

!["alt"](/img/my-first-project/proj1-img4.png "图 4 油罐识别结果局部（左：原始遥感影像；右：识别结果）")

## 1 基于GDAL的滑动窗口切片算法

储油罐在卫星遥感影像视场中所占比例较小，给目标检测带来较大难度，通过滑动窗口切片的方法提高储油罐在单幅影像中所占比例。大图幅遥感影像需要按照滑动窗口的位置被裁剪为诸多小图幅遥感影像，相邻窗口之间有一定的重叠比率，以确保目标地物至少能够完整出现在一个窗口中。

!["alt"](/img/my-first-project/proj1-img5-compressed.jpg "图 5 滑动窗口切片原理图（岙山国家战略石油储备基地）")

如图 5，滑动窗口将逐行从左至右滑动，红色部分为重叠区域，其在窗口中的占比即为overlay。每个窗口对应的遥感影像区域将被单独裁剪，输出为小窗口切片图像，其命名规则遵循：*ImageName-row_column_height_width.tif*。例如：*Aoshan-2120_3180_1248_1248.tif*。

为了使得裁切出的窗口影像是具有空间参考的GeoTiff文件，方便后续将目标检测结果的图片坐标转为地理坐标，我们采用GDAL库（地理空间数据抽象库，即Geospatial Data Abstraction Library）实现滑动窗口切片算法。该算法涉及两个关键参数：（1）*window_width*：即窗口的宽和高；（2）*overlay*：即相邻窗口间重叠比率，其默认值为0.15。若要保证目标地物至少在一个窗口中完整出现，需满足不等式：

> ***window_width×overlay≥object_width_max/resolution***。

其中，*object_width_max*为目标地物宽度的最大值，*resolution*为遥感影像分辨率。在给定*overlay*=0.15，*object_width_max*=*90m，resolution*=0.5m的情况下，应当满足：

> ***window_width>1200***

考虑到YOLOv5的特征图（Feature map）尺寸为416×416，将*window_width*设置为416的整数倍1248。

## 2 YOLOv5目标检测算法

搭建了系统运行YOLOv5所必须的环境，基于YOLOv5m模型和储油罐遥感影像数据集进行训练，得到了一个储油罐目标检测模型。

## 3 目标检测结果去冗余/非极大值抑制算法

由于滑动窗口之间有重叠部分，部分目标地物在不同窗口中会各自产生锚框，造成目标检测结果的冗余，影响后续油罐信息提取步骤。系统实现非极大值抑制(NMS)算法，通过计算区域交并比（IOU，即每两个检测结果框之间交集面积与并集面积之比），最终保留IOU小于阈值的、具有最大得分的目标框，达到去冗余锚框的目的。图 6展示了岙山国家战略石油储备基地区域的遥感影像，从左至右分别为未经NMS处理的锚框、经过NMS处理的锚框、经过NMS处理的锚框与霍夫变换圆检测效果图。可见，合理的IOU阈值既能筛掉滑动窗口边界处的冗余锚框（图 6上部），也能保留间距较近、有小部分重合的油罐锚框（图 6下部），为霍夫变换提供良好的数据来源。

!["alt"](/img/my-first-project/proj1-img6.png "图 6 非极大值抑制与霍夫变换圆检测效果图")

## 4 储油罐高度信息提取

采用长春地区一景高分二号影像作为测试数据，测试了基于单幅遥感影像的储油罐高度提取方法。依次进行了遥感影像预处理、遥感影像融合、RGB-HSV色彩空间变换、波段比值计算、OTSU自适应阈值分割、形态学优化等图像处理步骤。

对于单幅遥感影像，获取太阳与卫星的方位角与高度角，计算储油罐阴影长度，即可根据相关空间关系计算出油罐高度。首先需要将原始图像从RGB色彩空间转换至HSV空间（图 7左上图），此后根据下述公式计算比值图像（图 7中上图）：

> ***I_ij=(H_ij+1)/(V_ij+1)***

其中，*H_ij*和*V_ij*分别为像元(i,j)的色调和亮度通道的分量。随后进行OTSU阈值分割，得到如图 7右上图所示的二值图像。最后，通过面积筛选优化（阈值分别为200,500,2500；或形态学开运算）得到如图 7下部三张图所示的结果。

!["alt"](/img/my-first-project/proj1-img7.png "图 7 提取油罐高度图像处理过程")

根据亚像素细分定位思想提取阴影中与太阳光线平行的线段，根据下列公式计算线段长度S:

> ***S(p,q)=[(s-x)^2+(t-y)^2 ]^(1/2)×k***

其中，*(s,t)*和*(x,y)*分别是线段两个端点*p*和*q*的坐标，*k*为遥感影像空间分辨率，*S*是储油罐的阴影长度。

根据卫星成像方位角和高度角的不同，以及太阳角度的不同，可根据阴影长度计算出储油罐的高度。有3种可能的情况：1）卫星方位角与太阳方位角的差值大于180°；2）卫星方位角等于太阳方位角；3）卫星方位角与太阳方位角的差值小于180°。第1种情况下，卫星可拍摄到完整的油罐阴影。以此种情况为例，列出油罐高度计算公式：

> ***h=S×tanα***

其中，*α*为太阳高度角，*S*是上一步提取的油罐阴影长度，*h*是储油罐高度。其他情况的具体数学公式可以此类推。

## 5 霍夫变换圆检测

调用Python OpenCV库的cv2.HoughCircles()函数，进行霍夫变换圆检测。该函数的定义如下：

```Python
HoughCircles(image, method, dp, minDist, circles=None, param1=None, param2=None, minRadius=None,
    maxRadius=None)
```

其中，image为图像；method为方法，目前仅可选择霍夫梯度法，即cv2.HOUGH_GRADIENT；dp默认选择1；minDist为圆之间的最小距离，若过小则可能导致误检；param1为Canny检测器的较高阈值，较低阈值为其值的1/2，默认设置为100；param2为检测阶段圆心的累加器阈值，过小会检测出本不存在的圆，越大则检测到的圆就越接近完美的圆形，但过大时可能会检测不出圆形造成漏检。

参考OpenCV文档，当关键参数param2参数较小导致返回多个圆形时，圆形将按照累加值降序排列，即可信度最高的圆将排在首位。基于此特点，及系统输入给霍夫圆检测函数的图像应为经过NMS去冗余后的储油罐目标检测结果，图片尺寸较小，储油罐的圆形特征突出，故本文将param2设置得尽量小，并只取函数返回的第一个圆形作为结果，以尽可能提高储油罐顶圆形的检出率和识别准确性。系统调用函数的具体参数如下：

```Python
circles = cv2.HoughCircles(gray, cv2.HOUGH_GRADIENT, dp=1, minDist=30, param1=100, param2=30,
    minRadius=15, maxRadius=200)
```

霍夫变换圆检测的效果图在上文中已经列出，见图 6。以岙山国家战略石油储备基地为实验区域，检验系统检测油罐的精度。在半径大于10m的198个油罐中，有192个被准确检测出来，识别率达96.97%。

## 6 归属港口判别算法

为了判别识别出的储油罐所归属的港口，笔者开发了储油罐归属港口判别算法。为了减少运算的次数，避免每次运算都要遍历所有的港口坐标以寻找给定坐标点的最近港口，笔者生成了港口点文件的泰森多边形，通过将目标点与港口泰森多边形做Intersection运算，来判别距离该点最近的港口点，以达到判别归属港口的目的。

首先输入储油罐的地理坐标，并构造点类型的WKT，并创建该点对应的数据源（DataSource）；此后读取泰森多边形，将其与目标点数据源相交；最终读取结果文件的字段信息，以获得包括港口ID、港口代码、港口名称等的归属港口的相关信息。

## 7 要素服务自动更新

通过学习ArcGIS Server、SQL Server与企业级地理数据库相关知识，实现了通过SQL语句在地理数据库中插入要素以达到更新服务目的的算法。

SQL Server提供了一种平面坐标系下的空间数据类型，即geometry。该类型的字段以二进制的形式存储地理空间中的几何图形信息，并可通过开放地理空间信息联盟（OGC）的熟知文本（WKT）形式写入和输出。对于识别出的储油罐，我们将其ID、半径、面积、高度、体积、空间信息以及所属港口信息整合为SQL语句，样例如下：

```SQL
INSERT INTO AUTOUPDATETEST VALUES(401, 35.498910, 3958.949018, 23.443860, 92813.046525, 
    geometry::STGeomFromText('POLYGON ((x1 y1, x2 y2, ... xn yn, x1, y1))', 0), 5717,
    ‘CN DLC’, DALIAN)
```

表示油罐图斑地理信息的WKT由霍夫变换所得的圆心坐标与半径计算得来，每 *(360/n)°* 取一个圆周上的点，计算出坐标 *(x_i,y_i )* ，将各点坐标按照标准格式依次相连，最后加上第一个点的坐标以使多边形闭合，即可表达一个近似圆的n边形。在本系统中n取36。
