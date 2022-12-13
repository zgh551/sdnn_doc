====================
Slimai资源利用率统计
====================

--------
使用说明
--------

debugfs增加文件/sys/kernel/debug/xvp0/stat供应用程序查看，执行如下指令可以查看统计内容：

.. code-block:: bash

   $ cat /sys/kernel/debug/xvp0/stat
   run(ms) 2000
   cnt 40
   max_time(ms) 55
   interval_time(s) 5
   loading 40

统计参数说明
============

1. 运行时间：统计时间段内slimAI运行的总时间，每次运行从发cmd给slimAI开始算起，一直到驱动收到slimAI完成后发来的中断为止，精度为ms
2. 运行次数：统计时间段内slimAI运行的总次数
3. 最大运行时间：统计时间段内slimAI运行最长的一次时间，单位为ms
4. 统计间隔时间：统计时间段的长度，默认为5s，可配置
5. 负载：运行时间/统计间隔时间x100%

----------------
配置统计时间间隔
----------------

通过/sys/module/xrp/parameters/running_period配置，例如：

.. code-block:: bash

   echo 10 > /sys/module/xrp/parameters/running_period

将时间间隔设为10s