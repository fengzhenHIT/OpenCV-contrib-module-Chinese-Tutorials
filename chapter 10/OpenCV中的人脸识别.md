## 10.1.1人脸识别介绍
人脸识别对于人类而言是一项轻松的任务。实验表明，即使是1至3天大的婴儿也能够区分已知面孔。那么，对于计算机来说有多难呢？事实证明，到目前为止，计算机对人类的认识还知之甚少。内部特征（眼睛，鼻子，嘴巴）或外部特征（头部形状，发际线）是否可成功用于的面部识别？我们如何分析图像，大脑如何编码呢？David Hubel和Torsten Wiesel证明，我们的大脑具有专门的神经细胞，可以响应场景的特定局部特征，例如线条，边缘，角度或运动。由于我们不认为世界是零散的碎片，因此我们的视觉皮层必须以某种方式将不同的信息源组合成有用的模式。自动人脸识别就是从图像中提取那些有意义的特征，将它们变成有用的表示并对其进行某种分类。

基于面部的几何特征的面部识别可能是最直观的面部识别方法。Picture processing system by computer complex and recognition of human faces中描述了最早的自动面部识别系统之一：标记点（眼睛，耳朵，鼻子等器官的的位置）用于构建特征向量（点之间的距离，它们之间的角度等 ）。识别是通过计算探针和参考图像的特征向量之间的欧式距离来实现的。 这种方法由于其性质使得抵抗照明变化的能力很强，但是有一个巨大的缺点就是即使使用最先进的算法，标记点的精确配准也很复杂。Face recognition through geometrical features中进行了一些有关几何人脸识别的最新工作。通过使用22维特征向量对大型数据集进行的实验表明，仅几何特征可能无法携带足够的信息来进行人脸识别。

Eigenfaces for recognition中描述的Eigenfaces采用了一种整体方法来进行脸部识别：脸部图像是来自高维图像空间的一个点，并且找到了低维表示，使得分类变得容易。使用主成分分析可以找到较低维的子空间，该子空间可以识别具有最大方差的轴。从重建的角度来看，这种转换是最佳的，但它没有考虑任何类标签。想象一种情况，其中的变化是由外部来源产生的，方差最大的轴根本不必包含任何判别信息，因此分类变得不可能。因此，在Eigenfaces vs. fisherfaces: Recognition using class specific linear projection中将具有线性判别分析的特定类投影应用于面部识别。 基本思想是最小化一个类中的方差，同时最大化两个类之间的方差。

最近出现了各种用于局部特征提取的方法。 为了避免输入数据的高维性，仅描述了图像的局部区域，提取的特征（希望）对部分遮挡，照明和较小的样本量更加健壮。 用于局部特征提取的算法是Gabor小波，离散余弦变换和局部二值模式。 应用空间特征提取时，保留空间信息的最佳方法是什么仍然是一个悬而未决的问题，因为空间信息可能是有用的信息。

## 10.1.2人脸数据集
人脸识别需要一些人脸图像，我们可以创建自己的数据集，也可以从可用的人脸数据库中获取图像， http://face-rec.org/databases/提供最新的数据库。 接下来介绍三个有趣的数据库：

- AT&T Facedatabase AT＆T人脸数据库（有时也称为ORL人脸数据库）包含40个不同主题中每个主题的十幅不同图像。 对于某些对象，图像是在不同的时间拍摄的，光线，面部表情（睁开/闭合的眼睛，微笑/不微笑）和面部细节（眼镜/不戴眼镜）均不同。 所有图像都是在深色均匀背景下拍摄的，对象处于直立的，正面的位置（允许某些侧向移动）。

- Yale Facedatabase A, 也称为Yalefaces。AT＆T Facedatabase适用于初始测试，但是它是一个相当简单的数据库。Eigenfaces方法已经具有97％的识别率，因此使用其他算法不会看到任何重大改进。Yale Facedatabase A（也称为Yalefaces）是更适合初始实验的数据集，因为识别问题比较困难。该数据库由15个人（男性14位，女性1位）组成，每个人具有11张320×243像素的灰度图像。光线条件（中心光线，左光线，右光线），面部表情（快乐，正常，悲伤，困倦，惊讶，眨眼）和眼镜（眼镜，无眼镜）都有变化。但是原始图像未裁剪和对齐。

- Extended Yale Facedatabase B 扩展的Yale Facedatabase B在其裁剪版本中包含3814个不同人物的2414张图像。该数据库的重点放在提取对照明稳定的特征上，图像的情感/遮挡/ ...几乎没有变化。我个人认为，对于我在本文中执行的实验而言，该数据集太大。 您最好使用AT＆T Facedatabase进行初始测试。Yale Facedatabase B的第一个版本曾用于查看在剧烈光照变化下Eigenfaces和Fisherfaces方法的性能。

## 10.1.3准备数据
一旦获取了一些数据，就需要在程序中读取它们。在演示应用程序中，我决定从一个非常简单的CSV文件读取图像。因为这是最简单的与平台无关的方法。基本上，所有CSV文件都需要包文件名和标签（作为整数），组成代码清单10-1中的形式。

```cpp
代码清单10-1
1./path/to/image.ext;0
```

/path/to/image.ext是图像的路径，如果在Windows中，则可能是这样的：C：/faces/person0/image0.jpg。 然后是分隔符“; ”最后，我们给图像分配标签0。将标签视为该图像所属的主题（人），因此相同的主题（人）应具有相同的标签。

从AT＆T Facedatabase下载AT＆T Facedatabase，并从at.txt下载相应的CSV文件，该文件中的内容如代码清单10-2中所示

```cpp
代码清单10-2
./at/s1/1.pgm;0
./at/s1/2.pgm;0
...
./at/s2/1.pgm;1
./at/s2/2.pgm;1
...
./at/s40/1.pgm;39
./at/s40/2.pgm;39
```

想象一下，我已经将文件提取到D:/ data / at，并将CSV文件下载到D:/data/at.txt。然后，我们只需要用D:/ data /搜索并替换./。我们可以在自己选择的编辑器中执行此操作，每个足够高级的编辑器都可以执行此操作。拥有包含有效文件名和标签的CSV文件后，我们可以通过将路径作为参数传递到CSV文件来运行任何示例程序，方式如代码清单10-3中所示。


```cpp
代码清单10-3
facerec_demo.exe D:/data/at.txt
```
	
## 10.1.4Eigenfaces介绍
我们得到的图像表示的最大问题是它的大尺寸。二维p×q灰度图像跨越m = pq维矢量空间，因此100×100像素的图像已经位于10000维图像空间中。 但是并不是所有的维度对我们都有用，我们只能决定数据是否存在差异，因此我们要寻找的是构成大多数信息的组件。 主成分分析（PCA）由Karl Pearson（1901）和Harold Hotelling（1933）独立提出，将一组可能相关的变量转换为较小的一组不相关变量。具体想法是，高维数据集通常由相关变量来描述，因此，只有少数几个有意义的维度才能说明大多数信息。PCA方法找到数据中方差最大的方向，称为主成分。

接下来介绍Eigenfaces方法算法

令X={x<sub>1</sub>,x<sub>2</sub>,…,x<sub>n</sub> }是带有观测值x<sub>i</sub>∈R<sup>d</sup>的随机向量。计算平均值μ

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex=\mu&space;=&space;\frac{1}{n}\sum\limits_{i&space;=&space;1}^n&space;{{x_i}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu&space;=&space;\frac{1}{n}\sum\limits_{i&space;=&space;1}^n&space;{{x_i}}" title="\mu = \frac{1}{n}\sum\limits_{i = 1}^n {{x_i}}" /></a></p>
计算协方差矩阵S

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex=S&space;=&space;\frac{1}{n}\sum\limits_{i&space;=&space;1}^n&space;{\left(&space;{{x_i}&space;-&space;\mu&space;}&space;\right){{\left(&space;{{x_i}&space;-&space;\mu&space;}&space;\right)}^T}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S&space;=&space;\frac{1}{n}\sum\limits_{i&space;=&space;1}^n&space;{\left(&space;{{x_i}&space;-&space;\mu&space;}&space;\right){{\left(&space;{{x_i}&space;-&space;\mu&space;}&space;\right)}^T}}" title="S = \frac{1}{n}\sum\limits_{i = 1}^n {\left( {{x_i} - \mu } \right){{\left( {{x_i} - \mu } \right)}^T}}" /></a></p>

计算S的特征值λ<sub>i</sub>和特征向量v<sub>i</sub>
<center>Sv<sub>i</sub>=λ<sub>i </sub>v<sub>i</sub>，i=1,2,…,n</center>

对特征向量按其特征值降序进行排序。k个主要成分是与k个最大特征值相对应的特征向量。所观察到的向量x的k个主要成分如下：

<center>y=W<sup>T</sup> (x-μ)</center>

其中W={v<sub>1</sub>,v<sub>2</sub>,…,v<sub>k</sub> }。

从PCA基础上进行的重构，方式由下式给出：
<center>x=Wy+μ</center>

其中W={v<sub>1</sub>,v<sub>2</sub>,…,v<sub>k </sub>}。

然后，Eigenfaces方法通过以下方式执行人脸识别：

- 将所有训练样本投影到PCA子空间中。
- 将查询图像投影到PCA子空间中。
- 在计划的训练图像和计划的查询图像之间找到最近的邻居

这里仍然有一个问题需要解决。想象一下，我们得到了400张尺寸为100×100像素的图像。主成分分析求解协方差矩阵S = XXT，在本教程中为size（X）= 10000×400。我们最终将得到10000×10000矩阵，大约为0.8GB。 解决此问题是不可行的，因此我们需要应用技巧。从线性代数课程中，您知道M> N的M×N矩阵只能具有N-1个非零特征值。 因此，有可能采用大小为N×N的特征值分解S = X<sup>T</sup>X：
<center>X<sup>T</sup> Xv<sub>i</sub>=λ<sub>i </sub>vi</center>

并使用数据矩阵的左乘法获得S = XX<sup>T</sup>的原始特征向量：
<center>XX<sup>T</sup> (Xv<sub>i </sub>)=λ<sub>i </sub>(Xv<sub>i</sub>)</center>

所得的特征向量是正交的，为了获得正交特征向量，需要将它们标准化为单位长度。具体方式读者可以Pattern classification中了解。

## 10.1.5Eigenface示例

```cpp
代码清单10-4
/*
 * Copyright (c) 2011. Philipp Wagner <bytefish[at]gmx[dot]de>.
 * Released to public domain under terms of the BSD Simplified license.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *   * Redistributions of source code must retain the above copyright
 *     notice, this list of conditions and the following disclaimer.
 *   * Redistributions in binary form must reproduce the above copyright
 *     notice, this list of conditions and the following disclaimer in the
 *     documentation and/or other materials provided with the distribution.
 *   * Neither the name of the organization nor the names of its contributors
 *     may be used to endorse or promote products derived from this software
 *     without specific prior written permission.
 *
 *   See <http://www.opensource.org/licenses/bsd-license>
 */
#include "opencv2/core.hpp"
#include "opencv2/face.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"
#include <iostream>
#include <fstream>
#include <sstream>
using namespace cv;
using namespace cv::face;
using namespace std;
static Mat norm_0_255(InputArray _src) {
    Mat src = _src.getMat();
    // Create and return normalized image:
    Mat dst;
    switch(src.channels()) {
    case 1:
        cv::normalize(_src, dst, 0, 255, NORM_MINMAX, CV_8UC1);
        break;
    case 3:
        cv::normalize(_src, dst, 0, 255, NORM_MINMAX, CV_8UC3);
        break;
    default:
        src.copyTo(dst);
        break;
    }
    return dst;
}
static void read_csv(const string& filename, vector<Mat>& images, vector<int>& labels, char separator = ';') {
    std::ifstream file(filename.c_str(), ifstream::in);
    if (!file) {
        string error_message = "No valid input file was given, please check the given filename.";
        CV_Error(Error::StsBadArg, error_message);
    }
    string line, path, classlabel;
    while (getline(file, line)) {
        stringstream liness(line);
        getline(liness, path, separator);
        getline(liness, classlabel);
        if(!path.empty() && !classlabel.empty()) {
            images.push_back(imread(path, 0));
            labels.push_back(atoi(classlabel.c_str()));
        }
    }
}
int main(int argc, const char *argv[]) {
    // Check for valid command line arguments, print usage
    // if no arguments were given.
    if (argc < 2) {
        cout << "usage: " << argv[0] << " <csv.ext> <output_folder> " << endl;
        exit(1);
    }
    string output_folder = ".";
    if (argc == 3) {
        output_folder = string(argv[2]);
    }
    // Get the path to your CSV.
    string fn_csv = string(argv[1]);
    // These vectors hold the images and corresponding labels.
    vector<Mat> images;
    vector<int> labels;
    // Read in the data. This can fail if no valid
    // input filename is given.
    try {
        read_csv(fn_csv, images, labels);
    } catch (const cv::Exception& e) {
        cerr << "Error opening file \"" << fn_csv << "\". Reason: " << e.msg << endl;
        // nothing more we can do
        exit(1);
    }
    // Quit if there are not enough images for this demo.
    if(images.size() <= 1) {
        string error_message = "This demo needs at least 2 images to work. Please add more images to your data set!";
        CV_Error(Error::StsError, error_message);
    }
    // Get the height from the first image. We'll need this
    // later in code to reshape the images to their original
    // size:
    int height = images[0].rows;
    // The following lines simply get the last images from
    // your dataset and remove it from the vector. This is
    // done, so that the training data (which we learn the
    // cv::BasicFaceRecognizer on) and the test data we test
    // the model with, do not overlap.
    Mat testSample = images[images.size() - 1];
    int testLabel = labels[labels.size() - 1];
    images.pop_back();
    labels.pop_back();
    // The following lines create an Eigenfaces model for
    // face recognition and train it with the images and
    // labels read from the given CSV file.
    // This here is a full PCA, if you just want to keep
    // 10 principal components (read Eigenfaces), then call
    // the factory method like this:
    //
    //      EigenFaceRecognizer::create(10);
    //
    // If you want to create a FaceRecognizer with a
    // confidence threshold (e.g. 123.0), call it with:
    //
    //      EigenFaceRecognizer::create(10, 123.0);
    //
    // If you want to use _all_ Eigenfaces and have a threshold,
    // then call the method like this:
    //
    //      EigenFaceRecognizer::create(0, 123.0);
    //
    Ptr<EigenFaceRecognizer> model = EigenFaceRecognizer::create();
    model->train(images, labels);
    // The following line predicts the label of a given
    // test image:
    int predictedLabel = model->predict(testSample);
    //
    // To get the confidence of a prediction call the model with:
    //
    //      int predictedLabel = -1;
    //      double confidence = 0.0;
    //      model->predict(testSample, predictedLabel, confidence);
    //
    string result_message = format("Predicted class = %d / Actual class = %d.", predictedLabel, testLabel);
    cout << result_message << endl;
    // Here is how to get the eigenvalues of this Eigenfaces model:
    Mat eigenvalues = model->getEigenValues();
    // And we can do the same to display the Eigenvectors (read Eigenfaces):
    Mat W = model->getEigenVectors();
    // Get the sample mean from the training data
    Mat mean = model->getMean();
    // Display or save:
    if(argc == 2) {
        imshow("mean", norm_0_255(mean.reshape(1, images[0].rows)));
    } else {
        imwrite(format("%s/mean.png", output_folder.c_str()), norm_0_255(mean.reshape(1, images[0].rows)));
    }
    // Display or save the Eigenfaces:
    for (int i = 0; i < min(10, W.cols); i++) {
        string msg = format("Eigenvalue #%d = %.5f", i, eigenvalues.at<double>(i));
        cout << msg << endl;
        // get eigenvector #i
        Mat ev = W.col(i).clone();
        // Reshape to original size & normalize to [0...255] for imshow.
        Mat grayscale = norm_0_255(ev.reshape(1, height));
        // Show the image & apply a Jet colormap for better sensing.
        Mat cgrayscale;
        applyColorMap(grayscale, cgrayscale, COLORMAP_JET);
        // Display or save:
        if(argc == 2) {
            imshow(format("eigenface_%d", i), cgrayscale);
        } else {
            imwrite(format("%s/eigenface_%d.png", output_folder.c_str(), i), norm_0_255(cgrayscale));
        }
    }
    // Display or save the image reconstruction at some predefined steps:
    for(int num_components = min(W.cols, 10); num_components < min(W.cols, 300); num_components+=15) {
        // slice the eigenvectors from the model
        Mat evs = Mat(W, Range::all(), Range(0, num_components));
        Mat projection = LDA::subspaceProject(evs, mean, images[0].reshape(1,1));
        Mat reconstruction = LDA::subspaceReconstruct(evs, mean, projection);
        // Normalize the result:
        reconstruction = norm_0_255(reconstruction.reshape(1, images[0].rows));
        // Display or save:
        if(argc == 2) {
            imshow(format("eigenface_reconstruction_%d", num_components), reconstruction);
        } else {
            imwrite(format("%s/eigenface_reconstruction_%d.png", output_folder.c_str(), num_components), reconstruction);
        }
    }
    // Display if we are not writing to an output folder:
    if(argc == 2) {
        waitKey(0);
    }
    return 0;
}
```

我们使用了喷射色图，因此我们可以看到灰度值如何在特定的Eigenface内分布。可以看到，特征不仅编码面部特征，而且还编码图像中的光照（请参见特征4中的左灯，特征5中的右灯）。具体结果在图10-1给出。

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226135302981.png" height="300">
</p>
 
我们可以看到，可以从其较低维度的近似值重建面孔。因此，让我们看看一个好的重建需要多少本征面。我们将用10,30，…，310个本征面做一个子图，具体操作如代码清单10-5中所示。

```cpp
代码清单10-5
// Display or save the image reconstruction at some predefined steps:
for(int num_components = 10; num_components < 300; num_components+=15) {
    // slice the eigenvectors from the model
    Mat evs = Mat(W, Range::all(), Range(0, num_components));
    Mat projection = LDA::subspaceProject(evs, mean, images[0].reshape(1,1));
    Mat reconstruction = LDA::subspaceReconstruct(evs, mean, projection);
    // Normalize the result:
    reconstruction = norm_0_255(reconstruction.reshape(1, images[0].rows));
    // Display or save:
    if(argc == 2) {
        imshow(format("eigenface_reconstruction_%d", num_components), reconstruction);
    } else {
        imwrite(format("%s/eigenface_reconstruction_%d.png", output_folder.c_str(), num_components), reconstruction);
    }
}
```

10个特征向量显然不足以进行良好的图像重建，而50个特征向量可能已经足以编码重要的面部特征。如果使用300个特征向量描述AT＆T Facedatabase，我们将获得良好的重构。 根据经验，要成功识别人脸，应该选择多少个特征脸，但这在很大程度上取决于输入数据。 如图10-5是特征向量增加对同一个图像重建的效果的影响。
<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226135512221.png" height="300">
</p>
 

## 10.1.6Fisherfaces介绍
主成分分析（PCA）是Eigenfaces方法的核心，它找到特征的线性组合，该特征使数据的总方差最大化。 尽管这显然是表示数据的一种有效方法，但是它不考虑任何类，因此在丢弃组件时可能会丢失很多区分性信息。想象一种情况，我们的数据差异是由外部来源生成的，那就容易了。 PCA标识的组件根本不必包含任何歧视性信息，因此将投影的样本混在一起，并且无法进行分类（有关示例： http://www.bytefish.de/wiki/pca_lda_with_gnu_octave）。

线性判别分析是特定类的降维，它是由伟大的统计学家Sir R. A. Fisher提出的。他在1936年的论文《分类学问题中多次测量的使用》中成功地将其用于花朵分类。为了找到在类别之间最佳分离的特征组合，线性判别分析使类别之间对类别内部散布的比率最大化，而不是使整体散布最大化。 这个想法很简单：相同的类应该紧密地聚在一起，而不同的类在低维表示中则要尽可能地远离彼此。 Belhumeur，Hespanha和Kriegman也认识到这一点，因此他们在[14]中将判别分析应用于人脸识别。

接下来介绍**Fisherfaces**方法的算法

令X为具有从c类中抽取的样本的随机向量：
<center>X={X<sub>1</sub>,X<sub>2</sub>,…,X<sub>C </sub>}</center>

<center>X<sub>I</sub>={x<sub>1</sub>,x<sub>2</sub>,…,x<sub>n</sub> }</center>

散射矩阵S<sub>B</sub>和S_ {W}的计算公式为：

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex={S_B}&space;=&space;\sum\limits_{i&space;=&space;1}^c&space;{{N_i}\left(&space;{{\mu&space;_i}&space;-&space;u}&space;\right){{\left(&space;{{\mu&space;_i}&space;-&space;u}&space;\right)}^T}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{S_B}&space;=&space;\sum\limits_{i&space;=&space;1}^c&space;{{N_i}\left(&space;{{\mu&space;_i}&space;-&space;u}&space;\right){{\left(&space;{{\mu&space;_i}&space;-&space;u}&space;\right)}^T}}" title="{S_B} = \sum\limits_{i = 1}^c {{N_i}\left( {{\mu _i} - u} \right){{\left( {{\mu _i} - u} \right)}^T}}" /></a></p>



<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex={S_W}&space;=&space;\sum\limits_{i&space;=&space;1}^c&space;{\sum\limits_{{x_j}&space;\in&space;{X_i}}&space;{\left(&space;{{x_j}&space;-&space;{u_i}}&space;\right){{\left(&space;{{x_j}&space;-&space;{u_i}}&space;\right)}^T}}&space;}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{S_W}&space;=&space;\sum\limits_{i&space;=&space;1}^c&space;{\sum\limits_{{x_j}&space;\in&space;{X_i}}&space;{\left(&space;{{x_j}&space;-&space;{u_i}}&space;\right){{\left(&space;{{x_j}&space;-&space;{u_i}}&space;\right)}^T}}&space;}" title="{S_W} = \sum\limits_{i = 1}^c {\sum\limits_{{x_j} \in {X_i}} {\left( {{x_j} - {u_i}} \right){{\left( {{x_j} - {u_i}} \right)}^T}} }" /></a></p>

其中，μ是总平均值：

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex=\mu&space;=&space;\frac{1}{N}\sum\limits_{i&space;=&space;1}^N&space;{{x_i}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu&space;=&space;\frac{1}{N}\sum\limits_{i&space;=&space;1}^N&space;{{x_i}}" title="\mu = \frac{1}{N}\sum\limits_{i = 1}^N {{x_i}}" /></a></p>

μ<sub>i</sub>是类i∈{1，…，c}的平均值：

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex={\mu&space;_i}&space;=&space;\frac{1}{{\left|&space;{{X_i}}&space;\right|}}\sum\limits_{{x_j}&space;\in&space;{X_i}}^N&space;{{x_j}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{\mu&space;_i}&space;=&space;\frac{1}{{\left|&space;{{X_i}}&space;\right|}}\sum\limits_{{x_j}&space;\in&space;{X_i}}^N&space;{{x_j}}" title="{\mu _i} = \frac{1}{{\left| {{X_i}} \right|}}\sum\limits_{{x_j} \in {X_i}}^N {{x_j}}" /></a></p>

Fisher的经典算法现在寻找投影W，该投影W使类可分离性标准最大化：

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex={W_{opt}}&space;=&space;\arg&space;{\max&space;_W}\frac{{\left|&space;{{W^T}{S_B}W}&space;\right|}}{{\left|&space;{{W^T}{S_W}W}&space;\right|}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{W_{opt}}&space;=&space;\arg&space;{\max&space;_W}\frac{{\left|&space;{{W^T}{S_B}W}&space;\right|}}{{\left|&space;{{W^T}{S_W}W}&space;\right|}}" title="{W_{opt}} = \arg {\max _W}\frac{{\left| {{W^T}{S_B}W} \right|}}{{\left| {{W^T}{S_W}W} \right|}}" /></a></p>

之后，通过解决一般特征值问题，给出了该优化问题的解决方案：

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex={S_B}{\upsilon&space;_i}&space;=&space;{\lambda&space;_i}{S_\omega&space;}{\upsilon&space;_i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{S_B}{\upsilon&space;_i}&space;=&space;{\lambda&space;_i}{S_\omega&space;}{\upsilon&space;_i}" title="{S_B}{\upsilon _i} = {\lambda _i}{S_\omega }{\upsilon _i}" /></a></p>

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex=S_W^{&space;-&space;1}{S_B}{\upsilon&space;_i}&space;=&space;{\lambda&space;_i}{\upsilon&space;_i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?S_W^{&space;-&space;1}{S_B}{\upsilon&space;_i}&space;=&space;{\lambda&space;_i}{\upsilon&space;_i}" title="S_W^{ - 1}{S_B}{\upsilon _i} = {\lambda _i}{\upsilon _i}" /></a></p>

还有一个问题要解决：SW的等级最多为（N-c），具有N个样本和c个类。 在模式识别问题中，样本数N几乎总是比输入数据的维数（像素数）小，因此散射矩阵SW变得奇异（请参阅[173]）。这是通过对数据执行主成分分析并将样本投影到（NC）维空间来解决的。 然后对简化的数据执行线性判别分析，因为SW不再是奇异的。

然后可以将优化问题重写为：
<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex={W_{pac}}&space;=&space;\arg&space;{\max&space;_W}\left|&space;{{W^T}{S_T}W}&space;\right|" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{W_{pac}}&space;=&space;\arg&space;{\max&space;_W}\left|&space;{{W^T}{S_T}W}&space;\right|" title="{W_{pac}} = \arg {\max _W}\left| {{W^T}{S_T}W} \right|" /></a></p>


<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex={W_{fld}}&space;=&space;\arg&space;{\max&space;_W}\frac{{\left|&space;{{W^T}W_{pca}^T{S_B}{W_{pca}}W}&space;\right|}}{{\left|&space;{{W^T}W_{pca}^T{S_W}{W_{pca}}W}&space;\right|}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?{W_{fld}}&space;=&space;\arg&space;{\max&space;_W}\frac{{\left|&space;{{W^T}W_{pca}^T{S_B}{W_{pca}}W}&space;\right|}}{{\left|&space;{{W^T}W_{pca}^T{S_W}{W_{pca}}W}&space;\right|}}" title="{W_{fld}} = \arg {\max _W}\frac{{\left| {{W^T}W_{pca}^T{S_B}{W_{pca}}W} \right|}}{{\left| {{W^T}W_{pca}^T{S_W}{W_{pca}}W} \right|}}" /></a></p>


之后，将样本投影到（c-1）维空间的变换矩阵W由下式给出

<p align="center"><a href="https://www.codecogs.com/eqnedit.php?latex=W&space;=&space;W_{fld}^TW_{pca}^T" target="_blank"><img src="https://latex.codecogs.com/gif.latex?W&space;=&space;W_{fld}^TW_{pca}^T" title="W = W_{fld}^TW_{pca}^T" /></p>


## 10.1.7	Fisherfaces示例

```cpp
代码清单10-6
/*
 * Copyright (c) 2011. Philipp Wagner <bytefish[at]gmx[dot]de>.
 * Released to public domain under terms of the BSD Simplified license.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *   * Redistributions of source code must retain the above copyright
 *     notice, this list of conditions and the following disclaimer.
 *   * Redistributions in binary form must reproduce the above copyright
 *     notice, this list of conditions and the following disclaimer in the
 *     documentation and/or other materials provided with the distribution.
 *   * Neither the name of the organization nor the names of its contributors
 *     may be used to endorse or promote products derived from this software
 *     without specific prior written permission.
 *
 *   See <http://www.opensource.org/licenses/bsd-license>
 */
#include "opencv2/core.hpp"
#include "opencv2/face.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"
#include <iostream>
#include <fstream>
#include <sstream>
using namespace cv;
using namespace cv::face;
using namespace std;
static Mat norm_0_255(InputArray _src) {
    Mat src = _src.getMat();
    // Create and return normalized image:
    Mat dst;
    switch(src.channels()) {
    case 1:
        cv::normalize(_src, dst, 0, 255, NORM_MINMAX, CV_8UC1);
        break;
    case 3:
        cv::normalize(_src, dst, 0, 255, NORM_MINMAX, CV_8UC3);
        break;
    default:
        src.copyTo(dst);
        break;
    }
    return dst;
}
static void read_csv(const string& filename, vector<Mat>& images, vector<int>& labels, char separator = ';') {
    std::ifstream file(filename.c_str(), ifstream::in);
    if (!file) {
        string error_message = "No valid input file was given, please check the given filename.";
        CV_Error(Error::StsBadArg, error_message);
    }
    string line, path, classlabel;
    while (getline(file, line)) {
        stringstream liness(line);
        getline(liness, path, separator);
        getline(liness, classlabel);
        if(!path.empty() && !classlabel.empty()) {
            images.push_back(imread(path, 0));
            labels.push_back(atoi(classlabel.c_str()));
        }
    }
}
int main(int argc, const char *argv[]) {
    // Check for valid command line arguments, print usage
    // if no arguments were given.
    if (argc < 2) {
        cout << "usage: " << argv[0] << " <csv.ext> <output_folder> " << endl;
        exit(1);
    }
    string output_folder = ".";
    if (argc == 3) {
        output_folder = string(argv[2]);
    }
    // Get the path to your CSV.
    string fn_csv = string(argv[1]);
    // These vectors hold the images and corresponding labels.
    vector<Mat> images;
    vector<int> labels;
    // Read in the data. This can fail if no valid
    // input filename is given.
    try {
        read_csv(fn_csv, images, labels);
    } catch (const cv::Exception& e) {
        cerr << "Error opening file \"" << fn_csv << "\". Reason: " << e.msg << endl;
        // nothing more we can do
        exit(1);
    }
    // Quit if there are not enough images for this demo.
    if(images.size() <= 1) {
        string error_message = "This demo needs at least 2 images to work. Please add more images to your data set!";
        CV_Error(Error::StsError, error_message);
    }
    // Get the height from the first image. We'll need this
    // later in code to reshape the images to their original
    // size:
    int height = images[0].rows;
    // The following lines simply get the last images from
    // your dataset and remove it from the vector. This is
    // done, so that the training data (which we learn the
    // cv::BasicFaceRecognizer on) and the test data we test
    // the model with, do not overlap.
    Mat testSample = images[images.size() - 1];
    int testLabel = labels[labels.size() - 1];
    images.pop_back();
    labels.pop_back();
    // The following lines create an Fisherfaces model for
    // face recognition and train it with the images and
    // labels read from the given CSV file.
    // If you just want to keep 10 Fisherfaces, then call
    // the factory method like this:
    //
    //      FisherFaceRecognizer::create(10);
    //
    // However it is not useful to discard Fisherfaces! Please
    // always try to use _all_ available Fisherfaces for
    // classification.
    //
    // If you want to create a FaceRecognizer with a
    // confidence threshold (e.g. 123.0) and use _all_
    // Fisherfaces, then call it with:
    //
    //      FisherFaceRecognizer::create(0, 123.0);
    //
    Ptr<FisherFaceRecognizer> model = FisherFaceRecognizer::create();
    model->train(images, labels);
    // The following line predicts the label of a given
    // test image:
    int predictedLabel = model->predict(testSample);
    //
    // To get the confidence of a prediction call the model with:
    //
    //      int predictedLabel = -1;
    //      double confidence = 0.0;
    //      model->predict(testSample, predictedLabel, confidence);
    //
    string result_message = format("Predicted class = %d / Actual class = %d.", predictedLabel, testLabel);
    cout << result_message << endl;
    // Here is how to get the eigenvalues of this Eigenfaces model:
    Mat eigenvalues = model->getEigenValues();
    // And we can do the same to display the Eigenvectors (read Eigenfaces):
    Mat W = model->getEigenVectors();
    // Get the sample mean from the training data
    Mat mean = model->getMean();
    // Display or save:
    if(argc == 2) {
        imshow("mean", norm_0_255(mean.reshape(1, images[0].rows)));
    } else {
        imwrite(format("%s/mean.png", output_folder.c_str()), norm_0_255(mean.reshape(1, images[0].rows)));
    }
    // Display or save the first, at most 16 Fisherfaces:
    for (int i = 0; i < min(16, W.cols); i++) {
        string msg = format("Eigenvalue #%d = %.5f", i, eigenvalues.at<double>(i));
        cout << msg << endl;
        // get eigenvector #i
        Mat ev = W.col(i).clone();
        // Reshape to original size & normalize to [0...255] for imshow.
        Mat grayscale = norm_0_255(ev.reshape(1, height));
        // Show the image & apply a Bone colormap for better sensing.
        Mat cgrayscale;
        applyColorMap(grayscale, cgrayscale, COLORMAP_BONE);
        // Display or save:
        if(argc == 2) {
            imshow(format("fisherface_%d", i), cgrayscale);
        } else {
            imwrite(format("%s/fisherface_%d.png", output_folder.c_str(), i), norm_0_255(cgrayscale));
        }
    }
    // Display or save the image reconstruction at some predefined steps:
    for(int num_component = 0; num_component < min(16, W.cols); num_component++) {
        // Slice the Fisherface from the model:
        Mat ev = W.col(num_component);
        Mat projection = LDA::subspaceProject(ev, mean, images[0].reshape(1,1));
        Mat reconstruction = LDA::subspaceReconstruct(ev, mean, projection);
        // Normalize the result:
        reconstruction = norm_0_255(reconstruction.reshape(1, images[0].rows));
        // Display or save:
        if(argc == 2) {
            imshow(format("fisherface_reconstruction_%d", num_component), reconstruction);
        } else {
            imwrite(format("%s/fisherface_reconstruction_%d.png", output_folder.c_str(), num_component), reconstruction);
        }
    }
    // Display if we are not writing to an output folder:
    if(argc == 2) {
        waitKey(0);
    }
    return 0;
}
```

对于代码清单10-6中的程序，我将使用Yale Facedatabase A，因为结果显示效果更好。每个Fisherface的长度与原始图像的长度相同，因此可以将其显示为图像。该演示显示（或保存）第一张，最多16张Fisherfaces，程序运行结果在图10-3中给出。

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226144508114.png" height="300">
</p>

Fisherfaces方法学习特定于类的变换矩阵，因此它们不像Eigenfaces方法那样明显捕获照明。判别分析改为找到可区分人的面部特征。值得一提的是，Fisherfaces的性能也很大程度上取决于输入数据。 实用地说：如果您仅学习具有良好照明效果的图片的Fisherfaces，并尝试识别照明条件较差的场景中的人脸，则该方法很可能会找到错误的成分（只是因为这些功能在照明不良的图像中可能并不占主导地位）。这有点合乎逻辑，因为该方法没有机会学习照明。

Fisherfaces可以像Eigenfaces一样重建投影图像。但是，由于我们仅识别出可区分主题的功能，因此不能指望对原始图像进行很好的重建。对于Fisherfaces方法，我们将示例图像投影到每个Fisherfaces上，将获得一个不错的可视化效果。具体操作在代码清单10-7中给出，可视化效果在图10-4给出，对于人眼来说，差异可能是微妙的，但是任然能够看到一些差异：

```cpp
代码清单10-7
// Display or save the image reconstruction at some predefined steps:
for(int num_component = 0; num_component < min(16, W.cols); num_component++) {
    // Slice the Fisherface from the model:
    Mat ev = W.col(num_component);
    Mat projection = LDA::subspaceProject(ev, mean, images[0].reshape(1,1));
    Mat reconstruction = LDA::subspaceReconstruct(ev, mean, projection);
    // Normalize the result:
    reconstruction = norm_0_255(reconstruction.reshape(1, images[0].rows));
    // Display or save:
    if(argc == 2) {
        imshow(format("fisherface_reconstruction_%d", num_component), reconstruction);
    } else {
        imwrite(format("%s/fisherface_reconstruction_%d.png", output_folder.c_str(), num_component), reconstruction);
    }
}
```
<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226144628258.png" height="300">
</p>

