ArUco标记板是一组如图2-13所示的标志。在为相机提供单个姿势上，他们就像单个标记一样工作。常用的方式是所有标记都在同一平面上的标志板，因为我们可以直接打印出来。但是，标记板不限于这种布置，还可以用2维或3维布局表示。

 <center>
 <img src="https://img-blog.csdnimg.cn/20200223141221760.jpg">
</center>

板和一组独立标记之间的区别在于，板中标记之间的相对位置是先验的。这样可以将所有标记的角点用于估计摄像机相对于整个标记板的姿态。当我们使用一组独立的标记时，由于我们不知道标记在环境中的相对位置，因此可以分别估计每个标记的姿态。

使用标记板的最主要好处有：
-	姿态估计更加通用。执行姿态估计仅需要一部分标记。因此，即便存在一些遮挡或仅是局部视图，也可以估算出姿态。
-	由于采用了大量的点对应关系标记角点，因此获得的姿态通常更加准确。

Aruco模块允许使用标记板，使用的是cv::aruco::Board类，它用代码清单2-15中的方式定义了板的布局。
```cpp
代码清单2-15 图像修复
1.	class Board {
2.	public:
3.	    std::vector<std::vector<cv::Point3f> > objPoints;
4.	    cv::Ptr<cv::aruco::Dictionary> dictionary;
5.	    std::vector<int> ids;
6.	};
```

Board类对象具有三个参数：
-	ObjPoints结构是在三维板参考系统中角点位置的列表，即其布局。对于每个标记，其四个角点均以标准顺序储存，即以顺时针顺序并从左上角开始存储。
-	字典参数指示板中标记属于哪个标识字典。
-	最后，ids结构指示objPoints中相对于指定字典的每一个标记的标记符。

## 2.2.1	标志板检测
Aruco模块提供了一个特定的函数estimatePoseBoard()用于执行标记板的姿态估计，该函数的使用方式在代码清单2-16中给出。
```cpp 
代码清单2-16 板姿态估计
1.	cv::Mat inputImage;
2.	// 从某处读取相机参数
3.	cv::Mat cameraMatrix, distCoeffs;
4.	readCameraParameters(cameraMatrix, distCoeffs);
5.	// 假设我们有一个创建板对象的函数
6.	cv::Ptr<cv::aruco::Board> board = cv::aruco::Board::create();
7.	...
8.	std::vector<int> markerIds;
9.	std::vector<std::vector<cv::Point2f>> markerCorners;
10.	cv::aruco::detectMarkers(inputImage, board.dictionary, markerCorners, markerIds);
11.	// 如果至少检测到一个标识
12.	if(markerIds.size() > 0) {
13.	cv::Vec3d rvec, tvec;
14.	int valid = cv::aruco::estimatePoseBoard(markerCorners, markerIds, board, cameraMatrix, distCoeffs, rvec, tvec);
15.	}

```
estimatePoseBoard()函数的每个参数含义为：

-	makerCorners和markerlds：从detectMarkers()函数检测到的标记的结构。
-	board：定义板布局及其ID。
-	cameraMatrix和distCoeffs：姿态估计所需的相机标定参数。
-	该函数返回值用于估计板姿态的标记总数。需要注意，由于仅考虑了其ID在Board::ids结构中列出的标记，因此不应使用markerCorners和markerlds中提供的所有标记。

drawAxis()函数可用于绘制获得的姿态，绘制结果如图2-14所示。即使标记版存在部分遮挡情况，依然可以估计姿态并绘制坐标轴，具体如图2-15所示。
 <center>
 <img src="https://img-blog.csdnimg.cn/20200223142039640.jpg">
</center>

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
1.	cv::aruco::GridBoard board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
```
-	第一和第二参数分别是在X轴和Y轴方向上的标记数；
-	第三和第四参数分别是标记边长和标记间隔。考虑到该板的估计姿态将以相同的单位进行测量，因此可以以任何单位提供它们（通常使用米）。
-	最后，提供了标记的字典。

代码清单2-17中的代码创建的标记板由5×7=35个标识组成。默认情况下，每个标识的ID均以从0开始的升序分配，因此他们的编号为0、1、2，…，34。可以通过从Board.ids访问ID向量来轻松地对其进行自定义。

在创建标记板之后，接下来将打印并使用它。cv::aruco::GridBoard::draw()中提供了生成GridBoard图像的功能，具体方法在代码清单2-18中给出。
```cpp
代码清单2-18 生成GridBoard图像
1.	cv::Ptr<cv::aruco::GridBoard> board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
2.	cv::Mat boardImage;
3.	board->draw( cv::Size(600, 500), boardImage, 10, 1 );
```
-	第一个参数是输出图像的大小（以像素为单位）。在这种情况下为600x500像素。如果这与（标识）板尺寸不成比例，它将在图像上居中。
-	boardImage：带有板的输出图像。
-	第三个参数是（可选）以像素为单位的边距，因此所有标记都没有触及图像边界。在这种情况下，边距为10。
-	最后，标记边框的大小，类似于drawMarker()函数。预设值为1。

使用代码清单2-18中的代码将输入如图2-16所示的标记板。为了方便用户使用，aruco模块中提供了创建标记板的示例程序create_board.cpp，可以直接调用创建标记板。该程序调用方式在代码清单2-19中给出。

```cpp
代码清单2-19：调用示例程序
1.	"_output path_/aboard.png" -w=5 -h=7 -l=100 -s=10 -d=10
```

<center>
 <img src="https://img-blog.csdnimg.cn/20200223162544554.jpg">
</center>
 
```cpp
接下来在代码清单2-20中给出检测标记板并估计位姿完整的示例程序。
代码清单1-24 板检测样例
1.	cv::VideoCapture inputVideo;
2.	inputVideo.open(0);
3.	cv::Mat cameraMatrix, distCoeffs;
4.	// camera parameters are read from somewhere
5.	readCameraParameters(cameraMatrix, distCoeffs);
6.	cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
7.	cv::Ptr<cv::aruco::GridBoard> board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
8.	while (inputVideo.grab()) {
9.	    cv::Mat image, imageCopy;
10.	    inputVideo.retrieve(image);
11.	    image.copyTo(imageCopy);
12.	    std::vector<int> ids;
13.	    std::vector<std::vector<cv::Point2f> > corners;
14.	    cv::aruco::detectMarkers(image, dictionary, corners, ids);
15.	    // if at least one marker detected
16.	    if (ids.size() > 0) {
17.	        cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
18.	        cv::Vec3d rvec, tvec;
19.	        int valid = estimatePoseBoard(corners, ids, board, cameraMatrix, distCoeffs, rvec, tvec);
20.	        // if at least one board marker detected
21.	        if(valid > 0)
22.	            cv::aruco::drawAxis(imageCopy, cameraMatrix, distCoeffs, rvec, tvec, 0.1);
23.	    }
24.	    cv::imshow("out", imageCopy);
25.	    char key = (char) cv::waitKey(waitTime);
26.	    if (key == 27)
27.	        break;
28.	}
```

为了方便使用，OpenCV在aruco模块中提供了detect_board.cpp文件，可以调用实现标记板的检测与位姿估计。使用该文件的参数输入在代码清单2-21中给出。
```cpp
代码清单2-21
1.	-c="_path_"/calib.txt" "_path_/aboard.png" -w=5 -h=7 -l=100 -s=10 -d=10
```
标记板检测视频的截图在图2-17给出。
 <center>
 <img src="https://img-blog.csdnimg.cn/20200223163022569.jpg">
</center>

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
1.	cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
2.	cv::Ptr<cv::aruco::GridBoard> board = cv::aruco::GridBoard::create(5, 7, 0.04, 0.01, dictionary);
3.	std::vector<int> markerIds;
4.	std::vector<std::vector<cv::Point2f>> markerCorners, rejectedCandidates;
5.	cv::aruco::detectMarkers(inputImage, dictionary, markerCorners, markerIds, cv::aruco::DetectorParameters(), rejectedCandidates);
6.	cv::aruco::refineDetectedMarkersinputImage, board, markerCorners, markerIds, rejectedCandidates);
7.	// 调用此函数后，如果检测到任何新标识，它将从被拒的候选对象中删除，
8.	// 并将其包含在markerCorners和MarkerIds的末尾
```
还必须注意，在某些情况下，如果首先检测到的标记数过少（例如仅1个或2个标记），则丢失标记的投影质量可能很差，从而产生错误的对应关系。
# 2.3	ChArUco角的检测
ArUco标识和标记板由于其快速检测和多功能性而非常有用。然而，ArUco标记存在即便是在应用亚像素细化之后，其角点的位置精度也不太高的问题。相反，如果每个角点都被两个黑色正方形包围，那么棋盘图案的角点可以更精确地细化。但是，找到棋盘图案并不像找到ArUco板那样通用，它要求它必须完全可见，并且不允许有任何遮挡。ChArUco标记板试图将这两种方法的优点结合起来，具体形式如图2-18所示。

 <center>
 <img src="https://img-blog.csdnimg.cn/2020022316374383.jpg">
</center>

ArUco标记用于插补棋盘的角点的位置，因此它允许遮挡或使用局部视图，故其具有标记板的通用性。而且由于内插的角点属于棋盘，因此就压像素精度而言它们非常精确。当在需要高精度场合时，例如在相机校准中，ChArUco标记板比标准ArUco标记板更好。

## 2.3.1	创建ChArUco标志板
aruco模块提供了cv::aruco::CharucoBoard类代表Charuco标记板，并继承自Board类。此类被定义在<opencv2/aruco/charuco.hpp>头文件中，在使用时需要将该头文件包含进去。该类中需要如下信息：
-	X轴方向上的棋盘方块数
-	Y轴方向上的棋盘方块数
-	方块的边长
-	标识的边长
-	标识的字典
-	标识的ID

aruco模块提供了创建CharucoBoard类的静态函数cv::aruco::CharucoBoard::create()，该函数的使用方式在代码清单2-23中给出
```cpp
代码清单2-23 创建Charuco Board
1.	cv::aruco::CharucoBoard board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
```
-	第一个和第二个参数分别是X轴、Y轴方向上的正方形数。
-	第三个和第四个参数分别是正方形和标记的边长。考虑到估计姿态时将以相同的单位进行测量（通常使用米），因此可以使用任何单位。
-	最后一个参数提供了标记的字典。

在默认情况下，每个标识的ID都如GirdBoard::create()一样从0开始按升序分配。也如同Board父类一样，可以通过board.ids访问ID向量，从而轻松地对其进行自定义。一旦有了CharucoBoard对象，我们就能创建图像并进行打印。我们可以通过CharucoBoard::draw()函数实现生成图像，该函数的使用方式在代码清单2-24中给出。
```cpp
代码清单2-24 创建图像
1.	cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
2.	cv::Mat boardImage;
3.	board->draw( cv::Size(600, 500), boardImage, 10, 1 );
```
- 第一个参数是输出图像的大小（以像素为单位）。在示例中为600×500像素。
-	boardImage：带有标记板的输出图像；
-	第三个参数（可选）是以像素为单位的边距，在示例中，边距为10；
-	最后是标识边框的大小，类似于函数drawMarker()。预设值为1。

调用代码清单2-24中的程序，输出图像如图2-19所示。
 <center>
 <img src="https://img-blog.csdnimg.cn/20200223164241928.jpg">
</center>
 
    OpenCV中aruco模块提供了create_board_charuco.cpp文件，文件中含有生成ChArUco标记板的完整程序，使用该程序需要出入代码清单2-25中的参数。
```cpp
代码清单2-25
1.	"_ output path_/chboard.png" -w=5 -h=7 -sl=200 -ml=120 -d=10
```
## 2.3.2	ChArUco标记板检测
当我们检测一个ChArUco标记板时，实际检测到的是板上的每个棋盘角点。ChArUco板上的每个角点都分配有唯一的标识符（id）。这些id从0到板上角点的总数。所以，ChArUco板的检测由以下几部分组成：

-	std::vector<cv::Point2f> charucoCorners:检测到的角的图像位置列表；
-	std::vector<int> charucoIds:在charucoCorners中的每个检测到的角的ID。

ChArUco角点的检测基于先前检测到的标识。因此，首先检测标识，然后在标识中插入ChArUco角点。aruco模块中提供了cv::aruco::interpolateCornersCharuco()函数用于检测ChArUco角点，该函数的的使用方式在代码清单2-26中给出。
```cpp
代码清单2-26 检测ChArUco角点
1.	cv::Mat inputImage;
2.	cv::Mat cameraMatrix, distCoeffs;
3.	// 从某处读取相机参数
4.	readCameraParameters(cameraMatrix, distCoeffs);
5.	cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
6.	cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
7.	...
8.	std::vector<int> markerIds;
9.	std::vector<std::vector<cv::Point2f>> markerCorners;
10.	cv::aruco::detectMarkers(inputImage, board.dictionary, markerCorners, markerIds);
11.	// 如果至少检测到一个标识
12.	if(markerIds.size() > 0) {
13.	std::vector<cv::Point2f> charucoCorners;
14.	std::vector<int> charucoIds;
15.	cv::aruco::interpolateCornersCharuco(markerCorners, markerIds, inputImage, board, charucoCorners, charucoIds, cameraMatrix, distCoeffs);
16.	}
```
interpolateCornersCharuco()函数的每个参数的含义为：

-	markerCorners和markerIds：从函数detectMarkers()检测到的标识；
-	inputImage：检测到标识的原始图像。若要在ChArUco角中进行亚像素化，则必须使用该图像；
-	board：CharucoBoard对象；
-	charucoCorners和charucoIds：输出的插入的Charuco角；
-	cameraMatrix和distCoeffs：可选的相机校准参数；
-	该函数返回插入的Charuco角的数目。

在代码清单-26中，我们调用interpolateCorneresCharuco()函数时提供了相机标定参数。但这些参数页可以不使用，不使用相机标定参数的示例程序在代码清单2-27中给出。
```cpp
代码清单1-32 （未使用相机校准参数）检测ChArUco角
1.	cv::Mat inputImage;
2.	cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
3.	cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
4.	...
5.	std::vector<int> markerIds;
6.	std::vector<std::vector<cv::Point2f>> markerCorners;
7.	cv::Ptr<cv::aruco::DetectorParameters> params;
8.	params->cornerRefinementMethod = cv::aruco::CORNER_REFINE_NONE;
9.	cv::aruco::detectMarkers(inputImage, board.dictionary, markerCorners, markerIds, params);
10.	// 如果至少有一个标识被检测到
11.	if(markerIds.size() > 0) {
12.	std::vector<cv::Point2f> charucoCorners;
13.	std::vector<int> charucoIds;
14.	cv::aruco::interpolateCornersCharuco(markerCorners, markerIds, inputImage, board, charucoCorners, charucoIds);
15.	}
```
如果提供了校准参数，则首先通过ArUco标记粗略估计姿态，然后将ArUco角点重新投影回图像来对ChArUco角点进行插值。如果未提供校准参数，则可以通过计算ChArUco平面和ChArUco图像投影之间对应的单应性变换来对ChArUco角点进行插值。使用单应性变换的主要问题是插值对图像失真更敏感。实际上，在使用每个ChArUco角点的最接近的标记以减少失真效果时，才会执行单应性变换。

当检测ChArUco板的标识时，尤其是在使用单应性变换时，建议禁用标记的边角细化。因为棋盘方块的形状相似，压像素处理会在拐角位置产生较大的偏差，并且这些偏差会传递到ChArUco角点的插值中，从而得到较差的结果。

此外，仅返回那些找到两个周围标记的角。如果未检测到两个周围标记中的任何一个，则通常意味着存在某些遮挡或该区域的图像质量不佳。无论哪用情况，最好不要考虑这种角点，因为我们要确保插入的ChArUco角点非常准确。在对ChArUco角点进行插值后，再执行压像素细化。一旦我们插入ChArUco角后，我们可能希望绘制它们以查看检测是否正确。OpenCV提供了drawDetectedCornersCharuco()函数用于显示检测结果，该函数的使用方式在代码清单2-28中给出。检测结果如图2-20和图2-21所示。
```cpp
代码清单2-28绘制
1.	cv::aruco::drawDetectedCornersCharuco(image, charucoCorners, charucoIds, color);
```

-	image是将要绘制角点的图像（通常与检测到角点图像相同），同时也是绘制后的输出图像；
-	charucoCorners和charucoIds是从函数interpolateCornersCharuco()中检测到的Charuco角点；
-	最后一个参数（可选）是我们要用来绘制角的颜色，类型为cv::Scalar。
 
<center>
 <img src="https://img-blog.csdnimg.cn/20200223165150456.jpg">
</center> 

不使用相机标定参数情况下对ChArUco标记板检测的示例程序在代码清单2-29中给出。
```cpp
代码清单2-29 ChArUco检测的完整示例
1.	cv::VideoCapture inputVideo;
2.	inputVideo.open(0);
3.	cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
4.	cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
5.	cv::Ptr<cv::aruco::DetectorParameters> params;
6.	params->cornerRefinementMethod = cv::aruco::CORNER_REFINE_NONE;
7.	while (inputVideo.grab()) {
8.	    cv::Mat image, imageCopy;
9.	    inputVideo.retrieve(image);
10.	    image.copyTo(imageCopy);
11.	    std::vector<int> ids;
12.	    std::vector<std::vector<cv::Point2f>> corners;
13.	    cv::aruco::detectMarkers(image, dictionary, corners, ids, params);
14.	    // if at least one marker detected
15.	    if (ids.size() > 0) {
16.	        cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
17.	        std::vector<cv::Point2f> charucoCorners;
18.	        std::vector<int> charucoIds;
19.	        cv::aruco::interpolateCornersCharuco(corners, ids, image, board, charucoCorners, charucoIds);
20.	        // if at least one charuco corner detected
21.	        if(charucoIds.size() > 0)
22.	            cv::aruco::drawDetectedCornersCharuco(imageCopy, charucoCorners, charucoIds, cv::Scalar(255, 0, 0));
23.	    }
24.	    cv::imshow("out", imageCopy);
25.	    char key = (char) cv::waitKey(waitTime);
26.	    if (key == 27)
27.	        break;
28.	}
```
aruco模块中提供了检测ChArUco标记板的完整的示例程序，完整程序存放在detect_board_charuco.cpp文件中，我们可以通过代码清单2-30中的代码直接调用该文件。检测ChArUco标记板的结果在图2-22给出。

**提示
*完整视频可以在小白学视觉微信公众号后台回复“ChArUco标记板检测”获取。***
```cpp
代码清单2-30 样本输入
1.	-c="_path_/calib.txt" -dp="_path_/detector_params.yml" -w=5 -h=7 -sl=0.04 -ml=0.02 -d=10
```
 <center>
 <img src="https://img-blog.csdnimg.cn/20200223165438347.jpg">
</center>


## 2.3.3	ChArUco姿态估计
ChArUco标记板的最终目的是非常精确地找到角点以进行高精度校准或姿态估计。aruco模块提供了estimatePoseCharucoBoard()函数用于ChArUco姿态估计。ChArUco标记板的坐标系与GridBoard一样，坐标系放置在板平面中，Z轴指向外侧，并居中于板的左下角。estimatePoseCharucoBoard()函数的使用方法在代码清单2-31中给出。
```cpp
代码清单2-31姿态估计函数
1.	cv::aruco::estimatePoseCharucoBoard(charucoCorners, charucoIds, board, cameraMatrix, distCoeffs, rvec, tvec);
```
-	charucoCoerners和charucoIds参数是从函数interpolateCornersCharuco()中检测到的charuco角点；
-	第三个参数是CharucoBoard对象；
-	cameraMatrix和distCoeffs是姿态估计所需的相机标定参数；
-	rvec和tvec参数是Charuco板的输出姿态；
-	如果正确估计了姿态，则函数返回true，否则返回false。失败的主要原因是没有足够的角点进行姿态估计或者它们不在同一条线。

可以使用drawAxis()绘制轴轴以检查姿态是否正确估计。绘制结果如所示（X轴：红色；Y轴：绿色；Z轴：蓝色）。
 <center>
 <img src="https://img-blog.csdnimg.cn/20200223170110524.jpg">
</center>

代码清单2-33中给出了姿态估计完整示例程序。该程序被存放在aruco模块中的detect_board_charuco.cpp文件中，可以通过代码清单2-32中的参数对该文件进行调用。
```cpp
代码清单2-32 样本输入
1.	"_path_/calib.txt" -dp="_path_/detector_params.yml" -w=5 -h=7 -sl=0.04 -ml=0.02 -d=10
```
```cpp
代码清单2-33：带有姿态估计的ChArUco检测
1.	cv::VideoCapture inputVideo;
2.	inputVideo.open(0);
3.	cv::Mat cameraMatrix, distCoeffs;
4.	// camera parameters are read from somewhere
5.	readCameraParameters(cameraMatrix, distCoeffs);
6.	cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
7.	cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
8.	while (inputVideo.grab()) {
9.	    cv::Mat image, imageCopy;
10.	    inputVideo.retrieve(image);
11.	    image.copyTo(imageCopy);
12.	    std::vector<int> ids;
13.	    std::vector<std::vector<cv::Point2f>> corners;
14.	    cv::aruco::detectMarkers(image, dictionary, corners, ids);
15.	    // if at least one marker detected
16.	    if (ids.size() > 0) {
17.	        std::vector<cv::Point2f> charucoCorners;
18.	        std::vector<int> charucoIds;
19.	        cv::aruco::interpolateCornersCharuco(corners, ids, image, board, charucoCorners, charucoIds, cameraMatrix, distCoeffs);
20.	        // if at least one charuco corner detected
21.	        if(charucoIds.size() > 0) {
22.	            cv::aruco::drawDetectedCornersCharuco(imageCopy, charucoCorners, charucoIds, cv::Scalar(255, 0, 0));
23.	            cv::Vec3d rvec, tvec;
24.	            bool valid = cv::aruco::estimatePoseCharucoBoard(charucoCorners, charucoIds, board, cameraMatrix, distCoeffs, rvec, tvec);
25.	            // if charuco pose is valid
26.	            if(valid)
27.	                cv::aruco::drawAxis(imageCopy, cameraMatrix, distCoeffs, rvec, tvec, 0.1);
28.	        }
29.	    }
30.	    cv::imshow("out", imageCopy);
31.	    char key = (char) cv::waitKey(waitTime);
32.	    if (key == 27)
33.	        break;
34.	}
```
