=============
快速入门
=============

概述
======

目前9系列 **SOC** 只有 **V9752** 系列芯片包含双 ``SlimAI`` 设备,其余系列包含单个 ``SlimAI`` 设备。

硬件准备
========

#. 9系列SOC参考板
#. **USB** 数据线
#. 电脑（Windows）


通过 ``SDToolBox`` 软件给参考板下载需要的pac包。通过拨码开关配置下载模式。


软件准备
========

参考章节1、2、3，在电脑上创建sdnn编译环境，主要包括安装license，配置docker开发环境，安装sdnn_build工具等。

.. note::

   目前只提供docker开发环境，请确保电脑安装docker软件。


编译第一个模型
==============

编译流程
--------

实际应用场景中，一般需要将多个模型部署到单个SlimAI设备上，此时建议先对每个模型单独编译，确保模型的算子加速器都能支持。最后引用每个模型单模型编译时自动生成的json文件，即可完成多模型的编译。


单模型
^^^^^^

确认模型是否可以编译
""""""""""""""""""""

对于一个新模型，首先要确认该模型中的算子类型、算子属性参数和网络图结构等加速设备是否支持，此时只需执行如下命令：

.. code-block:: bash

   $ sdnn_build -f xxx.onnx -a slimai

上述命令执行完后，模型开始编译，如果编译成功，会自动在模型所在目录生成 ``.autogen.cfg`` 和 ``.json`` 文件。

.. sidebar:: Tips

   如果编译失败，请向FAE提供完整的编译log信息，如果log信息不足以定位问题，还需要提供相应的模型文件。

如果已有slimai配置文件，则在模型编译时指定 ``--cfg`` 参数：

.. code-block:: bash

   $ sdnn_build -f xxx.onnx -a slimai --cfg xxx.cfg

指定 ``--cfg`` 参数后，会在自动生成 **json文件** 中添加cfg文件路径参数。

确认SlimAI cfg文件参数
""""""""""""""""""""""

当通过第一步模型编译后，需要打开自动生成 ``.autogen.cfg`` 文件，根据模型实际情况修改内部参数；

.. note::

   验证和校验数据的选择对模型的精度影响较大，请合理选择相关数据。


多模型编译
^^^^^^^^^^

确认每个新模型可以正确编译后，便可以执行多模型编译：

.. code-block::

   $ sdnn_build -f model1.json -f model2.json -f model3.json --os android

.. note::

   #. ``--os`` 参数根据实际部署系统确定，目前可选 **linux** 、**android** 和 **qnx**

部署流程
--------

模型部署
^^^^^^^^

将编译生成的模型so库和SlimAI固件elf文件，拷贝到目标板，可以通过 ``scp`` 或 ``adb`` 实现文件的传输。
其中，elf文件需要放置到如下固定目录：

- Linux: ``/lib/firmware``
- Android: ``/vendor/firmware``
- QNX: ``/lib/firmware``

对于模型so库文件可以放置任意位置。


运行库部署
^^^^^^^^^^

参考章节4，配置模型运行环境。


推理流程
--------

建议使用 ``sdnn_test`` 工具对模型进行推理测试，详细指令如下：

.. code-block:: shell

   $ ./sdnn_test cat.png model1.deploy.json -d -p

运行上述程序后输出如下信息：

.. code-block:: shell

   |-----------------------|
   |    SlimAI SelfCheck   |
   |-----------------------|
   |      Item    | Status |
   |--------------|--------|
   |  xrp driver  |  Pass  |
   |  xrp node    |  Pass  |
   |  elf load    |  Pass  |
   |-----------------------|


   ===> [./mobilenet_v2.so]
   |-----------------------|
   |    Node    | Layout   |
   |------------|----------|
   |   input    |  input:[1, 3, 224, 224]
   |   output   |  0:[1, 1000]
   |-----------------------|

   ===> DataSet Method: [ImageNet]
   ===> Metric Method: [TopK]
   |-----------------------|
   |    Software Version   |
   |-----------------------|
   |    Params   | Version |
   |-------------|---------|
   |SDNN Test    | V1.0.2  |
   |SDNN Runtime | V2.2.1  |
   |-----------------------|

   |---------------------|
   |    Inference Time   |
   |---------------------|
   |  Params  | Time[ms] |
   |----------|----------|
   |   mean   |   8.421
   |   std    |   0.000
   |---------------------|

   |--------------------------|
   |   Inference Frame Rate   |
   |--------------------------|
   | Params | Frame Rate[fps] |
   |--------|-----------------|
   |  FPS   |     118.76
   |--------|-----------------|

