
姿态估计在许多计算机视觉应用程序中非常重要，如机器人导航，增强现实等。该过程基于发现真实环境中的点与2D图像投影之间的对应关系。 由于投影是从3D到2D，缺少了一个维度，使得位姿估计变成一个比较困难的问题，因此通常需要使用一些人为制作的标记或这基准标记来使增加信息量，以此使位姿估计更容易一些。

最受欢迎的方法之一是使用二进制方形基准标记。 这些标记的主要优点是单个标记可以提供足够的对应关系（四个角）来获得相机的姿态。 而且，内部二进制编码原理使得它们识别稳定，并且具有一定的容错性，从而允许应用错误检测和纠正技术。

aruco模块基于ArUco库，该库是由RafaelMuñoz和Sergio Garrido开发的用于检测方形基准标记的最受欢迎的库之一。

可以通过“#include <opencv2/aruco.hpp>”代码在程序中加入aruco模块中的相关函数。
## 2.1.1 标记和字典	
ArUco标记是由宽黑色边框和确定其标识符（id）的内部二进制矩阵组成的正方形标记。黑色边框有助于其在图像中的快速检测，内部二进制编码用于识别标记和提供错误检测和纠正。标记尺寸的大小决定内部矩阵的大小，例如尺寸为4x4的标记由16位组成二进制组成。在图2-1中给出了一些ArUco标记的示例。

 <center>
 <img src="https://img-blog.csdnimg.cn/20200223104948460.jpg">
 </center>
需要注意的是，在环境中ArUco标记可能会发生旋转或者倒着的情况，但是在检测时借助二进制编码能够确定其旋转的情况，以便明确标识每个角。因此ArUco标记不会出现中心对称和轴对称的图案。

标记字典是在具体应用时对标记的识别依据，它是对每个二进制编码及对应含义的存储。
字典具有两个主要特性，分别是字典大小和标记尺寸：

-	字典大小是组成字典中存储的标记数目。
-	标记尺寸是这些标记的位数。


aruco模块中具有一些预定义的词典，这些词典具有不同的字典大小和标记尺寸。

有些标记是将信息直接存储在二进制中，在读取时将二进制转换成十进制从而得到真实的数字信息。但是ArUco标记不是这样做的，因此如果标记的尺寸较大，，那么二进制数据将有较多的位数，管理较多的二进制数据在实时系统中是不实际的。但是，通过标记id来寻找字典中的信息将极大的缩减二进制位数。例如，字典中的前5个标记的id为：0、1、2、3和4，而这5个标记可以存储任意的数据。
## 2.1.2 创建标记
在检测到标记之前，需要先打印标记，然后将其放置在环境中。可以使用drawMarker()函数生成标记图像。例如可以通过代码清单2-1中的代码生成标记图像。
```cpp
代码清单2-1：生成标记图像

1.	cv::Mat markerImage;
2.	cv::Ptr<cv::aruco::Dictionary> dictionary = 
3.	               cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
4.	cv::aruco::drawMarker(dictionary, 23, 200, markerImage, 1)

```
在生成标记时，首先通过选择aruco模块中的预定义词典来创建Dictionary对象。代码清单2-1中创建的是一个具有250个标记和标记尺寸6×6位（DICT_6X6_250）字典。

绘制ArUCo标记的drawMarker()函数具有五个参数，每个参数的含义具体为：
-	第一个参数是之前创建的Dictionary对象。
-	第二个参数是标记id，代码清单2-1中设置的是字典DICT_6X6_250的标记23。需要注意的是，每个字典由不同数量的标记组成。在DICT_6X6_250字典中，有效id从0到249。任何超出有效范围的特定id都会产生异常。
-	第三个参数200是输出标记图像的大小。在这种情况下，输出图像的大小将为200x200像素。请注意，此参数应足够大以存储特定字典的位数。例如，对于6×6位的标记，我们无法生成5×5像素的图像（并且这不考虑标记边界）。此外，为避免变形，此参数应与位数+边界大小成比例，或者至少比标记大小大得多，以使变形不明显。
-	第四个参数是输出图像。
-	最后一个参数是一个可选参数，用于指定标记黑色边框的宽度。指定的大小与位数成正比。例如，值2表示边框的宽度等于两个内部位的大小。预设值为1。

代码清单2-1生成的图像在图2-2给出。
 
 <center>
<img src="https://img-blog.csdnimg.cn/20200223105957333.jpg">
</center>

aruco模块样例中提供了create_marker.cpp文件用于生成指定的ArRco标记，调用该函数时输入参数需要与代码清单2-2中的格式相同。
```cpp
代码清单2-2：create_marker.cpp的参数
"/Users/Sarthak/Dropbox/OpenCV_GSoC/marker.png" -d=10 -id=1
```
## 2.1.3 标记检测
对ArUco标记进行检测时，需要返回检测结果的一系列信息，包括：
-	图像中四个角的位置（按原始顺序）。
-	标记的ID。

标记检测过程主要分为两步：
1.	筛选候选标记。在此步中，将对图像进行分析，找到可以用作标记的正方形。首先从自适应阈值分割标记开始，然后从阈值图像中提取轮廓，之后去除那些非凸形或不近似于正方形的轮廓。此外还可以设置一些额外筛选条件，例如删除太小或太大的轮廓，删除彼此太近的轮廓等。
2.	在确定候选标记后，需要通过分析其内部编码来确定它们是否为ArUco标记。这步中需要提取每个标记的标记位。首先应用透视变换获得规范形式的标记，然后使用Otsu算法对规范图像进行阈值处理进行白色位和黑色位分离。根据标记大小和边框大小将图像划分为不同的单元，并对每个单元上的黑色或白色像素进行计数，以确定其是白色位还是黑色位。最后，分析这些像素位以确定标记是否属于特定词典，并在必要时使用纠错技术。

例如需要检测的图片如图2-3所示，检测出的标记如图2-4所示。候选标记在第二步被去除的情况如图2-5所示。
  <center>
<img src="https://img-blog.csdnimg.cn/20200223111435710.jpg">
</center>
<img src="https://img-blog.csdnimg.cn/20200223111650824.png">
</center>
<img src="https://img-blog.csdnimg.cn/2020022311195836.jpg">
</center>

在aruco模块中，detectMarkers()函数实现标志的检测。此函数是该模块中是最重要的函数，其他函数都是基于detectMarkers()函数的返回值进行再处理。代码清单2-3给出了检测标记的示例程序。
```cpp
代码清单2-3：检测标记
1.	cv::Mat inputImage;
2.	...
3.	std::vector<int> markerIds;
4.	std::vector<std::vector<cv::Point2f>> markerCorners, rejectedCandidates;
5.	cv::Ptr<cv::aruco::DetectorParameters> parameters;
6.	cv::Ptr<cv::aruco::Dictionary> dictionary = 
7.	              cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
8.	cv::aruco::detectMarkers(inputImage, dictionary, markerCorners, 
9.	                                   markerIds, parameters, rejectedCandidates);
```			
detectMarkers()函数具有六个参数，每个参数的含义为：
1.	第一个参数是需要检测标记的图像。
11.	第二个参数是字典对象，本教程中使用的是DICT_6X6_250预定义词典。
12.	第三个参数是检测到的标记的角点列表。对于每个标记，其四个角均按其原始顺序返回（从左上角开始顺时针旋转）。因此，第一个角是左上角，然后是右上角，右下角和左下角。
13.	第四个参数是检测到的每个标记的id。需要注意的是第三个参数和第四个参数具有相同的大小。
14.	第五个参数是类型DetectionParameters的对象。该对象包括在检测过程中可以自定义的所有参数。下一部分将详细说明此参数。
15.	第六个参数是去除的候选标记列表，即找到的但未提供有效编码的正方形。每个候选标记也由其四个角定义，其格式与第三个参数相同，该参数可以省略。

检测并识别标记之后之后，可以利用aruco模块提供的drawDetectedMarkers()函数在输入图像中绘制检测到的标记，该函数的使用方法在代码清单2-4中给出。
```cpp
代码清单2-4：绘制标记
16. cv::Mat outputImage
17.	cv::aruco::drawDetectedMarkers(image, markerCorners, markerIds);
```
drawDetectedMarkers()函数具有三个参数，每个参数的含义如下：
18. .	第一个参数是将绘制标记的输入/输出图像（通常是与检测到标记的图像相同）
19.	第二个参数是检测到的标记的角点列表
20.	第三个参数是检测到的每个标记的id

标记绘制结果在图2-6给出。
**注意**
*此功能仅用于可视化，在实际项目中可以省略。*

<center>
<img src="https://img-blog.csdnimg.cn/202002231129324.jpg">
</center>

使用上面两个功能，我们可以创建一个基本的标记检测循环程序，对相机拍摄的场景直接检测标记，该例程在代码清单2-5中给出。

```cpp
代码清单2-5：相机检测标记
21.	cv::VideoCapture inputVideo;
22.	inputVideo.open(0);
23.	cv::Ptr<cv::aruco::Dictionary> dictionary =
24.	              cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
25.	while (inputVideo.grab()) {
26.	    cv::Mat image, imageCopy;
27.	    inputVideo.retrieve(image);
28.	    image.copyTo(imageCopy);
29.	    std::vector<int> ids;
30.	    std::vector<std::vector<cv::Point2f> > corners;
31.	    cv::aruco::detectMarkers(image, dictionary, corners, ids);
32.	    // if at least one marker detected
33.	    if (ids.size() > 0)
34.	        cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
35.	    cv::imshow("out", imageCopy);
36.	    char key = (char) cv::waitKey(waitTime);
37.	    if (key == 27)
38.	        break;
39.	}
```
 完整的示例程序在aruco模块文件夹内的detect_markers.cpp中。可以通过代码清单2-6中的命令执行该项目。

```cpp
代码清单2-6：detect_markers.cpp文件需要的参数
40.	-c="_path_/calib.txt" -d=10
```
## 2.1.4 姿态估计
检测到标记后，我们需要从标记中获取相机姿态。要执行相机姿态估计，我们需要了解相机的标定参数。这是相机内参矩阵和畸变系数。使用OpenCV基础库中标定函数即可，这里不详细介绍如何对相机进行标定。我们默认读者已经完成了相机的标定。

使用ArUco标记估计姿态时，可以分别估计每个标记的姿态。如果要从一组标记中估计一个姿态，则要使用aruco Boards（请参阅ArUco Boards教程，这里不做过多介绍）。

相机相对于标记的姿态是从标记坐标系到相机坐标系的3d转换。它由旋转量和平移矢量确定。aruco模块提供了cv::aruco::estimatePoseSingleMarkers()函数用于估计所有检测到的标记的姿态，具体使用方法在代码清单2-7中给出。

```cpp
代码清单2-7：估计姿态
1.	 cv::Mat cameraMatrix, distCoeffs;
2.	...
3.	std::vector<cv::Vec3d> rvecs, tvecs;
4.	cv::aruco::estimatePoseSingleMarkers(corners, 0.05, cameraMatrix, 
5.	                                                       distCoeffs, rvecs, tvecs);
```
cv::aruco::estimatePoseSingleMarkers()函数有六个参数，每个参数的含义如下：
1. 第一个参数是detectMarkers()函数返回的标记角点的向量。
7.	第二个参数是标记的大小，以米或其他长度单位为单位。
8.	第三个参数和第四个参数是相机的标定参数。
9.	最后两个参数分别是角中每个标记的旋转和平移向量。

此外aruco模块还提供了绘制坐标轴的cv::aruco::drawAxis()函数，可以检查姿态估计结果，该函数的使用方法在代码清单2-8中给出。

```cpp
代码清单2-8：绘制坐标轴
10.	cv::aruco::drawAxis(image, cameraMatrix, distCoeffs, rvec, tvec, 0.1);
```
cv::aruco::drawAxis()函数具有六个参数，每个参数的含义如下
1.	第一个参数是绘制坐标轴轴的输入/输出图像（通常是与检测到标记的图像相同）。
13.	第二个和第三个是相机的标定参数。
14.	第四个和第五个是要绘制坐标轴物体的姿态参数。
15.	第六个参数是轴的长度，单位与第五个参数（通常为米）相同

此函数假定标记坐标系位于Z轴指向的标记中心。 坐标轴颜色对应为X：红色，Y：绿色，Z：蓝色，具体如图2-7所示。

 <center>
 <img src="https://img-blog.csdnimg.cn/20200223114221549.jpg">
</center>

**提示**
***在小白学视觉微信公众号后台回复“坐标轴检测”，即可获得该检测视频完整版*。** 

可以通过代码清单2-9实现基于单个标记利用相机进行实时姿态估计，程序运行结果的视频截图在图2-8给出。

```cpp
代码清单2-9：实时姿态估计
1.	cv::VideoCapture inputVideo;
2.	inputVideo.open(0);
3.	cv::Mat cameraMatrix, distCoeffs;
4.	// camera parameters are read from somewhere
5.	readCameraParameters(cameraMatrix, distCoeffs);
6.	cv::Ptr<cv::aruco::Dictionary> dictionary = 
7.	               cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
8.	while (inputVideo.grab()) {
9.	    cv::Mat image, imageCopy;
10.	    inputVideo.retrieve(image);
11.	    image.copyTo(imageCopy);
12.	    std::vector<int> ids;
13.	    std::vector<std::vector<cv::Point2f>> corners;
14.	    cv::aruco::detectMarkers(image, dictionary, corners, ids);
15.	    // if at least one marker detected
16.	    if (ids.size() > 0) {
17.	        cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);
18.	        std::vector<cv::Vec3d> rvecs, tvecs;
19.	        cv::aruco::estimatePoseSingleMarkers(corners, 0.05, cameraMatrix, 
20.	                                                       distCoeffs, rvecs, tvecs);
21.	        // draw axis for each marker
22.	        for(int i=0; i<ids.size(); i++)
23.	            cv::aruco::drawAxis(imageCopy, cameraMatrix, distCoeffs, 
24.	                                                         rvecs[i], tvecs[i], 0.1);
25.	    }
26.	    cv::imshow("out", imageCopy);
27.	    char key = (char) cv::waitKey(waitTime);
 if (key == 27)
29.	        break;
30.	}

 ```
 <center>
<img src="https://img-blog.csdnimg.cn/20200223114941709.jpg">
</center>


完整的代码程序在aruco模块文件夹内的detect_markers.cpp中。可以通过代码清单2-10中的命令执行该项目。
```cpp
代码清单2-10：调用实时姿态的参数
1.	-c="_path_/calib.txt" -d=10
```
## 2.1.5 选择字典
aruco模块了提供Dictionary类来表示标记字典。除了标记的尺寸和字典中标记的数量之外，标记间的距离也是字典的重要参数。标记间的距离是指字典中所有标记之间的最小距离，它决定了词典的错误检测能力和纠错能力。

一般来说，字典大小越小，标记尺寸越大，标记间的距离越远，反之亦然。但是，如果标记尺寸变大，那么需要冲图像中提取的数据量也更大，这使得对较大尺寸标记的检测更加复杂，因此通常采用尽量缩小字典大小的方式来增加标记间的距离。例如，如果我们只需要10个标记，那么此时使用由10个标记组成的字典要比使用由1000个标记组成的字典更好。因为由10个标记组成的字典会有更高的标记间的距离，从而对错误有更强的鲁棒性。为了让用户可以选择合适的字典来增加程序的鲁棒性，aruco模块提供了多种字典供拥护选择，接下来将详细介绍每种字典构建的方法。
### 1. 预定义的字典

使用预定义字典是选择字典最简单的方法。aruco模块内提供了一组预定义的字典，其中包含多种标记尺寸和标记数量，可以通过代码清单2-11中的代码选择字典的大小和标记的尺寸。
```cpp
代码清单2-11：选择预定义字典
2.	cv::Ptr<cv::aruco::Dictionary>dictionary=cv::aruco::getPredefinedDictionary(cv::aruco::DICT_6X6_250);
```
代码清单2-11中的代码表示创建一个字典大小为250，标记尺寸为6×6的字典。使用预定于的字典，标记尺寸可以在4×4到7×7之间自由选择。同样，字典大小可以在50、100、250和1000中自由选择。在实际需求中，我们需要选择合适的字典。例如在同时满足需求的前提下，DICT_6X6_250的字典要好于DICT_6X6_1000，因为字典越小，标记间的距离就越大。
### 2. 	自动选择字典

有时我们并不知道应该选择多大的字典，因此可以将选择字典的任务交给程序，由程序自动选择一个合适的字典，来保证字符间的距离最大。可以用代码清单2-12中的命令实现自动选择字典。
```cpp
代码清单2-12：自动选择字典
4.	cv::Ptr<cv::aruco::Dictionary> dictionary = 
5.	                                    cv::aruco::generateCustomDictionary(36, 5);
```
代码清单2-12中的命令将会生成一个由36个尺寸为5×5的标记组成的自定义字典。根据参数的不同，这个过程可能会花费几秒钟的时间(对于较大的字典和较高的标记尺寸，这个过程会慢一些)。

### 3. 	自定义字典:

有时我们可以根据自己的需要自定义字典，这种方式的好处是我们可以使用任意的编码方式。在自定义字典之前，我们需要重新定义Dictionary类，定义方式在代码清单2-13中给出。

```cpp
代码清单2-13：重定义Dictionary类
7.	class Dictionary {
8.	    public:
9.	    Mat bytesList;
10.	    int markerSize;
11.	    int maxCorrectionBits;    ...
12.	}
```
代码清2-13中每个参数的含义如下：字典参数为：
-	bytesList是包含关于标记代码的所有信息的数组。
-	markerSize是每个标记尺寸的大小(例如，尺寸5×5标记为5)。
-	maxCorrectionBits是标记检测过程中可以纠正的错误比特数的最大值。如果这个值过高，就会导致大量的误报。

**注意**
***除非有特殊的需求，否则建议使用模块中自带的预定义字典。***

bytesList中的每一行表示一个字典标记。但是，这个标记不是以二进制形式存储的，而是以一种特殊的格式存储的，以简化它们的检测。Dictionary类中提供了Dictionary::getByteListFromBits()函数将标记转换为这种形式，具体如代码清单2-14中所示。
```cpp
代码清单2-14：自定义字典示例
1.	cv::aruco::Dictionary dictionary;
2.	// markers of 6x6 bits
3.	dictionary.markerSize = 6;
4.	// maximum number of bit corrections
5.	dictionary.maxCorrectionBits = 3;
6.	// lets create a dictionary of 100 markers
7.	for(int i=0; i<100; i++)
8.	{
9.	    // assume generateMarkerBits() generate a new marker in binary 
10.	    //format, so that markerBits is a 6x6 matrix of CV_8UC1 type,
11.	    // only containing 0s and 1s
12.	    cv::Mat markerBits = generateMarkerBits();
13.	    cv::Mat markerCompressed = 
14.	                     cv::aruco::Dictionary::getByteListFromBits(markerBits);
15.	    // add the marker as a new row
16.	    dictionary.bytesList.push_back(markerCompressed);
17.	}
```

## 2.1.6参数检测器
前文我们见到detectmarker()函数中的DetectorParameters对象。该对象包含标记检测过程中可以自定义的所有选项。在本小节中，我们将详细介绍这些参数。

### 1. 阈值

标记检测过程的第一步是对输入图像进行自适应阈值化。例如，本教程所使用图像的阈值图像如所示。
阈值参数可以通过如下参数进行定义：
-	int  adaptiveThreshWinSizeMin, 
-	int  adaptiveThreshWinSizeMax, 
-	int  adaptiveThreshWinSizeStep
 <center>
 <img src="https://img-blog.csdnimg.cn/20200223120037423.jpg">
</center>
adaptivewellwinsizemin和adaptivewellwinsizemax参数表示为自适应阈值选择阈值窗口大小(以像素为单位)的区间，adaptiveThreshWinSizeStep表示自适应窗口每次改变的大小。例如，默认值adaptivewellwinsizemin=3、adaptivewellwinsizemax=23以及adaptiveThreshWinSizeStep=10。默认值表示阈值窗口依次为3×3、13×13和23×23。
此外，标记的尺寸也也对图像阈值处理产生影响，关于阈值尺寸的相关参数为：

-  double  minMarkerPerimeterRate
-	double  maxMarkerPerimeterRate

这些参数决定了标记点的最小和最大尺寸，具体来说就是标记点的最大和最小周长。这两个参数的单位不是像素值，而是相对于输入图像的最大尺寸的比例。例如，大小为640×480，相对标记最小周长为0.05的图像，其最小标记周长为640×0.05 = 32像素，因为640是图像的最大尺寸。
如果minMarkerPerimeterRate太小，则会大大降低检测性能，因为在未来阶段需要考虑更多的轮廓。对于maxMarkerPerimeterRate参数，这种影响不是很明显，因为通常小轮廓比大轮廓要多得多。如果minMarkerPerimeterRate值为0和maxMarkerPerimeterRate值为4(或更大)则等效于考虑图像中的所有轮廓，但是出于性能原因不建议这样做。
有时标记在图像中会显示成多边形，可以用多边形率来表示形变的程度：
-	double  polygonalApproxAccuracyRat

这个值决定了多边形近似可以产生的最大误差。此参数是相对于候选对象周长长度的比例。例如，如果候选对象的周长为100像素，且polygonalApproxAccuracyRate的值为0.04，则最大误差为100x0.04=5.4像素。在大多数情况下，使用该参数的默认值即可正常工作，但是对于高失真的图像，可能需要更高的数值。该参数的默认值为0.05。
加下来详细介绍多个标记之间的距离参数：
-	double  minCornerDistanceRate

同一标记上任意对角之间的最小距离。它是相对于界标周长表示的。像素的最小距离是周长* minCornerDistanceRate。该参数的默认值为0.05。
-	double  minMarkerDistanceRate

两个不同的标记之间的最小距离。它是相对于两个标记的最小标记周长来表示的。如果两个候选标记太接近，较小的那个就会被忽略。该参数的默认值为0.05。
-	int  minDistanceToBorder

任何标记角到图像边界的最小距离(以像素为单位)。该参数的默认值为3。
2.	分析标志图像的位信息

在候选标志检测完成后需要对每个候选标志进行数位分析，以确定它们是否是ArUco标记。“位”就是组成标志的最小单元，每个位表示一位二进制，例如一个6×6的标志位数就是36位。

在分析标志中二进制代码之前，需要提取二进制位。为了能够精准的提取二进制信息，首先需要消除了视角形变，之后使用Otsu算法对去除形变的图像进行阈值处理，以分离黑白像素。图2-10就是去除形变和二值化后的结果。
 <center>
 <img src="https://img-blog.csdnimg.cn/20200223120719951.jpg">
</center>

之后将图像划分到一个网格中，网格的单元格数目与标记中的位数目相同。在每个单元格上，计算黑白像素的数量来决定最终单元格的的颜色，从而确定该位表示的是0还是1。图2-11是划分单元格结果。
 <center>
 <img src="https://img-blog.csdnimg.cn/202002231208441.jpg">
</center>

在这个过程中有几个重要的参数可以选择，具体内容如下：

-	int  markerBorderBits

此参数指示标记边框的宽度。它与每个位的大小有关。例如，数值2表示边界的宽度为每个位宽度的2倍。此参数需要与我们正在使用标记的边框大小一致。我们可以在标记绘制函数drawMarker()中设置边框的大小，并及时记录它。该参数的默认值为1.

-	double  minOtsuStdDev

这个值决定了执行Otsu阈值算法的像素值的最小标准偏差。如果偏差很小，可能所有的单元格都是黑色(或白色)，应用Otsu没有意义。如果是这种情况，所有的位都被设置为0(或1)，这取决于平均值与128的关系。该参数的默认值为5.0。

-	int  perspectiveRemovePixelPerCell

该参数为在去除透视失真(包括边框)后得到的图像中每个单元格像素数目，即图2-11中红色方框的大小。例如，我们处理的是5×5的标记尺寸和1边界的标记图像，那么每个的单元格像素宽度为5 + 2*1 = 7(边界必须计数两次)。像素总数为7×7。如果tiveremovepixelpercell的值为10，则每个单元格内有70×70个像素。该参数值越高，提取过程越快，但性能越差。该参数的默认值为4。

-	double  perspectiveRemoveIgnoredMarginPerCell

在提取每个单元的位元时，将计算黑白像素的数目。通常，不建议考虑所有的单元格像素。相反，最好忽略单元格边缘的一些像素。这样做的原因是，在消除了透视失真之后，每个像素的颜色通常并不是完全分离的，白色单元可以入侵黑色单元的一些像素(反之亦然)。因此，为了避免计算错误的像素，最好忽略一些像素，如图2-12所示只考虑绿色方块内的像素。在右边的图像中可以看到，生成的像素包含来自邻居单元的更低的噪声。参数指示了红色和绿色方块之间的差异。

 <center>
 <img src="https://img-blog.csdnimg.cn/20200223121121631.jpg">
</center>

此参数表示绿色方框外忽略部分相对于单元格的总大小。例如，如果单元格大小为40像素，并且该参数的值为0.1，则单元格中忽略40*0.1=4像素的空白。这意味着在每个单元上要分析的像素总数实际上是32x32，而不是40x40。该参数默认值为0.13。

### 3.	标记识别

提取标记中二进制信息后，接下来检查提取的二进制代码是否属于标记字典，如果需要，可以执行错误纠正。接下来依次介绍在这个过程中涉及的参数：

-	double  maxErroneousBitsInBorderRate

标记边界的单元格应该是黑色的。此参数指定边界中允许的错误率，即边界中白色单元格的最大数量。它是相对于标记中总的单元格数目来表示的。该参数的默认值为0.35。

-	double  errorCorrectionRate

每个标记字典都有一个理论上可以更正的最大位数，这个值可以通过errorCorrectionRate参数进行修改。例如，如果允许可以更正的位数(与字典种类相关)是6，并且errorCorrectionRate的值是0.5，那么可以更正的实际最大比特数是6*0.5=3位。这个值有助于降低错误纠正能力，以避免误报，默认值为0.6。

### 4.	角点位置细化

检测并识别标记后，最后一步是对角点位置执行亚像素细化。这一步是可选的，只有在标记角点位置必须为精确值的情况下才有意义，例如姿态估计、运动测量等问题中。角点位置细化通常是一个耗时的步骤，默认情况下是不使用的。接下来依次介绍在这个过程中涉及的参数：

-	int  cornerRefinementMethod

此参数确定是否执行角亚像素处理。如果不需要精确的角点坐标，可以进行角点位置细化。该参数默认值为CORNER_REFINE_NONE，表示不进行角点位置细化。

-	int  cornerRefinementWinSize

此参数确定亚像素细化过程的窗口大小。数值较大会产生窗口区域包含临近的图像角点的效果，使得标记角在处理过程中移动到错误位置。并且，数值较大会影响计算时间。该参数的默认值为5。

-	int  cornerRefinementMaxIterations      double  cornerRefinementMinAccuracy

这两个参数决定了亚像素细化过程的停止条件。cornerRefinementMaxIterations表示迭代的最大次数，cornerRefinementMinAccuracy表示停止迭代之前的最小错误值，两者满足任意一个条件即停止细化迭代。如果迭代次数太多，计算时间将会极大延长，但是如果过低，则会产生较差的亚像素细化。这两个参数的默认值分别为：30和0.1。

