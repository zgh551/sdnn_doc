===========
License配置
===========

申请license
===========

由于SlimAI工具链需要安装license，否则无法进行编译。相关license获取及使用方法可以联系我司FAE人员，并提供如下信息：

- **软件环境**：Linux Ubuntu 版本
- **用户名**：中英文都可
- **MAC地址**：获取Server(主机) MAC 地址
- **hostname**：获取系统的主机名

.. note::

	#. MAC地址对应实际运行主机的地址，并非Docker容器内获取的地址。
	#. hostname可以通过 ``cat /etc/hostname`` 命令获得。
	#. 主机环境建议使用 **ubuntu** 系统

安装license
===========

拿到License文件，请参考文件包中**install instruction linux.pdf**文档把环境搭建好，文件包目录结构如下：

.. sidebar:: Tips

	执行 ``start_license.sh`` 文件将会生成 **xtensa.lic**。


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


检查license
===========

执行 ``check_license.sh`` ，获取 **license** 安装状态。

.. code-block:: shell
	:linenos:

	$ ./check_license.sh 

如下所示代表安装成功：

.. code-block:: bash

	Feature                Version    #license    Vendor    Expires
	------------------------------------------------------------------
	Xt_ISS_BASE_E2B6056E   14.0          1        xtensad  21-jul-2022
	Xt_XCC_FUSA_E2B6056E   14.0          1        xtensad  21-jul-2022
	Xt_XPLORER_SE          14.0          1        xtensad  21-jul-2022

