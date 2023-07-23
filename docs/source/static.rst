静态数据
================
用于全球模式的静态数据制作
----------------
  依赖软件：
    - NetCDF库
    - Lapack库
    - 编译器

  原始数据集：  
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
  该步骤和编译GRIST主程序相似，进入编译目录bld修改Makefile中 NETCDF和LAPACK路径，修改 EXEDIR 指定执行文件（grist_static.exe）路径。
  开始编译：

    $ ./make.sh

  如果编译成功，执行目录中会出现可执行文件: grist_static.exe。



运行GRIST_static
  根据用户需求设置grist.nml、grist_init.nml、topo.nl，执行

    $ ./sbatch.sh

  串行运行执行文件，从原始数据制作静态数据大概需要25分钟左右，运行完成会生成static.nc，即为主程序所需的同分辨率静态数据。

    #. grist.nml设置参考：
       gridFilePath：模式网格数据路径；
       gridFileHeadName：模式网格数据名称，静态数据将与该网格相匹配；
       gridRegionFileHeadName：有限区域模式网格数据名称，详见有限区域模式的静态数据制作；
       mesh_nv：模式网格数
    #. grist_init.nml设置参考:
       geog_data_path：原始静态数据集路径；
       static_path：指定static.nc路径；
       config_do_staic：是否从原始数据制作；
       do_regional_domain：是否生成有限区域模式的静态数据；
       read_static：是否读取当前路径下已有的全球static.nc，与config_do_staic相反，主要用于有限区域模式。
    #. topo.nl为内置地形处理软件NCAR_topography的namelist，设置参考：
       raw_data_filepath：原始地形数据；
       do_cube_smooth：是否平滑地形；
       smooth_times：平滑次数；
       smooth_method：平滑方法（可选'linear'，'shapiro'，'fv3'，'avg'）。

用于有限区域模式的静态数据制作
----------------
  有限区域模式静态数据制作流程与全球模式类似，需准备有限区域网格数据（包括有限区域网格、同路径下对应的全球网格，全球-区域index对应关系文件，详见网格数据制作），在grist.nml和grist_init.nml里设置：
  gridRegionFileHeadName：有限区域模式网格数据名称；
  do_regional_domain：.true.
  read_static：如果已存在对应全球网格的静态数据，可以直接使用，会极大减少运行时间


