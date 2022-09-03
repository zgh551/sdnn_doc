==========
Docker配置
==========

目前Docker是主要的SDNN开发环境，包括如下内容：

- 开发应用必须的软件包
- 目标设备相关工具链
- 不同系统的工具链

Docker安装
==========

用户需要在本地 **PC** 环境安装Docker环境, 对于PC环境下Docker的安装请参见：https://docs.docker.com/get-docker/

.. note::

   本地PC已验证环境为Ubuntu 18.04.5 LTS。

导入镜像
==========

本地PC环境安装好Docker环境后需要导入SDNN Docker镜像，通过 `客户支持系统 <https://www.semidrive.com/>`_ 下载最新镜像包。

Docker环境由多个镜像组合而成，用户需要根据自己的部署目标环境及可使用的网络计算加速器，下载所需的镜像模块，逐个解压并导入镜像，最后创建并组合容器完成环境准备，下面以SlimAI为网络加速器、部署目标环境为linux情况示意环境准备过程。

首先下载需要的镜像文件，镜像文件由基础环境、加速器工具链及操作系统工具链3大部分组成。假设加速器为SlimAI、操作系统为linux，这3大部分如下:

#. 基础环境: ``sdrv_tvm_base_1_0.zip``
#. 加速器工具链: ``xnnc_2.4.zip`` 和 ``xtensa_2021_7.zip``
#. 操作系统工具链: ``linux_env_yocto_2_5_3.zip``

.. note::

   目前支持 **linux** 、 **android** 和 **qnx** 三类操作系统的模型部署，需要根据实际板子运行的操作系统选择相应的镜像包下载。


解压镜像文件
-------------

通过如下命令解压已下载的docker镜像文件。

.. code-block:: bash
   :linenos:

   $ unzip sdrv_tvm_base_1_0.zip
   $ unzip xnnc_2_4.zip
   $ unzip xtensa_2021_7.zip
   $ unzip linux_env_yocto_2_5_3.zip


执行完解压命令后，当前目录下会产生对应的tar包:

.. code-block:: shell
   :linenos:

   $ ls *.tar
   linux_env_yocto_2_5_3.tar  sdrv_tvm_base_1_0.tar  xnnc_2_4.tar  xtensa_2021_7.tar

加载镜像文件
-------------

执行 ``docker load`` 命令可导入镜像 **tar** 包:

.. code-block:: shell
   :linenos:

   $ docker load < xnnc_2_4.tar
   $ docker load < xtensa_2021_7.tar
   $ docker load < linux_env_yocto_2_5_3.tar
   $ docker load < sdrv_tvm_base_1_0.tar

导入完成后通过 ``docker images`` 查看镜像是否导入成功:

.. code-block:: shell
   :linenos:

   $ docker images

   REPOSITORY      TAG           IMAGE ID       CREATED        SIZE
   tvm_base        1.0           0b2ffdde775d   17 hours ago   12.7GB
   xnnc            2.4           8e9ee914c256   2 days ago     1.7GB
   linux_env       yocto_2.5.3   6726aca913ff   9 days ago     4.82GB
   xtensa          2021.7        e4c0657ac1e8   2 weeks ago    10.2GB

创建容器
========

本地PC环境成功导入docker镜像包后，分两步完成运行环境容器创建。

volume容器创建
--------------

首先创建工具链volume容器，详细命令如下：

.. code-block:: shell
   :linenos:

   $ docker create --name xtensa_2021_7 xtensa:2021.7 /bin/sh
   $ docker create --name xnnc_2_4 xnnc:2.4 /bin/sh
   $ docker create --name linux_env_yocto_2_5_3 linux_env:yocto_2.5.3 /bin/sh

.. note::

   - volume容器的名字可以任意选取，下一步导入volume时，名字对应即可。
   - 多人开发共用volume容器时，如果一方修改工具链的内容，修改影响是全局的，所有引用该volume的开发容器都会受影响，所以建议多人开发时，每个人创建自己的volume容器。

SDNN开发环境容器创建
--------------------

创建SDNN开发容器时，需要导入工具链volume，并挂载本地工作文件夹，创建容器命令如下：

.. code-block:: shell

   docker run -it --rm
          --name ${容器名} \
          --volumes-from {volume容器名} \
          -v ${挂载主机文件夹}:${容器内被映射到的文件夹} \
          -e ${配置容器的环境变量} \
          ${仓库名}：${标签名}  /bin/bash \

具体示例如下：

.. code-block:: shell
   :linenos:

   $ docker run -it \
            --name tvm_$USER \
            -v ${PWD}:$HOME \
            --volumes-from xnnc_2_4 \
            --volumes-from xtensa_2021_7 \
            --volumes-from linux_env_yocto_2_5_3 \
            -e XTENSAD_LICENSE_FILE="27030@10.18.10.241" \
            -e XTENSA_SYSTEM="/opt/xtensa/XtDevTools/install/builds/RI-2021.7-linux/vision_dsp/config" \
            -e PATH="/sdrv/llvm-12.0.1-linux-gnu/bin:/sdrv/llvm-10.0.1/bin:/opt/xtensa/XtDevTools/install/tools/RI-2021.7-linux/XtensaTools/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
            -e XTENSA_CORE="vision_dsp" \
            -e XTENSA_CORE_TYPE="dsp" \
            tvm_base:1.0 /bin/bash

.. note::

   #. 容器名可以自定义，仓库名与标签名需要与导入到本地PC中镜像的仓库名与标签名一致。
   #. **XTENSAD_LICENSE_FILE** ="27030@10.18.10.241"是重要的环境变量的配置，仅 **XTENSAD_LICENSE_FILE** 需要根据用户实际环境更改，其它环境变量采用上述默认设置即可。
   #. "27030@10.18.10.241" 需要被替换为用户自己的license服务器名(IP)和端口。
   #. 参考 **章节1** 的license检查，执行./check_license.sh 获取License server status 内容可得用户自己的所需license配置值。


进入容器
========

创建容器后退出，可以用过下述步骤再次进入容器：

查看系统已经创建的容器
----------------------

.. code-block:: bash
   :linenos:

   $ docker ps -a
   7c5993971858   tvm_base:1.0            "/bin/bash"              3 weeks ago    Up 3 weeks                          tvm_base_linux_1_0_user
   009902f3ed26   qnx_env:710             "/bin/sh"                5 weeks ago    Created                             qnx_env_710
   e3a098b395a5   android_env:ndk_r23b    "/bin/sh"                5 weeks ago    Created                             android_env_ndk_r23b
   f37197b75f18   linux_env:yocto_2.5.3   "/bin/sh"                5 weeks ago    Created                             linux_env_yocto_2_5_3
   8aff9d99ef74   xnnc:2.4                "/bin/sh"                5 weeks ago    Created                             xnnc_2_4
   f2270c3a9439   xtensa:2021.7           "/bin/sh"                5 weeks ago    Created                             xtensa_2021_7

启动容器
--------

通过步骤一查看需要运行的容器ID。对于”STATUS”为”Exited”状态的容器，才需要执行此步骤，否则可以跳过此步骤。启动命令如下：

.. code-block:: bash

   $ docker start ${容器名}

进入容器
--------

.. code-block:: shell

   $ docker exec -it ${容器名} bash

.. note::

   对于容器的其它相关命令参见：https://docs.docker.com/engine/reference/commandline/cli/

QNX Docker配置
==============

由于QNX 开发需要license，因此SDNN不提供Docker image，由用户自己生成，步骤如下：

QNX Docker镜像生成与容器创建
----------------------------

镜像创建详见 ``path/to/tvm/docker/qnx_create_readme.txt`` 。容器创建与linux相似。

配置QNX license
---------------

创建完完整docker运行环境后，需基于命令行方式进行QNX工具链license安装，否则编译时提示“license check failed”。

.. code-block:: bash

   $ qnxsoftwarecenter/qnxsoftwarecenter_clt
   -url https://qnx.com/swcenter -syncLicenseKeys -myqnx.user=<MYQNX_USER>
   -myqnx.password=<MYQNX_PASSWORD> -addLicenseKey <build_server_license_key>
   -listLicenseKeys

``qnxsoftwarecenter`` 文件目录及MYQNX_USER，MYQNX_PASSWORD，build_server_license_key等请联系QNX FAE获取。