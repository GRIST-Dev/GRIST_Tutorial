强迫数据
================
用于全球模式的强迫数据制作
----------------
本章描述制作运行GRIST模式所必须的强迫数据。
强迫数据制作前需配置好依赖环境，以下几个软件是制作静态数据所必需的，括号中为推荐的软件版本：
    - NCO（推荐版本：4.7.6）
    - CDO（推荐版本：2.0.5）
   
构建强迫数据需要以下两个步骤：
    #. 数据前处理
    #. 数据后处理

数据前处理
~~~~~~~~~~~~~~~~
此步骤需先下载ERA5、GFS或者其它资料集中的海温和海冰数据作为原始强迫数据，然后将其转化为nc格式。
下载对应数据后，运行step1_convert_sstsic.sh对数据格式进行转换，以下为step1_convert_sstsic.sh设置参考（以ERA5数据为例）：
::
  pathin='/fs2/home/zhangyi/zhouyh/data/download/mcs/sstsic' #设置原始初值数据读取路径
  pathou='../download/netcdf/20080714/sstsic' #设置初值数据生成路径
  hres="G8UR" #网格名
  cdo_grid_file=/path/to/grist_scrip_gridnum.nc #设置模式网格描述文件路径（模式网格描述文件的生成请参考模式网格生成部分）
  cdo -f nc copy ${pathin}/${file} ${pathou}/${file}.tmp0.nc #将GRIB格式文件转化为nc格式
  cdo chname,var34,sst,var235,tsk,var31,sic ${pathou}/${file}.tmp0.nc  ${pathou}/${file}.nc #修改变量名
  rm -rf ${pathou}/${file}.*tmp.nc ${pathou}/${file}.tmp0.nc #删除中间数据

强迫数据模态设置
~~~~~~~~~~~~~~~~
目前GRIST模式支持三种强迫数据读取模式，用户可根据不同需求制作相应类型的强迫场（仅时间维有差异）分别为：

**1.	AMIP模态：该模态遵循经典大气模式比较计划试验（AMIP）规定的逐月海温强迫模式。每个文件包含一个整数十年及相邻的两个月份数据（例如，196912-198001)。**

*AMIP模态参考namelist设置：（参考文件名：realNoMissERA5SstSic.1980.GRIST.2621442.nc)*
::
  numMonSST              = 122 #AMIP模式必须为122，且每10年一个文件
  sstFile_year_beg       = 1980 #起始日期
  real_sst_style         = 'AMIP' #强迫数据模态
  sstFileNameHead        = 'realNoMissERA5SstSic.' #强迫数据文件前缀
  sstFileNameTail        = '.GRIST.2621442.nc'#强迫数据文件后缀

**2.	CYCLE模态：海表温度文件为年循环的12个月数据。该模态主要用于进行模式自由气候积分，从而检验模式的基本气候态。此外，该模态也可用于开展海温固定条件下的天气预报型试验，仅需将12个月份的海温数据设为同一数值即可。**

*CYCLE模态参考namelist设置：（参考文件名：realNoMissERA5SstSicCYCLE.GRIST.2621442.nc）*
::
 numMonSST              = 12 #AMIP模式必须为12
 sstFile_year_beg       = 2021  #起始日期
 real_sst_style         = 'CYCLE' #强迫数据模态
 sstFileNameHead        = 'realNoMissERA5SstSicCYCLE.' #强迫数据文件前缀
 sstFileNameTail        = '.GRIST.2621442.nc' #强迫数据文件后缀
**3.	DAILY模态：该模态的海温数据为逐日，每个文件涵盖一日数据。运行时，GRIST模式会逐日读取海温数据，并用于强迫大气。此外，该模态也可以广义化为以任意时间间隔读取文件。**

*DAILY模态参考namelist设置：（参考文件名：realNoMissERA5SstSic.daily20080717.G9B3.nc）*
::
 numMonSST              = 1 #AMIP模式必须为1，然后逐文件读取
 sstFile_year_beg       = 2021  #起始日期
 real_sst_style         = 'DAILY' #强迫数据模态
 sstFileNameHead        = 'realNoMissERA5SstSic.' #强迫数据文件前缀
 sstFileNameTail        = '.G9B3.nc' #强迫数据文件后缀

强迫数据后处理
~~~~~~~~~~~~~~~~
制定好强迫模态后，运行step2_possion.sh对初值文件变量进行泊松插值来处理缺测值，以下step2_possion.sh参考设置：
::
  filein=${pathin}/${file} #输入文件名那个
  fileou=${pathou}/realNoMissCDOYconsstsic.6hr${date}.${res}.nc #输出文件名
  echo 'Step 1: possion inte to :  ' ${fileou}
  cat > poisson.ncl << EOF #泊松插值脚本
  begin
  f1=addfile("${filein}","r") #读文件
  sst    = f1->sst #读取变量

  guess     = 1                ; use zonal means
  is_cyclic = True             ; cyclic [global]
  nscan     = 1500             ; usually much less than this
  eps       = 0.001            ; variable dependent
  relc      = 0.6              ; relaxation coefficient
  opt       = 0                ; not used
  poisson_grid_fill( sst, is_cyclic, guess, nscan, eps, relc, opt) #泊松插值

  b1=addfile("${fileou}", "c") #写文件
  b1->sst=sst
  end

  EOF

  ncl poisson.ncl #运行脚本
  rm  poisson.ncl #删除脚本
泊松插值完成后，需运行step3_post_sstsic.sh将初值文件变量插值到模式网格，以下step3_post_sstsic.sh参考设置：
::
  filein=${pathin}/${file} #输入文件名
  fileou=${pathou}/realNoMissCDOYconsstsic.6hr${date}.${res}.nc #中间文件名
  fileouf=${pathou}/realNoMissCDOYconsstsic.daily${date}.${res}.nc #输出文件名
  cdo_grid_file=/fs2/home/zhangyi/public/g9b3_grids/grist_scrip_23592962.nc #模式网格文件
  filemask=/fs2/home/zhangyi/wangym/GRIST_Data-master/static/static.g9b3.mpiscvt.nc #海陆mask

  cdo -f nc4c -P 6 remapycon,${cdo_grid_file} ${filein} ${pathou}/remap.tmp.nc #将初值插值到模式网格
  cdo selname,sic ${pathou}/remap.tmp.nc ${pathou}/remap.sic.tmp.nc #提取海冰文件
  cdo selname,MASK ${filemask} ${pathou}/mask.tmp.nc #提取海路mask
  cdo chname,MASK,sic ${pathou}/mask.tmp.nc ${pathou}/remap.masksic.tmp.nc #将MASK重命名为sic作为sic变量的mask
  cdo ifnotthen ${pathou}/remap.masksic.tmp.nc ${pathou}/remap.sic.tmp.nc ${pathou}/remap.sicnew.tmp.nc #将陆地部分设为缺测
  cdo setmisstoc,0 ${pathou}/remap.sicnew.tmp.nc ${pathou}/remap.sicnew.tmp1.nc #将缺测设为0
  cdo selname,tsk,sst ${pathou}/remap.tmp.nc ${fileou} #提取tsk，sst
  ncks -4 -A ${pathou}/remap.sicnew.tmp1.nc ${fileou} #拼接sst，sic，tsk
  cdo -f nc2 timmean ${fileou} ${fileouf} #生成daily强迫场
  

强迫数据制作脚本参考样例（使用G8分辨率网格）
----------------
**1.step1_convert_sstsic.sh**
::
  pathin='/fs2/home/zhangyi/wangym/data/script/sstsic/case2'
  pathou='../download/netcdf/19980101/sstsic'
  mkdir -p ${pathou}
  rm -rf ${pathou}/*

  hres="G9B3"
  cdo_grid_file=/fs2/home/zhangyi/g9b3_grids/grist_scrip_23592962.nc
 
  for file in `ls ${pathin}` ;do

  if [ "${file##*.}"x = "grib"x ] ;then

  echo ${file}
  echo "1) convert grib to netcdf"
  cdo -f nc copy ${pathin}/${file} ${pathou}/${file}.tmp0.nc

  echo "2) rename sst tsk sic"
  cdo chname,var34,sst,var235,tsk,var31,sic ${pathou}/${file}.tmp0.nc  ${pathou}/${file}.nc

  echo "3) clean"
  rm -rf ${pathou}/${file}.*tmp.nc ${pathou}/${file}.tmp0.nc
  echo "done"
  fi

  done


**2.step2_possion.sh（以DAILY模态为例）**
::
  #!/bin/bash
  lev_type=sf
  year=2020
  pathin='../download/netcdf/19980101/sstsic'
  pathou='/fs2/home/zhangyi/wangym/GRIST_Data-master/init/geniniFromERA5/download/G9B3-case2/sstsic'
  res=G9B3

  mkdir -p ${pathou}
  rm -rf ${pathou}/*.nc

  for file in `ls ${pathin}` ;do

  if [ "${file##*.}"x = "nc"x ] ;then


  echo ${file}
  date=${file:8:8}
  echo ${date}

  filein=${pathin}/${file}
  fileou=${pathou}/realNoMissCDOYconsstsic.6hr${date}.${res}.nc
  rm -rf ${fileou}
  echo 'Step 1: possion inte to :  ' ${fileou}

  cat > poisson.ncl << EOF
  load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/gsn_code.ncl"
  load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/gsn_csm.ncl"
  load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/contributed.ncl"
  load "$NCARG_ROOT/lib/ncarg/nclscripts/csm/shea_util.ncl"

  begin

  f1=addfile("${filein}","r")
  sic    = f1->sic
  sst    = f1->sst
  tsk    = f1->tsk

  guess     = 1                ; use zonal means
  is_cyclic = True             ; cyclic [global]
  nscan     = 1500             ; usually much less than this
  eps       = 0.001            ; variable dependent
  relc      = 0.6              ; relaxation coefficient
  opt       = 0                ; not used
  poisson_grid_fill( sst, is_cyclic, guess, nscan, eps, relc, opt)

  b1=addfile("${fileou}", "c")

  b1->sic=sic
  b1->sst=sst
  b1->tsk=tsk

  end
  EOF
  ncl poisson.ncl
  rm  poisson.ncl

  fi
  done
**3.step3_post_sstsic.sh（以DAILY模态为例）**
::
  pathin=/fs2/home/zhangyi/wangym/GRIST_Data-master/init/geniniFromERA5/download/G9B3-case2/sstsic
  pathou=/fs2/home/zhangyi/wangym/GRIST_Data-master/init/geniniFromERA5/download/G9B3-case2/sstsic/new

  if [ ! -d ${pathou} ];then
     mkdir -p ${pathou}
  fi

  rm -rf ${pathou}/*

  res=G9B3

  echo 'Step 3:   NC   Data from:  '  $pathin
  echo 'Step 3:   NC   Data To  :  '  $pathou

  lev_type=sf

  for file in `ls ${pathin}` ;do

  if [ "${file##*.}"x = "nc"x ] ;then

  echo ${file}
  datetmp=${file#*.}
  date=${datetmp:3:8}
  echo ${date}

  filein=${pathin}/${file}
  fileou=${pathou}/realNoMissCDOYconsstsic.6hr${date}.${res}.nc
  fileouf=${pathou}/realNoMissCDOYconsstsic.daily${date}.${res}.nc
  cdo_grid_file=/fs2/home/zhangyi/public/g9b3_grids/grist_scrip_23592962.nc
  filemask=/fs2/home/zhangyi/wangym/GRIST_Data-master/static/static.g9b3.mpiscvt.nc
  if [ -f ${filein} ]; then
      echo 'Remaps :'${filein}

      rm -rf  ${fileou}
      cdo -f nc4c -P 6 remapycon,${cdo_grid_file} ${filein} ${pathou}/remap.tmp.nc
      cdo selname,sic ${pathou}/remap.tmp.nc ${pathou}/remap.sic.tmp.nc
      cdo selname,MASK ${filemask} ${pathou}/mask.tmp.nc
      cdo chname,MASK,sic ${pathou}/mask.tmp.nc ${pathou}/remap.masksic.tmp.nc
      cdo ifnotthen ${pathou}/remap.masksic.tmp.nc ${pathou}/remap.sic.tmp.nc ${pathou}/remap.sicnew.tmp.nc
      cdo setmisstoc,0 ${pathou}/remap.sicnew.tmp.nc ${pathou}/remap.sicnew.tmp1.nc
      cdo selname,tsk,sst ${pathou}/remap.tmp.nc ${fileou}
      ncks -4 -A ${pathou}/remap.sicnew.tmp1.nc ${fileou}
      cdo -f nc2 timmean ${fileou} ${fileouf}
      rm -rf  ${pathou}/*tmp*

  echo "Done"
  else
      echo 'NO file in '${filein}
  fi
  fi
  done


