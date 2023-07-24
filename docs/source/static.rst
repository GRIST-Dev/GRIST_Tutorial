静态数据
================
用于全球模式的静态数据制作
----------------

静态数据制作前需配置好依赖环境，以下几个软件是制作静态数据所必需的，括号中为推荐的软件版本：
    - NetCDF库（推荐版本：3.6.3）
    - Lapack库（推荐版本：3.8.0）
    - 编译器和MPI.（推荐版本：Intel2018）

制作静态数据需要以下几个步骤：
    #.	构建输入数据集。
    #.	编译GRIST_static。
    #.	运行GRIST_static。

构建输入数据集:  
>>>>>>>>>
GRIST所需的原始静态数据集封装在geog_raw_data中，默认数据如下：
      #. 地形高度：ncar_cube_topo_data/ usgs-rawdata.nc（800m分辨率，由内置的NCAR_Topography软件处理，详见topo.nl）;
      #. 陆面类型：modis_landuse_20class_30s_with_lakes（800m分辨率，21种类型，水体编号17、冰编号15、湖编号21，详见数据index文件）；
      #. 土表类型：soiltype_top_30s（800m分辨率，16种类型）；
      #. 土壤温度：soiltemp_1deg（1°分辨率）；
      #. 雪最大反照率：maxsnowalb（1°分辨率）；
      #. 逐月平均植被覆盖比：greenfrac_fpar_modis（800m分辨率）；
      #. 逐月平均地表反照率：albedo_ncep（0.144°分辨率）
      #. 重力波拖曳相关量：根据模式网格分辨率自动选择orogwd_2deg（2°分辨率）、orogwd_1deg（1°分辨率）、orogwd_30m（800m分辨率）。

编译GRIST_static
>>>>>>>>>
该步骤和编译GRIST主程序相似，进入编译目录bld修改Makefile中 NETCDF和LAPACK路径，修改 EXEDIR 指定执行文件（grist_static.exe）路径。

开始编译
:::::::::
::

     # 进入编译目录
     $ cd /path/to/bld #bld文件路径

     # 修改Makefile中NETCDF和LAPACK路径
     # 修改EXEDIR指定执行文件（grist_static.exe）路径
     # 编译
     $ ./make.sh

     # 如果编译成功，执行目录中会出现可执行文件: grist_static.exe。

运行GRIST_static
>>>>>>>>>
根据用户需求设置grist.nml、grist_init.nml、topo.nl，执行::

     $ ./sbatch.sh

串行运行执行文件，从原始数据制作静态数据大概需要25分钟左右，运行完成会生成static.nc，即为主程序所需的同分辨率静态数据。

**1. grist.nml设置参考:**
::
     gridFilePath='/path/to/gridFile' #模式网格数据路径；
     gridFileHeadName='FileHeadName' #模式网格数据名称，静态数据将与该网格相匹配；
     gridRegionFileHeadName='RegionFileHeadName' #有限区域模式网格数据名称，详见有限区域模式的静态数据制作；
     mesh_nv=${mesh} #模式网格数

**2. grist_init.nml设置参考:**
::
       geog_data_path='/path/to/geog_raw_data' #原始静态数据集路径；
       static_path='path/to/static.nc' #指定static.nc路径；
       config_do_staic=.true. #是否从原始数据制作；
       do_regional_domain=.true. #是否生成有限区域模式的静态数据；
       read_static=.false. #是否读取当前路径下已有的全球static.nc，与config_do_staic相反，主要用于有限区域模式。

**3. topo.nl为内置地形处理软件NCAR_topography的namelist，设置参考:**
::
       raw_data_filepath='/path/to/raw_data' #原始地形数据；
       do_cube_smooth=.true. #是否平滑地形；
       smooth_times=num #平滑次数；
       smooth_method='linear' #平滑方法（可选'linear'，'shapiro'，'fv3'，'avg'）。

用于有限区域模式的静态数据制作
----------------
有限区域模式静态数据制作流程与全球模式类似，需准备有限区域网格数据（包括有限区域网格、同路径下对应的全球网格，全球-区域index对应关系文件，详见网格数据制作），在grist.nml和grist_init.nml里设置:
::
    gridRegionFileHeadName='RegionFileHeadName' #有限区域模式网格数据名称；
    do_regional_domain=.true. #设为true开启有限区域模式静态数据制作;
    read_static=.true. #如果已存在对应全球网格的静态数据，可以直接使用，会极大减少运行时间;

namelist参考样例（使用G8分辨率网格）
----------------
**1. grist.nml**
::
    &ctl_para
    outdir                 = './'
    gridFilePath           = '/THL8/home/zhangyi/public/GRIST/data/uniform-g8/grid/'    
    gridFileHeadName       = 'grist.grid_file.g8.ccvt'
    /
    &swe_para
    /
    &dycore_para
    /
    &tracer_para
    /
    &mesh_para
    mesh_nv                = 655362
    /
    &ccvt_para
    /

**2. grist_init.nml**
::
    &share
    start_date = '2012-05-26_00:00:00'
    end_date   = '2012-05-26_00:00:00'
    interval_seconds = 21600
    io_form_geogrid = 2,
    /
    &mesh_plot
    config_mesh_plot = .false.
    /
    &ungrib
    out_format = 'WPS',
    prefix = 'ForGrist',
    /
    &static_para
    geog_data_path = '/THL8/home/zhangyi/grist_static/geog_raw_data/',
    static_path = './',
    config_do_staic = .true.
    do_regional_domain     = .false.
    read_static     = .false.
    config_do_init_condition = .false.
    /
    &gfs_para
    grist_data_date  = '2012-05-26_00'
    config_nfglevels = 27
    nSoilLevels = 4
    /
    &physics_para
    ozone_data_path  = '/g13/zhangyi/mac/run/grist_landData/grist_init/geog_data/ozone_1.9x2.5_L26_2000clim_c091112.nc'
    config_do_ozone  = .false.
    /

**3. topo.nl**
::
    &topoparams
    raw_data_filepath = '/THL8/home/zhangyi/grist_static/geog_raw_data/ncar_cube_topo_data/'
    externally_smoothed_topo_file   = 'inputdata/externally-smoothed-PHIS/USGS-gtopo30_ne30np4_16xdel2.nc'
    lsmooth_terr = .false.
    lexternal_smooth_terr = .true.
    lzero_out_ocean_point_phis = .false.
    res_cube  = 18
    do_sgh    = .false.
    do_cube_smooth= .false.
    smooth_times  = 1
    smooth_method = 'linear'
    /


