在 Windows 上安装 ESP-IDF-v4.0.1 
========================================

本章完整文档见 `这里`__。

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/get-started/index.html#get-started-get-prerequisites


本文档旨在指导用户搭建 ESP32 硬件开发的软件环境，

通过一个简单的示例展示如何使用 ESP-IDF (Espressif IoT Development Framework) 配置菜单，并编译、下载固件至 ESP32 开发板等步骤。


概述
---------

ESP32 SoC 芯片支持以下功能：

* 2.4 GHz Wi-Fi
* 蓝牙 4.2
* 高性能双核
* 超低功耗协处理器
* 多种外设
* ESP32 采用 40 nm 工艺制成，具有最佳的功耗性能、射频性能、稳定性、通用性和可靠性，适用于各种应用场景和不同功耗需求。

乐鑫为用户提供完整的软、硬件资源，进行 ESP32 硬件设备的开发。其中，乐鑫的软件开发环境 ESP-IDF 旨在协助用户快速开发物联网 (IoT) 应用，可满足用户对 Wi-Fi、蓝牙、低功耗等方面的要求。


准备工作
---------------

**硬件：**

* 一款 ESP32 开发板
* **USB 数据线** (A 转 Micro-B)
* PC（Windows、Linux 或 Mac OS）

**软件：**

* 设置 **工具链**，用于编译 ESP32 代码；
* **编译工具** —— CMake 和 Ninja 编译工具，用于编译 ESP32 **应用程序**；
* 获取 ESP-IDF 软件开发框架。该框架已经基本包含 ESP32 使用的 API（软件库和源代码）和运行 **工具链** 的脚本
* 安装 C 语言编程（**工程**）的 文本编辑器，例如 `Eclipse`__

.. __: https://www.eclipse.org/


开发板简介
------------------

请点击下方连接，了解有关具体开发板的详细信息。

* `ESP32-DevKitC`__
* `ESP-WROVER-KIT`__
* `ESP32-PICO-KIT`__
* `ESP32-Ethernet-Kit`__。

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/hw-reference/get-started-devkitc.html
.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/hw-reference/get-started-wrover-kit.html
.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/hw-reference/get-started-pico-kit.html
.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/hw-reference/get-started-ethernet-kit.html


.. _get-started-step-by-step:

详细安装步骤
-------------

请根据下方详细步骤，完成安装过程。

**设置开发环境**

* :ref:`get-started-get-prerequisites`。
* :ref:`get-started-get-esp-idf`
* :ref:`get-started-set-up-tools`
* :ref:`get-started-set-up-env`

**创建您的第一个工程**

* :ref:`get-started-start-project`
* :ref:`get-started-connect`
* :ref:`get-started-configure`
* :ref:`get-started-build`
* :ref:`get-started-flash`
* :ref:`get-started-build-monitor`


设置开发环境
-----------------


.. _get-started-get-prerequisites:

第一步：安装准备 — Windows 平台工具链的标准设置
**************************************************

本节完整文档见 `这里`__。 

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/get-started/windows-setup.html

.. note::

    目前，基于 CMake 的构建系统仅支持 64 位 Windows 版本。32 位 Windows 版本的用户可根据 `传统 GNU Make 构建系统`__ 中的介绍进行操作。*

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/get-started-legacy/windows-setup.html


工具链概述
###############

    ESP-IDF 需要安装一些必备工具，才能围绕 ESP32 构建固件，包括 Python、Git、交叉编译器、menuconfig 工具、CMake和 Ninja 编译工具等。

    在本入门指南中，我们通过 **命令提示符** 进行有关操作。不过，您在安装 ESP-IDF 后还可以使用 `Eclipse`__ 或其他支持 CMake 的图形化工具 IDE。

    .. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/get-started/eclipse-setup.html

    .. note::

        较早 ESP-IDF 版本使用 `传统 GNU Make 编译系统`__ 和 `MSYS2 Unix`__ 兼容环境。但如今已非必需，用户可直接通过 Windows 命令提示符使用 ESP-IDF。

    .. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/get-started-legacy/windows-setup.html
    .. __: https://msys2.github.io/


.. _get-started-windows-tools-installer:

ESP-IDF 工具安装器
########################

    要安装 ESP-IDF 必备工具，最简易的方式是下载 ESP-IDF 工具安装器，地址如下：

    `https://dl.espressif.com/dl/esp-idf-tools-setup-2.3.exe`__
        
    .. __: https://dl.espressif.com/dl/esp-idf-tools-setup-2.3.exe

    本安装器可为您安装所需的交叉编译器、OpenOCD、`cmake`__ 和 `Ninja`__ 编译工具，以及一款 `mconf-idf`__ 配置工具。此外，本安装器还可在有需要时下载、运行 `Python`__ 3.7 和 Git For Windows 的安装器。

    .. __: https://cmake.org/download/
    .. __: https://ninja-build.org/
    .. __: https://github.com/espressif/kconfig-frontends/releases/
    .. __: https://www.python.org/downloads/windows/

    本安装器还可用于下载任意 ESP-IDF 发布版本。

    本实例安装于 ``I:\esp32\esp-idf-v4.0.1``。


使用命令提示符
#####################

    在后续步骤中，我们将使用 Windows 的命令提示符进行操作。

    ESP-IDF 工具安装器可在 :guilabel:`开始` 菜单中，创建一个打开 ESP-IDF 命令提示符窗口的快捷方式。本快捷方式可以打开 Windows 命令提示符（即 cmd.exe），并运行 ``export.bat`` 脚本以设置各环境变量（比如 ``PATH``，``IDF_PATH`` 等）。此外，您可还以通过 Windows 命令提示符使用各种已经安装的工具。

    注意，本快捷方式仅适用 ESP-IDF 工具安装器中指定的 ESP-IDF 路径。如果您的电脑上存在多个 ESP-IDF（比如您需要不同的 ESP-IDF 版本）需要使用快捷方式，您可以：

    1. 为 ESP-IDF 工具安装器创建的快捷方式创建一个副本，并将新快捷方式的“当前路径”指定为您希望使用的 ESP-IDF 路径。
    2. 运行 ``cmd.exe``，并更新至您希望使用的 ESP-IDF 目录，然后运行 ``export.bat``。注意，这种方法要求 ``PATH`` 中存在 Python 和 Git。如果您在使用时遇到有关“找不到 Python 或 Git” 的错误信息，请使用第一种方法。


后续步骤
##################

    当 ESP-IDF 工具安装器安装完成后，则开发环境设置也到此结束。后续开发步骤，请前往 :ref:`get-started-start-project` 查看。


.. _get-started-get-esp-idf:


第二步：获取 ESP-IDF
*********************

.. note::

    在本文档中，Windows 操作系统的默认路径为 ``%userprofile%\esp``。您也可以将 ESP-IDF 安装在任何其他路径下，但请注意在使用命令行时进行相应替换。**注意，ESP-IDF 不支持带有空格的路径**。

在围绕 ESP32 构建应用程序之前，请先获取乐鑫提供的软件库文件 `ESP-IDF 仓库`__。

获取 ESP-IDF 的本地副本：打开终端，切换到您要保存 ESP-IDF 的工作目录，使用 ``git clone`` 命令克隆远程仓库。针对不同操作系统的详细步骤，请见下文。

.. note::

    Windows 操作系统下，除了安装必要工具外，第一步中介绍的 :ref:`get-started-windows-tools-installer` 也能同时下载 ESP-IDF 本地副本。**如果执行了第一步，您可以跳过这一步。**

请前往 `ESP-IDF 版本简介`__，查看 ESP-IDF 不同版本的具体适用场景。

除了使用 :ref:`get-started-windows-tools-installer`，您也可以参考 `指南`__ 手动下载 ESP-IDF。

.. __: https://github.com/espressif/esp-idf
.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/versions.html
.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/get-started/windows-setup-scratch.html#get-esp-idf-windows-command-line


.. _get-started-set-up-tools:

第三步：设置工具
*********************

除了 ESP-IDF 本身，您还需要安装 ESP-IDF 使用的各种工具，比如编译器、调试器、Python 包等。

Windows 操作系统下，请根据第一步中对 Windows (:ref:`get-started-windows-tools-installer`) 的介绍，安装所有必需工具。

除了使用 ESP-IDF 工具安装器，您也可以通过**命令提示符**窗口手动安装这些工具。具体步骤见下：

.. code:: batch

    cd %userprofile%\esp\esp-idf
    install.bat

本步骤中介绍的脚本将 ESP-IDF 所需的编译工具默认安装在用户根文件夹中，Linux 和 MacOS 系统为 ``$HOME/.espressif``，Windows 系统为 ``%USERPROFILE%\.espressif``。此外，您可以可以将工具安装到其他目录中，但请在运行安装脚本前，重新设置环境变量 ``IDF_TOOLS_PATH``。注意，请确保您的用户已经具备了读写该路径的权限。

如果修改了 ``IDF_TOOLS_PATH`` 变量，请确保该变量在每次执行 ``install.bat/install.sh`` 和 ``export.bat/export.sh`` 脚本时均保持一致。


.. _get-started-set-up-env:

第四步：设置环境变量
***********************

此时，您刚刚安装的工具尚未添加至 ``PATH`` 环境变量，无法通过“命令窗口”使用这些工具。因此，必须设置一些环境变量，这可以通过 ESP-IDF 提供的另一个脚本完成。

Windows 安装器 (:ref:`get-started-windows-tools-installer`) 可在 :guilabel:`开始` 菜单创建一个 “ESP-IDF Command Prompt” 快捷方式。该快捷方式可以打开命令提示符窗口，并设置所有环境变量。您可以点击该快捷方式，然后继续下一步。

此外，如果您希望在当下命令提示符窗口使用 ESP-IDF，请使用下方代码：

.. code:: batch

    %userprofile%\esp\esp-idf\export.bat


创建您的第一个工程
--------------------

.. _get-started-start-project:

第五步：开始创建工程
*********************

现在，您可以开始准备开发 ESP32 应用程序了。您可以从 ESP-IDF 中 `examples`__ 目录下的 `get-started/hello_world`__ 工程开始。

将 `get-started/hello_world`__ 复制至您本地的 ``~/esp`` 目录下：

.. code:: batch

    cd %userprofile%\esp
    xcopy /e /i %IDF_PATH%\examples\get-started\hello_world hello_world


ESP-IDF 的 `examples`__ 目录下有一系列示例工程，都可以按照上面的方法进行创建。您可以按照上述方法复制并运行其中的任何示例，也可以直接编译示例，无需进行复制。

.. __: https://github.com/espressif/esp-idf/tree/v4.0.1/examples
.. __: https://github.com/espressif/esp-idf/tree/v4.0.1/examples/get-started/hello_world
.. __: https://github.com/espressif/esp-idf/tree/v4.0.1/examples/get-started/hello_world
.. __: https://github.com/espressif/esp-idf/tree/v4.0.1/examples

.. important::
    
    ESP-IDF 编译系统不支持带有空格的路径。

.. _get-started-connect:

第六步：连接设备
*********************

现在，请将您的 ESP32 开发板连接到 PC，并查看开发板使用的串口。通常，串口在不同操作系统下显示的名称有所不同：

- **Windows 操作系统**： ``COM1`` 等
- **Linux 操作系统**： 以 ``/dev/tty`` 开始
- **MacOS 操作系统**： 以 ``/dev/cu.`` 开始

.. note:：

    请记住串口名，您会在下面的步骤中用到。


.. _get-started-configure:

第七步：配置工程
**********************

请进入 :ref:`get-started-start-project` 中提到的 ``hello_world`` 目录，并运行工程配置工具 ``menuconfig``。Windows 操作系统

.. code:: batch

    cd %userprofile%\esp\hello_world
    idf.py menuconfig

如果之前的步骤都正确，则会显示菜单。

``menuconfig`` 工具的常见操作见下。

* 上下箭头：移动
* ``回车``：进入子菜单
* ``ESC 键``：返回上级菜单或退出
* ``英文问号``：调出帮助菜单（退出帮助菜单，请按回车键）。
* ``空格`` 或 ``Y 键``：选择 ``[*]`` 配置选项；``N 键``：禁用 ``[*]`` 配置选项
* ``英文问号`` （查询配置选项）：调出有关该选项的帮助菜单
* ``/ 键``：寻找配置工程

.. note::

    如果您使用的是 ESP32-DevKitC（板载 ESP32-SOLO-1 模组），请在烧写示例程序前，前往 ``menuconfig`` 中使能单核模式（`CONFIG_FREERTOS_UNICORE`__）。

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/api-reference/kconfig.html#config-freertos-unicore


.. _get-started-build:

第八步：编译工程
**********************

请使用以下命令，编译烧录工程:

.. code:: batch

    idf.py build

运行以上命令可以编译应用程序和所有 ESP-IDF 组件，接着生成 bootloader、分区表和应用程序二进制文件。

.. code:: batch

    $ idf.py build
    Running cmake in directory /path/to/hello_world/build
    Executing "cmake -G Ninja --warn-uninitialized /path/to/hello_world"...
    Warn about uninitialized values.
    -- Found Git: /usr/bin/git (found version "2.17.0")
    -- Building empty aws_iot component due to configuration
    -- Component names: ...
    -- Component paths: ...

    ... (more lines of build system output)

    [527/527] Generating hello-world.bin
    esptool.py v2.3.1

    Project build complete. To flash, run this command:
    ../../../components/esptool_py/esptool/esptool.py -p (PORT) -b 921600 write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x10000 build/hello-world.bin  build 0x1000 build/bootloader/bootloader.bin 0x8000 build/partition_table/partition-table.bin
    or run 'idf.py -p PORT flash'

如果一切正常，编译完成后将生成 .bin 文件。

.. _get-started-flash:

第九步：烧录到设备
************************

请使用以下命令，将刚刚生成的二进制文件烧录至您的 ESP32 开发板：

.. code:: batch

    idf.py -p PORT [-b BAUD] flash


请将 PORT 替换为 ESP32 开发板的串口名称，具体可见 :ref:`get-started-connect`。

您还可以将 BAUD 替换为您希望的烧录波特率。默认波特率为 460800。

更多有关 idf.py 参数的详情，请见 `idf.py`__。

.. __: (https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/api-guides/build-system.html#idf-py

.. tip::

    勾选 ``flash`` 选项将自动编译并烧录工程，因此无需再运行 ``idf.py build``。

.. code:: none

    Running esptool.py in directory [...]/esp/hello_world
    Executing "python [...]/esp-idf/components/esptool_py/esptool/esptool.py -b 460800 write_flash @flash_project_args"...
    esptool.py -b 460800 write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x1000 bootloader/bootloader.bin 0x8000 partition_table/partition-table.bin 0x10000 hello-world.bin
    esptool.py v2.3.1
    Connecting....
    Detecting chip type... ESP32
    Chip is ESP32D0WDQ6 (revision 1)
    Features: WiFi, BT, Dual Core
    Uploading stub...
    Running stub...
    Stub running...
    Changing baud rate to 460800
    Changed.
    Configuring flash size...
    Auto-detected Flash size: 4MB
    Flash params set to 0x0220
    Compressed 22992 bytes to 13019...
    Wrote 22992 bytes (13019 compressed) at 0x00001000 in 0.3 seconds (effective 558.9 kbit/s)...
    Hash of data verified.
    Compressed 3072 bytes to 82...
    Wrote 3072 bytes (82 compressed) at 0x00008000 in 0.0 seconds (effective 5789.3 kbit/s)...
    Hash of data verified.
    Compressed 136672 bytes to 67544...
    Wrote 136672 bytes (67544 compressed) at 0x00010000 in 1.9 seconds (effective 567.5 kbit/s)...
    Hash of data verified.

    Leaving...
    Hard resetting via RTS pin...

如果一切顺利，烧录完成后，开发板将会复位，应用程序 “hello_world” 开始运行。


.. _get-started-build-monitor:

第十步：监视器
*****************

您可以使用 ``idf.py -p PORT monitor`` 命令，监视 “hello_world” 的运行情况。注意，不要忘记将 PORT 替换为您的串口名称。

运行该命令后，`IDF 监视器`__ 应用程序将启动:

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/api-guides/tools/idf-monitor.html

.. code:: batch

    $ idf.py -p /dev/ttyUSB0 monitor
    Running idf_monitor in directory [...]/esp/hello_world/build
    Executing "python [...]/esp-idf/tools/idf_monitor.py -b 115200 [...]/esp/hello_world/build/hello-world.elf"...
    --- idf_monitor on /dev/ttyUSB0 115200 ---
    --- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
    ets Jun  8 2016 00:22:57

    rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
    ets Jun  8 2016 00:22:57
    ...

此时，您就可以在启动日志和诊断日志之后，看到打印的 “Hello world!” 了。

.. code:: none

    ...
    Hello world!
    Restarting in 10 seconds...
    I (211) cpu_start: Starting scheduler on APP CPU.
    Restarting in 9 seconds...
    Restarting in 8 seconds...
    Restarting in 7 seconds...


您可使用快捷键 ``Ctrl+]``，退出 IDF 监视器。

如果 IDF 监视器在烧录后很快发生错误，或打印信息全是乱码，很有可能是因为您的开发板采用了 26 MHz 晶振，而 ESP-IDF 默认支持大多数开发板使用的 40 MHz 晶振。

此时，请您：

1. 退出监视器。
2. 打开 :ref:`menuconfig <get-started-configure>`，
3. 进入 ``Component config`` –> ``ESP32-specific`` –> ``Main XTAL frequency`` 进行配置，将 `CONFIG_ESP32_XTAL_FREQ_SEL`__ 设置为 26 MHz。
4. 然后，请重新 :ref:`编译和烧录 <get-started-flash>` 应用程序。

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/api-reference/kconfig.html#config-esp32-xtal-freq-sel

.. note::

    您也可以运行以下命令，一次性执行构建、烧录和监视过程:

    .. code:: batch

        idf.py -p PORT flash monitor

此外，

* 请前往 `IDF 监视器`__，了解更多使用 IDF 监视器的快捷键和其他详情。
* 请前往 `idf.py`__，查看更多 ``idf.py`` 命令和选项。

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/api-guides/tools/idf-monitor.html
.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/api-guides/build-system.html#idf-py

**恭喜，您已完成 ESP32 的入门学习！**

现在，您可以尝试一些其他 `examples`__，或者直接开发自己的应用程序。

.. __: https://github.com/espressif/esp-idf/tree/v4.0.1/examples


更新 ESP-IDF
--------------

乐鑫会不时推出更新版本的 ESP-IDF，修复 bug 或提出新的特性。因此，您在使用时，也应注意更新您本地的版本。最简单的方法是：直接删除您本地的 ``esp-idf`` 文件夹，然后按照 :ref:`get-started-get-esp-idf` 中的指示，重新完成克隆。

此外，您可以仅更新变更部分。具体方式，请前往 `更新`__ 章节查看。

注意，更新完成后，请执行 ``install.sh`` (Windows 系统中为 ``install.bat``) 脚本，避免新版 ESP-IDF 所需的工具也所更新。具体请参考 :ref:`get-started-set-up-tools`。

一旦重新安装好工具，请使用 ``export.sh`` (Windows 系统中为 ``export.bat``) 脚本更新环境，具体请参考 :ref:`get-started-set-up-env`。

.. __: https://docs.espressif.com/projects/esp-idf/zh_CN/v4.0.1/versions.html#updating
