模式重启
=================

  GRIST模式的重启动运行，确保计算结果与连续积分的位一致性。重启动运行仅需设置grist.nml中 run_type='restart'。模式将根据step_restart.txt中的时次，自动读取对应的restart文件并开始重启动积分。
  
  一组典型的Restart文件如下：
  
  GRIST.RST.Dyn.G6.1d.amipw.1980-05-01.00850-01200.nc

  GRIST.RST.Dyn.G6.2d.amipw.1980-05-01.00850-01200.nc  
  
  GRIST.RST.Dyn.G6.3d.amipw.1980-05-01.00850-01200.nc  
  
  GRIST.RST.Phy.G6.1d.amipw.1980-05-01.00850-01200.nc  
  
  GRIST.RST.Phy.G6.2d.amipw.1980-05-01.00850-01200.nc  
  
  GRIST.RST.Lnd.G6.1d.amipw.1980-05-01.00850-01200.nc  
  
  GRIST.RST.Lnd.G6.2d.amipw.1980-05-01.00850-01200.nc  

  这里，00850 表示当前的“积分天数”，01200表示模式的时间步长。step_restart.txt的时次对应为：86400/模式时间步长*积分天数+1。此时，模式将读取这一组文件并开始重启动计算。step_restart.txt中默认会自动记录最近一个write_restart文件时次的时次数。
  write_restart文件的频次可以由用户自定义。
