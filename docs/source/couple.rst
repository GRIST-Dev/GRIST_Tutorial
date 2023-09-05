耦合模拟
=================================
模式组成介绍
~~~~~~~~~~~~~~~
整个耦合模式包括四部分：   
    - ESMF  （推荐版本：8.3.0.）
    - GRIST （仅支持版本：23.6.26.）
    - MOM   （推荐版本：MOM6 + SIS2）
    - WW3   （推荐版本：6.07）
  
**1.	耦合器：ESMF 8.3.0.**

ESMF(Earth System Modeling Frame) 是由NASA提出并维护的面向地球系统模式的并行软件开发框架。它不含任何分量模式代码，而是提供了一套包括核心框架、天气及气候建模、数据同化应用等的标准接口工具。借助ESMF耦合器可以快速将大气、海洋、海浪模式中的各个分量进行网格匹配插值和数据通信。目前ESMF已被应用于SKRIPS（Sun,et al., 2019[1], 2023[2]）、COAMPS（Hodur, 1997[3]）等多个耦合模式。采用耦合器技术搭建耦合模型，一方面便于区域耦合模式各个子分量模式的发展和维护，是耦合模式的主流技术和发展方向。另一方面，耦合器技术将海流和海浪模式接口可插拔式耦合在GRIST模式上，确保不同用户可自由选择耦合模式和分量。
下载地址：https://github.com/esmf-org/esmf

**2.	大气模式：GRIST 23.6.26**
目前耦合版本的GRIST模式仅支持：GRIST-23.6.26 版本。

**3.	海流+海冰模式：MOM6+SIS2**
本项目使用海流、海冰耦合版本的MOM6+SIS2模式
下载地址：https://github.com/NOAA-GFDL/MOM6-examples

**4.	海浪模式：WW3 6.07[未上线]**
WW3 海浪模式
下载地址：https://github.com/NOAA-EMC/WW3


编译和运行
~~~~~~~~~~~~~~~
编译
---------------------------------
编译需要统一以下几条，可设在环境变量中：
::
    export NETCDF=/fs2/software/netcdf/4.8.0-icc19.0-mpi-x
    export PNETCDF=/fs2/software/pnetcdf/1.12.2-icc19.0-mpi-x
    export METIS=/fs2/home/zhangyi/softwares/metis-5.1.0
    export LAPACK=/fs2/software/lapack/3.10.0-icc19.0

    其中，LAPACK仅用于 GRIST的编译运行


**1.	耦合器：ESMF 8.3.0.**
耦合器下载地址：https://earthsystemmodeling.org/download/
安装-修改脚本：
:: 
    # esmf-8.3.0/compile.bash
    #指定FC、CC、CXX 【环境是mpich2】
    FC=mpifort
    CC=mpicc
    CXX=mpicxx
    #
    export ESMF_OS=Linux
    export ESMF_DIR=esmf-8.3.0
    export ESMF_BOPT=O
    export ESMF_COMM=mpich  #intelmpi
    export ESMF_COMPILER=intel
    export ESMF_INSTALL_PREFIX=GRISTMOM/esmf-8.3.0-intel2019
    export ESMF_NETCDF=nc-config
    export ESMF_PIO=OFF #internal
    #
    gmake distclean
    gmake -j32
    gmake install

安装-运行程序：
::
    cd esmf-8.3.0
    ./compile.bash

安装好ESMF后，需要在环境变量中配置ESMF相关变量
环境变量配置:
::
    export ESMF=GRISTMOM/esmf-8.3.0-intel2019
    #export ESMFMKFILE=${ESMF}/lib/libO/Linux.intel.64.mpich.default/esmf.mk
    export ESMF_LIBDIR=${ESMF}/lib/libO/Linux.intel.64.mpich.default
    export ESMF_MODDIR=${ESMF}/mod/modO/Linux.intel.64.mpich.default

**2.	大气模式：GRIST-23.6.26**

环境变量配置需要链接以下软件： NETCDF， PNETCDF， METIS， LAPACK。已经集成到 环境变量中
修改脚本：
::
    # ParGRIST-A23.6.26v1/src/grist_lib/bld/MakeFile
    FC  = mpifort ${oplevel3}
    CC  = mpicc   ${oplevel3}
    CXX = mpicxx  ${oplevel3}

    # ParGRIST-A23.6.26v1/bld/build_amipw/Makefile
    EXEDIR      =
    EXENAME     = ParGRIST-amipw.exe
    FC          = ${FCompiler} -convert big_endian -r8 -DAMIPW_PHYSICS -DAMIPW_CLIMATE -DUSE_NOAHMP -DCDATE -DCOUPLE

编译命令:
::
    cd ParGRIST-A23.6.26v1/src/grist_lib/bld
    make lib
    cd ParGRIST-A23.6.26v1/bld/build_amipw
    ./make.sh

**3.	海流+海冰模式：MOM6+SIS2**

环境变量配置需要链接以下软件： NETCDF， PNETCDF， METIS。已经集成到环境变量中
修改脚本：
::
    cd MOM6-v4/build/intel/
    # 需要修改FC、CC（以 hpc5 上的mpich2为例）
    # MOM6-v4/build/intel/ncrc-intel.mk
    FC = mpifort -DCOUPLE
    CC = mpicc
编译命令：
::
    cd MOM6-v4/build/intel/shared/
    make NETCDF=4 REPO=1 –j8    
    cd MOM6-v4/build/intel/ice_ocean_SIS2/repro/
    make couple

**4.	海浪模式：WW3 [未上线]**
环境变量配置
::
    export WW3_DIR=WW3-6.07.1
    export PATH=${WW3_DIR}/model/bin:${PATH} 
    export PATH=${WW3_DIR}/model/exe:${PATH}
    export WWATCH3_NETCDF=NC4
    export NETCDF_LIBDIR=${NETCDF}/lib
    export NETCDF_INCDIR=${NETCDF}/include
    export NETCDF_CONFIG=${NETCDF}/bin/nc-config

编译之前需要指定  WW3-6.07.1/model/bin/  中的文件：
    link
    comp
    switch
其中，link 相关的库的链接
    comp   是编译选项
    switch是WW3的源函数/子程序的选项，
具体配置可参考该目录下的相关例子

编译命令:
::
    cd WW3-6.07.1/
    ./compile_clean.sh 
    #./model/bin/w3_setup model
    #./model/bin/w3_make

**5.	耦合模式：**

环境变量，需指定相关库和模式的位置
这里用到的有  NETCDF， PNETCDF， METIS， LAPACK
修改脚本：
::
    # wrfphys_esmf8.3_G23.6/makefile
    INDIR=/GRISTMOM
    #GRIST
    GRISTDIR = $(INDIR)/ParGRIST-A23.6.26v1
    #MOM6
    MOMDIR  = $(INDIR)/MOM6-v4
    #ESMF 

编译命令：
::
    cd wrfphys_esmf8.3_G23.6
    make 

运行
---------------------------------


数据前处理
~~~~~~~~~~~~~~~
**1.	耦合器：ESMF 8.3.0.**

所需初始场、强迫场数据
制作方法

**2.	大气模式：GRIST-23.6.26**

所需初始场、强迫场数据
制作方法

**3.	海流+海冰模式：MOM6+SIS2**

所需初始场、强迫场数据
制作方法

**4.	海浪模式：WW3 [未上线]**

所需初始场、强迫场数据
制作方法



运行namelist
~~~~~~~~~~~~~~~
**1.	大气模式：GRIST-23.6.26**


**2.	海流+海冰模式：MOM6+SIS2**



**3.	海浪模式：WW3 [未上线]**


数据后处理
~~~~~~~~~~~~~~~
**1.	大气模式：GRIST-23.6.26**

输出变量
可视化代码

**2.	海流+海冰模式：MOM6+SIS2**

输出变量
可视化代码

**3.	海浪模式：WW3 [未上线]**

输出变量
可视化代码

其他
~~~~~~~~~~~~~~~

