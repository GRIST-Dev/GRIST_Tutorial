代码获取
==================
GRIST模式代码
------------------
  
  GRIST模式的源代码的演进版本目前在github中托管: https://github.com/GRIST-Dev/GRIST

  冻结版本（Frozen version）：一些已发表文献的附件中包含了GRIST模式发展过程中的一些冻结版本，可在这些论文的附件下载：


  1. Chen, S., Zhang, Y., Wang, Y., Liu, Z., Li, X., and Xue, W.: Mixed-precision computing in the GRIST dynamical core for weather and climate modelling, Geosci. Model Dev., 17, 6301–6318, https://doi.org/10.5194/gmd-17-6301-2024, 2024.  


  2. Li, X., Zhang, Y., Peng, X., Zhou, B., Li, J., and Wang, Y.: Intercomparison of the weather and climate physics suites of a unified forecast–climate model system (GRIST-A22.7.28) based on single-column modeling, Geosci. Model Dev., 16, 2975–2993, https://doi.org/10.5194/gmd-16-2975-2023, 2023.  
  

  3. Fu, Z., Zhang, Y., Li, X. et al. Intercomparison of two model climates simulated by a unified weather-climate model system (GRIST), part II: Madden–Julian oscillation. Clim Dyn 63, 55 (2025). https://doi.org/10.1007/s00382-024-07527-1.   


Bug提交
------------------
  如果用户在代码中遇到bug(即，它没有按照文档所建议的方式运行)，可以以邮件（grist_dev@163.com）方式报告该问题。在编写bug报告时，需要提供足够的信息，以便bug可以被重现。下面的列表列出了报告中应包含的最少信息：
    
    #. GRIST的版本号。
    #. 运行代码的环境。包括如Fortran编译器，MPI库等相关信息。
    #. 编译和运行命令行或脚本。
    #. 打印模式输出。其中应该包含打印到输出日志中的所有错误消息。
