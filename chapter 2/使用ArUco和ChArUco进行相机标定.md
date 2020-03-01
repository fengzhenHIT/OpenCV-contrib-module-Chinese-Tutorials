ArUco模块也可以用来标定相机。相机标定包括获取相机的内参系数和畸变系数。除非相机光学系统发生变化，否则这一参数保持不变。因此，相机标定只需要做一次。但是，如果相机长时间未使用，建议再次使用时重新标定一次。

相机机标定通常使用标准库中的calibrateCamera()函数。该函数要求空间点与图像中投影点之间存在一定的对应关系。一般来说，这些对应是从棋盘图案的角点获得的。有关更多详细信息，可以阅读基础库中的calibrateCamera()函数文档或OpenCV标定教程。在《OpenCV 4计算机视觉编程实战》书中也有详细的标定算法和流程介绍。

使用ArUco模块，可以基于ArUco标记角点或ChArUco角点进行标定。与使用传统棋盘图案相比，使用ArUco进行标定的功能要更广泛得，因为即使部分标定板被遮挡了，一样可以实现相机的标定。

可以同时使用ArUco标记角点或ChArUco角点进行标定。但是，强烈推荐使用ChArUco的方法，因为它提供的角点更精确。但是如果由于限制而无法使用ChArUco Board的情况下，也可以使用ArUco标定板板进行标定。

## 2.5.1	使用 ChArUco Boards 进行标定
要使用ChArUco进行标定，必须获得不同的角度的棋盘图像，这与传统标定方法对棋盘格的检测方法相同。使用ChArUco的好处是即使标定板部分被遮挡，只要任然有角点可以被检测，就可以实现标定。标定过程中获取的含有ChArUco标定板的图像如图2-28所示。
 <center>
<img src="https://img-blog.csdnimg.cn/20200224113108960.png">
</center>

使用ChArUco标定板进行标定的函数是calibrateCameraCharuco()，其使用方法在代码清单2-40给出。

```cpp
代码清单2-40 创建Mat类
cv::Ptr<aruco::CharucoBoard> board = ... // create charuco board
cv::Size imgSize = ... // camera image size
std::vector<std::vector<cv::Point2f>> allCharucoCorners;
std::vector<std::vector<int>> allCharucoIds;
// Detect charuco board from several viewpoints and fill allCharucoCorners and allCharucoIds
...
...
// After capturing in several viewpoints, start calibration
cv::Mat cameraMatrix, distCoeffs;
std::vector<cv::Mat> rvecs, tvecs;
int calibrationFlags = ... // Set calibration flags (same than in calibrateCamera() function)
double repError = cv::aruco::calibrateCameraCharuco(allCharucoCorners, allCharucoIds, board, imgSize, cameraMatrix, distCoeffs, rvecs, tvecs, calibrationFlags);
```
在每个视角上捕获的ChArUco角点和ChArUco标识符存储在vector容器allCharucoCorners和allcharucoid中，每个视角一个元素。

calibrateCameraCharuco()函数将相机参数存储在数组cameraMatrix和distCoeffs中。该函数的返回值是标定的重投影误差。rvecs和tvecs中存储每幅图对应的相机姿态(相对于ChArUco板)。最后，calibrationFlags参数确定一些用于标定的选项。它的格式与基础库中calibrateCamera()函数中的flags参数格式相同。

完整的工作示例包含在模块示例文件夹中的calibrate_camera_charuco.cpp中，读者可以在扩展模块的文件中找到它。该函数调用时需要输入的参数在代码清单2-41中给出。

```cpp
代码清单2-41 ：调用参数
_output path_" -dp="_path_/detector_params.yml" -w=5 -h=7 -sl=0.04 -ml=0.02 -d=10
```

## 2.5.2	使用 ArUco Boards 进行标定
如前所述，因为ChArUco角点比更准确ChAruco角点更精确，因此建议使用ChAruco标定板进行相机标定。进行摄像机校准，。但是在某些特殊情况下不得不使用ArUco标定板进行标定。使用ArUco标定板时，需要使用calibrateCameraAruco()函数。同样该函数也需要输入不同角度拍摄到的ArUco标定板图像。标定过程中获取的含有ArUco标定板的图像如图2-29所示

 <center>
<img src="https://img-blog.csdnimg.cn/20200224113411996.png">
</center>

calibrateCameraAruco()函数的使用方法在代码清单2-42中给出。
```cpp
代码清单2-42 创建Mat类
cv::Ptr<aruco::Board> board = ... // create aruco board
cv::Size imgSize = ... // camera image size
std::vector<std::vector<cv::Point2f>> allCornersConcatenated;
std::vector<int> allIdsConcatenated;
std::vector<int> markerCounterPerFrame;
// Detect aruco board from several viewpoints and fill allCornersConcatenated, allIdsConcatenated and markerCounterPerFrame
...
...
// After capturing in several viewpoints, start calibration
cv::Mat cameraMatrix, distCoeffs;
std::vector<cv::Mat> rvecs, tvecs;
int calibrationFlags = ... // Set calibration flags (same than in calibrateCamera() function)
double repError = cv::aruco::calibrateCameraAruco(allCornersConcatenated, allIdsConcatenated, markerCounterPerFrame, board, imgSize, cameraMatrix, distCoeffs, rvecs, tvecs, calibrationFlags);
```

与calibrateCameraCharuco（）函数相反，在每幅图中检测到的标记将被串联存储在数组allCornersConcatenated和allCornersConcatenated（该函数的前两个参数）中。第三个参数（数组markerCounterPerFrame）指示在每幅图中检测到的标记数。

完整的工作示例包含在模块示例文件夹中的calibrate_camera.cpp中，读者可以在扩展模块的文件中找到它。该函数调用时需要输入的参数在代码清单2-43中给出。

```cpp
代码清单2-43 ：调用参数
"_path_/calib.txt" -w=5 -h=7 -l=100 -s=10 -d=10
```

