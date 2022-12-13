
==================
Slimai运行地址修改
==================

------------------
查看开发板内存配置
------------------

可以通过如下命令查看开发板上的实际加载地址：

.. code-block:: bash

   $ cat /sys/kernel/debug/xvp0/stdout

或者查看编译pac包的源码定位地址修改情况，例如x9_high_ms_serdes的板子，根据vdsp的内存空间在kernel的dts里设定，其kernel中的memmap配置位于 ``include/dt-bindings/memmap/x9_high/projects/ms_serdes/image_cfg.h`` 文件中，文件中默认分配的vdsp内存空间为：

.. code-block:: c++

   VDSP_MEMBASE=0x40a00000
   VDSP_MEMSIZE=0x03000000

假设现在用户将基地址修改如下:

.. code-block:: c++

   VDSP_MEMBASE=0x41000000
   VDSP_MEMSIZE=0x03000000

则需要在semidrive提供的sdnn开发环境里修改vdsp的链接脚本(**lds文件**)，从而编译生成可在 ``0x41000000`` 运行的elf文件。

--------------
docker容器修改
--------------

**lds文件** 位于docker容器的 ``/opt/toolchains/XtDevTools/install/builds/RI-2021.7-linux/vision_dsp/xtensa-elf/lib/lsp_semidrive`` 目录，如果该目录下存在 ``mpu_table.c`` 文件则需要删除该文件。

修改memmap.xmm文件
==================

文件中默认的空间为 ``0x40a00000 - 0x43a00000`` ，文件详细信息如下：

.. code-block:: SQL
   :linenos:

   VECBASE = 0x40a00800
   VECSELECT = 1
   VECRESET = 0x40a00000

   BEGIN dram1
   0xa00000: dataRam : dram1 : 0x20000 : writable ;
    dram1_0 : C : 0xa00000 - 0xa1ffff : .dram1.rodata .dram1.data .dram1.bss;
   END dram1

   BEGIN dram0
   0xa20000: dataRam : dram0 : 0x20000 : writable ;
    dram0_0 : C : 0xa20000 - 0xa3ffff : .dram0.rodata .dram0.data .dram0.bss;
   END dram0

   BEGIN sram
   0x40a00000: sysram : sram : 0x03000000 : executable, writable ;
    sram0 : F : 0x40a00000 - 0x40a0032f : .ResetVector.text .ResetHandler.literal .ResetHandler.text .MemoryExceptionVector.literal;
    sram1 : F : 0x40a00330 - 0x40a007ff : .MemoryExceptionVector.text;
    sram2 : F : 0x40a00800 - 0x40a0097f : .WindowVectors.text .Level2InterruptVector.literal;
    sram3 : F : 0x40a00980 - 0x40a009bf : .Level2InterruptVector.text .DebugExceptionVector.literal;
    sram4 : F : 0x40a009c0 - 0x40a009ff : .DebugExceptionVector.text .NMIExceptionVector.literal;
    sram5 : F : 0x40a00a00 - 0x40a00a3f : .NMIExceptionVector.text .KernelExceptionVector.literal;
    sram6 : F : 0x40a00a40 - 0x40a00a7f : .KernelExceptionVector.text .UserExceptionVector.literal;
    sram7 : F : 0x40a00a80 - 0x40a00aff : .UserExceptionVector.text .DoubleExceptionVector.literal;
    sram8 : F : 0x40a00b00 - 0x439fffff :  STACK :  HEAP : .DoubleExceptionVector.text .sram.rodata .clib.rodata .rtos.rodata .rodata .sram.literal .literal .rtos.literal .clib.literal .sram.text .text .clib.text .rtos.text .clib.data .clib.percpu.data .rtos.percpu.data .rtos.data .sram.data .data __llvm_prf_names .clib.bss .clib.percpu.bss .rtos.percpu.bss .rtos.bss .sram.bss .bss;
   END sram
   // Following lines are added by XNNC compiler to standard sim LSP
   PLACE SECTIONS(.dram0.rodata .dram0.literal .dram0.data .dram0.bss) IN_SEGMENT(dram1_0)
   PLACE SECTIONS(.dummy.nodata_in_dram0) IN_SEGMENT(dram0_0)
   PLACE SECTIONS(STACK) IN_SEGMENT(dram0_0)

根据开发板的运行地址配置，需要将起始地址修改为 ``0x41000000`` ，大小 0x03000000 保持不变。因为dram0和dram1是vdsp内部的ram，地址固定，无需修改，只要修改入口地址和sram区域：

.. code-block:: SQL
   :linenos:

   VECBASE = 0x41000800
   VECSELECT = 1
   VECRESET = 0x41000000

   BEGIN sram
   0x41000000: sysram : sram : 0x03000000 : executable, writable ;
    sram0 : F : 0x41000000 - 0x4100032f : .ResetVector.text .ResetHandler.literal .ResetHandler.text .MemoryExceptionVector.literal;
    sram1 : F : 0x41000330 - 0x410007ff : .MemoryExceptionVector.text;
    sram2 : F : 0x41000800 - 0x4100097f : .WindowVectors.text .Level2InterruptVector.literal;
    sram3 : F : 0x41000980 - 0x410009bf : .Level2InterruptVector.text .DebugExceptionVector.literal;
    sram4 : F : 0x410009c0 - 0x410009ff : .DebugExceptionVector.text .NMIExceptionVector.literal;
    sram5 : F : 0x41000a00 - 0x41000a3f : .NMIExceptionVector.text .KernelExceptionVector.literal;
    sram6 : F : 0x41000a40 - 0x41000a7f : .KernelExceptionVector.text .UserExceptionVector.literal;
    sram7 : F : 0x41000a80 - 0x41000aff : .UserExceptionVector.text .DoubleExceptionVector.literal;
    sram8 : F : 0x41000b00 - 0x43ffffff :  STACK :  HEAP : .DoubleExceptionVector.text .sram.rodata .clib.rodata .rtos.rodata .rodata .sram.literal .literal .rtos.literal .clib.literal .sram.text .text .clib.text .rtos.text .clib.data .clib.percpu.data .rtos.percpu.data .rtos.data .sram.data .data __llvm_prf_names .clib.bss .clib.percpu.bss .rtos.percpu.bss .rtos.bss .sram.bss .bss;
   END sram

sram每一段的格式为 ``start - end`` ，每一段的大小一般情况下不用修改。

修改完毕后保存文件，并在当前目录下运行如下命令：
   
.. code-block:: bash

   $ xt-genldscripts -b .
   New linker scripts generated in ./ldscripts

最后使用sdnn工具重新编译模型即可。
