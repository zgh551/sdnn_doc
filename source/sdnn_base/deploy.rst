============
部署环境配置
============

可以通过 `客户支持系统 <https://www.semidrive.com/>`_ ，获取相应系统的runtime库文件，也可通过如下链接点击下载。

.. tabs::

   .. tab:: Linux

      1. `linux_runtime库 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/aarch64-linux.tgz>`_
      2. `linux_opencv库 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/opencv_deploy_linux.run>`_

   .. tab:: Android

      1. `环境配置脚本 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/android_envsetup.sh>`_
      2. `android_libcpp库 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/libcpp_shared_android.run>`_
      3. `android_runtime库 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/aarch64-android.tgz>`_
      4. `android_opencv库 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/opencv_deploy_android.run>`_

   .. tab:: QNX

      1. `qnx_runtime库 <https://gitee.com/zgh551/sdnn_doc/releases/download/2.2.3/aarch64-qnx.tgz>`_

.. important::

   #. 部署环境中的runtime库版本一定要与sdnn_build工具版本一致，不要新旧版本混淆使用，否则模型推理会出现不可预知的错误；
   #. 每次升级一次sdnn_build工具，板子上的runtime库建议也同时升级；

Linux环境
=========

mount操作
---------

因为一些依赖库需要部署到系统目录，当板子启动后，如果需要部署依赖库，先确保当前用户具备系统根目录的读写权限，具体操作如下：

.. code-block:: bash

   $ mount -o remount,rw /

runtime库部署
-------------

首先确认目前开发的板子上是否包含该库文件，一般搜索路径如下：

- ``/usr/lib``
- ``/usr/local/lib``
- ``/usr/sdrv/tvm``


已部署过runtime库情况
^^^^^^^^^^^^^^^^^^^^^

如果板子上已有runtime库，且升级了sdnn_build工具，则需要更新板上的runtime库版本，该版本要与sdnn_build工具版本一致。此时只需要下载对应系统的libtvm_runtime.so库文件，覆盖已有的库文件即可。

未部署过runtime库情况
^^^^^^^^^^^^^^^^^^^^^

自动部署包
""""""""""

下载 **tvm_runtime_linux-deploy.run** 包，放到板子上任意路径，执行下述命令，可实现库的部署：

.. code-block:: bash

   $ sh ./tvm_runtime_linux-deploy.run

上述操作后，**libtvm_runtime.so** 库文件将部署到 ``/usr/sdrv/tvm`` 目录。


手动部署库
""""""""""

下载libtvm-runtime.so库文件，然后手动将其放置到目标路径。为了保证程序运行时能够搜索到该运行库，需要手动配置库文件的加载路径。

1. **ldconfig配置**

确认 ``/etc`` 目录下存在 ``ld.so.conf`` 文件，如果不存在则创建该文件，并在文件内写入下述内容：

.. code-block:: bash

   include /etc/ld.so.conf.d/*.conf

确认是否存在 ``/etc/ld.so.conf.d`` 目录，如果不存在则创建，然后在该目录下创建xxx.conf文件，在该文件中写入库文件的绝对路径。
最后运行如下命令刷新加载路径：

.. code-block:: bash

   $ ldconfig

2. **环境变量配置**

通过环境变量 **LD_LIBRARY_PATH** 指定库的路径，其命令如下：

.. code-block:: bash

   $ export LD_LIBRARY_PATH=path/to/your_lib_path

opencv部署
----------

目前提供的测试程序依赖于opencv库，如果板子上未部署opencv库，可以从 ``examples/vendor/OpenCV`` 目录下拷贝部署包到目标板，执行如下命令实现库的部署：

.. code-block:: bash

   $ sh ./opencv_deploy_linux.run

上述操作后，**opencv** 库将会部署到 ``/usr/sdrv/opencv`` 目录。

Android环境
===========

adb环境配置
-----------

确认USB先插入adb接口，则通过abd工具执行如下操作：

root操作
^^^^^^^^

.. code-block:: bash

   $ adb root

mount操作
^^^^^^^^^

.. code-block:: bash

   $ adb remount

进入shell
^^^^^^^^^

.. code-block:: bash

   $ adb shell

环境变量配置
------------

从 ``examples/vendor/Android`` 目录拷贝 **android_envsetup.sh** 脚本到板子上任意路径，执行下述命令，完成环境变量初始化。

.. code-block:: bash

   $ source android_envsetup.sh

C++库部署
---------

查看 ``vendor/lib64`` 目录下是否存在 **libc++_shared.so** 文件，如果不存在，则从 ``examples/vendor/Android`` 目录拷贝 **libc++_shared_android.run** 部署文件，执行下述命令：

.. code-block:: bash

   $ sh libc++_shared_android.run

runtime库部署
-------------

首先确认目前开发的板子上是否包含该库文件，一般搜索路径如下：

- ``/vendor/lib``
- ``/vendor/lib64``
- ``/vendor/sdrv/tvm``

已部署过runtime库情况
^^^^^^^^^^^^^^^^^^^^^

如果板子上已有runtime库，且升级了sdnn_build工具，则需要更新板上的runtime库版本，该版本要与sdnn_build工具版本一致。此时只需要下载对应系统的libtvm_runtime.so库文件，覆盖已有的库文件即可。

未部署过runtime库情况
^^^^^^^^^^^^^^^^^^^^^

自动部署包
""""""""""

下载tvm_runtime_android-deploy.run包，放到板子上任意路径，执行下述命令，可实现库的部署：

.. code-block:: bash

   $ sh ./tvm_runtime_linux-deploy.run

上述操作后，**libtvm_runtime.so** 库文件将部署到 ``/vendor/sdrv/tvm`` 目录。

手动部署库
""""""""""

下载libtvm-runtime.so库文件，然后手动将其放置到 ``/vendor/lib64/`` 目录。如果想指定到其它路径，可以通过环境变量 **LD_LIBRARY_PATH** 指定库的路径，其命令如下：

.. code-block:: bash

   $ export LD_LIBRARY_PATH=path/to/your_lib_path

opencv部署
-------------

目前提供的测试程序依赖于opencv库，如果板子上未部署opencv库，可以从 ``examples/vendor/OpenCV`` 目录下拷贝部署包到目标板，执行如下命令实现库的部署：

.. code-block:: bash

   $ sh ./opencv_deploy_android.run

上述操作后，opencv库将会部署到 ``/vendor/sdrv/opencv`` 目录。

QNX环境
=======

runtime库部署
-------------

拷贝 **libtvm_runtime.so** 库到 ``/proc/boot/`` 目录完成运行库的更新。

其它环境
========

如果板子上运行的环境不满足上述系统环境，或者所使用的工具链与预编译的库不一致，为了避免兼容性问题，需要获取sdnn源码，编译新的进runtime库。否则直接从客户支持系统下载已经编译好的runtime库，可跳过该章节内容阅读。

系统环境
--------

目前支持linux、android和qnx系统，如果板子上运行的系统不满足上述三种，则需要源码编译运行库。

工具链
------

- **Linux**

目前linux系统使用的交叉编译工具链是aarch64-gcc7.5 ，如果使用的是其它版本的gcc，可以考虑源码编译；

- **Android**

目前android系统使用的aarch64-linux-android29工具链编译，如果实际使用的NDK不一致，可以考虑源码编译。

- **QNX**

目前qnx是的编译工具是qcc8.3版本，如果使用的不一致，可以考虑源码编译。

获取SDNN源码
------------

登录客户支持系统下载sdnn源码压缩包，解压后便可得到编译所需的源码文件。

编译SDNN运行库
--------------

进入TVM代码根目录(如图中示例为/workspace/tvm)，依以下步骤逐步操作：

环境变量配置
^^^^^^^^^^^^

执行下述命令设置环境变量：

.. code-block:: bash

   $ source envsetup.sh

运行库编译
^^^^^^^^^^

根据部署平台的系统类型，指定目标平台部署包的编译选项：

1. **linux** 系统编译命令

.. code-block:: bash

   $ ./build.sh linux-deploy

编译完后生成 ``build_aarch64-linux`` 目录，部署包 **tvm_runtime_linux-deploy.run** 存在于此目录，也可以直接拷贝 **libtvm_runtime.so** 文件到板子上可加载的目录。

2. **android** 系统编译命令

.. code-block:: bash

   $ ./build.sh android-deploy

编译完成后部署包 **tvm_runtime_android-deploy.run** 生成在 ``build_aarch64-android`` 目录。后续把 **tvm_runtime_xxx-deploy.run** 文件拷贝到目标平台系统下任意文件夹。在文件所在目录执行以下指令：

.. code-block:: bash

   $ chmod 775 tvm_runtime_xxx-deploy.run
   $ ./tvm_runtime_xxx-deploy.run

其中 ``xxx`` 表示不同平台，执行完以上命令后，linux下则生成 ``/usr/sdrv/tvm`` 目录，android下则生成 ``/vendor/sdrv/tvm`` 目录 **libtvm-runtime.so** 被拷贝安装到此目录下，至此tvm target端部署完毕。

3. **qnx** 系统编译命令

.. code-block:: bash

   $ ./build.sh qnx-deploy

编译完成后在 ``build_aarch64-qnx`` 目录生成 **libtvm_tuntime.so** 文件。后续需要把该文件放到sd卡/u盘或者打包到ifs中 ``/proc/boot`` 目录。如果使用sd卡或者u盘挂载方式部，需要添加 **libtvm_runtime.so** 路径到 **LD_LIBRARY_PATH** 环境变量。

.. code-block:: bash

   $ export LD_LIBRARY_PATH=/path/to/sdcard-mount-point:$LD_LIBRARY_PATH

