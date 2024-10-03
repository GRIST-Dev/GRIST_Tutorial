重启动计算（restart）
==================
使用方式
------------------
     类似history输出，GRIST模式可以按月（h0）或按用户定义的时间间隔（h1)输出restart文件，用于模式续算。一组典型的重启动文件如下：  
     
     GRIST.RST.Dyn.G6.1d.amipw.1981-10-01.11596-01200.nc（动力框架）
     GRIST.RST.Dyn.G6.2d.amipw.1981-10-01.11596-01200.nc（动力框架）
     GRIST.RST.Dyn.G6.3d.amipw.1981-10-01.11596-01200.nc（动力框架）
     GRIST.RST.Phy.G6.1d.amipw.1981-10-01.11596-01200.nc（物理过程）
     GRIST.RST.Phy.G6.2d.amipw.1981-10-01.11596-01200.nc（物理过程）
     GRIST.RST.Lnd.G6.1d.amipw.1981-10-01.11596-01200.nc（陆面模式）
     GRIST.RST.Lnd.G6.2d.amipw.1981-10-01.11596-01200.nc（陆面模式）

     这是一组可用于从1981年10月01日的1200s开始进行restat计算的restart文件。‘11596’表示模式从‘init’run开始运行了‘11596’days，‘01200’表示‘model_timestep in second’（时间步长）。
     在‘init’run的基础上，重启动计算需要修改如下设置：   

     1. step_restart.txt，这里需要设置为restart时刻在一次‘init’运行中的step index, 等于 (86400s/day)/(1200s/step)*11596 day + 1= 834913 step。(按需计算相应时次）
     2. grist.nml里，ctl_para下 run_type = 'restart'。如果运行时的real_sst_style=‘AMIP’，则还需确保 sstFile_year_beg =  'first year of the restart-year decade' (上述示例设为1980)

注意事项
------------------
  1. Restart计算推荐策略   

     即使在restart计算本身正确的情况下，某些诊断输出变量（如时间平均变量）可能会依赖于step_restart前一个时间步的数据（这些数据在restart时未被保留）。因此，restart时无法利用这些数据来生成诊断输出，导致init运行和restart运行之间出现不一致。为了确保restart过程中的诊断输出正确性，并预防其他相关问题（详见下述第2点），建议采取以下策略进行restart操作：   

     将'step_restart'设置为目标续算文件（如h0或h1）的提前2个输出频次间隔所对应的步数。例如，在上述示例中，如果模型在1981年10月12日中断，并且init运行已经生成了直到1981年09月的h0月平均文件，则step_restart设置为从1981年8月1日开始的时间步数。这样，1981年8月的h0文件将被废弃，而1981年9月的h0文件可用于检查restart与init运行的一致性，1980年10月的为用户所需的输出文件。   

  2. Restart计算的正确性  

     尽管GRIST基线版本能够严格保证'restart'和'init'运行的一致性，但在某些尚未支持此特性的特殊配置下（如采用slab_ocean计算sst），或者用户对模式代码进行了修改的情况下（特别是新增物理过程），可能会造成init运行和restart运行不一致。此外，由于某些硬件故障会导致restart文件在写入时发生错误（罕见但出现），也将无法保证restart和init运行的一致性。因此，建议在使用restart时，保留一份可供与'init'运行进行一致性检验的数据文件，以便对比。检验可以使用如下命令：‘cdo diff 1.nc 2.nc’