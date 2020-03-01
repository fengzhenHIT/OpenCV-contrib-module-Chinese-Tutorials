# 2.6	Aruco模块常见问题
我们在使用aruco模块是可能经常会遇到一些问题，我们整理了部分常见的内容，在下方详细列出：
-	如果只想标记一些对象，应该使用什么?

如果只需要单个ArUco标记，那可以在要识别的每个对象中放置一个或多个具有不同ID的标记。

-	标记检测用的哪种算法？

aruco模块基于原始的aruco库。检测过程算法的完整介绍可以阅读Automatic generation and detection of highly reliable fiducial markers under occlusion文论了解。

-	如果标记没有被正确探测到，应该怎么办?

许多因素都可能影响对标记的正确检测。我们可能需要调整DetectorParameters对象中的一些参数。首先检查detectmarker()函数是否将标记作为被拒绝的候选标记返回。根据这一点，可以尝试修改不同的参数。如果使用ArUco board，也可以尝试refineDetectedMarker()函数。

-	ArUco board有什么优点？有什么缺点？

使用ArUco board，可以从一组标记而不是单个标记中获得摄像机的姿态。这样就可以解决标记版局部视图被遮挡的问题，因为仅需一个标记就可获得摄像机的姿态。大多数情况下，使用更多的标记来估计姿态将比使用单一标记的结果更准确。
主要缺点是不如单个标记那样通用。

-	与ArUco board相比，ChArUco board有什么优点和缺点?

ChArUco board将棋盘和ArUco board结合起来。因此ChArUco board提供的角点比ArUco board (或单标记)提供的角点更准确。
主要缺点是ChArUco board不如ArUco board用途广泛。ChArUco board是具有特定标记布局的平面板，而ArUco board甚至可以在3d中具有任何布局。此外，ChArUco板上的标记通常更小，更难以检测。

-	如果不需要相机姿态估计，应该使用ChArUco board吗?

不，ChArUco board的主要目标是为摄像机姿态估计和摄像机标定提供高精度的角点。

-	应该将ArUco board上的所有标记放在同一个平面上吗?

不，ArUco board上的标记可以放置在其3d坐标系中的任何位置。

-	应该将ChArUco board上的所有标记放在同一个平面上吗?

是的，ChArUco棋盘上的所有标记都需要在同一个平面上，它们的布局由棋盘形状决定。

-	想进行相机标定，可以使用这个模块吗?

可以，aruco模块提供了使用aruco board和ChArUco board标定摄像机的功能。

- 应该使用ChArUco board还是ArUco board进行标定？

强烈推荐使用ChArUco板进行标定，因为它精度更高。

-	可以使用这个模块来检测基于二进制基准标记的其他标记吗?

可能可以，您需要将原始库的字典转换成符合aruco模块的格式。

-	是否需要将字典信息存储在一个文件中，以便在不同的应用中读取它?

如果正在使用某个预定义的字典，则没有必要这样做。否则，建议将其保存到文件中。

-	是否需要将板子信息存储在一个文件中，以便在不同的执行中使用它?

如果使用的是GridBoard或ChArUco board，则只需要存储提供给GridBoard::create()或ChArUco::create()函数的板的测量值。如果手动修改板的标记id，或者使用不同类型的板，就应该将它保存到文件中。

-	aruco模块是否提供将字典或板信息保存到文件的功能?

现在还没有。Dictionary和Board类的数据成员都是公共的，可以很容易地存储。
