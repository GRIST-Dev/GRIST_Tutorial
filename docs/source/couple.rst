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

ESMF(Earth System Modeling Frame) 是由NASA提出并维护的面向地球系统模式的并行软件开发框架。它不含任何分量模式代码，而是提供了一套包括核心框架、天气及气候建模、数据同化应用等的标准接口工具。借助ESMF耦合器可以快速将大气、海洋、海浪模式中的各个分量进行网格匹配插值和数据通信。目前ESMF已被应用于SKRIPS、COAMPS等多个耦合模式。采用耦合器技术搭建耦合模型，一方面便于区域耦合模式各个子分量模式的发展和维护，是耦合模式的主流技术和发展方向。另一方面，耦合器技术将海流和海浪模式接口可插拔式耦合在GRIST模式上，确保不同用户可自由选择耦合模式和分量。

下载地址：https://github.com/esmf-org/esmf

**2.	大气模式：GRIST 23.6.26**

目前耦合版本的GRIST模式仅支持：GRIST-23.6.26 版本。

**3.	海流+海冰模式：MOM6+SIS2**

本项目使用海流、海冰耦合版本的MOM6+SIS2模式

下载地址：https://github.com/NOAA-GFDL/MOM6-examples

**4.	海浪模式：WW3 6.07[未上线]**

下载地址：https://github.com/NOAA-EMC/WW3


安装和编译
~~~~~~~~~~~~~~~
安装 ESMF：
---------------------------------
**1.	安装包准备**

- 在线下载ESMF耦合器

下载地址：https://earthsystemmodeling.org/download/
    
- 使用TOOLs中的安装包
    
GRISTMOM/Tools/esmf-8.3.0

**2.	修改脚本**

修改安装文件 compile.bash:
:: 
    cd  GRISTMOM/Tools/esmf-8.3.0/
    # compile.bash
    # 指定FC、CC、CXX 【环境是mpich2】
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
    gmake -j56
    gmake install

**3.	运行程序**
::
    cd GRISTMOM/Tools/esmf-8.3.0
    ./compile.bash
**4.	环境变量配置**
    修改 GRISTMOM/Make/coupler-intel-mpich.sh, 具体修改内容见章节#编译

编译
---------------------------------

所有的编译已经集成到Make 文件夹中， 包括以下部分：
::
    ├── Makefile                        # MakeFile
    ├── README                          # README文件 
    ├── coupler-intel-mpich.sh          # 环境变量
    ├── GRIST.Makefile.grist_lib        # ${GRISTDIR}/src/grist_lib/bld/Makefile
    ├── GRIST.Makefile.build_amipw      # ${GRISTDIR}/bld/build_amipw/Makefile
    ├── MOM.mk                          # ${MOMDIR}/build/intel/ncrc-intel.mk
    ├── MOM.Makefile.shared             # ${MOMDIR}/build/intel/shared/repro/Makefile
    ├── MOM.Makefile.ice_ocean_SIS2     # ${MOMDIR}/build/intel/ice_ocean_SIS2/repro/Makefile
    └── COUPLER.Makefile                # ${COUPLERDIR}/Makefile

**1.	修改环境变量文件coupler-intel-mpich.sh**
::
  module add hdf5/1.12.0-icc19.0-mpi-x
  module add pnetcdf/1.12.2-icc19.0-mpi-x
  module add netcdf/4.8.0-icc19.0-mpi-x
  module add lapack/3.10.0-icc19.0
  module add zlib/1.2.11-icc19.0
  module add cdo/2.0.5-gcc8.5.0
  module add nco/5.1.1-icc19.0
  module add ncl
  module add ncview/2.1.8

  export FC=mpifort
  export CC=mpicc
  export CXX=mpicxx
  export FCompiler=${FC}

  # 环境变量中配置 NETCDF_PATH，PNETCDF_PATH，METIS_PATH，LAPACK_PATH
  export NETCDF_PATH={}
  export PNETCDF_PATH={}
  export METIS_PATH={}
  export LAPACK_PATH={}
  # 其中，LAPACK仅用于 GRIST的编译运行

  # 环境变量中配置ESMF：
  export ESMF_PATH=GRISTMOM/esmf-8.3.0-intel2019
  #export ESMFMKFILE=${ESMF_PATH}/lib/libO/Linux.intel.64.mpich.default/esmf.mk
  export ESMF_LIBDIR=${ESMF_PATH}/lib/libO/Linux.intel.64.mpich.default
  export ESMF_MODDIR=${ESMF_PATH}/mod/modO/Linux.intel.64.mpich.default

  # 环境变量中修改  PATH，LDFLAGS，CPPFLAGS，LD_LIBRARY_PATH
  export PATH=${NETCDF_PATH}/bin:${PATH}
  export LDFLAGS=-L${NETCDF_PATH}/lib/:${LDFLAGS}
  export CPPFLAGS=-I${NETCDF_PATH}/include/:${CPPFLAGS}
  export LD_LIBRARY_PATH=${NETCDF_PATH}/lib/:${LD_LIBRARY_PATH}
  export LD_LIBRARY_PATH=${HDF5_PATH}/lib/:${LD_LIBRARY_PATH}
  export LD_LIBRARY_PATH=${LAPACK_PATH}/lib/:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH=${METIS_PATH}/lib/:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH=${ESMF_LIBDIR}:$LD_LIBRARY_PATH

  # 为WW3做准备
  # export WW3_DIR=WW3-6.07.1
  # export PATH=${WW3_DIR}/model/bin:${PATH} 
  # export PATH=${WW3_DIR}/model/exe:${PATH}
  # export WWATCH3_NETCDF=NC4
  # export NETCDF_PATH_LIBDIR=${NETCDF_PATH}/lib
  # export NETCDF_PATH_INCDIR=${NETCDF_PATH}/include
  # export NETCDF_PATH_CONFIG=${NETCDF_PATH}/bin/nc-config
  #-------------------------------------------------

  ulimit -s unlimited
  export LD_LIBRARY_PATH=${ncl_path}/lib:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH=${mpich_path}/lib:$LD_LIBRARY_PATH

**2.	运行编译命令**

GRISTMOM/Make/Makefile 文件的使用命令：
::
  make         # 编译 所有模式 env GRIST MOM GRIST-MOM 
  make env     # 启动 环境变量 coupler-intel-mpich.sh
  make GRIST   # 编译 ParGRIST-A23-v1
  make MOM     # 编译 MOM6-v4  
  make COUPLER # 编译 wrfphys_esmf8.3-G23.6 
  make clean           # clean 所有模式 clean-GRIST clean-MOM clean-COUPLER
  make clean-GRIST     # clean ParGRIST-A23-v1
  make clean-MOM       # clean MOM6-v4
  make clean-COUPLER   # clean wrfphys_esmf8.3-G23.6

具体各模式的编译命令可在 Makefile 中查看

各个模式编译需要的Makefile 也在文件夹中列出。
                                      



**3.	海浪模式：WW3 [未上线]**

编译之前需要指定  WW3-6.07.1/model/bin/ 中的文件： link、 comp、  switch

其中，link 相关的库的链接、comp 是编译选项、switch 是WW3的源函数/子程序的选项，具体配置可参考该目录下的相关例子。

编译命令:
::
    cd WW3-6.07.1/
    ./compile_clean.sh 
    #./model/bin/w3_setup model
    #./model/bin/w3_make

运行
~~~~~~~~~~~~~~~
**1.	修改运行环境**
::
    source GRISTMOM/Make/coupler-intel-mpich.sh 

**2.	修改并行计算节点数**

    对于MOM耦合来说，需要修改MOMSIS_layout，SIS_layout，cplcfg.rc 中对应的节点数，以和run.sh中使用的一致。

**3.	运行命令**
::
    cd GRISTMOM/run/
    ./batch.sh

数据前处理
~~~~~~~~~~~~~~~

**1.	大气模式：GRIST-23.6.26**

GRIST的前处理方法可参考章节#模式输入文件: 初值数据；#模式输入文件: 强迫数据

在这里我们提供了一些简单的可以生成GRIST 初始场和强迫场的脚本。

GRISTMOM/TOOLS/gendata-GRIST
文件夹下主要的内容有：
::
    ├── README                    # README文件 
    ├── G6                        # G6 网格
    ├── G8                        # G8 网格
    ├── G9                        # G9 网格
    ├── geniniFromERA5            # 利用ERA5 数据做初始场
    ├── geniniFromGFS             # 利用GFS  数据做初始场
    ├── gensstFromERA5            # 利用ERA5 数据做强迫场
    ├── namelist                  # 生成namelist的脚本
    ├── wrf-data                   
    └── noahmp_data

- 网格 
目前提供G6/G8/G9 三套网格的基本信息，将通过namelist引入模式计算
::
    ├── G6
        ├── grist.grid_file.g6.ccvt.0d.nc      
        ├── grist.grid_file.g6.ccvt.2d.nc
        ├── static_uniform_g6.nc             #静态数据，制作方法参考章节#模式输入文件: 静态数据
        ├── grist_scrip_655362.nc
    ├── G8      
    └── G9

- 所需初始场、强迫场数据 
包括：
::
    ├── geniniFromERA5      # 利用ERA5数据制作初始场数据
    ├── geniniFromGFS       # 利用ERA5数据制作初始场数据
    └── gensstFromERA5      # 利用ERA5数据制作强迫场数据
- 制作方法
运行：
::
    ./ GRISTMOM/TOOLS/gendata-GRIST/geniniFromERA5/scripts/pre_process.sh  
    ./ GRISTMOM/TOOLS/gendata-GRIST/geniniFromGFS/scripts/pre_process.sh 
    ./ GRISTMOM/TOOLS/gendata-GRIST/gensstFromERA5/scripts/pre_process.sh 

**2.	海流+海冰模式：MOM6+SIS2**

- 网格

- 所需初始场、强迫场数据

- 制作方法

**3.	海浪模式：WW3 [未上线]**

- 网格

- 所需初始场、强迫场数据

- 制作方法

运行namelist
~~~~~~~~~~~~~~~
**1.	大气模式：GRIST-23.6.26**

GRIST模式的namelist主要有以下：
::
    ├── grist.nml
    ├── grist_lsm_noahmp.nml
    └── grist_amipw_phys.nml 
在本耦合模式中，grist.nml 和其他GRIST配置一样，需要考虑网格、输入文件的路径等进行配置。
因为耦合的通量部分仅配置在部分物理包中，需要特别注意 grist_amipw_phys.nml 中使用的物理包。以下是grist_amipw_phys.nml 的参考配置：
::
    &wrfphys_para    
     wrfphys_cu_scheme      = 'NTDKV381'
     wrfphys_cf_scheme      = 'RANDALL'
     wrfphys_ra_scheme      = 'RRTMGV381'
     wrfphys_rasw_scheme    = 'RRTMGV381'
     wrfphys_ralw_scheme    = 'RRTMGV381'
     wrfphys_mp_scheme      = 'WSM6V381'
     wrfphys_bl_scheme      = 'YSUV381'
     wrfphys_sf_scheme      = 'SFCLAYV381'
     wrfphys_lm_scheme      = 'noahmp'
     wphys_has_req          = 1
     unuse_cu               = .false.
     step_cu                = 5

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

