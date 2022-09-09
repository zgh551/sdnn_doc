======================
SlimAI模型量化配置文件
======================

SlimAI cfg文件说明
------------------

由于SlimAI设备硬件计算单元是定点计算，因此模型部署到SlimAI上需要对模型进行量化编译。将浮点模型量化为定点模型，需要配置校验数据等量化信息，这些配置信息通过cfg文件进行量化设置。cfg文件组织按段进行组织，``[path]`` 、 ``[network]`` 、 ``[dataset]`` 、 ``[quantization]`` 、 ``[performance]`` 等是段的起始标记，分别表示 **相关路径配置** 、 **网络信息配置** 、 **数据集信息配置** 、 **量化配置** 以及 **性能配置** 等。

下面脚本介绍cfg文件的各配置项：

.. code-block:: shell

   [path]
   output_dir = output
   image_dir = ../../dataset/ILSVRC2012

   [network]
   class = Classification
   model_type = float
   input_data_layout = NHWC
   channel_order = RGB
   mean = 123.680,116.779,103.9390
   stddev = 128.0,128.0,128.0

   [metric]
   output_debug = 0

   [dataset]
   calibration_set = imagenet_224_png_train.txt
   calibration_count = 5
   validation_set = imagenet_224_png_val.txt
   validation_count = 8

   [quantization]
   accuracy_level = 0
   range = 96:99:1

   [performance]
   optimization = level2

path
^^^^

量化相关输出信息保存路径和数据集路径配置段。

output_dir
""""""""""

指定调试输出和性能评测输出的文件路径，请配置 **相对于cfg文件** 的路径，最终会由编译工具补全为绝对路径。

image_dir
"""""""""

指定用于标定和验证的imagelist列表所在路径。

network
^^^^^^^

模型相关参数配置段

class
"""""

模型类别，可能取值为 **Classification** , **Segmentation** or **Detection**. 其中 **Segmentation** 是一个通用类别，诸如super resolution这样的以3d blob为输出的类别均可用其指代。

model_type
""""""""""

模型参数类型，对于 **TFLite Fully Quantized Models** ，model_type需要设为 ``quantized`` ，其他情况均设为 ``float`` 。

input_data_layout
"""""""""""""""""

可设置 ``NHWC`` 、 ``NCHW`` 和 ``NonImage`` 具体可取值及数据集的介绍见数据集介绍章节。

channel_order
"""""""""""""

给定模型的图像数据通道顺序，常见 ``RGB`` 或 ``BGR`` ，非图像时无效。

mean 和 stddev
""""""""""""""

预处理参数， **mean** 和 **stddev** 取值来自训练时设置的数据归一化值，数据个数与通道数相等，如果数据通道不是3则需要
随着通道数调整。由于SlimAI接收的数据为uint8，即[0,255]区间，若训练数据的归一化参数不是作用在uint8数据上的，则需要对原参数进行scale使其等效于作用在uint8数据再填到mean或stddev中。

.. math::

   output_{float} = \frac{input_{uint8} - mean}{std}

metric
^^^^^^

量化验证的度量配置段，Classification模型无需配置，其他模型需要用户自定义metric插件，具体可见用户自定义量化度量。

dataset
^^^^^^^

calibration_set，validation_set等具体见数据准备。

quantization
^^^^^^^^^^^^

关于模型量化相关配置参数。

accuracy_level
""""""""""""""

可取0-4，具体等级解释见下，多个配置用逗号分割可启动智能量化搜索,例如'accuracy_level = 0,1,2,3'

下面介绍量化准确度等级:

- Accuracy Level 0. 使用对称有符号8bit量化，以尽可能获得更快速的推理。权值和激活以tensor或者层为粒度（per-tensor）进行量化。
- Accuracy Level 1. 仍使用对称有符号8bit量化，但对于convolution 和 deconvolution的权重使用channel粒度（per-channel）量化以提高计准确度，而激活仍保持tensor粒度量化。对于一些网络这能在不增加推理时间的情况下有更好的准确度。
- Accuracy Level 2. 这个等级下除了level1的优化功能开启外，为合适的网络层（例如relu的输出）选择无符号8bit 对称量化，以获得更好的准确度。
- Accuracy Level 3. 这个等级下除了level1的优化功能开启外，尽可能的使用非对称无符号8bit量化来提高准确性。注意非对称量化只在激活上实施，而不在权重上实施，另外自定义算子不使用非对称量化。
- Accuracy Level 4. 这个等级下尽可能使用对称有符号 ``16bit`` 量化，对于 **convolution** 和 **deconvolution** 的权重使用channel粒度量化，以获得更好的准确度，而激活仍保持tensor粒度量化。

range
"""""

每层激活量化后值域范围覆盖原浮点激活范围的百分比搜索范围，例如下面配置将搜索输出值能覆盖原浮点激活值96%到99%的范围，以1%为步进进行量化分析搜索，量化结果不好时可以尝试扩大此范围以提高模型精度。


performance
^^^^^^^^^^^

optimization
""""""""""""

- level1 – reduced search space
- level2 – larger search space than level1 (default)


更具体的模型量化配置文件可参考sdnn_model_zoo内每个模型文件夹下的cfg文件。总之为了尽可能的加速计算，权重仅使用对称量化，8bit或16bit，为了提高准确度可实施per-channel粒度的量化；而激活始终使用per-tensor粒度的量化，支持8bit对称量化、8bit非对称量化、16bit对称量化。自定义算子不使用非对称量化。

