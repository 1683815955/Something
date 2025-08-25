<h1 align = "center">WGS算法使用指南</h1>

<h4 align = "center">Author:  Haopeng Wang</h4>

<h4 align = "center">Date:  250713</h4>





[TOC]

## 一、搭建环境

> 代码与环境适用于滨松SLM和大恒水星相机，使用其他设备需要略微修改代码与环境

### 1、安装环境管理工具Conda

主流的集成Conda的环境管理软件有Anaconda和Miniconda。Anaconda包含了Conda并且预装了许多科学计算所依赖的Python库，Miniconda则为Anaconda的精简版，包含了Conda和少量Python库。本文使用Miniconda配置环境。

```html
Miniconda下载地址：https://repo.anaconda.com/miniconda/ 
```

选择与Python版本和电脑系统相对应的安装包下载，比如Python3.13版本和Windows系统，则选择如下安装包下载

![](https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252146820.png)

注意安装时将“Add Miniconda3 to my PATH environment variable”选项勾上，该选项可以将Conda添加到系统环境变量中，无需自己配置系统环境变量。

<img src="https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252146935.png" style="zoom: 80%;" />

### 2、配置虚拟环境

在包含“environment.yaml”的文件夹目录空白处右键，选择“在终端中打开”

<img src="https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252146357.png" style="zoom:67%;" />

输入以下代码，回车，等待虚拟环境安装完成

```
conda env create -f environment.yaml
```
> 如果通过yaml文件安装环境失败，也可以点击链接下载环境：https://rec.ustc.edu.cn/share/40954e60-6082-11f0-9c8d-47d83e2be8f1，解压后将文件夹slm放入Miniconda3\envs中

本文使用VSCode进行代码编辑，也可使用其他代码编辑器，本文以VSCode为示例。虚拟环境安装完成后，在VSCode右下角点击使用的解释器，选择名称为slm的解释器，即可将编程环境切换到虚拟环境slm

![](https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252147125.png)

> 还需要在VSCode在左侧工具栏“扩展”中下载Python插件，才能运行Python代码
>
> ![](https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252147257.png)

此外，使用滨松SLM还需要slmpy库，使用大恒相机还需要gxipy库，我已经将两个库放进了“SLM code”文件夹中，无需进行额外配置。
<img src="https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252148368.png" style="zoom:67%;" />



## 二、介绍代码

### 1、主文件

打开“SLM code”文件夹，该文件夹存放了WGS的代码，其中主文件为“TweezerDaheng.py”，用于调用其他模块文件和配置文件，启动整个项目。主要分为五个部分，每一部分都添加了注释

```
SLM初始化->WGS生成相位图->加入相位修正->投屏->相机反馈
```

在主文件中，有些设置可以自定义，比如修改以下代码可以设置菲涅尔透镜与修正的参数，以及选择对相位图进行何种修正

```python
# 菲涅尔透镜相位 
fresnel_lens_screen, _ = SLM.fresnel_lens_phase_generate(50000,1024/2,1024/2)

# 考虑菲涅尔透镜相位
slm_screen_corrected = slm_screen + fresnel_lens_screen

#考虑菲涅尔透镜相位和波前修正
# slm_screen_corrected = slm_screen + fresnel_lens_screen + correction

#LUT修正
# slm_screen_corrected = np.around(slm_screen_corrected*200.0/255.0).astype('uint8')
```

将电脑连接滨松SLM和大恒水星相机后，运行主文件即可，运行结果会给出来相机反馈的时间、最终不均匀度等信息。

### 2、配置文件

“SLM code”中的”Config.yaml“文件为配置文件，需要根据实际情况修改。其中“SLM_Config"包含了SLM及光镊阵列基本的配置，参数加载到”SLMGeneration.py“文件中，以下是示例配置

```yaml
SLM_Config:
  pixelpitch: 12.5 # SLM液晶像素尺寸，单位um
  SLMRes: [1272,1024] # SLM屏幕分辨率

  arraySizeBit: [10,10] # 计算用图片分辨率
  threshold:  0.01  # 相位固定阈值

  beamwaist: 5000 # 入射高斯光束腰半径，单位um
  focallength: 150000 # 物镜焦距，单位um
  magnification: 1  # SLM到物镜间成像系统放大率
  wavelength: 0.759 # 光波长，单位um
  mask: false  # 是否有光阑
  maskradius: 0  # 光阑半径，单位um

  distance: 0  # 光镊阵列中最近点到原点的物理距离，单位um，注意必须为正值，否则相机反馈时顺序会乱
  spacing: [150,150]  # 光镊阵列物理间隔，单位um
  arraysize: [20,20]  # 光镊阵列大小

  translate: true  # 将SLM阵列移到屏幕中央，注意此时输入的目标图应该是未旋转过的
  rotate: false  # 旋转SLM阵列，来匹配AOD阵列。请注意！！！旋转后可能会使相机反馈输入顺序错乱，到时要注意好顺序！！！
  angle: 0  # 旋转角度，单位：度
  modify: true  # 根据SLM的衍射效率作对目标图作修正，如果阵列体系较大，建议使用
      
  AddZernike: false # 是否加上Zernike修正
  ind_Zernike_list: 0  # 需要添加的Zernike项，注意项数从0到27，对应Zernike的径向n从0到6。注意该图显示是1-28，但实际函数调用应该是0-27
  percent_list: 0  # percent表示对应Zernike项的强度，范围-1到1。物理上，它对应着对应像差的rms值
  zernike_aperture_radius: 0  # 要添加的Zernike圆半径，通常与光阑半径一致，单位um
  isZernikePhaseContinous: 0  # 是否需要将圆形的Zernike相位连续的扩展到方形来匹配SLM尺寸，通常不需要 
```

”WGS_Config“包含了WGS中关于GPU加速的配置，参数加载到”WGS.py“文件中，以下是示例配置

```yaml
WGS_Config:
  CUDA: true # 是否启用GPU加速WGS
  cudnn_benchmark: false # 是否启用cudnn_benchmark
```

”Feedback_Config“包含了关于相机反馈的配置，参数加载到”TweezerDaheng.py“的相机反馈部分中，以下是示例配置

```yaml
Feedback_Config:
  optimize: true # 是否启用相机反馈
  interval: 0.1 # 等待SLM切换的时间间隔，单位s
  rep: 10 # 相机反馈次数
```

### 3、模块文件

“SLM code”中以下部分文件为模块文件，该部分文件用于给“TweezerDaheng.py”提供支持。

<img src="https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252149029.png" style="zoom:80%;" />

下面对模块文件一一进行简单的介绍。

```
Aberration.py:定义了Zernike类，封装了许多用于Zernike校正像差的函数
DahengCamera.py：定义了DahengCamera类，封装了许多用于操作大恒相机的函数
Gaussian_2D_Fit.py：封装了用于生成2D高斯分布和进行2D高斯拟合的函数
SLMGeneration.py：定义了SLM_class类，封装了许多关于操作SLM和生成目标振幅的函数
TweezerArrayRegionsExtraction.py：封装了用于生成网格划分光斑的函数
WGS.py：封装了用于运行WGS的函数
```

## 三、一些测试数据

压缩包内还有一些我整理的测试数据，对比了不同反馈方式的均匀性，以及记录了相机的反馈速度，设备型号记录于”相机反馈速度测试数据“文件中。

![](https://cdn.jsdelivr.net/gh/1683815955/Something@master/img/202508252149527.png)

