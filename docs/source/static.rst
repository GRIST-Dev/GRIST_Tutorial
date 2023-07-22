静态数据
================
用于全球模式的静态数据制作
----------------
  依赖软件：
    - NetCDF库
    - Lapack库
    - 编译器

  namelist设置：
    #. 编译grs
    #. 编译GRIST主程序
    #. 准备前处理数据
    #. 运行模式

编译GRIST_lib库
  该步骤包括使用GNU make命令(gmake)编译和链接GRIST_lib库文件。用户需根据计算机运行环境在编译目录中修改Makefile文件。然后执行gmake命令完成编译。

编译GRIST主程序
  该步骤和上述步骤相似。其流程和编译GRIST_lib一致：进入编译目录，修改Makefile文件，修改后执行make.sh命令完成编译。

准备前处理数据
  该步骤生成模式运行所必要的前处理文件，包括网格文件，初/边值条件文件以及静态数据文件。

运行模式
  这一步包括可执行文件的实际调用。当运行模式并行运行时，需要调用MPI可执行文件。在大多数高性能计算（HPC）平台上，一般通过批处理队列系统访问计算资源。（不同HPC平台批处理命令各有差异，需根据实际情况修改）。

用于有限区域模式的静态数据制作
----------------
  下面几小节中会给出交互式shell命令来编译和运行GRIST模式。大部分情况下，建议将这些步骤封装在shell脚本中。这样做最重要的优点是它可以记录每一步用户所做的运行步骤。

编译GRIST_lib库
~~~~~~~~~~~~~~~~
  首先进入grist_lib库的编译目录::

    $ cd ${GRIST_HOME}/src/grist_lib/bld
  
  然后修改Makefile文件中的编译选项：修改 FC、 CC 和 CXX选项来指定 Fortran、 C 和 CXX的编译器(对于 Intel 编译器，示例配置为: FC = mpifort，CC = mpiicc，CXX = mpiicpc)。然后修改 METIS _ LIB 指定 METIS lib 目录。
  以上步骤完成后，输入::

    $ make lib
  等待编译完成。

编译GRIST主程序
~~~~~~~~~~~~~~~~
  GRIST_lib编译完成后，可对GRIST主程序进行编译。首先选择编译的GRIST工作模式并进入该工作模式目录（包括amipc, amipw, gcm, scm, swe），这里以gcm模式的编译为例::

    $ cd ${GRIST_HOME}/bld/build_gcm
  修改该目录下的Makefile文件：修改 NETCDF、 PNETCDF、 LAPACK 和 METIS来指定对应软件的路径。修改 EXEDIR 指定 GRIST 的可执行文件目录。然后修改 FC 指定 Fortran 编译器。
  修改完成后，输入::

    $ ./make.sh
  对GRIST可执行程序进行编译。
  如果编译成功，执行目录中会出现两个可执行文件: ${model}.exe 和 parttion.exe。这代表GRIST整个编译流程完成。

准备前处理数据
~~~~~~~~~~~~~~~~
  在模式运行前需准备前处理数据。其中，网格文件和静态数据为开发者事先生成，用户可根据自身试验需求下载或联系模式支持获取。一般不推荐用户自行生成网格和静态数据，这样做可能会产生意料之外的bug。
  初/边值条件则需要提前下载相应数据（例如，ERA5、GFS），数据下载后可放入${GRIST_HOME}/data中完成对初/边值条件的初始化，该目录下有gesSST, init和post三个文件，分别存放生成边值条件，生成初始场和初值后处理的示例脚本，详情参考README.md文件，根据自身需求修改示例脚本并运行后生成初/边值条件。

运行模式
~~~~~~~~~~~~~~~~
  以上步骤完成后，即可运行GRIST。需要指出，所有前处理文件都可以生成后重复使用，如服务器中已存在所需前处理文件，则可以直接进入模式运行阶段。
  GRIST提供了多个示例脚本来自定义模式配置，这里仍对gcm试验的示例脚本（run_dcmip_tc_g6_rk3o3_rj.sh）进行介绍。其目录在${GRIST_HOME}/run/run_mode_dtp_dcmiptc下。该脚本主要负责修改namelist对模式的各模块进行基本设置（附录namelist中会详细介绍），生成运行模式和提交批处理任务的脚本。
  用户可根据试验需求修改上述内容，完成自定义设置，完成后，运行脚本::
    $ ./run_dcmip_tc_g6_rk3o3_rj.sh
  应当指出，由于GRIST模式发展较为迅速。一些运行脚本可能未能根据实际情况及时更新。如遇到问题，可联系模式支持。
  等待脚本运行完毕后会生成run.sbatch文件，即模式运行和提交批处理任务脚本。以下是run.sbatch文件的内容，它负责设置环境变量和运行GRIST可执行程序，用户需根据自身计算机环境进行修改::

    #!/bin/sh
    #!/usr/bin/bash
    #SBATCH --comment=GCM 
    #SBATCH -J GCM #任务名称
    #SBATCH -n ${nproc}#总节点数
    #SBATCH -p normal #节点名称
    #SBATCH -o gcm_%j.out #输出
    #SBATCH -e gcm_%j.err #错误输出
    
    ##set runtime environment variables
    
    ulimit -s unlimited
    ulimit -c unlimited
    
    module load compiler/intel/composer_xe_2017.2.174 #加载inetl编译器
    module load mpi/intelmpi/2017.2.174 #加载mpi，以上均需根据计算机环境指定
    export I_MPI_PMI_LIBRARY=/opt/gridview/slurm17/lib/libpmi.so #加载MPI库
    export LD_LIBRARY_PATH=/g13/zhangyi/softwares/intel2017/metis-5.1.0/build/Linux-x86_64/libmetis/:${LD_LIBRARY_PATH} #加载
    srun ./par.exe #运行程序

  修改完run.sbatch文件后，使用sbatch命令提交批处理任务::

    $ sbatch run.sbatch
  运行完成后等待模式输出结果。
