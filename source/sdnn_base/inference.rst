========
模型推理
========

完成模型编译后，就可以调用API接口编写应用代码。关于API接口的使用方法，请获取 **examples** 仓库代码，里面包含详尽的接口说明。目前提供　``TVM API`` 和 ``SDNN API``　两套接口，可以根据实际场景选择合适的API接口。
如果目前不关注API接口如何使用，只想验证模型推理性能或结果，可以直接下载sdnn_test程序，将其拷贝到板子上，便可以进行模型推理。

模型layout判断
==============

对于slimai模型，在启动智能搜索的模式下，实际模型推理的输入和输出的layout格式可能与配置文件中的不一致，此时需要参考编译log输出信息，确定最终的layout格式。
一般Optimizing 阶段结束后，会打印出如下关键信息：

.. code-block:: bash

   INFO: Optimizing for bandwidth (96%): Clip_clip_1300__1_minClip_4_whd_dwh
   INFO: Optimizing for bandwidth (96%): Add_add_1330__3_whd
   INFO: Optimizing for bandwidth (97%): Conv_nn_conv2d_1350__8_whd
   INFO: Optimizing for bandwidth (97%): Clip_clip_1360__1_minClip_4_whd
   INFO: Optimizing for bandwidth (97%): Clip_clip_1390__1_minClip_4_whd
   INFO: Optimizing for bandwidth (98%): Conv_nn_conv2d_1430__7_whd
   INFO: Optimizing for bandwidth (98%): Clip_clip_1440__1_minClip_4_whd
   INFO: Optimizing for bandwidth (98%): Clip_clip_1470__1_minClip_4_whd
   INFO: Optimizing for bandwidth (99%): Conv_nn_conv2d_1510__7_whd
   INFO: Optimizing for bandwidth (99%): Clip_clip_1520__1_minClip_4_whd
   INFO: Optimizing for bandwidth (99%): Clip_clip_1550__1_minClip__4_whd
   INFO: Neural Network Inputs:
   INFO:   Input #0: slimai_0_i0_whd, layout: WHD, channel order: RGB, dimensions: [224, 224, 3]
   INFO: Neural Network Outputs:
   INFO:   Output #0: nn_softmax_30_dwh, layout: DWH, dimensions: [1, 1, 1001]



Neural Network Inputs
---------------------

该字段包含模型输入节点的信息，其中layout属性需要特别注意，编译log中的layout循序与通用的排列格式是反向。例如，上述layout格式为WHD，对应通用的CHW格式。

Neural Network Outputs
----------------------

该字段与输入一致，其中layout和dimensions需要编译后特别关注。


examples
========

examples主要用于说明如何使用 ``API`` 接口和如何在不同的目标设备上部署模型。

获取源码
--------

获取examples代码，代码仓库地址从客户支持系统上获取。

编译APP
-------

使用example进行示例编译前，需要进入examples的根目录，执行以下指令进行环境变量的设置：

.. code-block:: bash

   $ source envsetup.sh


然后进入examples/apps目录，调用build_app.sh脚本进行app的编译，例如，编译aarch64-linux平台的应用，命令如下：

.. code-block:: bash

   $ ./build_app.sh -p aarch64-linux -d app_api/sdnn_api/


sdnn_test工具
=============

``sdnn_test`` 工具包含如下功能：

#. SlimAI设备自检；
#. 模型性能评估；
#. 模型精度评估(分类)；


获取工具
--------

获取不同平台的测试工具：

.. tabs::

   .. tab:: Linux-x86_64

      `sdnn_test_x86 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/x86_64-linux_sdnn_test.tgz>`_

   .. tab:: Linux-aarch64

      `sdnn_test_linux <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/aarch64-linux_sdnn_test.tgz>`_

   .. tab:: Android-aarch64

      `sdnn_test_android <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/aarch64-android_sdnn_test.tgz>`_

   .. tab:: QNX-aarch64

      `sdnn_test_qnx <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/aarch64-qnx_sdnn_test.tgz>`_


帮助信息
--------

``sdnn_test`` 运行时添加 ``--help`` 或 ``-h`` 参数，可以输出软件相关信息：

.. code-block:: bash
   :linenos:

   sdnn_test version: 1.0.2
   Usage: sdnn_test [params] image deploy_json

      -a, --accuracy (value:false)
              Whether evaluate the accuracy of model.
      -d, --debug (value:false)
              Whether enable debug information.
      -h, --help (value:true)
              Print help message.
      -n (value:10)
              The count of loop inference.
      -p, --performance (value:false)
              Whether evaluate the performancb of model.

      image
              the path of test image.
      deploy_json
              The path of deploy json file.

关于指令参数，详细说明如下：

.. table:: 应用参数
   :name: sdnn_test_params

   +---------------+------+--------+------+--------------------------+
   | 参数          | 缩略 | 默认值 | 状态 | 说明                     |
   +===============+======+========+======+==========================+
   | image         |      |        | 必须 | 设置数据集路径或单张图片 |
   +---------------+------+--------+------+--------------------------+
   | deploy_json   |      |        | 必须 | 设置部署json文件         |
   +---------------+------+--------+------+--------------------------+
   | --help        | -h   | true   | 可选 | 打印帮助信息             |
   +---------------+------+--------+------+--------------------------+
   | --debug       | -d   | false  | 可选 | 使能调试信息输出         |
   +---------------+------+--------+------+--------------------------+
   | --performance | -p   | false  | 可选 | 使能性能信息输出         |
   +---------------+------+--------+------+--------------------------+
   | --accuracy    | -a   | false  | 可选 | 使能精度信息输出         |
   +---------------+------+--------+------+--------------------------+
   |               | -n   | 10     | 可选 | 设置模型推理次数         |
   +---------------+------+--------+------+--------------------------+

自检
----

运行 ``sdnn_test`` 程序可以用于运行环境的自检，输出如下 **LOG** ：

.. code-block:: bash
   :linenos:

   |-----------------------|
   |    SlimAI SelfCheck   |
   |-----------------------|
   |      Item    | Status |
   |--------------|--------|
   |  xrp driver  |  Pass  |
   |  xrp node    |  Pass  |
   |  elf load    |  Pass  |
   |-----------------------|

#. **xrp driver** : 检查xrp驱动是否安装；
#. **xrp node** ： 检查设备节点是否正确生成；
#. **elf load** : 检查elf文件是否正确加载；


调试信息
--------

``sdnn_test`` 运行时添加 ``--debug`` 或 ``-d`` 参数，使能调试信息输出，输出 **LOG** 如下：

.. code-block:: bash
   :linenos:

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

调试信息输出包括，加载模型的路径、数据集方法和测试方法等。

软件版本
^^^^^^^^

包含 ``sdnn_test`` 软件版本号和tvm runtime库的版本号。其中，runtime库的版本用于判断模型so库是否于runtime库版本匹配。

节点结构
^^^^^^^^

包含输入和输出节点的结构信息，通过该信息可以判断部署网络的输入输出结构信息。


性能评估
--------

``sdnn_test`` 运行时添加 ``--performance`` 或 ``-p`` 参数，使能模型性能评估功能，输出LOG如下：

.. code-block:: bash
   :linenos:

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

- **mean** : 模型推理 ``n`` 次的平均值；
- **std** : 模型推理 ``n`` 次的方差值；
- **FPS** : 模型推理的帧率；


精度评估
--------

``sdnn_test`` 运行时添加 ``--accuracy`` 或 ``-a`` 参数，使能模型精度评估功能，输出LOG如下：

.. code-block:: bash
   :linenos:

   |------------------------------|
   |            Top5              |
   |           0.00 %
   |------------------------------|
   | Index | Score |     Label    |
   |-------|-------|--------------|
   |  282  | 9.989 | tiger cat
   |  287  | 9.161 | lynx, catamount
   |  285  | 9.068 | Egyptian cat
   |  278  | 9.019 | kit fox, Vulpes macrotis
   |  281  | 8.856 | tabby, tabby cat
   |------------------------------|

目前支持分类模型的 **TopK** 精度评估，其它类别模型的精度评估，可以采用 **BinData** 形式，先将模型推理输出结果保存为 **bin** 格式文件，然后通过python等脚本语言对数据进行模型的后处理，来评估模型精度。


部署json格式说明
----------------

``sdnn_build`` 工具编译模型会同时生成 ``模型库文件(so)`` 和 ``部署配置文件(.deploy.json)`` ，通过编译时指定 ``--save`` 参数，可指定文件的保存路径。 关于json文件的格式详见如下：

.. code-block:: json

   {
     "model": {
    "name": "mobilenet_v2",
    "accelerator": "cpu",
    "path": "./mobilenet_v2.so",
    "domain": "classfication",
    "inputs": [
      {
        "name": "input",
        "layout": "NCHW",
        "channel_order": "RGB",
        "mean": [
          0.485,
          0.456,
          0.406
        ],
        "std": [
          0.229,
          0.224,
          0.225
        ]
      }
    ]
     },
     "dataset": {
    "name": "ImageNet"
     },
     "metric": {
    "method": "TopK",
    "params": [
      5
    ],
    "annotation": "label.txt"
     }
   }


.. note::

   使用sdnn_test程序评估模型前，请先确认.deploy.json配置文件的参数 **设置正确** 。


model字段
^^^^^^^^^

name
""""

设置模型别名，该属性编译时会自动生成，不需要特别关注。

accelerator
"""""""""""

设置模型推理的设备类型，编译时该属性值会自动生成，一般会与 path 字段的模型so文件匹配。如果修改该参数，需要确保 path 字段的so文件与新设置的加速器类型匹配。

path
""""

设置推理模型文件的加载路径，编译时该属性值会自动生成，如果模型 **so文件** 存放路径与 **部署json文件** 不在同级目录，需要修改该属性值，采用相对路径方式，相对该json文件。

domain
""""""

设置模型所属领域，该属性在 **dataset** 和 **metric** 字段未设置时，根据指定领域属性，设置模型前后处理的默认参数。

inputs
""""""

设置模型输入节点的属性，支持多输入格式，按照数组方式指定不同输入节点的参数。

1. **name**

   设置模型输入节点的名称，该属性值，编译时会字段填充。该属性用于模推理时，指定模型

2. **layout**

   设置模型输入节点的结构，该属性值，编译时会字段填充。

3. **channel_order**

   设置模型通道循序是否交换。

4. **mean**

   设置输入节点每个通道的平均值，如果编译时配置文件中已经设置，该属性会继承过来。

5. **std**

   设置输入节点每个通道的方差值，如果编译时配置文件中已经设置，该属性会继承过来。

.. note::

   **mean** 和 **std** 属性值对于slimai设备部署时会忽略，其在编译时已经集成到模型文件中。

dataset字段
^^^^^^^^^^^

数据集字段，设置模型推理时使用的数据集类型，该字段作用于模型推理的前处理。默认采用 **ImageNet** 数据集处理方式。

name
""""

根据模型训练的数据集类型，设置name属性。目前支持 **ImageNet** 数据集的前处理。

Metric字段
^^^^^^^^^^

关于模型精度测试，可以设置metric属性。编译时如果不指定，默认采用BinData格式输出。

method
""""""

目前支持的method方法有 **BinData** 和 **TopK** 两种。其中BinData支持所有模型，TopK可以支持常见的分类模型评估。

params
""""""

该字段设置method的具体参数，以数组形式保存，可以按照循序设置不同的参数。

annotation
""""""""""

该字段设置标注文件的路径，如果需要评估模型的精度，可以设置数据集的标注文件。目前支持ImageNet数据集的评估。

sdnn_test使用
-------------

使用 ``sdnn_test`` 工具可以快速评估模型的性能，关于模型推理的准确性，可以采用BinData的方式，将模型的输出张量结果保存为binary文件，后续可以通过python脚本读取该文件进行相关后处理评估。关于直接评估模型的准确性，目前支持TopK方法，可以指定数据集评估精度或者指定单张图片评估分类网络的准确性，其它类型网络的评估方法后续会陆续支持。


模型评估方式
^^^^^^^^^^^^

基于主机评估
""""""""""""

该方式在docker容器中直接评估模型，请下载x86_64格式的sdnn_test工具，目前可以支持cpu和slimai设备的评估，不过slimai设备需要开启-emu模式编译模型，会生成.sim.so模型文件。

基于芯片评估
""""""""""""

根据芯片所运行的os，下载合适的sdnn_test程序，目前支持linux,android和qnx操作系统。然后将模型(.so)和部署配置文件(.deploy.json)拷贝到板子上，最后确保板子上已经配置好runtime库和opencv库，就可以直接运行sdnn_test进行评估。


sdnn_test参数项
使用sdnn_test应用，有两个必需设置的参数。第一参数是数据的设置，第二个参数是部署配置文件的设置。
数据
数据项可以指定单张图片或者设置数据集的目录，测试图片可以从提供的examples/dataset目录获取。
配置文件
指定编译生成的部署配置文件(.deploy.json)，该文件包含模型预处理和后处理相关信息。该文件会在编译时自动生成，并和模型so文件在同级目录下。验证模型时，需要将xxx.deply.json和xxx.so文件同时拷贝到板子上验证。


示例
^^^^

.. code-block:: bash

   ./sdnn_test cat.png mobilenet_v2.deploy.json -d -p