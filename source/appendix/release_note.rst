============
Release Note
============

------
V3.0.0
------

#. sdnn工具升级到TVM正式版本0.10；
#. 简化sdnn build工具的编译选项并完善配置文件格式；
#. 新增sdnn run模型推理工具；
#. docker镜像升级，新增aipu工具链并升级python版本v3.8；
#. 解决slimai mpu设置bug，修复了slimai混合精度问题；
#. 新增aipu加速器，支持PC simulator和模型性能，带宽，精度的评估；
#. 将sdnn_test工具重命名为sdnn_run，并优化了工具选项和性能输出报告格式；

------
V2.2.0
------

#. 新增sdnn_build工具，用于模型编译和模型性能仿真；
#. 新增sdnn_test工具,可用于检测开发板端的部署环境和评估模型的性能与精度；
#. 新增模型编译slimai加速设备不支持的算子自动切换到CPU端执行；
#. slimai仿真编支持输出各层性能数据；
#. slimai编译支持输出相似度信息；
#. 模型编译支持输出节点增加layout通道变换算子；

------
V2.1.0
------

#. 新增SlimAI模拟器后端，可以基于模拟器输出网络各层性能数据；
#. 支持ONNX模型自动性能评估，用户可以将网络转为ONNX，进行性能评估；
#. 优化量化策略，支持自定义量化评价；
#. 新增elf path可自定义配置；
#. 支持quantized TFlite模型;

------
V2.0.0
------

#. 支持Muliti SlimAI设备部署；
#. 新增QNX系统支持部署SlimAI设备；
#. 支持应用程序多进程开发；
#. SlimAI编译工具链：XNNC 2.3.1升级到2.4;
#. 新增功能：KL散度量化;
#. 新增支持模型：BERT Wave2Letter ESRGAN;
#. 新增onnx算子：GreaterOrEqual;
#. TVM 升级到0.8 Release版本;
#. Docker重构及优化，将SDNN运行环境解构为多个docker镜像;
#. Example架构优化及重构;
