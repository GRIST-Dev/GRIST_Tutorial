初值数据
================
用于全球模式的初值数据制作
----------------
  初值数据制作前需配置好依赖环境，推荐使用以下几个文件处理软件制作初值数据，括号中为推荐的软件版本：
    - NCO（推荐版本：4.7.6）
    - CDO（推荐版本：2.0.5）

  构建初值数据需要以下几个步骤：
   #.	初值数据前处理
   #.	重命名初值变量
   #.	初值变量后处理


初值数据前处理
~~~~~~~~~~~~~~~~
此步骤需先下载ERA5或GFS再分析数据作为GRIST初值的原始数据，然后将其转化为nc格式，其中初值数据包括：
  1.	气压层数据：T（温度）、U（纬向风）、V（经向风）、比湿（Q）。
  2.	单层数据：PS（气压）、SOILH（重力位势）、SoilMoist（土壤湿度、一般有4层）、SoilTemp（土壤温度、一般有4层）、SKINTEMP（表面温度）、XICE（海冰面积）、SNOW（雪密度）、SNOWH（雪深）。
下载对应数据后，运行step1_convert.sh对数据格式进行转换，以下为step1_convert.sh设置参考（以ERA5数据为例）：
::
  pathin='/path/to/initdata' #设置原始初值数据读取路径
  pathou='/path/to/output' #设置初值数据生成路径
  res="G8UR" #网格名
  cdo_grid_file=/path/to/grist_scrip_gridnum.nc #设置模式网格描述文件路径（模式网格描述文件的生成请参考模式网格生成部分）
  cdo -f nc copy ${pathin}/${file} ${pathou}/${file}.tmp0.nc #将GRIB格式文件转化为nc格式
  cdo setmisstoc,0 ${pathou}/${file}.tmp0.nc ${pathou}/${file}.tmp.nc #将缺省值设为0
  cdo -P 6 remapycon,${cdo_grid_file} ${pathou}/${file}.tmp.nc ${pathou}/${file}.${res}.nc #将初始场数据插值到模式网格
  
重命名初值变量
~~~~~~~~~~~~~~~~
此步骤通过运行step2_rename.sh将插值好的初值文件变量名改为模式适用的初值，以下为step2_rename.sh参考设置:
::
  res=G8UR #网格分辨率
  pathou='/path/to/output' #输出文件路径
  lev_type=pl #数据垂直层类型
  cdo chname,var130,T,var131,U,var132,V,var133,Q ${pathin}/ERA5.${lev_type}.${year}${mon}${day}.00.grib.${res}.nc ${pathou}/initial_${res}_${lev_type}_${year}${mon}${day}.nc #将对应变量重命名为模式适用变量名
  cdo chname,var134,PS,var129,SOILH,var235,SKINTEMP,var31,XICE,var39,SoilMoist_lv1,var40,SoilMoist_lv2,var41,SoilMoist_lv3,var42,SoilMoist_lv4,\
  var139,SoilTemp_lv1,var170,SoilTemp_lv2,var183,SoilTemp_lv3,var236,SoilTemp_lv4,var33,SNOW,var141,SNOWH,\ 
  ${pathin}/ERA5.sf.${year}${mon}${day}.00.grib.${res}.nc    ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc #同上，但为单层变量设置
  ncks -v SNOW          ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/snow.nc #选取降雪数据并保存为snow.nc
  ncks -v SNOWH         ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/snowh.nc #选取雪深数据并保存为snowh.nc
  ncks -v SOILH         ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/soilh.nc #选取重力位势数据并保存为soilh.nc
  ncks -x -v SNOW,SOILH ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/newbase.nc #选取降雪，SOILH数据并保存为newbase.nc
  cdo mul ${pathou}/snow.nc      ${pathou}/snowh.nc ${pathou}/snow1.nc #计算得到snow初值
  cdo expr,'SOILH=SOILH/9.80616' ${pathou}/soilh.nc ${pathou}/soilh1.nc #计算得到soilh初值
  ncks -A ${pathou}/snow1.nc ${pathou}/newbase.nc #拼接snow1.nc和newbase.nc文件
  ncks -A ${pathou}/soilh1.nc ${pathou}/newbase.nc #拼接soilh1.nc和newbase.nc文件
  mv ${pathou}/newbase.nc ${pathou}/initial_${res}_sf_${year}${mon}${day}.nc #将newbase.nc重命名为模式初始场读取格式。

初值变量后处理
~~~~~~~~~~~~~~~~
此步骤通过运行step3_post.sh脚本进一步对初值文件进行整理，使其符合GRIST模式的读取需求，以下是step3_post.sh脚本的设置参考：
::
  lvname=plev #垂直坐标变量名
  ncks -d time,0 ${pathin}/initial_${res}_${lev_type}_${year}${mon}${day}.nc  initial_${res}.dim1.nc #选取第一个时间维度的变量作为初始场（如果有多个时间维度）
  ncwa -a time initial_${res}.dim1.nc tmp.nc #去除时间维度
  ncpdq -a ncells,${lvname} tmp.nc ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc #调换ncells和垂直坐标位置。
  cdo selname,SoilTemp_lv1 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.st1.nc #提取SoilTemp_lv1变量
  ncrename -v SoilTemp_lv1,SoilTemp tmp.st1.nc #将SoilTemp_lv1 重命名为SoilTemp
  … …
  cdo selname,SoilTemp_lv4 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.st4.nc #同上但为第四层
  ncrename -v SoilTemp_lv4,SoilTemp tmp.st4.nc #同上但为第四层
  cdo merge tmp.st?.nc tmp.st.nc #将四层土壤温度合并到depth维度
  ncpdq -a ncells,depth tmp.st.nc tmp.st.ncpdq.nc #将ncells和depth维度位置调换
  cdo selname,XICE,SOILH,SNOWH,SNOW,SKINTEMP,PS grist.initial.${res}_sf_${year}${mon}${day}.nc grist.init.${res}_sf_${year}${mon}${day}.nc #选取单层变量并存为GRIST模式读取格式文件
  ncks -A tmp.sn.ncpdq.nc grist.init.${res}_sf_${year}${mon}${day}.nc #拼接计算得到的土壤湿度变量
  ncks -A grist.init.${res}_sf_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc #将单层变量和气压层变量拼接为一个初始场文件。

大初值文件制作
^^^^^^^^^^^^^^^^^^^^^
需指出，GRIST模式在读取比G9网格更细的初值文件时，由于netcdf对文件容量的限制，需单独制作气压层的各变量，并逐一读取。以下为大初值文件的制作参考：
::
  cdo selname,U ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.U.${lev_type}.${res}_${year}${mon}${day}.nc #提取U变量并单独存放
  … …
  cdo selname,Q ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.Q.${lev_type}.${res}_${year}${mon}${day}.nc #提取Q变量并单独存放

有限区域模式的初值制作
----------------
有限区域模式的初值由GRIST全球模式提供，运行remap_lam.sh脚本对全球模式处理生成有限区域模式初值。以下为remap_lam.sh的参考设置：
::
  ncks -v lon_nv,lat_nv,ps,hps ${inpth}/1d/${fhead}.1d.h1.nc tmp.nc #提取经纬度和表层气压变量
  ncks -v uPC,vPC,temperature  ${inpth}/2d/${fhead}.2d.h1.nc tmp2.nc #提取U，V和温度等2维变量
  ncks -d ntracer,0  ${inpth}/3d/${fhead}.3d.h1.nc tmp3a.nc #提取Q变量
  ncpdq -a ntracer,location_nv,nlev tmp3a.nc tmp3.nc #将Q变量的维度调整为（ntracer,location_nv,nlev）
  ncrename -d ntracer,time tmp3.nc tmp3b.nc #将ntracer变量重命名为time（便于后面操作）
  ncks -A tmp2.nc tmp.nc #拼接1d和2d变量
  ncks -A tmp3b.nc tmp.nc #拼接3d变量
  cdo remapdis,r1440x720 tmp.nc GRIST.lamData.test.nc #水平插值到经纬度网格
  ncks --fix_rec_dmn time GRIST.lamData.test.nc GRIST.lamData.test1.nc #将time为设为unlimited
  cdo remapdis,/THL8/home/zhangyi/zhangyi/grid_generator/run/uniform-g9/lam_grid/grist_scrip_556704.nc GRIST.lamData.test1.nc GRIST.lamData.test2.nc #水平插值到有限区域网格
  ncpdq -a ncells,nlev,time GRIST.lamData.test2.nc GRIST.lamData.test3.nc #将3d变量的维度调整为（ncells,nlev,time）
  ncrename -d time,ntracer GRIST.lamData.test3.nc GRIST.lamData.test4.nc #将time维度重新设置为ntracer
  ncrename -v time,ntracer GRIST.lamData.test4.nc GRIST.lamData.test5.nc #将time变量重命名为ntracer
操作完成之后，运行rename_lamdata.sh对有限区域模式变量进行重命名，详情请参考下方的示例脚本

初值制作脚本参考样例（使用G8分辨率网格）
----------------
**1.step1_convert.sh**
::
  pathin='/fs2/home/zhangyi/zhouyh/data/download/mcs/init'
  pathou='../download/netcdf/20080714/'
  mkdir -p ${pathou}

  hres="G8UR"
  cdo_grid_file=/fs2/home/zhangyi/wangym/GRIST_Data-master/g8-uniform/grid/grist_scrip_655362.nc

  for file in `ls ${pathin}` ;do
  if [ "${file##*.}"x = "grib"x ] ;then
  echo ${file}
  echo "1) convert grib to netcdf"
  cdo -f nc copy ${pathin}/${file} ${pathou}/${file}.tmp0.nc
  # only sea ice fraction has missing, just set to 0
  cdo setmisstoc,0                 ${pathou}/${file}.tmp0.nc ${pathou}/${file}.tmp.nc
  echo "2) convert lat-lon to unstructured"
  cdo -P 6 remapycon,${cdo_grid_file} ${pathou}/${file}.tmp.nc ${pathou}/${file}.${hres}.nc
  echo "3) clean"
  rm -rf ${pathou}/${file}.tmp.nc ${pathou}/${file}.tmp0.nc
  echo "done"
  fi
  done

**2.step2_rename.sh**
::
  res=G8UR
  pathou='/fs2/home/zhangyi/wangym/GRIST_Data-master/init/geniniFromERA5/download/raw'
  lev_type=pl
  mkdir -p ${pathou}
  for year in 2008 ;do
  for mon in 07 ;do
  for day in 14 ;do
  pathin=../download/netcdf/${year}${mon}${day}/
  echo ${year} ${mon} ${day} 
  if true ;then
    cdo chname,var130,T,var131,U,var132,V,var133,Q ${pathin}/ERA5.${lev_type}.${year}${mon}${day}.00.grib.${res}.nc ${pathou}/initial_${res}_${lev_type}_${year}${mon}${day}.nc
    cdo chname,var134,PS,var129,SOILH,var235,SKINTEMP,var31,XICE,\
        var39,SoilMoist_lv1,var40,SoilMoist_lv2,var41,SoilMoist_lv3,var42,SoilMoist_lv4,\
        var139,SoilTemp_lv1,var170,SoilTemp_lv2,var183,SoilTemp_lv3,var236,SoilTemp_lv4,\
        var33,SNOW,var141,SNOWH \
        ${pathin}/ERA5.sf.${year}${mon}${day}.00.grib.${res}.nc    ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc
  fi
  if  true ; then 
    ncks -v SNOW          ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/snow.nc 
    ncks -v SNOWH         ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/snowh.nc
    ncks -v SOILH         ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/soilh.nc
    ncks -x -v SNOW,SOILH ${pathou}/initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/newbase.nc

    cdo mul ${pathou}/snow.nc      ${pathou}/snowh.nc ${pathou}/snow1.nc
    cdo expr,'SOILH=SOILH/9.80616' ${pathou}/soilh.nc ${pathou}/soilh1.nc
    mv ${pathou}/snow1.nc  ${pathou}/snow.nc
    mv ${pathou}/soilh1.nc ${pathou}/soilh.nc

    ncks -A ${pathou}/snow.nc ${pathou}/newbase.nc
    ncks -A ${pathou}/soilh.nc ${pathou}/newbase.nc
    mv ${pathou}/newbase.nc ${pathou}/initial_${res}_sf_${year}${mon}${day}.nc
    rm -rf initial_${res}_sf_${year}${mon}${day}.tmp.nc ${pathou}/snow.nc ${pathou}/snowh.nc ${pathou}/soilh.nc
  fi
  done
  done
  done
**3.step3_post.sh**
::
  res=G8UR
  pathin='/fs2/home/zhangyi/wangym/GRIST_Data-master/init/geniniFromERA5/download/raw'
  pathou='/fs2/home/zhangyi/wangym/GRIST_Data-master/init/geniniFromERA5/download/G8UR'
  lev_type=pl
  lvname=plev
  mkdir -p ${pathou}
  for year in 2008 ;do
  for mon in 07 ;do
  for day in 14 ;do
  echo ${year}${mon}${day}
  #2d
  ncks -d time,0 ${pathin}/initial_${res}_${lev_type}_${year}${mon}${day}.nc  initial_${res}.dim1.nc
  ncwa -a time initial_${res}.dim1.nc tmp.nc
  ncpdq -a ncells,${lvname} tmp.nc ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc
  cdo selname,U ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.U.${lev_type}.${res}_${year}${mon}${day}.nc
  cdo selname,V ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.V.${lev_type}.${res}_${year}${mon}${day}.nc
  cdo selname,T ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.T.${lev_type}.${res}_${year}${mon}${day}.nc
  cdo selname,Q ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.Q.${lev_type}.${res}_${year}${mon}${day}.nc
  rm -rf initial_${res}.dim1.nc tmp.nc
  #1d
  ncks -d time,0 ${pathin}/initial_${res}_sf_${year}${mon}${day}.nc  initial_${res}.dim1.nc
  ncwa -a time initial_${res}.dim1.nc grist.initial.${res}_sf_${year}${mon}${day}.nc
  cdo selname,PS grist.initial.${res}_sf_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.PS.${lev_type}.${res}_${year}${mon}${day}.nc
  cdo selname,SoilTemp_lv1 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.st1.nc
  ncrename -v SoilTemp_lv1,SoilTemp tmp.st1.nc
  cdo selname,SoilTemp_lv2 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.st2.nc
  ncrename -v SoilTemp_lv2,SoilTemp tmp.st2.nc
  cdo selname,SoilTemp_lv3 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.st3.nc  
  ncrename -v SoilTemp_lv3,SoilTemp tmp.st3.nc
  cdo selname,SoilTemp_lv4 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.st4.nc
  ncrename -v SoilTemp_lv4,SoilTemp tmp.st4.nc
  cdo merge tmp.st?.nc tmp.st.nc
  ncpdq -a ncells,depth tmp.st.nc tmp.st.ncpdq.nc
  cdo selname,SoilMoist_lv1 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.sn1.nc
  ncrename -v SoilMoist_lv1,SoilMoist tmp.sn1.nc
  cdo selname,SoilMoist_lv2 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.sn2.nc
  ncrename -v SoilMoist_lv2,SoilMoist tmp.sn2.nc
  cdo selname,SoilMoist_lv3 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.sn3.nc
  ncrename -v SoilMoist_lv3,SoilMoist tmp.sn3.nc
  cdo selname,SoilMoist_lv4 grist.initial.${res}_sf_${year}${mon}${day}.nc tmp.sn4.nc
  ncrename -v SoilMoist_lv4,SoilMoist tmp.sn4.nc
  cdo merge tmp.sn?.nc tmp.sn.nc
  ncpdq -a ncells,depth tmp.sn.nc tmp.sn.ncpdq.nc
  cdo selname,XICE,SOILH,SNOWH,SNOW,SKINTEMP,PS grist.initial.${res}_sf_${year}${mon}${day}.nc grist.init.${res}_sf_${year}${mon}${day}.nc
  ncks -A tmp.sn.ncpdq.nc grist.init.${res}_sf_${year}${mon}${day}.nc
  ncks -A tmp.st.ncpdq.nc grist.init.${res}_sf_${year}${mon}${day}.nc
  cp grist.init.${res}_sf_${year}${mon}${day}.nc ${pathou}/grist.init.${res}_sf_${year}${mon}${day}.nc
  #append  
  ncks -A grist.init.${res}_sf_${year}${mon}${day}.nc ${pathou}/grist.era5.ini.${lev_type}.${res}_${year}${mon}${day}.nc
  rm -rf initial_${res}.dim1.nc grist.initial.${res}_sf_${year}${mon}${day}.nc
  rm -rf tmp*.nc
  done
  done
  done
**4.remap_lam.sh**
::
  inpth=/THL8/home/zhangyi/public/GRIST/run/GRIST_NWP_2021_JJA/hdc/L30/HDC-Beg20210610-hadv33-hnrk3-vadv3-vnrk3/history/atm
  outpth=./GRIST_lamData
  mkdir -p ${outpth}
  cd ${outpth}
  for f in ${inpth}/1d/*ATM*00.1d.h1.nc ;do
  fhead=${f:119:37}
  echo ${fhead}
  fyear=${fhead:21:4}
  fmon=${fhead:26:2}
  echo ${fyear}
  echo ${fmon}
  fday=${fhead:29:2}
  fsec=${fhead:32:5}
  echo ${fday}
  echo ${fsec}
  ncks -v lon_nv,lat_nv,ps,hps ${inpth}/1d/${fhead}.1d.h1.nc tmp.nc
  cp  ${inpth}/2d/${fhead}.2d.h1.nc tmp1.nc
  ncks -v uPC,vPC,temperature tmp1.nc tmp2.nc
  ncks -d ntracer,0  ${inpth}/3d/${fhead}.3d.h1.nc tmp3a.nc
  ncpdq -a ntracer,location_nv,nlev tmp3a.nc tmp3.nc
  ncrename -d ntracer,time tmp3.nc tmp3b.nc
  ncks -A tmp2.nc tmp.nc
  ncks -A tmp3b.nc tmp.nc
  cdo remapdis,r1440x720 tmp.nc GRIST.lamData.test.nc
  ncks --fix_rec_dmn time GRIST.lamData.test.nc GRIST.lamData.test1.nc
  cdo remapdis,/THL8/home/zhangyi/zhangyi/grid_generator/run/uniform-g9/lam_grid/grist_scrip_556704.nc GRIST.lamData.test1.nc GRIST.lamData.test2.nc
  ncpdq -a ncells,nlev,time GRIST.lamData.test2.nc GRIST.lamData.test3.nc
  ncrename -d time,ntracer GRIST.lamData.test3.nc GRIST.lamData.test4.nc
  ncrename -v time,ntracer GRIST.lamData.test4.nc GRIST.lamData.test5.nc
  ncks --fix_rec_dmn ncells GRIST.lamData.test5.nc GRIST.lamData.${fyear}${fmon}${fday}${fsec}.nc
  ncatted -O -a calendar,ntracer,d,, GRIST.lamData.${fyear}${fmon}${fday}${fsec}.nc
  ncatted -O -a axis,ntracer,d,, GRIST.lamData.${fyear}${fmon}${fday}${fsec}.nc
  ncatted -O -a standard_name,ntracer,o,c,'tracer type' GRIST.lamData.${fyear}${fmon}${fday}${fsec}.nc
  ncatted -O -a units,ntracer,o,c,' ' GRIST.lamData.${fyear}${fmon}${fday}${fsec}.nc
  rm -rf tmp*.nc
  rm -rf GRIST.lamData.test*.nc
  done
**5.rename_lamdata.sh**
::
  for f in ./GRIST_lamData/GRIST*.nc
  do
    echo ${f}
    ncrename -v hps,HPS ${f}
    ncrename -v uPC,U ${f}
    ncrename -v vPC,V ${f}
    ncrename -v temperature,T ${f}
    ncrename -v ps,PS ${f}
    ncrename -v tracerMxrt,Q ${f}
  done
 
