====================
SlimAI模型量化数据集
====================

数据集准备与image list
======================

要得到较好质量的量化模型，需要准备两组图像数据： ``定标集和验证集`` 。其指导思想是尽可能每个类别都选择多张图像来做定标和验证。建议选用与训练数据集同分布的稀疏子集作为定标集，而选用验证集或者测试集同分布的稀疏子集作为验证集，数据集越大量化效果越好，但量化时间也越长，因此具体数量需要根据具体工程需要进行取舍。最后用户需要创建文件夹和 ``image list`` 文件来组织数据集的图像和对应label文件。
image list 具体格式如下：

.. code-block:: shell

   <number-of-images>
   <image-path>[,<image-path>…]   [<image-label or ground-truth>[,<image-label or ground-truth>…]]
   <image-path>[,<image-path>…]   [<image-label or ground-truth>[,<image-label or ground-truth>…]]
   ...

文件的第一行是样本的数量，即该文件中接下来由多少行数据被用于量化分析，该值不能大于该文件中实际数据条目数。接下来是数据行，每行有左右两项，左侧是图像相对路径，右侧是对应的图像label或者ground truth（定标集的image list右侧项可以为空），它们之间用空格间隔，左右两项内部存在多个成员时则用逗号分隔，中间不能再出现空格。定标集和验证集里列出的图像，其宽高大小必须与网络要求的输入宽高一致，格式建议选择png或者ppm。

.. note::
   image 和 labe文件名主干中不能出现 ``.`` 符号，请用下划线 ``_`` 代替，例如40_20190308_Clear_Dusk_Country_0001.avi_frames_4500.png宜改为40_20190308_Clear_Dusk_Country_0001_avi_frames_4500.png

下面是一些image list的要求和具体样例：

1. 分类数据label用数字标记，类别从0开始计，例如4类分类则类别标记为0，1，2，3。下面分类网络示例中65，970，...等数字即对应图像的分类类别编号：

.. code-block:: shell

   200
   input/ILSVRC2012_val_00000001.ppm       65
   input/ILSVRC2012_val_00000002.ppm       970
   input/ILSVRC2012_val_00000003.ppm       230
   input/ILSVRC2012_val_00000004.ppm       809
   input/ILSVRC2012_val_00000005.ppm       516
   ...
   #...表示省略掉后续的数据行

2. 有些数据集label文件与图像一一对应，image list中要求label项与图像文件名主干相同仅后缀差异，标注文件被统一放在一个文件夹下。该文件夹路径会作为参数填入cfg文件中的[metric]段，参见SlimAI量化调优中内容。检测网络pascal voc类型数据的image list示例：

.. code-block:: shell

   7
   PNGImages/1_20190128_Cloud_Day_City_001_avi_frames_2700.png 1_20190128_Cloud_Day_City_001_avi_frames_2700.xml
   PNGImages/1_20190128_Cloud_Day_City_001_avi_frames_31800.png 1_20190128_Cloud_Day_City_001_avi_frames_31800.xml
   PNGImages/1_20190128_Cloud_Day_City_001_avi_frames_37200.png 1_20190128_Cloud_Day_City_001_avi_frames_37200.xml
   PNGImages/12_20190128_Clear_Day_City_006_avi_frames_94200.png 12_20190128_Clear_Day_City_006_avi_frames_94200.xml
   PNGImages/13_20180901_Clear_Night_City_007_avi_frames_20000.png 13_20180901_Clear_Night_City_007_avi_frames_20000.xml
   PNGImages/13_20180901_Clear_Night_City_007_avi_frames_46800.png 13_20180901_Clear_Night_City_007_avi_frames_46800.xml
   PNGImages/13_20180901_Clear_Night_City_007_avi_frames_53600.png 13_20180901_Clear_Night_City_007_avi_frames_53600.xml
   #例如，图像13_20180901_Clear_Night_City_007_avi_frames_53600.png 和 label文件13_20180901_Clear_Night_City_007_avi_frames_53600.xml
   #拥有相同的文件名主干13_20180901_Clear_Night_City_007_avi_frames_53600

3. 分割网络image list 示例：

.. code-block:: shell

   200
   IM/2007_000033.ppm GT/2007_000033.pgm
   IM/2007_000042.ppm GT/2007_000042.pgm
   ...

.. note::

   Segmentation label以图像形式给出，图像必须是pgm格式。

1. npy image list file示例：

.. code-block:: shell

   200
   000000289393.npy 3
   000000289394.npy 9
   ...

这些image list文件会被填入cfg文件中的dataset段，例如：

.. code-block:: shell

   [dataset]
   calibration_set = imagenet_224_png_train.txt
   calibration_count = 5
   validation_set = imagenet_224_png_val.txt
   validation_count = 8

- calibration_set 是定标image list 文件，包含图像个数以及每个图像的path，可选的有label信息，但label信息不会被使用。
- calibration_count是用于定标的图像个数，它不能大于calibration_set的图像个数
- validation_set是验证 image list文件， 结构与上面定标image list一样，但必须包含label信息, 否则无法进行模型量化的度量，从而无法选出最优的量化参数以生成最终最优量化模型
- validation_count是用于验证的图像个数，它不能大于validation_set的图像个数

输入数据layout
==============
当输入数据为3通道彩色图像时，input_data_layout可取NCHW或者NHWC。而当通道数不是3时，需要用numpy的npy文件保存图像，其必须是3维或者4维的uint8张量。
如果input_data_layout 取NonImage时，输入数据则只能是numpy的npy文件，其可以是1-4维的uint8张量。

