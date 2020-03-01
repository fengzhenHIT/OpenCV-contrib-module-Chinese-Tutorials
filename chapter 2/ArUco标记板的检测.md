ArUco标记板是一组如图2-13所示的标志。在为相机提供单个姿势上，他们就像单个标记一样工作。常用的方式是所有标记都在同一平面上的标志板，因为我们可以直接打印出来。但是，标记板不限于这种布置，还可以用2维或3维布局表示。

<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223141221760.jpg" height="300">
</p>

板和一组独立标记之间的区别在于，板中标记之间的相对位置是先验的。这样可以将所有标记的角点用于估计摄像机相对于整个标记板的姿态。当我们使用一组独立的标记时，由于我们不知道标记在环境中的相对位置，因此可以分别估计每个标记的姿态。

使用标记板的最主要好处有：
-	姿态估计更加通用。执行姿态估计仅需要一部分标记。因此，即便存在一些遮挡或仅是局部视图，也可以估算出姿态。
-	由于采用了大量的点对应关系标记角点，因此获得的姿态通常更加准确。

Aruco模块允许使用标记板，使用的是cv::aruco::Board类，它用代码清单2-15中的方式定义了板的布局。
```cpp
代码清单2-15 图像修复
class Board {
public:
    std::vector<std::vector<cv::Point3f> > objPoints;
    cv::Ptr<cv::aruco::Dictionary> dictionary;
    std::vector<int> ids;
};
```

Board类对象具有三个参数：
-	ObjPoints结构是在三维板参考系统中角点位置的列表，即其布局。对于每个标记，其四个角点均以标准顺序储存，即以顺时针顺序并从左上角开始存储。
-	字典参数指示板中标记属于哪个标识字典。
-	最后，ids结构指示objPoints中相对于指定字典的每一个标记的标记符。

## 2.2.1	标志板检测
Aruco模块提供了一个特定的函数estimatePoseBoard()用于执行标记板的姿态估计，该函数的使用方式在代码清单2-16中给出。
```cpp 
代码清单2-16 板姿态估计
cv::Mat inputImage;
// 从某处读取相机参数
cv::Mat cameraMatrix, distCoeffs;
readCameraParameters(cameraMatrix, distCoeffs);
// 假设我们有一个创建板对象的函数
cv::Ptr<cv::aruco::Board> board = cv::aruco::Board::create();
...
std::vector<int> markerIds;
std::vector<std::vector<cv::Point2f>> markerCorners;
cv::aruco::detectMarkers(inputImage, board.dictionary, markerCorners, markerIds);
// 如果至少检测到一个标识
if(markerIds.size() > 0) {
    cv::Vec3d rvec, tvec;
    int valid = cv::aruco::estimatePoseBoard(markerCorners, markerIds, board, cameraMatrix, distCoeffs, rvec, tvec);
}

```
estimatePoseBoard()函数的每个参数含义为：

-	makerCorners和markerlds：从detectMarkers()函数检测到的标记的结构。
-	board：定义板布局及其ID。
-	cameraMatrix和distCoeffs：姿态估计所需的相机标定参数。
-	该函数返回值用于估计板姿态的标记总数。需要注意，由于仅考虑了其ID在Board::ids结构中列出的标记，因此不应使用markerCorners和markerlds中提供的所有标记。

drawAxis()函数可用于绘制获得的姿态，绘制结果如图2-14所示。即使标记版存在部分遮挡情况，依然可以估计姿态并绘制坐标轴，具体如图2-15所示。
<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223142039640.jpg" height="300">
</p>

## 2.2.2	标记板生成
创建标记板对象需要指定环境中每个标记的角位置。但是，在很多情况下，标记板只是在同一平面和网格布局中的一组标记。aruco模块提供了可以轻松创建并打印这些类型标记的相关函数。GridBoard类是一个继承自Board类的类，它代表一个Board及其所有标记都位于同一平面和网格布局中，如图2-13所示。

具体来说，标记板中的坐标系位于板平面中，坐标原点位于标记板左下角，同时Z轴指向外，具体形式如图2-14中所示（X轴：红色；Y轴：绿色；Z轴：蓝色）。

可以通过调整GirdBoard类型的参数来调整标记板样式，具体参数为：
-	X轴方向上的标识数；
-	Y轴方向上的标识数；
-	标识的边长；
-	标识的间隔长度；
-	标识的字典；
-	所有标识（X×Y标识）的ID。

在GirdBoard类中提供了cv::aruco::GridBoard::create()静态函数用于上述参数的设置，该函数具体使用方法在代码清单2-17中给出。
```cpp
代码清单2-17 创建GirdBoard对象
cv::aruco::GridBoard board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
```
-	第一和第二参数分别是在X轴和Y轴方向上的标记数；
-	第三和第四参数分别是标记边长和标记间隔。考虑到该板的估计姿态将以相同的单位进行测量，因此可以以任何单位提供它们（通常使用米）。
-	最后，提供了标记的字典。

代码清单2-17中的代码创建的标记板由5×7=35个标识组成。默认情况下，每个标识的ID均以从0开始的升序分配，因此他们的编号为0、1、2，…，34。可以通过从Board.ids访问ID向量来轻松地对其进行自定义。

在创建标记板之后，接下来将打印并使用它。cv::aruco::GridBoard::draw()中提供了生成GridBoard图像的功能，具体方法在代码清单2-18中给出。
```cpp
代码清单2-18 生成GridBoard图像
cv::Ptr<cv::aruco::GridBoard> board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
cv::Mat boardImage;
board->draw( cv::Size(600, 500), boardImage, 10, 1 );
```
-	第一个参数是输出图像的大小（以像素为单位）。在这种情况下为600x500像素。如果这与（标识）板尺寸不成比例，它将在图像上居中。
-	boardImage：带有板的输出图像。
-	第三个参数是（可选）以像素为单位的边距，因此所有标记都没有触及图像边界。在这种情况下，边距为10。
-	最后，标记边框的大小，类似于drawMarker()函数。预设值为1。

使用代码清单2-18中的代码将输入如图2-16所示的标记板。为了方便用户使用，aruco模块中提供了创建标记板的示例程序create_board.cpp，可以直接调用创建标记板。该程序调用方式在代码清单2-19中给出。

```cpp
代码清单2-19：调用示例程序
"_output path_/aboard.png" -w=5 -h=7 -l=100 -s=10 -d=10
```

<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223162544554.jpg" height="300">
</p>
 
```cpp
接下来在代码清单2-20中给出检测标记板并估计位姿完整的示例程序。
代码清单1-24 板检测样例
cv::VideoCapture inputVideo;
inputVideo.open(0);
cv::Mat cameraMatrix, distCoeffs;
// camera parameters are read from somewhere
readCameraParameters(cameraMatrix, distCoeffs);
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::Ptr<cv::aruco::GridBoard> board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
while (inputVideo.grab()) {
    cv::Mat image, imageCopy;
    inputVideo.retrieve(image);
    image.copyTo(imageCopy);
    std::vector<int> ids;
    std::vector<std::vector<cv::Point2f> > corners;
    cv::aruco::detectMarkers(image, dictionary, corners, ids);
    // if at least one marker detected
    if (ids.size() > 0) {
        cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
        cv::Vec3d rvec, tvec;
        int valid = estimatePoseBoard(corners, ids, board, cameraMatrix, distCoeffs, rvec, tvec);
        // if at least one board marker detected
        if(valid > 0)
            cv::aruco::drawAxis(imageCopy, cameraMatrix, distCoeffs, rvec, tvec, 0.1);
    }
    cv::imshow("out", imageCopy);
    char key = (char) cv::waitKey(waitTime);
    if (key == 27)
        break;
}
```

为了方便使用，OpenCV在aruco模块中提供了detect_board.cpp文件，可以调用实现标记板的检测与位姿估计。使用该文件的参数输入在代码清单2-21中给出。
```cpp
代码清单2-21
-c="_path_"/calib.txt" "_path_/aboard.png" -w=5 -h=7 -l=100 -s=10 -d=10
```
标记板检测视频的截图在图2-17给出。
<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223163022569.jpg" height="300">
</p>

**提示**
***完整视频可以在小白学视觉微信公众号后台回复“ArUco标记板检测”获取。***

## 2.2.3	增强标识检测
如果我们已经知道标记板的布局信息，则可以利用已有信息对存在遮挡的标记板进行增强，记找到标记版中没有被识别的标记。opencv提供了fineDetectedMarkers()函数来实现这个步骤，但是需要注意的是，我们应该在执行detectMarkers()函数之后再执行这个函数。

此功能的主要参数是检测到标记的原始图像、Board对象、检测到的标记的角点、检测到的标识的id和被拒标记的角点。我们可以从detectMarkers()函数获得被拒标记的角点，也被称为候选标记。这些候选对象是在原始图像中找到但未通过识别步骤的正方形（可能它们的内部编码存在太多错误），因此尚未被识别为标记。

由于图像中的高频噪声、低分辨率或影响二进制代码提取的其他相关问题，这些候选对象有时候无法正确识别为实际标记。函数fineDetectedMarkers()可查找这些候选标记与标记板被遮挡标记之间的对应关系。该搜索函数基于以下两个信息：
-	候选对象与缺失标记的投影之间的距离。为了获得这些投影，有必要检测检测至少一个在板中的标记。如果提供了相机参数，则可使用相机参数获得投影。如果没有，则从局部单应性变换得出，但是这种情况仅允许使用平面板（即所有标记的角点的Z轴坐标都应相同）。
fineDetectedMarkers()中的minRepDistance参数确定侯选角和投影标识角之间的最小欧几里得距离（默认值为10）。
-	二进制编码。如果候选对象超过最小距离条件，则再次分析其内部，以确定其是否确为投影标识。然而，在这种情况下，条件不是特别强并且允许的错误位数可能会更多。fineDetectedMarkers()函数中的参数errorCorrectionRate（默认值3.0）表示二进制编码。如果设置为负值，则根本不分析内部位而仅估计角点之间的距离。

refineDetectedMarkers()函数的使用方式再代码清单2-22中给出。
```cpp
代码清单2-22 标识检测完善示例
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::Ptr<cv::aruco::GridBoard> board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
std::vector<int> markerIds;
std::vector<std::vector<cv::Point2f>> markerCorners, rejectedCandidates;
cv::aruco::detectMarkers(inputImage, dictionary, markerCorners, markerIds, cv::aruco::DetectorParameters(), rejectedCandidates);
cv::aruco::refineDetectedMarkersinputImage, board, markerCorners, markerIds, rejectedCandidates);
// 调用此函数后，如果检测到任何新标识，它将从被拒的候选对象中删除，
// 并将其包含在markerCorners和MarkerIds的末尾
```
还必须注意，在某些情况下，如果首先检测到的标记数过少（例如仅1个或2个标记），则丢失标记的投影质量可能很差，从而产生错误的对应关系。
