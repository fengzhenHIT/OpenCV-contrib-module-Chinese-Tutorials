# OpenCV-contrib-module-Chinese-Tutorials
OpenCV 扩展模块教程

<center>
本文首发于<font color="#3CA9C4">“小白学视觉”</font>微信公众号，欢迎关注公众号</center>
<center>
本文作者为小白，版权归<font color="red">人民邮电出版社</font>发行所有，禁止转载，侵权必究！
</center>

***

<table><tr><td bgcolor=#E7E4E5>经过几个月的努力，小白终于完成了市面上第一本OpenCV 4入门书籍《OpenCV 4开发详解》。为了更让小伙伴更早的了解最新版的OpenCV 4，小白与出版社沟通，提前在公众号上连载部分内容，请持续关注小白。</table>

<p style="text-align:justify">我们在上一节程序中生成了一张只有黑色和白色的图像，这种“非黑即白”的图像像素的灰度值无论在什么数据类型中只有最大值和最小值两种取值，因此称其为二值图像。二值图像色彩种类少，可以进行高度的压缩，节省存储空间，将非二值图像经过计算变成二值图像的过程称为图像的二值化。在OpenCV 4中提供了threshold()和adaptiveThreshold()两个函数用于实现图像的二值化，我们首先介绍threshold()函数的使用方法，该函数的函数原型在代码清单3-17中给出。</p>

```cpp
代码清单3-17 threshold()函数原型
1.	double cv::threshold(InputArray  src,
2.		                     OutputArray  dst,
3.		                     double  thresh,
4.		                     double  maxval,
5.		                     int  type
6.	                          )
```

-	src：待二值化的图像，图像只能是CV_8U和CV_32F两种数据类型。对于图像通道数目的要求和选择的二值化方法相关。
-	dst：二值化后的图像，与输入图像具有相同的尺寸、数据类型和通道数。
-	thresh：二值化的阈值。
-	maxval：二值化过程的最大值，此函数只在THRESH_BINARY和THRESH_BINARY_INV两种二值化方法中才使用，但是在使用其他方法是也需要输入。
-	type：选择图像二值化方法的标志。

该函数是众多二值化方法的集成，所有的方法都实现了一个功能，就是给定一个阈值，计算所有像素灰度值与这个阈值关系，得到最终的比较结果。函数中有些阈值比较方法输出结果的灰度值并不是二值的，而是具有一个取值范围，不过为了体现其最常用的功能，我们仍然称其为二值化函数或者阈值比较函数。函数的部分参数和返回值都是针对特定的算法才有用，但是即使不使用这些算法在使用函数时也需要明确的给出，不可缺省。函数的最后一个参数是选择二值化计算方法的标志，可以选择二值化方法以及控制哪些参数对函数的计算结果产生影响，该标志可以选择的范围及含义在表3-2中给出。
<center><font size=1 face="宋体"> 表3-2 二值化方法可选择的标志及含义</font></center>

|标志参数	|简记|作用|
|--|--|--|
|THRESH_BINARY|	0|	灰度值大于阈值为最大值，其他值为0|
|THRESH_BINARY_INV	|1	|灰度值大于阈值为0，其他值为最大值|
|THRESH_TRUNC	|2	|灰度值大于阈值的为阈值，其他值不变|
|THRESH_TOZERO|	3	|灰度值大于阈值的不变，其他值为0|
|THRESH_TOZERO_INV	|4|	灰度值大于阈值的为零，其他值不变|
|THRESH_OTSU|	8|	大津法自动寻求全局阈值|
|THRESH_TRIANGLE	|16|	三角形法自动寻求全局阈值|

接下来将详细的介绍每种标志对应的二值化原理和需要的参数。

#### THRESH_BINARY和THRESH_BINARY_INV
这两个标志是相反的二值化方法，THRESH_BINARY是将灰度值与阈值（第三个参数thresh）进行比较，如果灰度值大于阈值就将灰度值改为函数中第四个参数maxval的值，否则将灰度值改成0。THRESH_BINARY_INV标志正好与这个过程相反，如果灰度值大于阈值就将灰度值改为0，否则将灰度值改为maxval的值。这两种标志的计算公式在式(3.7)中给出。
<center>
<img src="https://img-blog.csdnimg.cn/20200102160021215.png">

#### THRESH_TRUNC
这个标志相当于重新给图像的灰度值设定一个新的最大值，将大于新的最大值的灰度值全部重新设置为新的最大值，具体逻辑为将灰度值与阈值thresh进行比较，如果灰度值大于thresh则将灰度值改为thresh，否则保持灰度值不变。这种方法没有使用到函数中的第四个参数maxval的值，因此maxval的值对本方法不产生影响。这种标志的计算公式在式(3.8)中给出。
<center>
<img src="https://img-blog.csdnimg.cn/20200102160102158.png">

#### THRESH_TOZERO和THRESH_TOZERO_INV
这两个标志是相反的阈值比较方法， THRESH_TOZERO表示将灰度值与阈值thresh进行比较，如果灰度值大于thresh则将保持不变，否则将灰度值改为0。THRESH_TOZERO_INV方法与其相反，将灰度值与阈值thresh进行比较，如果灰度值小于等于thresh则将保持不变，否则将灰度值改为0。这种两种方法都没有使用到函数中的第四个参数maxval的值，因此maxval的值对本方法不产生影响。这两个标志的计算公式在式中给出。
<center>
<img src="https://img-blog.csdnimg.cn/20200102160143562.png">

前面五种标志都支持输入多通道的图像，在计算时分别对每个通道进行阈值比较。为了更加直观的理解上述阈值比较方法，我们假设图像灰度值是连续变化的信号，将阈值比较方法比做滤波器，绘制连续信号通过滤波器后的信号形状，结果如图3-14所示，图中红线为设置的阈值，黑线为原始信号通过滤波器后的信号形状。
 
 <center>
<img src="https://img-blog.csdnimg.cn/20200102160223863.png">
<center><font size=1 face="宋体">图3-14 上述5种阈值比较法的信号示意图</font></center>

#### THRESH_OTSU和THRESH_TRIANGLE
这两种标志是获取阈值的方法，并不是阈值的比较方法的标志，这两个标志可以和前面5种标志一起使用，例如“THRESH_BINARY| THRESH_OTSU”。前面5种标志在调用函数时都需要人为的设置阈值，如果对图像不了解设置的阈值不合理，会对处理后的效果造成严重的影响，这两个标志分别表示利用大津法（OTSU）和三角形法（TRIANGLE）结合图像灰度值分布特性获取二值化的阈值，并将阈值以函数返回值的形式给出。因此如果函数最后一个参数设置了这两个标志中的任何一个，那么函数第三个参数thresh将由系统自动给出，但是在调用函数的时候仍然不能缺省，只是程序不会使用这个数值。需要注意的是，目前为止OpenCV 4中针对这两个标志只支持输入CV_8UC1类型的图像。

threshold()函数全局只使用一个阈值，在实际情况中由于光照不均匀以及阴影的存在，全局只有一个阈值会使得在阴影处的白色区域也会被函数二值化成黑色，因此adaptiveThreshold()函数提供了两种局部自适应阈值的二值化方法，该函数的函数原型在代码清单3-18中给出。

```cpp
代码清单3-18 adaptiveThreshold()函数原型
1.	void cv::adaptiveThreshold(InputArray  src,
2.		                            OutputArray  dst,
3.		                            double  maxValue,
4.		                            int  adaptiveMethod,
5.		                            int  thresholdType,
6.		                            int  blockSize,
7.		                            double 	C
8.	                                 )
```

-	src：待二值化的图像，图像只能是CV_8UC1数据类型。
-	dst：二值化后的图像，与输入图像具有相同的尺寸、数据类型。
-	maxValue：二值化的最大值。
-	adaptiveMethod：自制应确定阈值的方法，分为均值法ADAPTIVE_THRESH_MEAN_C和高斯法ADAPTIVE_THRESH_GAUSSIAN_C这两种。 
-	thresholdType：选择图像二值化方法的标志，只能是THRESH_BINARY和THRESH_BINARY_INV。
-	blockSize：自适应确定阈值的像素邻域大小，一般为3，5，7的奇数。
-	C：从平均值或者加权平均值中减去的常数，可以为正，也可以为负。

该函数将灰度图像转换成二值图像，通过均值法和高斯法自适应的计算blockSize* blockSize邻域内的阈值，之后进行二值化，其原理与前面的相同，这里就不再赘述。

为了直观的体会到图像二值化的效果，在代码清单3-19中给出了分别对彩色图像和灰度图像进行二值化的示例程序，程序运行结果在图3-15、图3-16中给出。

```cpp
代码清单3-19 myThreshold.cpp图像二值化
1.	#include <opencv2\opencv.hpp>
2.	#include <iostream>
3.	#include <vector>
4.	
5.	using namespace std;
6.	using namespace cv;
7.	
8.	int main()
9.	{
10.		Mat img = imread("lena.png");
11.		if (img.empty())
12.		{
13.			cout << "请确认图像文件名称是否正确" << endl;
14.			return -1;
15.		}
16.	
17.		Mat gray;
18.		cvtColor(img, gray, COLOR_BGR2GRAY);
19.		Mat img_B, img_B_V, gray_B, gray_B_V, gray_T, gray_T_V, gray_TRUNC;
20.	
21.		//彩色图像二值化
22.		threshold(img, img_B, 125, 255, THRESH_BINARY);
23.		threshold(img, img_B_V, 125, 255, THRESH_BINARY_INV);
24.		imshow("img_B", img_B);
25.		imshow("img_B_V", img_B_V);
26.	
27.		//灰度图BINARY二值化
28.		threshold(gray, gray_B, 125, 255, THRESH_BINARY);
29.		threshold(gray, gray_B_V, 125, 255, THRESH_BINARY_INV);
30.		imshow("gray_B", gray_B);
31.		imshow("gray_B_V", gray_B_V);
32.	
33.		//灰度图像TOZERO变换
34.		threshold(gray, gray_T, 125, 255, THRESH_TOZERO);
35.		threshold(gray, gray_T_V, 125, 255, THRESH_TOZERO_INV);
36.		imshow("gray_T", gray_T);
37.		imshow("gray_T_V", gray_T_V);
38.	
39.		//灰度图像TRUNC变换
40.		threshold(gray, gray_TRUNC, 125, 255, THRESH_TRUNC);
41.		imshow("gray_TRUNC", gray_TRUNC);
42.	
43.		//灰度图像大津法和三角形法二值化
44.		Mat img_Thr = imread("threshold.png", IMREAD_GRAYSCALE);
45.		Mat img_Thr_O, img_Thr_T;
46.		threshold(img_Thr, img_Thr_O, 100, 255, THRESH_BINARY | THRESH_OTSU);
47.		threshold(img_Thr, img_Thr_T, 125, 255, THRESH_BINARY | THRESH_TRIANGLE);
48.		imshow("img_Thr", img_Thr);
49.		imshow("img_Thr_O", img_Thr_O);
50.		imshow("img_Thr_T", img_Thr_T);
51.	
52.		//灰度图像自适应二值化
53.		Mat adaptive_mean, adaptive_gauss;
54.		adaptiveThreshold(img_Thr, adaptive_mean, 255, ADAPTIVE_THRESH_MEAN_C, 
55.	                                                                         THRESH_BINARY, 55, 0);
56.		adaptiveThreshold(img_Thr, adaptive_gauss, 255, ADAPTIVE_THRESH_GAUSSIAN_C, 
57.	                                                                         THRESH_BINARY, 55, 0);
58.	
59.		imshow("adaptive_mean", adaptive_mean);
60.		imshow("adaptive_gauss", adaptive_gauss);
61.		waitKey(0);
62.		return 0;
63.	}
```

<center>
<img src="https://img-blog.csdnimg.cn/20200102160417441.png">
<center><font size=1 face="宋体">图3-15 threshold()函数处理结果</font></center>

 <center>
<img src="https://img-blog.csdnimg.cn/20200102160457142.png">
<center><font size=1 face="宋体">图3-16 adaptiveThreshold()函数处理结果</font></center>

&nbsp;
<table><tr><td bgcolor=#AE9595><font color="white">OpenCV 4开发详解</font></td></tr></table>

| 往期推荐 | 
|--|
| [【OpenCV 4开发详解】Windows系统中安装OpenCV 4](https://blog.csdn.net/qq_42722197/article/details/103179441)|
|[【OpenCV 4开发详解】Ubuntu系统中安装OpenCV 4](https://blog.csdn.net/qq_42722197/article/details/103247107)|
|[【OpenCV 4开发详解】opencv_contrib扩展模块的安装 ](https://blog.csdn.net/qq_42722197/article/details/103247255)|
|[【OpenCV 4开发详解】Image Watch插件的使用](https://blog.csdn.net/qq_42722197/article/details/103368782)|
|[【OpenCV 4开发详解】安装过程中问题解决方案](https://blog.csdn.net/qq_42722197/article/details/103369129)|
|[【OpenCV 4开发详解】了解OpenCV的模块架构](https://blog.csdn.net/qq_42722197/article/details/103369012)|
|[【OpenCV 4开发详解】Mat类介绍](https://blog.csdn.net/qq_42722197/article/details/103369443)|
|[【OpenCV 4开发详解】Mat类构造与赋值](https://blog.csdn.net/qq_42722197/article/details/103370372)|
|[【OpenCV 4开发详解】4种读取Mat类元素的的方法](https://blog.csdn.net/qq_42722197/article/details/103370691)|
|[【OpenCV 4开发详解】图像的读取与显示](https://blog.csdn.net/qq_42722197/article/details/103697455)|
|[【OpenCV 4开发详解】视频加载与摄像头调用](https://blog.csdn.net/qq_42722197/article/details/103752506)|
|[【OpenCV 4开发详解】图像与视频的保存](https://blog.csdn.net/qq_42722197/article/details/103803236)||
[【OpenCV 4开发详解】保存和读取XML和YMAL文件](https://blog.csdn.net/qq_42722197/article/details/103803654)|
[【OpenCV 4开发详解】颜色模型与转换](https://blog.csdn.net/qq_42722197/article/details/103803890)|
[【OpenCV 4开发详解】多通道分离与合并](https://blog.csdn.net/qq_42722197/article/details/103804432)|
[【OpenCV 4开发详解】图像像素统计](https://blog.csdn.net/qq_42722197/article/details/103804579)|
[【OpenCV 4开发详解】两图像间的像素操作](https://blog.csdn.net/qq_42722197/article/details/103805479)|

<table><td bgcolor=#E7E4E5>经过几个月的努力，市面上第一本OpenCV 4入门书籍《OpenCV 4开发详解》将春节后由<font color="red">人民邮电出版社</font>发行。如果小伙伴觉得内容有帮助，希望到时候多多支持！</table>

<table><td bgcolor=#E7E4E5>关注小白的小伙伴可以提前看到书中的内容，我们创建了学习交流群，欢迎各位小伙伴添加小白微信加入交流群，添加小白时请备注“学习OpenCV 4”。</table>

<center>
<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS9mMzNjYWMxYy0xNWJhLTQ2OTMtYjNlYi1kOTQ0ZTE0OWE1MmYuanBn?x-oss-process=image/format,png" width="50%">
  
 <center>
<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS84NzQwMjAyNy0yZGExLTRhNDEtOTk3Mi0zMzEwZGM1YzU4ZWUuanBn?x-oss-process=image/format,png" width="80%">
