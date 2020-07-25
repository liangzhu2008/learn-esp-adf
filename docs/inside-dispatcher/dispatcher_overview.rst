概述
#####

本节原文详见 `这里`__。

.. __: https://github.com/espressif/esp-adf/tree/master/components/esp_dispatcher


| 市场上的音频产品大多具有高度相似的特性，例如使用基本相同的人机交互模式，使用几乎相同的云服务器提供的云服务。区别仅在于产品的按键功能及其逻辑控制模块。这意味着您可以重用某些现有产品的其他功能。现在，Espressif 提供了一个新的设计 **分发器 ESP_Dispatcher** 来帮助重用这些函数。
| Most of audio products in the market have highly similar features, such as applying basically the same human-computer interaction mode, and using nearly identical cloud services provided by various cloud servers. What differentiates is only the key functions of a product and its logical control module, which means you can reuse the other functions of some existing products. Now, Espressif provides a new design, **ESP_Dispatcher**, to help reuse these functions.

| 基于 ESP_Dispatcher，产品功能划可以划分为 **事件输入模块 event input module** 、**逻辑控制模块 control module** 和 **执行输出模块 execution output module** ，每个模块又可进一步细分为独立的 **子功能单元 sub-functional units** 。各子功能单元可与其他单元灵活组合，也可定制逻辑控制设计。
| Based on ESP_Dispatcher, product functions are split into **event input module**, **logic control module** and **execution output module**, with each module further subdivided into separate **sub-functional units**. Each sub-functional unit can be combined flexibly with other units or with customized logic control design.

| ESP_Dispatcher 帮助您重用不同产品的相同功能，避免重复开发，使现有产品功能更具可重用性。同时，您可以自行设计不同的功能和逻辑控制模块，使产品开发更加灵活。这样可以缩短产品开发周期，加快产品上市时间和产品迭代。
| ESP_Dispatcher helps you reuse the same functions of different products, which avoids repeated development and makes the existing product functions more reusable. Meanwhile, you can design differentiated functions and logic control module at your own will, making product development more flexible. In such way, you can shorten product development cycle, and speed up the timing to market and product iteration.

| ESP_Dispatcher由一个 **分发器 dispatcher** 和一系列的 **服务 service** 组成。
| ESP_Dispatcher consists of a **dispatcher** and a series of **services**.

* | 分发器 dispatcher 采用分离执行的机制，执行功能单元按单元 ID 各自独立。
  | The dispatcher adopts a separate-execution mechanism and executes functional units separately according to unit ID.
  
* | **服务 service** 是一个高度抽象的功能类，只生成外部输入和输出，比如提供可执行的功能单元  executable functional units 和生成服务状态事件 service state events。非外部任务在服务内部处理。目前，ESP_Dispatcher支持2种服务 service:
  | The **service** is a highly-abstract functional class and only generates external input and output, such as providing executable functional units and generating service state events. Non-external tasks are handled internally in service. Currently, ESP_Dispatcher support 2 services:
  
    * 外设服务 Peripheral Service
        * | **外设服务 Peripheral Service** 用于处理GPIO、Touch、SDCARD、ADC、UART、Display、Wi-Fi、网络配置等外部任务。
          | **Peripheral service** is used to handle external tasks concerning GPIO, Touch, SDCARD, ADC, UART, Display, Wi-Fi and network configuration.
        * | 该服务还可以生成 GPIO 中断、触摸事件，并提供管理和控制功能，如网络配置控制和 LED 显示样式管理。
          | This service also can generate GPIO interrupts, Touch events, and provides management and control functions, such as network configuration control and LED display style management.

    * 音频服务 Audio Service
        * | **音频服务Audio service** 用于处理协议类功能，如云服务(DuerOS)、DLNA 和 VoIP，并提供控制类接口 interfaces 和状态事件 state events。
          | **Audio service** is used to handle protocol class functions, such as cloud service (DuerOS), DLNA and VoIP, and provides control-class interfaces and state events.

| ESP_Dispatcher 中的每个模块和子功能单元，作为一个单独的任务运行。它提供了一个架构，可以简化复杂的产品设计，并适应任何风格的产品。
| Each module and sub-functional unit in ESP_Dispatcher runs as a separate task, providing a structure that can simplify complex product styles and adapts to any product style.

| 一个标准的 ESP_Dispatcher 音频应用程序框图如下所示。
| A standard ESP_Dispatcher audio application block diagram as shown below.

.. image:: /_static/esp_dispatcher_audio_app_diagram.png

| 一个例子：`esp_dispatcher_dueros`__。
| There is an example in the folder of `esp_dispatcher_dueros`__.

.. __: https://github.com/espressif/esp-adf/tree/master/examples/advanced_examples/esp_dispatcher_dueros
.. __: https://github.com/espressif/esp-adf/tree/master/examples/advanced_examples/esp_dispatcher_dueros