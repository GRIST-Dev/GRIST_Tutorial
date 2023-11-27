常见问题
=================

模式初始场垂直层数设置
----------------
    为便于兼容不同垂直层的命名方式（plev,mlev,lev等），GRIST使用namelist控制初始场文件垂直层数的读取。GRIST模式namelist中&mesh_para：nlev_inidata（初始场垂直层数量）的设置需与模式实际使用的垂直层严格一致。如果不一致会产生初始场的垂直层向模式层插值出现问题导致模式出现较大误差。

初值垂直坐标设置
----------------
    根据不同的初值来源，GRIST模式支持多种垂直坐标类型的读取（包括：气压坐标、混合坐标等）。读取不同垂直坐标类型的初值需利用namelist设置对应的读取方式。当前模式支持的垂直坐标及namelist设置为：
        1. 气压坐标系：namelist：initialDataSorc="ERAIP/GFS", gcm_testcase="real-ERAIP/real-GFS"
        2. 混合坐标系：namelist：initialDataSorc="ERAIM", gcm_testcase="real-ERAIM"
        3. WRF模式坐标系：namelist:initialDataSorc="WRFDA", gcm_testcase="real-WRFDA"
 
模式时步设置
----------------
    GRIST模式时步包括：  
    model_timestep   (最外层)
    tracer_timestep （中间层）
    dycore_timestep （最内层）
    
    dycore_timestep<=tracer_timestep; tracer_timestep<=model_timestep；
    且两个相邻step必须为整数倍。比如，model_timestep=1200, tracer_timestep=600, dycore_timesetp=300。