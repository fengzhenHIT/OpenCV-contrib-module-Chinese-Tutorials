目前Contrib Modules中有五十个子模块，涵盖了从传统机器视觉到深度神经网络，从相机标定到立体视觉，从背景分割到图像识别等众多领域。本节主要目的是为了介绍子模块的功能，让读者了解被隐藏在OpenCV-Contrib中的众多强大功能，也让读者能够直观的了解到模块中都有哪些部分是直接需要的，做到在学习和使用时能够有的放矢。

全部的模块名称和模块用处在表1-1中给出。


<p align="center">
  <strong>表1-1 contrib中各模块名称和含义</strong>
</p>

<center>
  
模块名称|含义
---|---
aruco|ArUco标记检测
bgsegm|改进的背景前景分割方法
bioinspired.|受生物启发的视觉模型和衍生工具
ccalib|用于3D重建的自定义校准图案
cnn_3dobj|3D对象识别和姿态估计API
cudaarithm.|矩阵运算（CUDA）
cudabgsegm|背景分割
cudacodec|视频编码/解码
cudafeatures2d|特征检测与描述
cudafilters|图像过滤
cudaimgproc|图像处理
cudalegacy|版权支持
cudaobjdetect|物体检测
cudaoptflow|光流
cudastereo|立体视觉
cudawarping|图像映射
cudev|设备层
cvv|用于计算机视觉程序的交互式视觉调试的GUI
datasets|处理不同数据集的框架
dnn_objdetect|DNN用于物体检测
dpm|基于零件的可变形模型
face|人脸分析
freetype|用freetype / harfbuzz绘制UTF-8字符串
fuzzy|基于模糊数学的图像处理
hdf|分层数据格式I / O例程
hfs|分层特征选择，实现有效的图像分割
img_hash|该模块带来了不同图像哈希算法的实现。
line_descriptor|从图像提取的行的二进制描述符
optflow	|光流算法
ovis|OGRE 3D可视化工具
phase_unwrapping|相位展开API
plot|Mat数据的绘图功能
quality|图像质量分析（IQA）API
reg|图像配准
rgbd|RGB深度处理
saliency|显着性API
sfm|运动结构
shape|形状距离和匹配
stereo|立体对应算法
structured_light|结构化光源API
superres|超分辨率
surface_matching|表面匹配
text|场景文字检测与识别
tracking|追踪API
videostab|视频稳定
viz|3D可视化器
xfeatures2d|额外的2D功能框架
ximgproc|扩展图像处理
xobjdetect|扩展物体检测
xphoto|其他照片处理算法

</center>

下面是部分子模块的详细介绍：

-	aruco：全名是“ArUco and ChArUco Markers”，AR增强现实模块。ArUco and ChArUco是两个增强现实中常用的标记。ArUco是被嵌入在棋盘的白色区域内的标记。
-	bgsegm：全名是“Background segmentation”，背景分割算法。模块中主要包括统计背景图像估计和按像素的贝叶斯分割等。 
-	bioinspired：全名是“Biological Vision -- Biologically inspired vision model”，生物视觉-生物启发的视觉模型。模块中主要包括最小化噪声、亮度差异、瞬态事件分割、高动态范围色调映射方法等。
-	ccalib：全名是“Custom Calibration”，自定义标定。模块中主要包括三维重建、广角相机标定、随机模式标定和多相机标定等。
-	cnn_3dobj：全名是“cnn_ 3D object recognition and pose estimation”，深度对象识别和姿态检测。模块中主要包含基于Caffe深度神经网络库构建、训练和测试视觉对象识别和姿势的CNN模型。
-	cvv：全名是“Computer Vision Debugger”计算机视觉调试器。模块提供GUI调试界面，便于交互式的调试计算机视觉的程序。
-	datasets：全名是“Datasets Reader”，数据集读取器。模块用于读取现有计算机视觉数据库和使用该读取器训练、测试和运行该数据集提供的示例程序。
-	text：全名是“Visual Text Matching”，文字检测与识别。模块主要包括检测文字、分割词汇、识别文本等。
