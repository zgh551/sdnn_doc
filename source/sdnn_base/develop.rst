.. _develop_prepare:

============
开发环境准备
============

----
概述
----

本章节介绍了模型编译部署前所需的准备工作，主要涉及开发机端docker环境的配置和开发板端相关依赖库的部署。

--------
开发机端
--------

软硬件准备
==========

操作系统
--------

操作系统方面理论上只要支持Docker安装，都能作为开发机使用。推荐使用Linux系统，例如ubuntu 18.04等。

硬件指标
--------

首先确认开发用的个人电脑或服务器的硬件配置至少满足以下条件，否则会影响开发体验。

+------+------------------------------+
| 硬件 | 需求                         |
+======+==============================+
| CPU  | Intel i3或以上; AMD r3或以上 |
+------+------------------------------+
| 内存 | 16G或以上                    |
+------+------------------------------+

.. note::

   建议CPU选用单核性能较高的芯片型号。

Docker安装
==========

用户需要在本地开发机上安装Docker软件服务, 详细安装流程请参考官方 `Docker安装指南 <https://docs.docker.com/get-docker>`_ 。


SDNN镜像导入
============

本地开发机安装好Docker服务后需要导入SDNN Docker镜像，请通过 `客户支持系统 <https://support.semidrive.com/account/login>`_ 下载最新镜像包。

解压镜像包
----------

由于镜像体积较大，客户支持系统上发布的镜像做了压缩处理，故拿到压缩包后先进行解压操作，Linux系统下解压命令如下：

.. code-block:: bash
   :linenos:

   $ tar zxvf sdnn_docker_{版本号}.tgz

执行完解压命令后，当前目录下会生成对应的 ``img`` 包:

.. code-block:: shell
   :linenos:

   $ ls *.img

   sdnn_docker_{版本号}.img

加载镜像包
----------

执行 ``docker load`` 命令可加载镜像，具体命令如下:

.. code-block:: shell
   :linenos:

   $ docker load -i sdnn_docker_{版本号}.img

.. note::

   由于镜像体积较大，加载过程中会消耗系统大量内存，如果开发机的内存较小，可能会出现系统假死的现象。

导入完成后通过 ``docker images`` 查看镜像是否导入成功:

.. code-block:: shell
   :linenos:

   $ docker images

   REPOSITORY      TAG           IMAGE ID       CREATED        SIZE
   sdnn            v3.0.0        756501bfa3c0   17 hours ago   24.3GB

.. _sdnn_dev_container:

创建SDNN开发容器
================

通过 ``docker run`` 创建容器对象，如果是在服务器上开发，建议每个开发者都创建一个属于自己的容器。

.. code-block:: shell
   :linenos:

   $ docker run -it --rm \
            --name ${容器名} \
            -v ${挂载开发机中的目录}:${容器内所映射的目录} \
            -e ${容器中的环境变量} \
            ${镜像名}：${标签名}  /bin/bash

简单示例如下：

.. code-block:: shell
   :linenos:

   $ docker run -it --name sdnn_$USER -v ${PWD}:$HOME sdnn:v3.0.0 /bin/bash

SlimAI 开发容器配置
-------------------

如果部署的是 ``slimai`` 加速设备，则需要添加 **XTENSAD_LICENSE_FILE** 环境变量，用于配置 slimai 工具链的license服务器。请参考 :ref:`license_config` 章节在 **开发机** 上开启license服务器，可以从 **License server status** 字段获取主机名(IP)和端口信息。关于环境变量添加的示例如下：

.. code-block:: shell
   :linenos:

   $ docker run -it \
            --name sdnn_$USER \
            -v ${PWD}:$HOME \
            -e XTENSAD_LICENSE_FILE="27030@10.18.10.241" \
            sdnn:v3.0.0 /bin/bash

.. note::

   建议环境变量的配置使用IP地址而不用主机名，主机名有时会存在无法解析的问题。

QNX 开发容器配置
----------------

由于QNX工具链的license限制，无法在sdnn镜像中预先装载qnx的工具链，需要用户通过挂载的方式将开发机本地的qnx工具链挂载到容器相应目录( ``/opt/toolchains/qnx710`` )。关于qnx工具链的挂载示例如下：

.. code-block:: shell
   :linenos:

   $ docker run -it \
            --name sdnn_$USER \
            -v $PWD:$HOME \
            -v {开发机qnx工具链的绝对路径}:/opt/toolchains/qnx710
            sdnn:v3.0.0 /bin/bash


其它Docker操作
==============

退出容器
--------


可以通过快捷键 :kbd:`Ctrl+d` 或 输入 ``exit`` 退出容器。如果创建容器时添加了 ``--rm`` 参数，则容器直接关闭，会释放所有资源。如果未添加该选项，则容器只是暂时退出，处于 **Exited** 状态，通过 ``docker ps -a`` 命令可以查询到该容器。


启动容器
--------

如果容器处于 **Exited** 状态, 可以执行下述命令将容器从 **Exited** 状态切换到 **Up** 状态：

.. code-block:: bash

   $ docker start ${容器名}


重新进入容器
------------

当容器处于 **Up** 状态时, 可以通过如下命令重新进入容器：

.. code-block:: shell

   $ docker exec -it {容器名} /bin/bash

.. note::

   对于容器的其它相关命令参见：https://docs.docker.com/engine/reference/commandline/cli/

.. _develop_board:

--------
开发板端
--------

概述
====

请根据开发板端所运行的操作系统，通过 `客户支持系统 <https://support.semidrive.com/account/login>`_ ，获取对应的runtime库和部署文件。

.. important::

   #. 部署环境中的runtime库版本一定要与sdnn工具版本一致，不要新旧版本混淆使用，否则模型推理会出现不可预知的错误；
   #. 每次升级sdnn工具，板子上的runtime库建议也同步升级；

Linux环境
=========

mount操作
---------

因为一些依赖库需要部署到系统目录，当板子启动后，如果需要部署依赖库，先确保当前用户具备系统根目录的读写权限，具体操作如下：

.. code-block:: bash

   $ mount -o remount,rw /

依赖库部署
----------

首先确认目前开发的板子上是否包含该库文件，一般搜索路径如下：

- ``/usr/lib``
- ``/usr/local/lib``

自动部署包
**********

使用 **sdnn_linux_deploy.run** 部署包，将其放到板子上任意路径，执行下述命令，可实现库的自动部署：

.. code-block:: bash

   $ sh ./sdnn_linux_deploy.run

上述操作后，**libtvm_runtime.so** 和 **opencv** 库都将自动部署到 ``/usr/lib`` 目录。

手动部署库
**********

将相关依赖库放置到 ``/usr/lib`` 目录即可完成部署。如果需要设置自定义的路径，为了保证程序运行时能够搜索到该运行库，需要手动配置库文件的加载路径。

1. **ldconfig方式**

确认 ``/etc`` 目录下存在 ``ld.so.conf`` 文件，如果不存在则创建该文件，并在文件内写入下述内容：

.. code-block:: bash

   include /etc/ld.so.conf.d/*.conf

确认是否存在 ``/etc/ld.so.conf.d`` 目录，如果不存在则创建，然后在该目录下创建xxx.conf文件，在该文件中写入依赖库文件所要存放的自定义路径：

.. code-block:: bash

   $ echo "path/to/your_lib_path" > xxx.conf

最后运行如下命令刷新加载路径：

.. code-block:: bash

   $ ldconfig

2. **环境变量方式**

通过环境变量 **LD_LIBRARY_PATH** 指定库的自定义路径，其命令如下：

.. code-block:: bash

   $ export LD_LIBRARY_PATH=path/to/your_lib_path:$LD_LIBRARY_PATH

Android环境
===========

adb环境配置
-----------

确认USB先插入adb接口，然后通过 ``abd`` 工具执行如下操作：

root操作
********

.. code-block:: bash

   $ adb root

mount操作
*********

.. code-block:: bash

   $ adb remount

进入shell
*********

.. code-block:: bash

   $ adb shell

依赖库部署
----------

首先确认目前开发的板子上是否包含该库文件，一般搜索路径如下：

- ``/vendor/lib``
- ``/vendor/lib64``

自动部署包
**********

使用 **sdnn_android_deploy.run** 部署包，将其放到板子上任意路径，执行下述命令，可实现库的自动部署：

.. code-block:: bash

   $ sh ./sdnn_android_deploy.run

上述操作后，**libtvm_runtime.so** 、 **opencv** 和 **libc++_shared** 库都将自动部署到 ``/vendor/lib64`` 目录。

手动部署库
**********

将相关依赖库放置到 ``/vendor/lib64`` 目录即可完成部署。如果需要设置自定义的路径，可以通过环境变量 **LD_LIBRARY_PATH** 指定，其命令如下：

.. code-block:: bash

   $ export LD_LIBRARY_PATH=path/to/your_lib_path:$LD_LIBRARY_PATH

QNX环境
=======

runtime库部署
-------------

请先确认micro-usb插入adb接口，然后通过 ``abd`` 工具将开发机上的 ``libtvm_runtime.so`` 库推送到 *ap1* 端的 ``/share`` 目录:

.. code-block:: bash

   $ adb push libtvm_runtime.so /share

然后进入 *ap1* 端的 `shell`:

.. code-block:: bash

   $ adb shell

在 *ap1* 端的shell窗口输入如下命令，进入 *ap2* 端的shell:

.. code-block:: bash

   $ sdshell ap2

在 *ap2* 端配置动态库搜索路径，将 ``libtvm_runtime.so`` 库的路径加入搜索范围：

.. code-block:: shell

   $ export LD_LIBRARY_PATH=/nfs:$LD_LIBRARY_PATH

通过上述步骤，完成了 ``libtvm_runtime.so`` 库的部署。