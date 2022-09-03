==================
SlimAI固件路径配置
==================

linux
-----

SDNN编译生成的SlimAI固件(.elf)在Linux系统中通过kernel的firmware机制来加载。SDNN runtime默认其放在 ``/lib/firmware`` 目录。

Linux默认按以下顺序搜索rootfs里的路径：

.. code-block:: shell

   /lib/firmware/updates/UTS_RELEASE/
   /lib/firmware/updates/
   /lib/firmware/UTS_RELEASE/
   /lib/firmware/

若需要修改默认路径，可以通过如下方法修改fw_path_para参数（不超过256个字符）来实现：

1. Linux boot cmdline里设置此参数，如在dts里bootargs中加上:

.. code-block:: bash

   firmware_class.path= /path/to/firmware

2. 系统运行时通过shell配置：

.. code-block:: bash

   $ echo -n "/path/to/firmware" > /sys/module/firmware_class/parameters/path

通过上述配置后，通过 **sdnn_build** 编译的模型固件文件elf，即可放到 ``/path/to/firmware`` 路径。
