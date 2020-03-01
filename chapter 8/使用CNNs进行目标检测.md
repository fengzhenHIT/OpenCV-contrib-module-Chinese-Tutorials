本章中的内容需要使用“dnn_objectect”模块，并且在编译的过程中需要构建示例程序，即在编译过程中选择BUILD_EXAMPLES选项，该选项在CMake界面中如图8-1所示，在Linux系统中通过BUILD_EXAMPLES=ON进行设置。
<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226104443881.png" height="200">
</p>
 
**提示
*设置BUILD_EXAMPLES=ON后，会编译所有模块的示例程序，因此如果条件允许的情况下，建议单独编译“dnn_objectect”模块。***

## 8.1.1	目标
在本教程中，我们将利用自带例程模型对图像进行识别和分类，使用到的示例程序如下：

1.	example_dnn_objdetect_obj_detect
2.	example_dnn_objdetect_image_classification

本教程中需要使用的模型定义文件和权值文件和可以在opencv_extra/dnn_objdetect中找到。文件的名称分别是在表8-1给出。

<p align="center">
  表8-1 模型文件和权值文件名称
</p>

模型文件|权值文件
-----|-----
SqueezeDet_deploy.prototxt |SqueezeDet.caffemodel	
SqueezeNet_deploy.prototxt|SqueezeNet.caffemodel

**提示
*小白学视觉公众号后台回复“dnn_objdetect”同样可以获取本章教程使用的模型定义、权值文件以及测试使用的图片。***

## 8.1.2	目标检测
对目标进行检测可以通过代码清单8-1中的格式实现。
```cpp
代码清单8-1：目标检测格式
example_dnn_objdetect_obj_detect  <模型定义文件>  <模型权值文件>  <图片>
```

本章中所有的的例子都是运行在Intel(R) Core(TM)2 i3-4005U CPU @ 1.70GHz(不含GPU)的笔记本上。

在本教程中，该模型预测多个边界框的平均时间仅为0.172091秒，在不适用GPU的情况下，速度也十分的快。
```cpp
代码清单8-2：测试飞机
<bin_path>/example_dnn_objdetect_obj_detect  SqueezeDet_deploy.prototxt  SqueezeDet.caffemodel  tutorials/images/aeroplane.jpg
Total objects detected: 1 in 0.168792 seconds
------
Class: aeroplane
Probability: 0.845181
Co-ordinates: 41 116 415 254
------
 ```

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226110321167.png" height="300">
</p>

```cpp
代码清单8-3：测试公交车
<bin_path>/example_dnn_objdetect_obj_detect  SqueezeDet_deploy.prototxt  SqueezeDet.caffemodel  tutorials/images/bus.jpg
Total objects detected: 1 in 0.201276 seconds
------
Class: bus
Probability: 0.701829
Co-ordinates: 0 32 415 244
------
 ```
<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226110603905.png" height="300">
</p>

```cpp
代码清单8-4：测试猫
<bin_path>/example_dnn_objdetect_obj_detect  SqueezeDet_deploy.prototxt  SqueezeDet.caffemodel  tutorials/images/cat.jpg
Total objects detected: 1 in 0.190335 seconds
------
Class: cat
Probability: 0.703465
Co-ordinates: 34 0 381 282
------
 ```

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226110709223.png" height="300">
</p>

```cpp
代码清单8-5：测试人
<bin_path>/example_dnn_objdetect_obj_detect  SqueezeDet_deploy.prototxt  SqueezeDet.caffemodel  tutorials/images/persons_mutli.jpg
Total objects detected: 2 in 0.169152 seconds
------
Class: person
Probability: 0.737349
Co-ordinates: 160 67 313 363
------
Class: person
Probability: 0.720328
Co-ordinates: 187 198 222 323
------
 ```

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226110812788.png" height="300">
</p>


## 8.1.3	改变阈值
默认情况下，该模型的检测阈值为置信值0.53。它可以预测到一定数量的边界框，我们可以通过传递变量threshold的值来手动控制阈值，调用格式如代码清单8-6所示。
```cpp
代码清单8-6：改变阈值
<bin_path>/example_dnn_objdetect_obj_detect  <模型定义文件>  <模型权重文件>   <测试图片> <阈值>
```
将阈值更改为0，再次检测飞机，可以如图8-6所示的结果。

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226111045592.png" height="300">
</p>

## 8.1.4	图像分类
-	利用现有模型对图像进行分类的格式在代码清单8-7中给出。

```cpp
代码清单8-7：对图像进行分类
example_dnn_objdetect_image_classification  <模型定义文件 >  <模型权重文件>   <测试图片>
```

该模型的大小为4.9MB，对图像进行分类平均只需要0.136401秒。

-	代码清单8-8中给出了对飞机图像进行分类的代码及运行结果
```cpp
代码清单8-8：对飞机图像进行分类
<bin_path>/example_dnn_objdetect_image_classification  SqueezeNet_deploy.prototxt  SqueezeNet.caffemodel  tutorials/images/aeroplane.jpg
Best class Index: 404
Time taken: 0.137722
Probability: 77.1757
```

预测结果为404，通过查看synset_words.txt，可以找到它属于airliner类

-	继续对猫的图像进行分类，代码清单8-9给出示例代码和分类结果。

```cpp
代码清单8-9：对猫图像进行分类
<bin_path>/example_dnn_objdetect_image_classification  SqueezeNet_deploy.prototxt  SqueezeNet.caffemodel  tutorials/images/cat.jpg
Best class Index: 285
Time taken: 0.136401
Probability: 40.7111
```

预测结果为285，通过查看synset_words.txt，可以找到它属于Egyptian cat类

-	继续对航天飞机图像进行分类，代码清单8-10给出示例代码和分类结果。
```cpp
代码清单8-10：对航天飞机图像分类
<bin_path>/example_dnn_objdetect_image_classification  SqueezeNet_deploy.prototxt  SqueezeNet.caffemodel  tutorials/images/space_shuttle.jpg
Best class Index: 812
Time taken: 0.137792
Probability: 15.8467
```

预测结果为285，通过查看synset_words.txt，可以找到它属于space shuttle类
