# 运行在Win10环境下的安装教程

本教程运行在Win10专业版22H2版本，python环境配置推荐使用Anaconda管理

硬件环境：

Intel(R) Core(TM) i9-14900HX

64GB内存

NVIDIA GeForce RTX 4070 Laptop GPU

Cuda环境：cuda12.8+cudnn9.7.1

# 1. 环境搭建

### 1.1 conda 虚拟环境搭建

~~~
conda create -n tennis_one python=3.9
conda activate tennis_one
# install pytorch
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128
# install utils tool, TrackNetV3 and openpose requirements
pip3 install -r requirements.txt
~~~

### 1.2 orbbecsdk setup

<details>

#### 1.2.1 安装CMake和Visual Studio

此处提供推荐版本下载链接安装

[CMake-3.17.2-win64-x64](https://drive.google.com/file/d/10IPa59chh4vYpl6Li8kYqxZyY5ueaYNs/view?usp=sharing)

[Visual Studio 2019](https://drive.google.com/file/d/14jAmLRdFaZmASuxTzlXukN6-vLWvrUy-/view?usp=sharing)

#### 1.2.2 配置CMake和Visual Studio

在TennisOneV2/orbbec目录下创建build目录，打开CMake，配置项目路径和build文件夹后点击Configure按钮
![](md_images/cmake1.png)
配置vs版本，选择vs2019，x64版本，使用默认编译配置，点击Finish
![](md_images/cmake2.png)
若出现找不到Python3的错误，需要添加环境变量指定虚拟环境路径
![](md_images/cmake3.png)
点击Add Entry按钮，分别添加Python3_EXECUTABLE和Python3_LIBRARIES，分别是虚拟环境路径下的python.exe和虚拟环境路径下的Lib文件夹，重新点击Configure按钮
![](md_images/cmake4-1.png)
![](md_images/cmake4-2.png)
若出现找不到pybind11的错误，需要修改pybind11_DIR变量的值，指定到虚拟环境下的Lib/site-packages/pybind11/share/cmake/pybind11文件夹，重新点击Configure按钮
![](md_images/cmake5-1.png)
![](md_images/cmake5-2.png)
当出现Configuring done且没有报错后，点击Generate按钮生成项目
![](md_images/cmake6.png)
生成完成后，点击Open Project按钮打开VS2019
![](md_images/cmake7.png)
切换到Release模式，选择pyorbbecsdk项目，右键菜单选择重新生成
![](md_images/vs_1.png)
选择INSTALL项目，右键菜单选择生成
![](md_images/vs_2.png)

复制orbbec/install/lib路径下的所有文件至虚拟环境路径下的Lib\site-packages目录下
![](md_images/image10.png)
![](md_images/image11.png)

</details>

### 1.3 openpose编译

<details>
~~~
git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose.git
python prepare_cmake_openpose.py
~~~

在TennisOneV2/openpose目录下创建build目录，打开CMake，配置项目路径和build文件夹后点击Configure按钮
![](md_images/openpose/cmake1.png)
勾选BUILD_PYTHON，再次点解Configure按钮
![](md_images/openpose/cmake2.png)
显示缺少python解释器，路径里选择虚拟环境路径下的python.exe，再次点击Configure按钮
![](md_images/openpose/cmake3.png)
注意要取消勾选PYBIND11_TEST
![](md_images/openpose/cmake4.png)
当出现Configuring done且没有报错后，点击Generate按钮生成项目
![](md_images/openpose/cmake5.png)
生成完成后，点击Open Project按钮打开VS2019
切换到Release模式，选择pyopenpose项目，右键菜单选择属性
![](md_images/openpose/vs1.png)
在属性中打开"配置属性"-"C/C++"-"命令行"中加入 "/utf-8"，然后点击确定
![](md_images/openpose/vs2.png)
选择解决方案"OpenPose"，右键菜单选择重新生成解决方案
![](md_images/openpose/vs3.png)
生成完成后将openpose/build/bin路径和openpose/build/python/openpose/Release路径下所有文件以及openpose/build/x64/Release/openpose.dll文件复制到虚拟环境下的Lib/site-packages路径中
![](md_images/openpose/done.png)

</details>

### 1.4 下载并配置与训练模型

~~~
python download_pretrain_model.py
~~~

# 2. 运行

### 2.1 通过orbbecsdk录制视频

~~~
python ./multi_device_sync_record.py -dn 2
~~~

参数解释

-dn:摄像机的数量

### 2.2 使用rt-detr-v2对录制视频进行裁剪，提取关键动作视频片段

~~~
python ./rtdetrv2_video.py -c ./rtdetrv2/configs/rtdetrv2/rtdetrv2_r18vd_sp3_120e_coco.yml -r ./rtdetrv2/best.pth -vf ./video/input/zed_test.mp4 -p
~~~

参数解释

-c:模型配置文件，无需修改

-r:预训练模型权重位置

-vf:需要分析的视频

-p:生成3D姿势，若不需要则无需添加此参数

