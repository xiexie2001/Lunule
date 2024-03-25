# 论文复现：Lunule: An Agile and Judicious Metadata Load Balancer for CephFS

## 项目介绍

本项目为河海大学2023-2024学年秋学期分布式计算课程的小组作业项目，相关背景信息展示如下：

+ 论文题目：Lunule: An Agile and Judicious Metadata Load Balancer for CephFS  
+ 论文来源：SC '21: Proceedings of the International Conference for High Performance Computing, Networking, Storage and Analysis  
+ 论文链接：[Lunule: An Agile and Judicious Metadata Load Balancer for CephFS](https://dl.acm.org/doi/abs/10.1145/3458817.3476196)  
+ 源码链接：[Lunule v2.1](https://github.com/mdbal-lunule/lunule/tree/clean-if-dev)  
+ 小组成员：谢天祺（231307040028，组长），王冬雨（231307040024），张捷（231307040030）  

## 项目结构  

主要在原项目上添加了中文目录[复现文档](./复现文档)，用于放置与项目复现相关的软件文档、软件安装文档、软件测试文档等相关说明文档。

Lunule源项目的[README](./Code/README.md)文件

本项目的[CephFS环境安装文档](./复现文档/分布式验证代码/02%20Install%20CephFS%20CN.md)

本项目的[Lunule安装文档](./复现文档/分布式验证代码/03%20lunule.md)

本项目的[测试全流程代码文档](./复现文档/分布式验证代码/01%20Software%20Testing%20Code.md)

## 实验环境

+ 本实验复现项目使用[阿里云](https://aliyun.com)的ECS弹性云服务器部署实验环境；
+ 使用 16 个云服务器构建集群（1\*Magener，5\*MDS，9\*OSD，1\*Client）；
+ 每个服务器为 2 核心 4 GB 内存的配置；
+ 每个服务器为挂载 1 个 160 GB 云盘作为系统盘、 1 个 1 Mbps速率的带宽以或许公网数据；
+ OSD1-9 服务节点额外挂载 40 GB云盘作为OSD存储块，
+ Client 节点额外挂载 300 GB 云盘用于存储测试用例的数据集。

## 写在最后

部分遇到的问题的解决方法可以参考原作者的解答：[复现问题的解答](https://github.com/mdbal-lunule/lunule/issues/1)

由于时间、环境、资源的限制，本次复现项目的执行中仅使用了论文开源代码及项目中提及的测试程序：MXNet中的img2rec.py，利用该程序对论文所述的文件数据均衡器Lunule相关功能进行了一些测试。资源不够和对其他测试项目的不熟悉，是限制我们对对原论文提及的CNN、NLP、ZipFan、WebTrace、MD Create五个类别进行全面测试的主要原因。除此以外，在论文的实际复现过程中，遇到最多的问题是实验环境的构建过程，本项目复现的论文的开源代码质量较高，因此在实际的复现过程中节省了在项目源码的深度研究和代码调试过程中的大量时间，得以在有限的时间内顺利完成相关工作。

最后，分布式系统作为当下服务计算的基础系统，为云计算和边缘计算的实现提供了坚实的基础，本文所述数据均衡器Lunule及其核心思想基于实时监控的负载均衡算法，利用监视器Monitor、决策器Initiator、规划器Planner对实时的负载进行监视和迁移决策，当发生负载不均衡达到所设上限时将进行数据迁移和调整，以实现工作负载调整迁移规划的合理性和动作的实时性。  
