.. sdnn documentation master file, created by
   sphinx-quickstart on Tue Aug 30 08:09:45 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

SDNN 用户指南
=============
SDNN 是一个基于开源编译器框架TVM的端到端的AI编译器框架，SEMIDRIVE 对TVM编译器框架做了适配，主要特性如下：

- 基于TVM 0.8.0 Release
- 支持OS: Android\Linux\QNX 开发及部署
- 支持多后端：X86_64_CPU/AARCH64_CPU/GPU(OPENCL)/SlimAI
- 支持C++/PYTHON接口开发及部署方式
- 支持异构及同构多模型部署
- 支持多进程和多线程应用

.. toctree::
   :maxdepth: 2
   :caption: 快速入门
   :name: quickstart

   quickstart/quickstart

.. toctree::
   :maxdepth: 2
   :numbered:
   :caption: SDNN基础
   :name: sdnn_base

   sdnn_base/license
   sdnn_base/docker
   sdnn_base/sdnn_build
   sdnn_base/deploy
   sdnn_base/inference

.. toctree::
   :maxdepth: 2
   :caption: SDNN进阶
   :name: sdnn_advanced

   sdnn_advanced/slimai_quantize_config
   sdnn_advanced/slimai_data_prepare
   sdnn_advanced/slimai_quantize_optim
   sdnn_advanced/slimai_restrictions
   sdnn_advanced/slimai_firmware_path

.. toctree::
   :maxdepth: 2
   :caption: 附录

   appendix/operator_support


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
