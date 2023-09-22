模式输出
=================
  GRIST模式的输出为基于非结构网格分布的文件数据。非结构网格数据将二维空间的格点信息存储至一维数组。模式的输出文件因此根据不同维度，分为一维文件、二维文件和三维文件。一维文件存放降水、2m气温、500hPa高度场等二维单层空间变量。二维文件存放大气风场、温度场等三维空间变量。三维文件目前主要存放包含不同示踪物成分的四维变量，即在垂直层的基础上增加了“类型”这一维度。

  输出文件根据其应用类型，可分为三种：

    1. 月平均诊断输出（h0）。该类型下每月输出一次月平均数据文件。
    2. 根据自定义时间间隔的输出（h1）。该类型下根据用户自定义的输出文件时间间隔进行输出。最短时间间隔为模式运行时间步长。
    3. 重启动文件。该类型如果开启，将产生用于模式重启动计算所需的数据。根据重启动数据的来源，主要分为大气动力文件（Dyn），大气物理文件（Phy）和陆面文件（Lnd）。

自定义诊断变量输出
------------------
  为方便用户对模式进行诊断，GRIST提供了灵活的诊断变量诊断和I/O模块。其中，诊断模块主要负责定义和计算诊断变量，包括瞬时变量（inst）和时间平均变量（accu）。这些变量须为模式中无法直接输出需要诊断计算的变量，其它变量可直接在I/O模块中设置输出。I/O模块控制模式变量输出，包括诊断模块中计算的变量和可直接输出的变量。需指出，用户自定义变量输出建议都定义在诊断模块中或者直接在I/O模块中，不建议在其它模式的计算模块中额外定义。下面介绍自定义输出变量的输出流程。

  首先进入变量输出模块目录::

    $ cd ${GRIST_HOME}/src/atmosphere/gcm/io_UrTemplate

  这里io_UrTemplate为模式示例目录，用户可根据自身习惯更改文件名。该目录中目前只有readme文件，需执行::

    $ cp ../io_zy/* .

  将文件诊断和输出模块拷贝到当前目录。拷贝以后会出现四个F90文件，分别为月平均历史文件（h0）和历史输出文件（h1）的诊断和I/O模块。这里以h1文件中的时间平均纬向水平风场（u_wind）为例介绍自定义变量的输出流程。 
  首先修改grist_gcm_diagnose_h1_module.F90文件声明所需输出变量::

    type gcm_diag_accu_vars_2d
          ···
            type(scalar_2d_field)  :: uwind #添加声明变量（如1d-2d需有不同声明）
          ···
    end type gcm_diag_accu_vars_2d
    ···
    call wrap_allocate_data2d(mesh,nlev ,diag_phys_accu_vars_h1_2d%uwind)
    #为变量分配内存
    ···
    subroutine wrap_deallocate_accu_vars_h1_2d
       ···
    if(allocated(diag_phys_accu_vars_h1_2d%uwind%f))         deallocate(diag_phys_accu_vars_h1_2d%uwind%f) #如果为变量分配过内存，调用时删除内存
       ···
    end subroutine wrap_deallocate_accu_vars_h1_2d
    ···
       subroutine gcm_h1_2d_accu_physics_variables
          ···
          diag_phys_accu_vars_h1_2d%uwind%f          = diag_phys_accu_vars_h1_2d%uwind%f         +dycoreVarCellFull%scalar_U_wind_n%f #计算累计uwind
          ···
    end subroutine gcm_h1_2d_accu_physics_variables
    ···
       subroutine gcm_h1_2d_dump_physics_variables
          ···
               diag_phys_accu_vars_h1_2d%uwind%f          = diag_phys_accu_vars_h1_2d%uwind%f        /diag_phys_accu_vars_h1_2d%ncount #计算时间平均
          ···
       end subroutine gcm_h1_2d_dump_physics_variables
    ···
       subroutine gcm_h1_2d_rest_physics_variables
          ···
    diag_phys_accu_vars_h1_2d%uwind%f     = zero #将变量设为零
          ···
       end subroutine gcm_h1_2d_rest_physics_variables
    ···

  这样自定义变量的声明和计算设置就基本完成了。输出瞬时变量的流程和时间平均变量基本一致，也需声明、赋值和计算。这里计算与上述时间平均不同，包括垂直平均或选取某一垂直层等。详情可参考诊断模块中viqv等变量的输出流程添加自定义瞬时变量。
  自定义变量定义和计算完成后，修改grist_gcm_io_h1_module.F90添加所需输出变量。用户只需在对应类型文件的write_atm_file?d中添加即可，这里瞬时纬向风为例::
    call wrap_add_field_2d(dycoreVarCellFull%scalar_U_wind_n,"uPC","zonal wind speed","m/s")

  在以上wrap_add_field_?d函数中dycoreVarCellFull%scalar_U_wind_n为模式中的变量（模式定义变量或诊断模块中变量），"uPC"为输出文件变量名，"zonal wind speed"为变量描述，"m/s"为单位。以上为用户自定义变量输出的全部流程。
  需要指出，在编译之前需修改${GRIST_HOME}/bld/build_amipw/Filepath文件中对应路径，目前默认路径为：../../src/atmosphere/gcm/io_zy/，用户只需改为修改了诊断和I/O文件的目录即可。

模式数据后处理
------------------
GRIST输出文件格式支持多种绘图软件直接进行可视化，也可通过CDO/NCO等处理软件进行后处理（数据拼接、向经纬度插值等）。GRIST提供了一些后处理脚本，包括变量提取，水平插值和垂直插值等。例如水平插值可利用CDO中提供的remapdis或者remapycon函数完成。

cdo gendis,global_1 ../GRIST.ATM.G6.${case}.MonAvg.2001-05.1d.h0.nc weight_global_1.nc
for ((jr=2001; jr<=2010;jr++))
do
for mn in {01..12}
do
export filehead=GRIST.ATM.G6.${case}.MonAvg.${jr}-${mn}
cp ${filehead}.1d.h0.nc 1d.nc
#ncks -d ntracer,0 ${filehead}.3d.nc a.nc;
#ncwa -a ntracer a.nc 3da.nc;
cp ${filehead}.2d.h0.nc 2da.nc;
ncpdq -a nlev,location_nv 2da.nc 2db.nc;
ncpdq -a nlevp,location_nv 2db.nc 2d.nc;
#ncks 3da.nc 2d.nc<<EOF
#a
#EOF
ncks 2d.nc 1d.nc <<EOF
a
EOF
cdo -f nc copy 1d.nc 1d_new.nc
cdo -P 6 remap,global_1,weight_global_1.nc 1d_new.nc ${filehead}.grid.nc;
rm -rf 1d.nc 2d.nc 2da.nc 2db.nc 3da.nc a.nc 1d_new.nc;
done
done



