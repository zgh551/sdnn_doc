===========
License配置
===========

申请license
===========

由于SlimAI工具链需要安装license，否则无法进行编译。相关license获取及使用方法可以联系我司FAE人员，并提供如下信息：

- **软件环境**：Linux Ubuntu 版本
- **用户名**：中英文都可
- **MAC地址**：获取Server(主机) MAC 地址
- **HostName**：获取系统的主机名

.. note::

   #. MAC地址对应实际运行主机的地址，并非Docker容器内获取的地址。
   #. HostName可以通过 ``cat /etc/hostname`` 命令获得。
   #. 主机环境建议使用 **ubuntu** 系统

安装license
===========

阅读文档
--------

拿到License文件，请参考文件包中 **install instruction linux.pdf** 文档把环境搭建好，文件包目录结构如下：

.. code-block:: bash
   :linenos:

   ├── check_license.sh
   ├── install instruction linux.pdf
   ├── license.readme
   ├── licserv_linux_x64_v11_15
   │             └── x64_lsb
   │                 ├── lmgrd
   │                 ├── lmutil
   │                 ├── xtensad
   │                 └── xtensa_lic_test-linux-64
   ├── start_license.sh
   ├── xtensa.lic
   └── xtensa.log

执行安装
--------

启用普通用户权限，执行如下命令:

.. code-block:: bash

   $ ./start_license.sh


执行安装脚本后，将会生成 **xtensa.log** 文件。

检查license
===========

执行 ``check_license.sh`` 脚本，获取 **license** 安装状态。

.. code-block:: shell
   :linenos:

   $ ./check_license.sh

安装成功
--------

执行检查脚本后，如果输出如下类似信息则表示安装成功：

.. code-block:: bash

   Feature                Version    #license    Vendor    Expires
   ------------------------------------------------------------------
   Xt_ISS_BASE_E2B6056E   14.0          1        xtensad  21-jul-2022
   Xt_XCC_FUSA_E2B6056E   14.0          1        xtensad  21-jul-2022
   Xt_XPLORER_SE          14.0          1        xtensad  21-jul-2022

安装失败
--------

如果返回如下信息，则说明license安装失败，请查看 ``xtensa.log`` 文件，里面有详细的错误提示。

.. code-block:: bash

   Feature                Version    #license    Vendor    Expires
   ------------------------------------------------------------------

.. note::

   1. 确认PC状态是否发生变化，如 **MAC地址** 和 **HOST Name** 是否与申请license时一致；
   2. 确认本机端口号是否被其它应用占用；

端口占用问题
^^^^^^^^^^^^

如果 ``xtensa.log`` 文件显示如下信息，则说明license服务的端口已经被占用，无法打开。

.. code-block:: bash

   Failed to open the TCP port number in the license

此时可以通过如下指令查询占用端口的进程：

.. code-block:: bash

   $ netstat -natp | grep [port]


``kill`` 相关进程 **ID** ,重新执行 ``start_license.sh`` 脚本。