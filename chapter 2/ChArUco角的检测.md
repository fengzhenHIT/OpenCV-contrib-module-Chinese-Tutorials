ArUco标识和标记板由于其快速检测和多功能性而非常有用。然而，ArUco标记存在即便是在应用亚像素细化之后，其角点的位置精度也不太高的问题。相反，如果每个角点都被两个黑色正方形包围，那么棋盘图案的角点可以更精确地细化。但是，找到棋盘图案并不像找到ArUco板那样通用，它要求它必须完全可见，并且不允许有任何遮挡。ChArUco标记板试图将这两种方法的优点结合起来，具体形式如图2-18所示。

<p align="center">
 <img src="https://img-blog.csdnimg.cn/2020022316374383.jpg" height="250">
</p>

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
cv::aruco::CharucoBoard board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
```
-	第一个和第二个参数分别是X轴、Y轴方向上的正方形数。
-	第三个和第四个参数分别是正方形和标记的边长。考虑到估计姿态时将以相同的单位进行测量（通常使用米），因此可以使用任何单位。
-	最后一个参数提供了标记的字典。

在默认情况下，每个标识的ID都如GirdBoard::create()一样从0开始按升序分配。也如同Board父类一样，可以通过board.ids访问ID向量，从而轻松地对其进行自定义。一旦有了CharucoBoard对象，我们就能创建图像并进行打印。我们可以通过CharucoBoard::draw()函数实现生成图像，该函数的使用方式在代码清单2-24中给出。
```cpp
代码清单2-24 创建图像
cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
cv::Mat boardImage;
board->draw( cv::Size(600, 500), boardImage, 10, 1 );
```
- 第一个参数是输出图像的大小（以像素为单位）。在示例中为600×500像素。
-	boardImage：带有标记板的输出图像；
-	第三个参数（可选）是以像素为单位的边距，在示例中，边距为10；
-	最后是标识边框的大小，类似于函数drawMarker()。预设值为1。

调用代码清单2-24中的程序，输出图像如图2-19所示。
<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223164241928.jpg" height="300">
</p>
 
    OpenCV中aruco模块提供了create_board_charuco.cpp文件，文件中含有生成ChArUco标记板的完整程序，使用该程序需要出入代码清单2-25中的参数。
```cpp
代码清单2-25
"_ output path_/chboard.png" -w=5 -h=7 -sl=200 -ml=120 -d=10
```
## 2.3.2	ChArUco标记板检测
当我们检测一个ChArUco标记板时，实际检测到的是板上的每个棋盘角点。ChArUco板上的每个角点都分配有唯一的标识符（id）。这些id从0到板上角点的总数。所以，ChArUco板的检测由以下几部分组成：

-	std::vector<cv::Point2f> charucoCorners:检测到的角的图像位置列表；
-	std::vector<int> charucoIds:在charucoCorners中的每个检测到的角的ID。

ChArUco角点的检测基于先前检测到的标识。因此，首先检测标识，然后在标识中插入ChArUco角点。aruco模块中提供了cv::aruco::interpolateCornersCharuco()函数用于检测ChArUco角点，该函数的的使用方式在代码清单2-26中给出。
```cpp
代码清单2-26 检测ChArUco角点
cv::Mat inputImage;
cv::Mat cameraMatrix, distCoeffs;
// 从某处读取相机参数
readCameraParameters(cameraMatrix, distCoeffs);
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
...
std::vector<int> markerIds;
std::vector<std::vector<cv::Point2f>> markerCorners;
cv::aruco::detectMarkers(inputImage, board.dictionary, markerCorners, markerIds);
// 如果至少检测到一个标识
if(markerIds.size() > 0) {
    std::vector<cv::Point2f> charucoCorners;
    std::vector<int> charucoIds;
    cv::aruco::interpolateCornersCharuco(markerCorners, markerIds, inputImage, board, charucoCorners, charucoIds, cameraMatrix, distCoeffs);
}
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
cv::Mat inputImage;
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
...
std::vector<int> markerIds;
std::vector<std::vector<cv::Point2f>> markerCorners;
cv::Ptr<cv::aruco::DetectorParameters> params;
params->cornerRefinementMethod = cv::aruco::CORNER_REFINE_NONE;
cv::aruco::detectMarkers(inputImage, board.dictionary, markerCorners, markerIds, params);
// 如果至少有一个标识被检测到
if(markerIds.size() > 0) {
    std::vector<cv::Point2f> charucoCorners;
    std::vector<int> charucoIds;
    cv::aruco::interpolateCornersCharuco(markerCorners, markerIds, inputImage, board, charucoCorners, charucoIds);
}
```
如果提供了校准参数，则首先通过ArUco标记粗略估计姿态，然后将ArUco角点重新投影回图像来对ChArUco角点进行插值。如果未提供校准参数，则可以通过计算ChArUco平面和ChArUco图像投影之间对应的单应性变换来对ChArUco角点进行插值。使用单应性变换的主要问题是插值对图像失真更敏感。实际上，在使用每个ChArUco角点的最接近的标记以减少失真效果时，才会执行单应性变换。

当检测ChArUco板的标识时，尤其是在使用单应性变换时，建议禁用标记的边角细化。因为棋盘方块的形状相似，压像素处理会在拐角位置产生较大的偏差，并且这些偏差会传递到ChArUco角点的插值中，从而得到较差的结果。

此外，仅返回那些找到两个周围标记的角。如果未检测到两个周围标记中的任何一个，则通常意味着存在某些遮挡或该区域的图像质量不佳。无论哪用情况，最好不要考虑这种角点，因为我们要确保插入的ChArUco角点非常准确。在对ChArUco角点进行插值后，再执行压像素细化。一旦我们插入ChArUco角后，我们可能希望绘制它们以查看检测是否正确。OpenCV提供了drawDetectedCornersCharuco()函数用于显示检测结果，该函数的使用方式在代码清单2-28中给出。检测结果如图2-20和图2-21所示。
```cpp
代码清单2-28绘制
cv::aruco::drawDetectedCornersCharuco(image, charucoCorners, charucoIds, color);
```

-	image是将要绘制角点的图像（通常与检测到角点图像相同），同时也是绘制后的输出图像；
-	charucoCorners和charucoIds是从函数interpolateCornersCharuco()中检测到的Charuco角点；
-	最后一个参数（可选）是我们要用来绘制角的颜色，类型为cv::Scalar。
 
<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223165150456.jpg" height="300">
</p>

不使用相机标定参数情况下对ChArUco标记板检测的示例程序在代码清单2-29中给出。
```cpp
代码清单2-29 ChArUco检测的完整示例
cv::VideoCapture inputVideo;
inputVideo.open(0);
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
cv::Ptr<cv::aruco::DetectorParameters> params;
params->cornerRefinementMethod = cv::aruco::CORNER_REFINE_NONE;
while (inputVideo.grab()) {
    cv::Mat image, imageCopy;
    inputVideo.retrieve(image);
    image.copyTo(imageCopy);
    std::vector<int> ids;
    std::vector<std::vector<cv::Point2f>> corners;
    cv::aruco::detectMarkers(image, dictionary, corners, ids, params);
    // if at least one marker detected
    if (ids.size() > 0) {
        cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
        std::vector<cv::Point2f> charucoCorners;
        std::vector<int> charucoIds;
        cv::aruco::interpolateCornersCharuco(corners, ids, image, board, charucoCorners, charucoIds);
        // if at least one charuco corner detected
        if(charucoIds.size() > 0)
            cv::aruco::drawDetectedCornersCharuco(imageCopy, charucoCorners, charucoIds, cv::Scalar(255, 0, 0));
    }
    cv::imshow("out", imageCopy);
   char key = (char) cv::waitKey(waitTime);
    if (key == 27)
        break;
}
```
aruco模块中提供了检测ChArUco标记板的完整的示例程序，完整程序存放在detect_board_charuco.cpp文件中，我们可以通过代码清单2-30中的代码直接调用该文件。检测ChArUco标记板的结果在图2-22给出。

**提示
*完整视频可以在小白学视觉微信公众号后台回复“ChArUco标记板检测”获取。***
```cpp
代码清单2-30 样本输入
-c="_path_/calib.txt" -dp="_path_/detector_params.yml" -w=5 -h=7 -sl=0.04 -ml=0.02 -d=10
```
<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223165438347.jpg" height="300">
</p>


## 2.3.3	ChArUco姿态估计
ChArUco标记板的最终目的是非常精确地找到角点以进行高精度校准或姿态估计。aruco模块提供了estimatePoseCharucoBoard()函数用于ChArUco姿态估计。ChArUco标记板的坐标系与GridBoard一样，坐标系放置在板平面中，Z轴指向外侧，并居中于板的左下角。estimatePoseCharucoBoard()函数的使用方法在代码清单2-31中给出。
```cpp
代码清单2-31姿态估计函数
cv::aruco::estimatePoseCharucoBoard(charucoCorners, charucoIds, board, cameraMatrix, distCoeffs, rvec, tvec);
```
-	charucoCoerners和charucoIds参数是从函数interpolateCornersCharuco()中检测到的charuco角点；
-	第三个参数是CharucoBoard对象；
-	cameraMatrix和distCoeffs是姿态估计所需的相机标定参数；
-	rvec和tvec参数是Charuco板的输出姿态；
-	如果正确估计了姿态，则函数返回true，否则返回false。失败的主要原因是没有足够的角点进行姿态估计或者它们不在同一条线。

可以使用drawAxis()绘制轴轴以检查姿态是否正确估计。绘制结果如所示（X轴：红色；Y轴：绿色；Z轴：蓝色）。
<p align="center">
 <img src="https://img-blog.csdnimg.cn/20200223170110524.jpg" height="300">
</center>

代码清单2-33中给出了姿态估计完整示例程序。该程序被存放在aruco模块中的detect_board_charuco.cpp文件中，可以通过代码清单2-32中的参数对该文件进行调用。
```cpp
代码清单2-32 样本输入
"_path_/calib.txt" -dp="_path_/detector_params.yml" -w=5 -h=7 -sl=0.04 -ml=0.02 -d=10
```
```cpp
代码清单2-33：带有姿态估计的ChArUco检测
cv::VideoCapture inputVideo;
inputVideo.open(0);
cv::Mat cameraMatrix, distCoeffs;
// camera parameters are read from somewhere
readCameraParameters(cameraMatrix, distCoeffs);
cv::Ptr<cv::aruco::Dictionary> dictionary = cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
cv::Ptr<cv::aruco::CharucoBoard> board = cv::aruco::CharucoBoard::create(5, 7, 0.04, 0.02, dictionary);
while (inputVideo.grab()) {
    cv::Mat image, imageCopy;
    inputVideo.retrieve(image);
    image.copyTo(imageCopy);
    std::vector<int> ids;
    std::vector<std::vector<cv::Point2f>> corners;
    cv::aruco::detectMarkers(image, dictionary, corners, ids);
    // if at least one marker detected
    if (ids.size() > 0) {
        std::vector<cv::Point2f> charucoCorners;
        std::vector<int> charucoIds;
        cv::aruco::interpolateCornersCharuco(corners, ids, image, board, charucoCorners, charucoIds, cameraMatrix, distCoeffs);
        // if at least one charuco corner detected
        if(charucoIds.size() > 0) {
            cv::aruco::drawDetectedCornersCharuco(imageCopy, charucoCorners, charucoIds, cv::Scalar(255, 0, 0));
            cv::Vec3d rvec, tvec;
            bool valid = cv::aruco::estimatePoseCharucoBoard(charucoCorners, charucoIds, board, cameraMatrix, distCoeffs, rvec, tvec);
            // if charuco pose is valid
            if(valid)
                cv::aruco::drawAxis(imageCopy, cameraMatrix, distCoeffs, rvec, tvec, 0.1);
        }
    }
    cv::imshow("out", imageCopy);
    char key = (char) cv::waitKey(waitTime);
    if (key == 27)
        break;
}
```
