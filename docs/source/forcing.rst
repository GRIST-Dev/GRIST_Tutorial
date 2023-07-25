强迫数据
================
用于全球模式的强迫数据制作
----------------
本章描述制作运行GRIST模式所必须的强迫数据。
强迫数据制作前需配置好依赖环境，以下几个软件是制作静态数据所必需的，括号中为推荐的软件版本：
    - NCO（推荐版本：4.7.6）
    - CDO（推荐版本：2.0.5）
   
构建强迫数据需要以下两个步骤：
    #. 强迫数据前处理
    #. 重命名强迫变量

强迫数据前处理
~~~~~~~~~~~~~~~~
此步骤需先下载ERA5、GFS或者其它资料集中的海温数据作为原始强迫数据，然后将其转化为nc格式。
下载对应数据后，运行step1_convert_sstsic.sh对数据格式进行转换，以下为step1_convert_sstsic.sh设置参考（以ERA5数据为例）：
::
  pathin='/fs2/home/zhangyi/zhouyh/data/download/mcs/sstsic' #设置原始初值数据读取路径
  pathou='../download/netcdf/20080714/sstsic' #设置初值数据生成路径
  hres="G8UR" #网格名
  cdo_grid_file=/path/to/grist_scrip_gridnum.nc #设置模式网格描述文件路径（模式网格描述文件的生成请参考模式网格生成部分）
  cdo -f nc copy ${pathin}/${file} ${pathou}/${file}.tmp0.nc #将GRIB格式文件转化为nc格式
  cdo setmisstoc,0 ${pathou}/${file}.tmp0.nc ${pathou}/${file}.tmp.nc #将缺省值设为0
  cdo -P 6 remapycon,${cdo_grid_file} ${pathou}/${file}.tmp.nc ${pathou}/${file}.${hres}.nc #将初始场数据插值到模式网格

重命名强迫变量
~~~~~~~~~~~~~~~~
运行step2_rename_sstsic.sh将插值好的初值文件变量名改为模式适用变量，以下为step2_rename_sstsic.sh参考设置:
::
  cdo chname,var34,sst,var31,sic ${pathin}/${file} ${pathou}/realNoMissCDOYconsstsic.6hr${fdate}.${res}.nc #将var34改名为sst，将var31改名为sic
  cdo timmean ${pathou}realNoMissCDOYconsstsic.6hr${fdate}.${res}.nc ${pathou}/realNoMissCDOYconsstsic.daily${fdate}.${res}.nc #将6小时数据处理为daily数据

强迫数据模式设置
----------------
目前GRIST模式支持三种强迫数据读取模式分别为：

**1.	AMIP模态：该模态遵循经典大气模式比较计划试验（AMIP）规定的逐月海温强迫模式。每个文件包含一个整数十年及相邻的两个月份数据（例如，196912-198001)。**

*AMIP模态参考namelist设置：（参考文件名：realNoMissERA5SstSic.1980.GRIST.2621442.nc)*
::
  numMonSST              = 122 #AMIP模式必须为122，且每10年一个文件
  sstFile_year_beg       = 1980 #起始日期
  real_sst_style         = 'AMIP' #强迫数据模态
  sstFileNameHead        = 'realNoMissERA5SstSic.' #强迫数据文件前缀
  sstFileNameTail        = '.GRIST.2621442.nc'#强迫数据文件后缀
**2.	CYCLE模态：海表温度文件为年循环的12个月数据。该模态主要用于进行模式自由气候积分，从而检验模式的基本气候态。此外，该模态也可用于开展海温固定条件下的天气预报型试验，仅需将12个月份的海温数据设为同一数值即可。**

*CYCLE模态参考namelist设置：（参考文件名：realNoMissERA5SstSic20210610.GRIST.2621442.nc）*
::
 numMonSST              = 12 #AMIP模式必须为12
 sstFile_year_beg       = 2021  #起始日期
 real_sst_style         = 'CYCLE' #强迫数据模态
 sstFileNameHead        = 'realNoMissERA5SstSic20210610.' #强迫数据文件前缀
 sstFileNameTail        = '.GRIST.2621442.nc' #强迫数据文件后缀
**3.	DAILY模态：该模态的海温数据为逐日，每个文件涵盖一日数据。运行时，GRIST模式会逐日读取海温数据，并用于强迫大气。此外，该模态也可以广义化为以任意时间间隔读取文件。**

*DAILY模态参考namelist设置：（参考文件名：realNoMissERA5SstSic.daily20080717.G9B3.nc）*
::
 numMonSST              = 1 #AMIP模式必须为1，然后逐文件读取
 sstFile_year_beg       = 2021  #起始日期
 real_sst_style         = 'DAILY' #强迫数据模态
 sstFileNameHead        = 'realNoMissERA5SstSic.' #强迫数据文件前缀
 sstFileNameTail        = '.G9B3.nc' #强迫数据文件后缀

强迫数据制作脚本参考样例（使用G8分辨率网格）
----------------
**1.step1_convert_sstsic.sh**
::
  pathin='/fs2/home/zhangyi/zhouyh/data/download/mcs/sstsic'
  pathou='../download/netcdf/20080714/sstsic'
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
