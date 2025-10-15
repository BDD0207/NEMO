@[TOC](【NEMO 海洋模型】NEMO-PISCES 1D 案例)
# 1. NEMO-PISCES 1D 简介
[PISCES 模型](https://zenodo.org/records/7139521)  是基于这样一个假设构建的，即浮游植物的生长直接受到外部营养物质供应的限制。该模型包括24个组成部分（如[图1](https://i-blog.csdnimg.cn/direct/fb04a7fa35f5452e9582d3ece5040e99.png#pic_center)）：

 - 5 种不同营养物质的限制浮游植物的生长：硝酸盐、 铵盐、 磷酸盐、 硅酸盐和铁。 
 - 4 个生物池： 两类浮游植物（微型浮游植物和硅藻）和两类浮游动物（微型浮游动物和中型浮游动物）。硅藻与微型浮游植物的区别在于其对硅的需求、 对铁的更高需求以及由于硅藻平均尺寸较大而具有更高的半饱和常数。
 - 3 类非生物组分：半活性溶解有机物（DOM）、小颗粒和大颗粒。这两种颗粒大小类别因其下沉速度不同而有所区别（小颗粒为每天 2 米， 大颗粒为每天 50 至 200 米）。生物组分的碳氮磷被设定为恒定 Redfield 比例。然而，颗粒中的铁、硅和方解石库是完全模拟的。因此，它们相对于有机碳的比例可以变化。 模型中未考虑球粒矿物对颗粒下沉速度的影响。
 - 3 种不同的营养物质源：大气尘埃沉降、 河流以及海洋沉积物的再矿化。 
![PISCES 运行原理图](https://i-blog.csdnimg.cn/direct/fb04a7fa35f5452e9582d3ece5040e99.png#pic_center)
# 2. 模型先决条件
## 2.1 Netcdf 软件包
```bash
# 下载 Netcdf
sudo apt-get update
sudo apt-get install libnetcdf-dev libnetcdff-dev libhdf5-dev libhdf5-mpi-dev

# 查看下载路径
nf-config --fflags
nf-config --flibs
```
## 2.2 XIOS 软件包
 新建一个工作目录，在此工作目录中下载 XIOS 软件包
```bash
mkdir WORK  # 新建工作目录
cd WORK     # 进入工作目录
svn co http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/trunk@2331 xios  # 下载XIOS
```
## 2.3 新的架构文件
一个用于环境（.env），一个用于编译器（.fcm），一个用于路径（.path）。
```bash
cd WORK/xios/arch  # 进入arch文件夹
# 在arch中新建3个空白文件，"local"可自行更换，"arch-"不可更换
touch arch-local.env
touch arch-local.fcm
touch arch-local.path
```
分别进入3个文件，将下面的内容粘贴到文件内，保存并退出文件编辑
```bash
vi arch-local.env  # 使用vi编辑器
```
点击<kbd>I</kbd>，进入编辑模式
```vi
# arch.local.env

# MPI
export MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi
export PATH=$MPI_HOME/bin:$PATH
export LD_LIBRARY_PATH=$MPI_HOME/lib:$LD_LIBRARY_PATH

# NetCDF
export NETCDF_HOME=/usr
export LD_LIBRARY_PATH=$NETCDF_HOME/lib:$LD_LIBRARY_PATH

# HDF5
export HDF5_HOME=/usr
export LD_LIBRARY_PATH=$HDF5_HOME/lib:$LD_LIBRARY_PATH
```
点击<kbd>Esc</kbd>，退出编辑模式，输入<kbd>:wq</kbd>+<kbd>Enter</kbd>，保存并退出。另外两个文件内容如下：

```vi
# arch.local.fcm
%CCOMPILER      mpicc
%FCOMPILER      mpif90
%LINKER         mpif90

# C++ 编译选项
%BASE_CFLAGS    -I/usr/lib/x86_64-linux-gnu/netcdf/mpi/include \
                -I/home/bdd/netcdf-c-4.9.2/include \  # 此行需根据自己系统的储存位置进行更改
                -std=c++11 -w -D__XIOS_EXCEPTION \
                -DBOOST_NO_AUTO_PTR -DBOOST_NO_CXX98_FUNCTION_BASE \
                -Wno-deprecated-declarations -Wno-nonnull \
                -Wno-unused-variable -Wno-unused-parameter

%PROD_CFLAGS    -O3 -DBOOST_DISABLE_ASSERTS
%DEV_CFLAGS     -g -O2
%DEBUG_CFLAGS   -g -O0

# Fortran 编译选项
%BASE_FFLAGS    -D__NONE__ -ffree-line-length-none
%PROD_FFLAGS    -O3 -ffree-line-length-none
%DEV_FFLAGS     -g -O2 -ffree-line-length-none
%DEBUG_FFLAGS   -g -O0 -fcheck=all -fbacktrace -ffree-line-length-none


# 头文件路径
%BASE_INC       -I/usr/lib/x86_64-linux-gnu/netcdf/mpi/include \
				-I/home/bdd/netcdf-c-4.9.2/include \  # 此行需根据自己系统的储存位置进行更改
                -I/usr/include \
                -I/usr/include/boost \
                -I/usr/lib/x86_64-linux-gnu/openmpi/include \
                -I/usr/lib/x86_64-linux-gnu/openmpi/include/openmpi

# 链接库路径和库
%BASE_LD        -L/usr/lib/x86_64-linux-gnu/hdf5/openmpi \
                -L$HOME/local/lib -L/usr/lib/x86_64-linux-gnu \
                -lnetcdf -lnetcdff -lhdf5_hl -lhdf5 -lz -lcurl -lstdc++

# 预处理与构建工具
%CPP            cpp
%FPP            cpp -P
%MAKE           gmake
```

```vi
# arch.local.path

# --- Include paths (NetCDF + XIOS + MPI) ---
%BASE_INC     -I/usr/include \
              -I/usr/include/hdf5/serial \
              -I/usr/lib/x86_64-linux-gnu/openmpi/include \
              -I/usr/lib/x86_64-linux-gnu/openmpi/include/openmpi \
              -I$HOME/xios-2.5/inc

# --- Library paths (NetCDF + XIOS + MPI) ---
%BASE_LD      -L/usr/lib/x86_64-linux-gnu \
              -L/usr/lib/x86_64-linux-gnu/hdf5/serial \
              -L/usr/lib/x86_64-linux-gnu/openmpi/lib \
              -lnetcdff -lnetcdf -lhdf5_hl -lhdf5 \
              -lxios -lstdc++ -lmpi
```

> **注：如果下面的编译 XIOS 报错，多数是因为架构文件设置不合理，请自行检查各个路径是否正确！！！**
## 2.4 NEMO-PISCES 1D 包
下载[PISCES_Demonstrator](https://zenodo.org/records/7139521/files/PISCES_Demonstrator.tgz?download=1)压缩包，手动将其粘贴到前面新建的工作目录中，利用指令解压缩
```bash
cd WORK
ls  # 此时应该能看到粘贴好的压缩包
tar xvzf PISCES_Demonstrator.tgz
ls  # 此时应该能看到解压缩后的PISCES_Demonstrator文件夹
```
# 3. 编译 XIOS
进入xios文件夹，利用前面设置好的架构文件进行编译
```bash
cd WORK/xios
./make_xios --arch local --full --prod --job 8  # 清除之前的缓存，重新编译
# 此处的"local"对应步骤2.3设置的文件名
```
> **注：如果报错，多数是因为步骤2.3架构文件设置不合理，请自行检查各个路径是否正确！！！
如果报错，多数是因为步骤2.3架构文件设置不合理，请自行检查各个路径是否正确！！！
如果报错，多数是因为步骤2.3架构文件设置不合理，请自行检查各个路径是否正确！！！**

编译成功界面：
![XIOS编译成功啦](https://i-blog.csdnimg.cn/direct/ca4eee9b607940f1ac1d05e4b04fb0ff.png#pic_center)
# 4. 编译并创建 NEMO 可执行文件

 - 获取 NEMO-5.0 代码
```bash
cd ..  # 此时在WORK文件夹下
git clone https://forge.nemo-ocean.eu/nemo/nemo.git nemo-5.0
cd nemo-5.0
git switch --detach 5.0
```
 - 设置架构配置文件，以供编译NEMO。可使用官方的自动生成文件
```bash
cd arch
./build_arch-auto.sh
```

 - 编译
```bash
cd ..
rm -rf cfgs/ORCA_1D_PISCES
./makenemo -n ORCA_1D_PISCES -r GYRE_PISCES -m auto -j 4
```
编译成功界面如下：
![NEMO编译成功啦](https://i-blog.csdnimg.cn/direct/b78e9a87c86f4171976395e2cdb20693.png#pic_center)
- 运行

```bash
cd cfgs/ORCA_1D_PISCES/EXP00
cp -r ~/nemo/WORK/nemo-5.0/cfgs/ORCA_1D_PISCES/BLD/bin/nemo.exe .
./nemo.exe
```
成功运行界面如下：
![成功运行耶耶耶](https://i-blog.csdnimg.cn/direct/7324c08b52014c63b9c9d460cafe92b6.png#pic_center)
成功运行：<kbd>STOP 0</kbd>
实际运行时间：<kbd>313.937 s</kbd>
XIOS 性能优秀：I/O开销占比<kbd>0.00101682 %</kbd>

# 参考
[NEMO 官方用户指南](https://sites.nemo-ocean.io/user-guide/install.html)
[NEMO Zoo](https://www.nemo-ocean.eu/nemo-zoo-demonstrators-and-tutorials/)
[PISCES 社区](https://www.pisces-community.org/)



