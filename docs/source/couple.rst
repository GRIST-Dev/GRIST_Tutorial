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

**4.	海浪模式：WW3 6.07**

下载地址：https://github.com/NOAA-EMC/WW3


安装和编译
~~~~~~~~~~~~~~~
安装 ESMF：
---------------------------------
**1.	安装包准备**

- 在线下载ESMF耦合器

下载地址：https://earthsystemmodeling.org/download/
    
- 使用TOOLs中的安装包
    
${GRISTMOM_PATH}/Tools/esmf-8.3.0

**2.	修改脚本**

修改安装文件 compile.bash:

.. code-block:: bash
 
    cd  ${GRISTMOM_PATH}/Tools/esmf-8.3.0/
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
    export ESMF_INSTALL_PREFIX=${GRISTMOM_PATH}/esmf-8.3.0-intel2019
    export ESMF_NETCDF=nc-config
    export ESMF_PIO=OFF #internal
    #
    gmake distclean
    gmake -j56
    gmake install

**3.	运行程序**
::
    cd ${GRISTMOM_PATH}/Tools/esmf-8.3.0
    ./compile.bash

**4.	环境变量配置**

修改 ${GRISTMOM_PATH}/Make/coupler-intel-mpich.sh, 具体修改内容见章节#编译

编译
---------------------------------

所有的编译已经集成到Make 文件夹中， 包括以下部分：

.. code-block:: bash

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

.. code-block:: bash

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
  export ESMF_PATH=${GRISTMOM_PATH}/esmf-8.3.0-intel2019
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

  # WW3
    export WWATCH3_DIR=WW3-6.07
    export PATH=${WWATCH3_DIR}/model/bin:${PATH}
    export PATH=${WWATCH3_DIR}/model/exe:${PATH}
    export WWATCH3_NETCDF=NC4
    export NETCDF_LIBDIR=${NETCDF_PATH}/lib
    export NETCDF_INCDIR=${NETCDF_PATH}/include
    export NETCDF_CONFIG=${NETCDF_PATH}/bin/nc-config

  #-------------------------------------------------

  ulimit -s unlimited
  export LD_LIBRARY_PATH=${ncl_path}/lib:$LD_LIBRARY_PATH
  export LD_LIBRARY_PATH=${mpich_path}/lib:$LD_LIBRARY_PATH

**2.	运行编译命令**

${GRISTMOM_PATH}/Make/Makefile 文件的使用命令：

.. code-block:: bash

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

**3.	海浪模式WW3**

编译之前需要指定  WW3-6.07.1/model/bin/ 中的文件： link、 comp、 switch
其中，link 相关的库的链接、comp 是编译选项、switch 是WW3的源函数/子程序的选项，具体配置可参考WW3-6.07.1/regtests/目录下的相关例子。

为实现耦合修改以下文件：w3profsmd.ftn ww3_esmf.ftn w3_make w3_automake make_makefile.sh
    w3profsmd.ftn | 由于GRIST运行需配置LAPACK库，与WW3 中的w3profsmd.ftn 冲突，因此修改w3profsmd.ftn中一些变量和程序名（搜索 WITH-GRIST ）
    ww3_esmf.ftn  | 为适配ESMF/NUOPC耦合器需要调整的run、exchange和mesh_define 等子程序
    w3_make w3_automake make_makefile.sh | 修改WW3 配套的编译脚本以实现编译ww3_esmf.ftn  ---> 生成 ww3_esmf.o libww3_esmf.a


编译命令:

.. code-block:: bash

    cd WW3-6.07.1/
    ./compile_clean.sh 
    #./model/bin/w3_setup model
    #./model/bin/w3_make

运行
~~~~~~~~~~~~~~~
**1.	修改运行环境**

.. code-block:: bash

    source ${GRISTMOM_PATH}/Make/coupler-intel-mpich.sh 

**2.	修改并行计算节点数**

对于GRIST-MOM耦合来说，需要修改MOMSIS_layout，SIS_layout，cplcfg.rc 中对应的节点数，以和run.sh中使用的一致。
对于GRIST-WW3耦合来说，并行节点数需要和ww3_grid 中的频率、波向的分组有关，需配置对应的参数。

**3.	运行命令**

.. code-block:: bash

    cd ${GRISTMOM_PATH}/run/
    ./batch.sh

数据前处理
~~~~~~~~~~~~~~~

**1.	大气模式：GRIST-23.6.26**

GRIST的前处理方法可参考章节#模式输入文件: 初值数据；#模式输入文件: 强迫数据

在这里我们提供了一些简单的可以生成GRIST 初始场和强迫场的脚本。

TOOLS/gendata-GRIST
文件夹下主要的内容有：

.. code-block:: bash

    ├── README                    # README文件 
    ├── GRID                      # 网格 (G6\G8\G9\cptp_50_3.5km_20230917  ....)
    ├── geniniFromERA5            # 利用ERA5 数据做初始场
    ├── geniniFromGFS             # 利用GFS  数据做初始场
    ├── gensstFromERA5            # 利用ERA5 数据做强迫场
    ├── gensstFromGFS             # 利用GFS  数据做强迫场
    ├── namelist                  # 生成namelist的脚本以及namelist的例子
    ├── wrf-data                   
    └── noahmp_data

- 网格 
目前提供G6/G8/G9 三套网格的基本信息，将通过namelist引入模式计算

.. code-block:: bash
   ├── GRID
        ├── G6
            ├── grist.grid_file.g6.ccvt.0d.nc      
            ├── grist.grid_file.g6.ccvt.2d.nc
            ├── static_uniform_g6.nc             #静态数据，制作方法参考章节#模式输入文件: 静态数据
            ├── grist_scrip_655362.nc
        ├── G8      
        └── G9

- 所需初始场、强迫场数据 

.. code-block:: bash

    ├── geniniFromERA5      # 利用ERA5数据制作初始场
    ├── geniniFromGFS       # 利用GFS 数据制作初始场

    ├── gensstFromERA5      # 利用ERA5数据制作强迫场
    └── gensstFromGFS       # 利用GFS  数据做强迫场

- 制作方法

.. code-block:: bash

    ./${GRISTMOM_PATH}/TOOLS/gendata-GRIST/geniniFromERA5/scripts/pre_process.sh  
    ./${GRISTMOM_PATH}/TOOLS/gendata-GRIST/geniniFromGFS/scripts/pre_process.sh 
    ./${GRISTMOM_PATH}/TOOLS/gendata-GRIST/gensstFromERA5/scripts/pre_process.sh 
    ./${GRISTMOM_PATH}/TOOLS/gendata-GRIST/gensstFromGFS/scripts/pre_process.sh 

**2.	海流+海冰模式：MOM6+SIS2**

- 网格

- 所需初始场、强迫场数据

- 制作方法


**3.	海浪模式：WW3 **
路径： ~/gaoj/TOOLS/gendata-WW3/
- 网格
TOOLS/gendata-WW3/GRID

.. code-block:: bash


    ├── GLO_15m         #  1/4°分辨率的全球网格数据 
        ├── GLO_15m.depth_ascii
        ├── GLO_15m.maskorig_ascii
        ├── GLO_15m.meta
        ├── GLO_15m.obstr_lev1
        ├── gridgen.GLOB-15M_etopo1.nml
        ├── namelists_ww3_grid.nml

    ├── GLO_30m         #  1/2°分辨率的全球网格数据 
    ├── GLO_60m         #  1°分辨率的全球网格数据 

    ├── ncpus-10        #  1/2°分辨率的全球网格数据 + 10个节点运行时的ww3_grid 配置
    └── ncpus-20        #  1/2°分辨率的全球网格数据 + 20个节点运行时的ww3_grid 配置


- 所需初始场、强迫场数据
生成脚本：TOOLS/gendata-WW3/genWindFromERA5/script_make_ERA5_wind_for_ww3.sh
数据：
    1）下载脚本： TOOLS/gendata-WW3/genWindFromERA5/download_era5.py
    2）已有数据： ~/data/ERA5_DATA/*  【逐6小时，2000-2023】

- 制作方法
./~/gaoj/TOOLS/gendata-WW3/genWindFromERA5/download.sh
./~/gaoj/TOOLS/gendata-WW3/genWindFromERA5/script_make_ERA5_wind_for_ww3.sh


运行namelist

~~~~~~~~~~~~~~~
**1.	大气模式：GRIST-23.6.26**

GRIST模式的namelist主要有以下：

.. code-block:: bash

    ├── grist.nml
    ├── grist_lsm_noahmp.nml
    └── grist_amipw_phys.nml 

在本耦合模式中，grist.nml 和其他GRIST配置一样，需要考虑网格、输入文件的路径等进行配置。
因为耦合的通量部分仅配置在部分物理包中，需要特别注意 grist_amipw_phys.nml 中使用的物理包。以下是grist_amipw_phys.nml 的参考配置：

.. code-block:: bash

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

**3.	海浪模式：WW3 **
WW3模式的namelist主要有以下5个文件（按照运行顺序排列）：

.. code-block:: bash

    ├── ww3_grid.nml    # grid   相关，修改 SPECTRUM%（波谱、波向信息和并行度相关）、TIMESTEPS%（时间步长）、GRID%（网格文件信息）、RECT%（网格nx, ny信息）
    ├── ww3_strt.inp    # inital 相关，修改 ITPYE（建议： ITPYE=3）
    ├── ww3_prnc.nml    # force  相关，修改 FILE%，FORCING%
    ├── ww3_shel.nml    # run    相关，修改 DOMAIN%（运行时间、restart间隔等）、INPUT%FORCING（强迫变量），TYPE%FIELD%LIST（输出变量），DATE%FIELD（输出时间）
    └── ww3_ounf.nml    # output 相关，输出nc文件，修改 FIELD%（时间、变量）、FILE%（文件Flag）


数据后处理
~~~~~~~~~~~~~~~
**1.	大气模式：GRIST-23.6.26**

输出变量 {GRIST_PATH}/src/atmosphere/gcm/io_gaoj/grist_gcm_io_h1_module/F90

可视化代码

**2.	海流+海冰模式：MOM6+SIS2**

输出变量 ---< diag_table

可视化代码

**3.	海浪模式WW3**

输出变量 ---< ww3_shel.nml 、ww3_ounf.nml

可视化代码

其他
~~~~~~~~~~~~~~~

