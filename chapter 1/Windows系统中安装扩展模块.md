本小节将介绍如何安装contrib扩展模块，考虑到读者可能使用windows系统或者Ubuntu系统，本书中将分别介绍在这两个系统中安装contrib扩展模块的过程。


安装contrib扩展模块时不仅需要扩展模块安装包，同时需要基础库安装包。contrib扩展模块安装包以opencv_contrib-x.x.x的形式统一命名，在安装时基础库安装包的版本需要和contrib扩展模块的安装包版本相对应。可以在GitHub上获取与自己OpenCV版本相匹配的opencv_contrib安装包。

## 1.1.1Windows系统中安装扩展模块
在Windows系统中安装opencv_contrib扩展模块其实就是重新将OpenCV编译一遍，因此需要用到CMake编译器，在CMake官网下载.msi安装包，通过双击直接完成安装任务。本书使用的是CMake 3.7.0版本，这里需要注意，由于OpenCV版本更新速度慢于CMake，因此不推荐使用过高版本的CMake编译器，只要满足OpenCV 4.0要求的最低版本即可，因此为了减少读者编译过程中的错误，建议与读者使用同版本的CMake。打开安装好的CMake软件，可以看到如图1-1所示的页面。我们需要选择OpenCV源码所在地址与编译文件的输出地址。源码放在了“…\opencv\sources”文件夹中，为了与OpenCV原有文件区分，在 “…\opencv”中创建一个名为newbuild文件夹用于存放编译输出文件。

<p align="center">
  <img src="https://img-blog.csdnimg.cn/20200226130256257.png" align="center" height="300"></img>
</p>

 
之后通过点击【Configure】按钮，选择将源码编译成与Visual Studio版本相对应的项目工程文件，作者安装的是Visual Studio 2015，同时想使用64位的OpenCV，因此选择“Visual Studio 14 2015 Win64”选项，同时选择本地编辑器“Use default native compilers”。选择配置的操作界面如图1-2所示。

<p align="center">
<img src="https://img-blog.csdnimg.cn/2020022613035076.png" align="center" height="250">
</p>

之后再次点击【Configure】开始构建，当出现“Configuring done”后说明构建成功，在CMake界面会出现很多变量，如图1-3所示。首先找到“BUILD_opencv_world”和“OPENCV_ENABLE_NONFREE”这两个变量，在变量后面的方框内打上“√”。第一个变量的含义是生成一个大的.lib文件，在被配置链接器时只有一个“opencv_world400d.lib”文件。第二个变量的含义是为了在编译成功后可以使用具有专利保护的算法，如果该变量不被选中，就不能使用例如SIFT算法在内的具有专利保护的算法，之后找到“OPENCV_EXTRA_MODULES_PATH”变量，该变量的含义是告诉编译器扩展包的源码在哪里，选择我们刚才下载的opencv_contrib安装包里的modules文件夹。如果这个变量为空，在编译过程中也不会报错，只是安装了OpenCV的基础版。

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226130522407.png" align="center" height="300">
</p>

在编译完成的界面内，我们可以看到扩展模块中将要编译的子模块，部分模块如图1-4中所示，比如aruco模块、bgsegm模块等。我们可以在右侧的方框中来选择是否编译该模块，默认情况下contrib中的子模块都是进行编译的，不过为了稳妥期间，读者在编译时可以查看并确认一下这些数据。

<p align="center">
<img src="https://img-blog.csdnimg.cn/20200226130629146.png" align="center" height="250">
</p>

再次点击【Configure】，直到所有的红色变量变成白色，之后点击【Generate】开始编译。编译成功后会在newbuild文件夹中生成许多文件，找到OpenCV.sln文件，用Visual Studio 2015打开该文件并重新生成解决方案，这个过程会比较漫长。 经过漫长时间的等待，在资源管理器中找到CMakeTargets中的INSTALL文件，右键选择“仅用于项目”中的“仅生成INSTALL”，会在newbuild文件夹中生成一个名为install的文件夹，我们用来环境配置的所有文件都存放在这个文件夹中。按照配置OpenCV基础库环境的方式配置环境即可。

**提示
编译后newbuild 文件夹非常的大，会有几个G的文件，但是除了install文件夹最重要，绝大多数文件都是垃圾文件，如果觉得硬盘存储量有限，可以选择性的删除一些文件。**

**注意
*我们认为读者熟练的掌握OpenCV基础库的环境配置方案，如果读者对基础库环境配置过程不是很熟悉，可以参考我们推荐的配套书籍《OpenCV 4计算机视觉编程实战》。***

***警告
编译过程较为繁琐，并且由于网络、系统版本等问题容易遇到比较冷门的错误，若遇到相关棘手问题，可以在“小白学视觉”微信公众号交流学习。
如果读者只是想使用扩展模块，我们在“小白学视觉”微信公众号上准备了多个版本已经编译完成的含有contrib扩展模块的库文件，下载后直接配置环境即可。***

## 1.1.2Ubuntu系统中安装扩展模块
在Ubuntu系统中安装扩展模块比较容易，只需要在cmake编译时使用代码清单1-1中的命令，在“OPENCV_EXTRA_MODULES_PATH=”后面添加上扩展安装包的路径即可，其余步骤与安装OpenCV基础模块没有区别。

```cpp
代码清单1-1 编译OpenCV命令
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules ..
```

***注意
我们认为读者熟练的掌握OpenCV在Ubuntu系统中的编译和配置工作。这部分相关内容网络资源较多，这里不再过多叙述。***
