电池服务 Battery Servcie
#################################

.. tip:: 

    如果图片看不清楚。你可以 **在图片上点击鼠标右键** --> **在新标签页中打开图片** ，然后你可以放大、缩小、移动图片。

`battery_service.h`__ , `battery_service.c`__ , `voltage_monitor.h`__ , `voltage_monitor.c`__ , `battery_example.c`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/battery_service/include/battery_service.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/battery_service/battery_service.c

.. __: https://github.com/espressif/esp-adf/blob/master/components/battery_service/monitors/include/voltage_monitor.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/battery_service/monitors/voltage_monitor.c

.. __: https://github.com/espressif/esp-adf/blob/master/examples/system/battery/main/battery_example.c


概述
=========


**电池服务** Battery Service，是 外设服务 Periph Service 的子类。

* 前者完全是按后者的框架实现的。
* 后者实现时基本上完全覆盖（或继承）前者的方法。
* 除了创建 Create 与特殊 API 要调用外设子服务的API；一般都调用音频服务的 API。


类图
=========

.. uml::

    caption Battery Servcie 类图

    Peripha_Service <|-- Battery_Servcie

    Battery_Servcie *-- Voltage_Monitor

    class Battery_Servcie {
        __ battery_service_t __
        -xQueueHandle         serv_q;
        -EventGroupHandle_t   sync_events;
        -vol_monitor_handle_t vol_monitor;

        __ private method __
        -battery_task()
        -_battery_start()
        -_battery_stop()
        -_battery_destroy()

        __ public method __
        +battery_service_create()
        +battery_service_vol_report_switch()
        +battery_service_set_vol_report_freq()
    }

    class Voltage_Monitor {
        __ vol_monitor_ctx_t __
        -SemaphoreHandle_t mutex;
        -vol_monitor_param_t *config;
        -vol_monitor_event_cb event_cb;
        -void *user_ctx;
        -esp_timer_handle_t check_timer;

        __ private method __
        -vol_check_timer_hdlr()

        __ public method __
        +vol_monitor_create(vol_monitor_param_t *config)
        +vol_monitor_destroy()
        +vol_monitor_set_event_cb()
        +vol_monitor_start_freq_report()
        +vol_monitor_stop_freq_report()
        +vol_monitor_set_report_freq()
    }

我们用类图的形式，描述电池服务 Battery Service 的实现（实际上代码是用 C 语言实现的）：

Battery_Servcie 类
------------------------

* 结构体 ``battery_service_t`` ，是内部私有数据。

    * ``.serv_q``: API 等向内部任务 ``battery_task`` 发送消息的消息队列。
    * ``.sync_events``: ``battery_task`` 向 API 等发送事件的 Event Group 。
    * ``.vol_monitor``: 也就是 class Voltage_Monitor 。 电池服务内部的另一个用于监测电池电压的定时器任务。见下文 `Voltage_Monitor 类`_ 。

* **private method** 部分，是 Battery Servcie 内部函数。这些函数的实现大部分都很简单:

    * ``battery_task()``:  内部任务。
    * ``_battery_start()``: 启动 Battery Service。
    * ``_battery_stop()``: 停止 Battery Service。
    * ``_battery_destroy()``: 销毁 Battery Service。

* **public method** 部分，是 DuerOS Servcie 的提供的 API 函数。这些函数的实现大部分都很简单:

    * ``battery_service_create()`` 的实现略微复杂一点，看这里  `API`_ 。
    * ``battery_service_vol_report_switch()`` 开关电量上报。
    * ``battery_service_set_vol_report_freq()`` 设置电量上报频率。

Voltage_Monitor 类
------------------------

* 结构体 ``vol_monitor_ctx_t`` ，是内部私有数据。

    * ``.mutex``: 锁，用于保护结构体内的其它字段，防止 API 与 定时器同时访问/修改这些变量。
    * ``.config``: 一些配置，如下所示：

        .. code:: c

            /**
            * @brief Battery adc configure
            */
            typedef struct {
                bool (*init)(void *);   /*!< Voltage read init */
                bool (*deinit)(void *); /*!< Voltage read deinit */
                int (*vol_get)(void *); /*!< Voltage read interface */
                void *user_data;        /*!< Parameters for callbacks */
                int read_freq;          /*!< Voltage read frequency, unit: s*/
                int report_freq;        /*!< Voltage report frequency, voltage will be report with a interval calculate by （`read_freq` * `report_freq`） */
                int vol_full_threshold; /*!< Voltage threshold to report, unit: mV */
                int vol_low_threshold;  /*!< Voltage threshold to report, unit: mV */
            } vol_monitor_param_t;

    * ``.event_cb``: 事件回调，用于与 ``battery_task()`` 通信 。
    * ``.user_ctx``: 事件回调上下文参数 。
    * ``.check_timer``: 定时器 。

* **private method** 部分，是 Voltage Monitor 内部函数。

    * ``vol_check_timer_hdlr()``:  定时器任务。

* **public method** 部分，是 Voltage Monitor 的提供的 API 函数。这些函数的实现大部分都很简单:

    * ``vol_monitor_create(vol_monitor_param_t *config)`` 创建电压监测功能 。
    * ``vol_monitor_destroy()`` 销毁电压监测能。
    * ``vol_monitor_set_event_cb()`` 设置电压监测事件回调。
    * ``vol_monitor_start_freq_report()`` 下始电压监测周期上报。
    * ``vol_monitor_stop_freq_report()`` 停止电压监测周期上报。
    * ``vol_monitor_set_report_freq()`` 设置电压监测上报频率。

--------------------------------------------------------------------------


序列图
=============

.. uml::

    caption Battery Servcie 序列图

    box "battery_example"
    participant "battery_example.c" as adf_app  order 10
    end box

    box "esp_dispatcher"
    participant "periph_service.c"  as periph_service  order 20
    end box

    box "battery_service" #LightBlue 
    participant "battery_service.c" as battery_service order 30
    participant "battery_task()" as battery_task order 40

    participant "voltage_monitor.c" as vol_monitor order 50
    participant "vol_check_timer_hdlr()" as vol_timer order 60
    end box

    == Create voltage monitor ==
    autonumber 1 "<b>(<u>##</u>)"
    adf_app     -> vol_monitor  : vol_monitor = vol_monitor_create({.init = adc_init,.deinit = adc_deinit,.vol_get = vol_read})    
    vol_monitor -> vol_monitor  : .config->init()
    vol_monitor ->]             : .mutex = mutex_create()
    vol_monitor ->]             : .check_timer = esp_timer_create \n ({.callback = vol_check_timer_hdlr})
    vol_monitor ->]             : esp_timer_start_periodic({.config->read_freq})
    activate vol_timer


    == Create battery service & set callback ==
    autonumber 10 "<b>(<u>##</u>)"
    adf_app        -> battery_service : battery_service = battery_service_create( \n {.evt_cb = **battery_service_cb**, .vol_monitor = vol_monitor})
    battery_service ->] : .serv_q = xQueueCreate()
    battery_service ->] : .sync_events = xEventGroupCreate()
    periph_service  <- battery_service : periph_service_create({\n .service_destroy = _battery_destroy, \n .service_start = _battery_start, \n .service_stop = _battery_stop, \n .task_func  = battery_task, \n .user_data = (void *)battery_service})
    alt .task_func!=NULL (实际上是 .task_stack > 0)
    periph_service -> battery_task : audio_thread_create({.task_func})
    activate battery_task
    end

    periph_service  <- battery_service : periph_service_set_callback({.evt_cb});
   

    == Start battery service ==
    autonumber 20 "<b>(<u>##</u>)"
    adf_app       -> periph_service : periph_service_start()
    alt .service_start != NULL
    periph_service -> battery_service : .service_start() \n ==> _battery_start()
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_SERVICE_START)
    battery_task    -> vol_monitor  : vol_monitor_set_event_cb \n (**battery_vol_monitor_cb**)
    vol_monitor     -> vol_monitor  : .event_cb = battery_vol_monitor_cb
    end


    == Start battery voltage report ==
    autonumber 30 "<b>(<u>##</u>)"
    adf_app       -> battery_service : battery_service_vol_report_switch(true)
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_START)
    battery_task    -> vol_monitor  : vol_monitor_start_freq_report()
    vol_monitor     -> vol_monitor  : .report_start = config->report_freq


    == Change battery voltage report freqency ==
    autonumber 35 "<b>(<u>##</u>)"
    adf_app       -> battery_service : battery_service_set_vol_report_freq(freq)
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_SET_REPORT_FREQ)
    battery_task    -> vol_monitor  : vol_monitor_set_report_freq(freq)
    vol_monitor     -> vol_monitor  : .config->report_freq = freq, \n .report_start = freq


    == Battery voltage report ==
    autonumber 40 "<b>(<u>##</u>)"
    vol_timer <-]                   : (freq)
    alt 周期超时
    vol_timer -> battery_service    : .event_cb(VOL_MONITOR_EVENT_FREQ_REPORT) ==> battery_vol_monitor_cb()
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_FREQ)
    periph_service  <- battery_task : periph_service_callback(BAT_SERV_EVENT_VOL_REPORT)
    adf_app       <- periph_service : battery_service_cb()

    else 低电量
    vol_timer -> battery_service    : .event_cb(VOL_MONITOR_EVENT_BAT_LOW) ==> battery_vol_monitor_cb()
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_LOW)
    periph_service  <- battery_task : periph_service_callback(BAT_SERV_EVENT_BAT_LOW)
    adf_app       <- periph_service : battery_service_cb()
    
    else 电量满
    vol_timer -> battery_service    : .event_cb(VOL_MONITOR_EVENT_BAT_FULL) ==> battery_vol_monitor_cb()
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_FULL)
    periph_service  <- battery_task : periph_service_callback(BAT_SERV_EVENT_BAT_FULL)
    adf_app       <- periph_service : battery_service_cb()
    end


    == stop battery voltage report ==
    autonumber 55 "<b>(<u>##</u>)"
    adf_app       -> battery_service : battery_service_vol_report_switch(false)
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_STOP)
    battery_task    -> vol_monitor  : vol_monitor_stop_freq_report()
    vol_monitor     -> vol_monitor  : .report_start = 0


    == --Stop battery service-- ==
    autonumber 60 "<b>(<u>##</u>)"
    [-> periph_service : periph_service_stop()
    alt .service_stop != NULL
    periph_service -> battery_service : .service_stop() \n ==> _battery_stop()
    battery_service -> battery_task : battery_service_msg_send \n (BATTERY_SERVICE_STOP)
    battery_task   -> vol_monitor   : vol_monitor_set_event_cb(NULL)
    vol_monitor    -> vol_monitor   : .event_cb = NULL
    end


    == Destroy battery service ==
    autonumber 70 "<b>(<u>##</u>)"
    [-> periph_service                 : periph_service_destroy()
    alt .service_desotry != NULL
    periph_service  -> battery_service : .service_desotry() \n ==> _battery_destroy()
    battery_service -> battery_task    : battery_service_msg_send \n (BATTERY_SERVICE_DESTROY)
    battery_task   -> vol_monitor   : vol_monitor_set_event_cb(NULL)
    vol_monitor    -> vol_monitor   : .event_cb = NULL
    battery_service -> battery_task : (destory task)
    deactivate battery_task 
    battery_service ->]         : vQueueDelete(.serv_q)
    battery_service ->]         : vEventGroupDelete(.sync_events) 
    end

    == Destory voltage monitor ==
    autonumber 80 "<b>(<u>##</u>)"
    adf_app -> vol_monitor : vol_monitor_destroy(vol_monitor)
    vol_monitor ->]             : esp_timer_stop(.check_timer)
    vol_monitor ->]             : esp_timer_delete(.check_timer)
    vol_monitor -> vol_monitor  : .config->deinit()
    vol_monitor ->]             : mutex_destroy(.mutex)
    deactivate vol_timer
 

**对像说明：**

* **battery_example.c**: 某个应用程序
* **battery_service.c**: 电池服务
* **battery_task()**: 电池服务的内部任务
* **voltage_monitor.c**：电量监测
* **vol_check_timer_hdlr()**：电量监测内部的定时器

**流程说明：**

1. 应用程序 battery_example.c 调用 voltage_monitor.c 电量检测 ``vol_monitor_create()``。同时传入与电量检测相关的三个函数。
2. voltage_monitor.c 调用初始函数 ``.config->init()`` 。
3. 创建锁。
4. 创建定时器 ``.check_timer`` 。该定时器超时后会调用 ``vol_check_timer_hdlr()`` 。
5. 启动定时器 ``.check_timer`` 。该定时器超时每 ``.config->read_freq`` 会超时一次。

10. 应用程序 battery_example.c 调用 Battery Service 电池服务 ``battery_service_create()``。
11. 创建内部通信的消息队列。
12. 创建内部通信的事件组。
13. battery_service.c 调用 ``periph_service_create()``, 并会将 ``.service_destroy`` ， ``.service_start`` , ``.service_stop`` ,  等回调函数作为参数的字段传入。 同时也会将自已的地址，作为 ``.user_data`` 参数字段传入。 因 Battery Service 需要创建内部任务，也会将内部任务函数 battery_task() 作为 ``.task_func`` 参数字段传入。
14. periph_service.c 将上述回调函数和 ``.user_data`` 保存下来。因为 ``.task_func`` 不为空(实际上是 ``.task_stack > 0``)，故同时创建内部任务。
15. battery_service.c 调用 ``periph_service_set_callback({.evt_cb})`` 注册事件回调函数。

20. battery_example.c 调用 ``periph_service_start()`` 开始电池服务。
21. ``.service_start`` 不为空，被执行（ 实际上是执行 ``_battery_start()`` ）。
22. battery_service.c 发送消息 ``BATTERY_SERVICE_START`` 给内部任务。
23. 内部任务 battery_task() 调用 ``vol_monitor_set_event_cb(battery_vol_monitor_cb)`` 设置电量监测的事件回调函数。``battery_vol_monitor_cb()`` 就是该事件回调函数。
24. voltage_monitor.c 保存该事件回调函数。

30. battery_example.c 调用 ``battery_service_vol_report_switch(true)`` 开始电量上报。
31. battery_service.c 发送消息 ``BATTERY_VOL_REPORT_START`` 给内部任务。
32. 内部任务 battery_task() 调用 ``vol_monitor_start_freq_report()`` 开始周期上报。
33. voltage_monitor.c 保存新的电量上报周期。

35. battery_example.c 调用 ``battery_service_set_vol_report_freq(freq)`` 修改电量上报周期。
36. battery_service.c 发送消息 ``BATTERY_VOL_SET_REPORT_FREQ`` 给内部任务。
37. 内部任务 battery_task() 调用 ``vvol_monitor_set_report_freq(freq)`` 修改电量上报周期。
38. voltage_monitor.c 保存新的电量上报周期。

40. 定时器周期超时。
41. 定时器调用 ``.event_cb(VOL_MONITOR_EVENT_FREQ_REPORT)`` （实际上是调用 ``battery_vol_monitor_cb()`` ） 周期上报电量。
42. battery_service.c 向内部任务 battery_task 发送消息 ``BATTERY_VOL_REPORT_FREQ`` ，携带电量信息 。
43. 内部任务 battery_task 调用 periph_service_callback(BAT_SERV_EVENT_VOL_REPOR).
44. periph_service.c 调用应用的事件回调函数 ``battery_service_cb()`` 。

    45~48, 49~52 的流程与 40~44 完全类似，只不过是携带的信息不同: 电量低或电量满。

55. battery_example.c 调用 ``battery_service_vol_report_switch(false)`` 开始电量上报。
56. battery_service.c 发送消息 ``BATTERY_VOL_REPORT_STOP`` 给内部任务。
57. 内部任务 battery_task() 调用 ``vol_monitor_stop_freq_report()`` 开始周期上报。
58. voltage_monitor.c 保存新的电量上报周期。

60. 应用调用（实际上无些调用） ``periph_service_stop()`` 停止电池服务。
61. ``.service_stop`` 不为空，被执行（ 实际上是执行 ``_battery_stop()`` ）。
62. battery_service.c 发送消息 ``BATTERY_SERVICE_STOP`` 给内部任务。
63. 内部任务 battery_task() 调用 ``vol_monitor_set_event_cb(NULL)`` 清除电量监测的事件回调函数。
64. voltage_monitor.c 清除该事件回调函数。

70. battery_example.c 调用 ``periph_service_destroy()``, 销毁 Battery Service。
71. 因 ``.service_destroy`` 不为空， 其被 periph_service.c 调用（ 实际上是执行 ``_battery_destory()`` ）。
72. battery_server.c 发送 ``BATTERY_SERVICE_DESTROY`` 消息给内部任务。
73. 内部任务 battery_task() 调用 ``vol_monitor_set_event_cb(NULL)`` 清除电量监测的事件回调函数。
74. voltage_monitor.c 清除该事件回调函数。。
75. battery_service.c 中止内部任务 battery_task() 。
76. battery_service.c 清除消息队列。
77. battery_service.c 清除事件组。

80. 应用程序 battery_example.c 调用 voltage_monitor.c 电量检测 ``vol_monitor_destory()``。
81. voltage_monitor.c 停止定时器 ``.check_timer`` 。
82. voltage_monitor.c 销毁定时器 ``.check_timer`` 。
83. voltage_monitor.c 调用反初始化函数 ``.config->deinit()`` 。
84. voltage_monitor.c 删除锁 。


API
=================

* vol_monitor_create()

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Create voltage monitor ==
        autonumber 1 "<b>(<u>##</u>)"
        adf_app     -> vol_monitor  : vol_monitor = vol_monitor_create({.init = adc_init,.deinit = adc_deinit,.vol_get = vol_read})    
        vol_monitor -> vol_monitor  : .config->init()
        vol_monitor ->]             : .mutex = mutex_create()
        vol_monitor ->]             : .check_timer = esp_timer_create \n ({.callback = vol_check_timer_hdlr})
        vol_monitor ->]             : esp_timer_start_periodic({.config->read_freq})
        activate vol_timer


* battery_service_create()

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Create battery service & set callback ==
        autonumber 10 "<b>(<u>##</u>)"
        adf_app        -> battery_service : battery_service = battery_service_create( \n {.evt_cb = **battery_service_cb**, .vol_monitor = vol_monitor})
        battery_service ->] : .serv_q = xQueueCreate()
        battery_service ->] : .sync_events = xEventGroupCreate()
        periph_service  <- battery_service : periph_service_create({\n .service_destroy = _battery_destroy, \n .service_start = _battery_start, \n .service_stop = _battery_stop, \n .task_func  = battery_task, \n .user_data = (void *)battery_service})
        alt .task_func!=NULL (实际上是 .task_stack > 0)
        periph_service -> battery_task : audio_thread_create({.task_func})
        activate battery_task
        end

        periph_service  <- battery_service : periph_service_set_callback({.evt_cb});
    

* periph_service_start()

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Start battery service ==
        autonumber 20 "<b>(<u>##</u>)"
        adf_app       -> periph_service : periph_service_start()
        alt .service_start != NULL
        periph_service -> battery_service : .service_start() \n ==> _battery_start()
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_SERVICE_START)
        battery_task    -> vol_monitor  : vol_monitor_set_event_cb \n (**battery_vol_monitor_cb**)
        vol_monitor     -> vol_monitor  : .event_cb = battery_vol_monitor_cb
        end


* battery_service_vol_report_switch(true)

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Start battery voltage report ==
        autonumber 30 "<b>(<u>##</u>)"
        adf_app       -> battery_service : battery_service_vol_report_switch(true)
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_START)
        battery_task    -> vol_monitor  : vol_monitor_start_freq_report()
        vol_monitor     -> vol_monitor  : .report_start = config->report_freq


* battery_service_set_vol_report_freq()

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Change battery voltage report freqency ==
        autonumber 35 "<b>(<u>##</u>)"
        adf_app       -> battery_service : battery_service_set_vol_report_freq(freq)
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_SET_REPORT_FREQ)
        battery_task    -> vol_monitor  : vol_monitor_set_report_freq(freq)
        vol_monitor     -> vol_monitor  : .config->report_freq = freq, \n .report_start = freq


* *battery_service_cb()* (事件回调)

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Battery voltage report ==
        autonumber 40 "<b>(<u>##</u>)"
        vol_timer <-]                   : (freq)
        alt 周期超时
        vol_timer -> battery_service    : .event_cb(VOL_MONITOR_EVENT_FREQ_REPORT) ==> battery_vol_monitor_cb()
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_FREQ)
        periph_service  <- battery_task : periph_service_callback(BAT_SERV_EVENT_VOL_REPORT)
        adf_app       <- periph_service : battery_service_cb()

        else 低电量
        vol_timer -> battery_service    : .event_cb(VOL_MONITOR_EVENT_BAT_LOW) ==> battery_vol_monitor_cb()
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_LOW)
        periph_service  <- battery_task : periph_service_callback(BAT_SERV_EVENT_BAT_LOW)
        adf_app       <- periph_service : battery_service_cb()
        
        else 电量满
        vol_timer -> battery_service    : .event_cb(VOL_MONITOR_EVENT_BAT_FULL) ==> battery_vol_monitor_cb()
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_FULL)
        periph_service  <- battery_task : periph_service_callback(BAT_SERV_EVENT_BAT_FULL)
        adf_app       <- periph_service : battery_service_cb()
        end


* battery_service_vol_report_switch(false)

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == stop battery voltage report ==
        autonumber 55 "<b>(<u>##</u>)"
        adf_app       -> battery_service : battery_service_vol_report_switch(false)
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_VOL_REPORT_STOP)
        battery_task    -> vol_monitor  : vol_monitor_stop_freq_report()
        vol_monitor     -> vol_monitor  : .report_start = 0

* periph_service_stop()

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == --Stop battery service-- ==
        autonumber 60 "<b>(<u>##</u>)"
        [-> periph_service : periph_service_stop()
        alt .service_stop != NULL
        periph_service -> battery_service : .service_stop() \n ==> _battery_stop()
        battery_service -> battery_task : battery_service_msg_send \n (BATTERY_SERVICE_STOP)
        battery_task   -> vol_monitor   : vol_monitor_set_event_cb(NULL)
        vol_monitor    -> vol_monitor   : .event_cb = NULL
        end

* periph_service_destroy()

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Destroy battery service ==
        autonumber 70 "<b>(<u>##</u>)"
        [-> periph_service                 : periph_service_destroy()
        alt .service_desotry != NULL
        periph_service  -> battery_service : .service_desotry() \n ==> _battery_destroy()
        battery_service -> battery_task    : battery_service_msg_send \n (BATTERY_SERVICE_DESTROY)
        battery_task   -> vol_monitor   : vol_monitor_set_event_cb(NULL)
        vol_monitor    -> vol_monitor   : .event_cb = NULL
        battery_service -> battery_task : (destory task)
        deactivate battery_task 
        battery_service ->]         : vQueueDelete(.serv_q)
        battery_service ->]         : vEventGroupDelete(.sync_events) 
        end


* vol_monitor_destroy()

    .. uml::

        caption Battery Servcie 序列图

        box "battery_example"
        participant "battery_example.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "periph_service.c"  as periph_service  order 20
        end box

        box "battery_service" #LightBlue 
        participant "battery_service.c" as battery_service order 30
        participant "battery_task()" as battery_task order 40

        participant "voltage_monitor.c" as vol_monitor order 50
        participant "vol_check_timer_hdlr()" as vol_timer order 60
        end box

        == Destory voltage monitor ==
        autonumber 80 "<b>(<u>##</u>)"
        adf_app -> vol_monitor : vol_monitor_destroy(vol_monitor)
        vol_monitor ->]             : esp_timer_stop(.check_timer)
        vol_monitor ->]             : esp_timer_delete(.check_timer)
        vol_monitor -> vol_monitor  : .config->deinit()
        vol_monitor ->]             : mutex_destroy(.mutex)
        deactivate vol_timer
 