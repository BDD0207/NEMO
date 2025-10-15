@[TOC](【NEMO海洋模型】NEMO 海洋模型简介)


# 1. NEMO海洋模型简介

“Nucleus for European Modelling of the Ocean” ([NEMO](https://www.nemo-ocean.eu/))是一个先进的建模框架，用于海洋和气候科学领域的研究活动和预测服务。
![NEMO](https://i-blog.csdnimg.cn/direct/73f5b8260da541998c20c12e0790f74d.png#pic_center)
NEMO 由欧盟开发，旨在确保其长期可靠性和可持续性。NEMO 有三个重要的组成部分：
 - [NEMO-OCE](https://zenodo.org/records/14515373)：模拟海洋热动力学并求解原始方程。
 - [NEMO-ICE](https://zenodo.org/records/7534900#.Y8GIF-xKg-Q)：模拟海冰热动力学、盐水包裹体和亚网格尺度厚度变化
 - [NEMO-TOP-PISCES](https://zenodo.org/records/1471700)：模拟海洋示踪剂运输和生物地球化学过程
# 2. 准备工作
## 2.1 类 Unix 机器
例如 Linux 发行版、MacOS。Windows 系统如需安装双系统 Linux 子系统，可参考[Microsoft 官方教程](https://learn.microsoft.com/zh-cn/windows/wsl/install)，通过 WSL 安装 Linux 系统后，推荐使用 [VSCode](https://code.visualstudio.com/) 的 WSL 插件。
![WSL](https://i-blog.csdnimg.cn/direct/9b4bec3f5c764f9595dadea3ec45892e.png#pic_center)
以下代码以Ubuntu为例，更新Ubuntu:

```bash
sudo apt update && sudo apt upgrade -y
```

## 2.2 subversion（svn）库
用于 XIOS 资源的版本控制。可通过 Linux 指令下载
```bash
sudo apt install subversion git -y
```
## 2.3 git 库
用于 NEMO 资源的版本控制。
## 2.4 Perl 解释器
可通过 Linux 指令下载
```bash
sudo apt-get install liburi-perl
```
## 2.5 MPI 消息传递接口
例如 OpenMPI 或 MPICH 。默认情况下，NEMO 需要 MPI-3。
## 2.6 Fortran 编译器
包括<kbd>ifort</kbd>、<kbd>gfortran</kbd>、<kbd>pgfortran</kbd>、<kbd>ftn</kbd>等。以<kbd>gfortran</kbd>为例：

```bash
sudo apt install gfortran -y
```
## 2.7 网络通用数据格式 NetCDF 及其底层分层数据格式 HDF
对于 XIOS，需要使用 NetCDF-4。如果不想使用 XIOS，仍然可以在 NEMO 中使用 NetCDF-3。如果用的是 Ubuntu/Debian，可以通过 Linux 指令下载
```bash
sudo apt-get update
sudo apt-get install libnetcdf-dev libnetcdff-dev libhdf5-dev libhdf5-mpi-dev
```
检查是否安装成功
```bash
nc-config --all
nf-config --all
```
# 3. 下载并安装 NEMO 代码
 NEMO 源代码是用 *Fortran 2008* 编写的，下载包中已包括 AGRIF 网格细化库、FCM 构建系统、PPR 多项式重建库和用于部分输出的 IOIPSL 库。[NEMO 5.0](https://forge.nemo-ocean.eu/nemo/nemo/-/releases/5.0) 开源，可下载压缩包，也可通过Linux指令下载
```bash
git clone --branch 5.0 https://forge.nemo-ocean.eu/nemo/nemo.git nemo_5.0
```
其他版本的 NEMO
```bash
git clone https://forge.nemo-ocean.eu/nemo/nemo.git
cd nemo
```
NEMO 主要目录包括：
 - <kbd>arch</kbd>：编译设置
 - <kbd>cfgs</kbd>：参考配置
 - <kbd>ext</kbd>：包含依赖项<kbd>AGRIF</kbd>、<kbd>FCM</kbd>、<kbd>PPR</kbd>、<kbd>IOIPSL</kbd>
 - <kbd>mk</kbd>：编译脚本
 - <kbd>src</kbd>：NEMO 代码库
 - <kbd>tests</kbd>：测试用例
 - <kbd>tools</kbd>：数据预处理和后处理的实用程序
 - <kbd>sct</kbd>：PSyclone代码转换脚本
 - <kbd>sette</kbd>：SETTE 代码测试框架

# 本栏目未来规划
未来即将更新 NEMO 演示程序的使用心得，参考 [NEMO ZOO](https://www.nemo-ocean.eu/nemo-zoo-demonstrators-and-tutorials/)。
![NEMO ZOO](https://i-blog.csdnimg.cn/direct/039d7a92e452432b934611c4042c4e09.png#pic_center)
