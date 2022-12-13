============
应用程序开发
============

----
概述
----

如果需要开发自定义的应用程序，则需要调用模型推理相关的API接口。关于推理API接口的详细使用方法，请获取 **examples** 仓库代码，里面包含详尽的接口说明。目前提供 ``TVM API`` 和 ``SDNN API`` 两套接口，可以根据实际场景选择合适的API接口。

.. _examples:

--------
examples
--------

概述
====

examples代码库包含模型推理应用程序的源码，请仔细阅读代码和注释，了解 ``API`` 接口如何使用，以及如何在不同的目标设备上部署模型等。

获取源码
========

获取examples代码，代码仓库地址请从 `客户支持系统 <https://support.semidrive.com/account/login>`_  上获取。

编译APP
=======

使用example进行示例编译前，需要进入examples的根目录，进行环境变量的配置：

.. code-block:: bash

   $ source envsetup.sh


然后进入examples/apps目录，调用build_app.sh脚本进行app的编译，例如，编译aarch64-linux平台的应用，命令如下：

.. code-block:: bash

   $ ./build_app.sh -p aarch64-linux -d app_api/sdnn_api/