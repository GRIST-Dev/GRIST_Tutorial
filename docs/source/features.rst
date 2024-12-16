模式特点
================

总体设计
----------------
  21世纪的20年代，全球天气和气候建模在应用的空间和时间尺度上仍存显著差异。全球天气预报-气候预测行业将继续追求可达千米级的高分辨率模式。应用于气候及气候变化模拟的粗分辨率全球气候模式仍然被需要。学界对采用全球高分辨率公里尺度(k-scale)模型来彻底地统一全时空尺度的天气-气候模拟已有所呼吁，但是实现这一目标的过程中需要解决很多问题。

  为了维持一个兼顾当前和未来的可持续模式研发平台，GRIST的设计遵循应用驱动路线。这体现在该系统的几大技术特征。首先，动力框架被设计为能够在单一积分流中进行静力和非静力求解的切换。这对于粗网格分辨率（例如，>10 km）的应用尤有价值：在这些应用中使用非静力方程很难带来价值增益，通常只增加计算负担。其次，由于大气模式物理过程代码通常具有较长的使用生命周期，并且常针对不同尺度的应用进行调整。GRIST被设计为能够与不同的天气、气候模拟体系下的不同模式物理过程包兼容。这样，一套物理过程的原始应用场景可以被继承。同时，也为促进不同物理包之间的融合提供了基础。第三，GRIST系统不依赖于现有全面的软件框架或耦合系统。这有助于实现快速迭代开发，且模式发展更具灵活性。

动力内核框架及物理-动力耦合
----------------
  静力-非静力一体化：
  GRIST动力框架（内核）方程组源于原始方程，但在其基础上恢复了垂直加速项以实现非静力动力计算。离散化采用了基于层平均形式的控制方程。由于采用了质量坐标，垂直加速项可作为运行时选项，在积分过程中进行灵活切换。当垂直加速度项恢复时，非静力动力能够在粗网格间距下保持静力平衡，并且GRIST的这一能力已经在实践中得到验证。这意味着，垂直速度预报方程必须在粗网格间距下产生很小至接近于0的加速度。对于长期气候模拟应用，静力动力内核将继续保持良好作用。因为在分辨率较粗时，求解一组非静力方程应该在物理上重现静力平衡近似。模型开发经验表明，即使在具有复杂的物理-动力学相互作用和真实海陆分布的现实世界建模应用，GRIST非静力模型也可以在静力尺度产生与其静力版本非常一致的计算结果。

  全球-区域一体化：
  全球-区域一体化是支撑天气-气候一体化实现的重要路径，并带来许多价值增益。   

  层平均控制体积离散化：
  垂直离散化是使用基于层平均质量控制体积离散法展开的，其中控制方程中的所有坐标度量项首先转换为层厚度的物理定义。这将平流和压力梯度项转换为控制体积格式。通过合理制定的压力梯度算法，可以确保平流项和压力梯度项之间的一致性。这种一致性已通过非静力地形山波试验验证，不一致的计算将带来山波解的计算扭曲。层平均方法的方程形式里，要求水平和垂直压力梯度项之间存在适当的关系，因为广义的水平压力梯度项与垂直加速项存在联系。

  垂直干质量坐标：
  对于湿大气模式，采用基于干空气质量的混合垂直坐标系，可以确保模式大气动力准确地守恒干空气质量，无需后效订正器。

  水平非结构网格：
  GRIST采用真正意义的非结构网格，网格单元本身可适应任意非重叠网格结构，包括正二十面体、立方球等。非结构化网格建模方法在如今的动力框架开发中被广泛应用。但应该注意的是，它们并不等同于准均匀网格。水平数值算子采用“六边形-C”网格方法，在理论研究和简单模型实验中，六边形C网格可以支持优异的重力波频散关系，这是由于其优异的各向同性，使其对散度和梯度算子具备良好的数学表达。

  时间积分方案：
  内步积分采用forward-backward格式，外步循环采用Runge-Kutta格式。非静力项的垂直计算采用半隐式。所有预报变量采用原始形式进行积分，这区别于采用扰动项积分的分裂显式算法。

  水汽传输算法：
  正定传输算法采用了我国科学家自主设计的两步保形水汽平流方案作为主要计算格式之一。  

  通用物理-动力耦合：
  这是使GRIST动力框架具备实际天气-气候一体化模拟能力的关键。通过设计独立的物理-动力耦合算子和工作流，使得模式的动力框架可以采用不同方式与模式的物理过程进行耦合。在此基础上，已经发展了2套完整物理过程包（天气&气候套件），并均完成了AMIP大气积分试验。   

 
  1. Zhang, Y., J. Li, R. Yu, S. Zhang, Z. Liu, J. Huang, and Y. Zhou, (2019), A Layer-Averaged Nonhydrostatic Dynamical Framework on an Unstructured Mesh for Global and Regional Atmospheric Modeling: Model Description, Baseline Evaluation, and Sensitivity Exploration. Journal of Advances in Modeling Earth Systems, 11(6), 1685-1714.doi: https://doi.org/10.1029/2018MS001539    


  2. Zhang, Y., J. Li, R. Yu, Z. Liu, Y. Zhou, X. Li, and X. Huang, (2020), A Multiscale Dynamical Model in a Dry-Mass Coordinate for Weather and Climate Modeling: Moist Dynamics and Its Coupling to Physics. Monthly Weather Review, 148(7), 2671-2699.DOI: https://doi.org/10.1175/MWR-D-19-0305.1   


  3. Zhang, Y., Z. Liu, Y. Wang, and S. Chen, (2024), Establishing a limited-area model based on a global model: A consistency study. Quarterly Journal of the Royal Meteorological Society, 150(764), 4049-4065.doi:https://doi.org/10.1002/qj.4804.     

物理过程
----------------
   天气物理包(PhysW, baseline)：
   该物理包中的物理方案源自传统应用于中尺度模式的物理过程，并进行了适配和方案修改升级。   

   气候物理包(PhysC, baseline)：
   该物理包中的物理方案源自传统应用于全球气候模式的物理过程，并进行了适配和方案修改升级。   
   
   理想物理包：
   该物理包中的物理方案源自DCMIP动力框架比较计划中的简单物理方案，用于测试和检验最简化的三维动力模式。   
    
   参考文献：   
   1. Li, X., Y. Zhang, X. Peng, B. Zhou, J. Li, and Y. Wang, (2023), Intercomparison of the weather and climate physics suites of a unified forecast–climate model system (GRIST-A22.7.28) based on single-column modeling. Geosci. Model Dev., 16(10), 2975-2993.doi:https://doi.org/10.5194/gmd-16-2975-2023   

两套完整模型配置及基线AMIP测试对比
----------------
   
   1. Zhang, Y., R. Yu, J. Li, X. Li, X. Rong, X. Peng, and Y. Zhou, (2021), AMIP Simulations of a Global Model for Unified Weather-Climate Forecast: Understanding Precipitation Characteristics and Sensitivity Over East Asia. Journal of Advances in Modeling Earth Systems, 13(11), e2021MS002592.doi:https://doi.org/10.1029/2021MS002592.   


   2. Li, X., Y. Zhang, X. Peng, W. Chu, Y. Lin, and J. Li, (2022), Improved Climate Simulation by Using a Double-Plume Convection Scheme in a Global Model. Journal of Geophysical Research: Atmospheres, 127(11), e2021JD036069.doi:https://doi.org/10.1029/2021JD036069.   


   3. Fu, Z., Y. Zhang, X. Li, and X. Rong, (2024), Intercomparison of Two Model Climates Simulated by a Unified Weather-Climate Model System (GRIST), Part I: Mean State. Climate Dynamics.doi: https://doi.org/10.1007/s00382-024-07205-2.   

   
   4. Fu, Z., Y. Zhang, X. Li, C. Zhu, H. Liu, X. Rong, and C. li, (2025), Intercomparison of Two Model Climates Simulated by a Unified Weather-Climate Model System (GRIST), Part II: Madden-Julian Oscillation. Climate Dynamics.   

模式框架中英文简介
----------------
   
   1. 王一鸣，张祎，李晓涵，刘壮，周逸辉. 2024. GRIST天气-气候一体化模式系统框架功能设计和应用[J]. 气象科技进展. http://www.cmalibrary.cn/amst/2024/202404/fmbd/202409/t20240927_165283.htm   


   2. Zhang, Y., J. Li, H. Zhang, X. Li, L. Dong, X. Rong, C. Zhao, X. Peng, and Y. Wang, 2023: History and Status of Atmospheric Dynamical Core Model Development in China.   Numerical Weather Prediction: East Asian Perspectives, S. K. Park, Ed., Springer International Publishing, 3-36.   

关键应用评估
----------------
   
   1. Zhou, Y., Y. Zhang, J. Li, R. Yu, and Z. Liu, (2020), Configuration and evaluation of a global unstructured mesh atmospheric model (GRIST-A20.9) based on the variable-resolution approach. Geosci. Model Dev., 13(12), 6325-6348.doi:https://doi.org/10.5194/gmd-13-6325-2020. (变分辨率模拟)  


   2. Zhang, Y., X. Li, Z. Liu, X. Rong, J. Li, Y. Zhou, and S. Chen, (2022), Resolution Sensitivity of the GRIST Nonhydrostatic Model From 120 to 5 km (3.75 km) During the DYAMOND Winter. Earth and Space Science, 9(9), e2022EA002401.doi:https://doi.org/10.1029/2022EA002401. (全球风暴解析模拟)   


   ...
   
   
