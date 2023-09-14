模式输入文件: 网格数据
================

GRIST模式基于非结构网格设计，可灵活实现全球准均匀模拟、全球变分辨率模拟和有限区域模拟。使用以下软件可自定义产生GRIST模式全球准均匀网格、全球变分辨率网格以及有限区域网格。

  1. GRID_GENERATOR：基于FORTRAN的GRIST网格产生软件。
  2. MPI-SCVT：球面质心Voronoi型网格并行产生软件。
  3. JIGSAW：Delaunay型非结构网格产生软件。

全球准均匀网格
----------------

  方式一：GRID_GENERATOR

  方式二：MPI-SCVT

全球变分辨率网格
----------------

  方式一：GRID_GENERATOR

  方式二：MPI-SCVT

  方式三：JIGSAW+GRID_GENERATOR

有限区域网格
----------------

  GRID_GENERATOR


备注：建议直接使用已预生成的网格用于模式安装校验测试： https://pan.baidu.com/s/1oQnOMDltTKADiaVnqMHbcg (code: ffqs)。
