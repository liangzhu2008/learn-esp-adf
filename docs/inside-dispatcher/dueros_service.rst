小度助手服务 DuerOS Service
#######################################


.. tip:: 

    如果图片看不清楚。你可以 **在图片上点击鼠标右键** --> **在新标签页中打开图片** ，然后你可以放大、缩小、移动图片。

`dueros_service.c`__.

.. __: https://github.com/espressif/esp-adf/blob/master/components/dueros_service/dueros_service.c

.. role:: strike
   :class: strike


1. 概述
=========


**小度助手服务** DuerOS Service，是为 音频服务 Audio Service 的子类。

* 前者完全是按后者的框架实现的。
* 后者实现时基本上完全覆盖（或继承）前者的方法。
* 除了创建 Create 与特殊 API 要调用音频子服务的API；一般都调用音频服务的 API。


2. 类图
=========

.. uml::

    caption DuerOS Servcie 类图

    Audio_Service <|-- DuerOS_Servcie

    class DuerOS_Servcie {
        __ dueros_service_t __
        -xQueueHandle       duer_que;
        -service_state_t    duer_state;

        __ private method __
        -dueros_task()
        -dueros_start()
        -dueros_stop()
        -dueros_connect()
        -dueros_disconnect()
        -dueros_destroy()

        __ public method __
        +dueros_service_create()
        +dueros_service_state_get()
    }

我们用类图的形式，描述小度助手服务 DuerOS Service 的实现（实际上代码是用 C 语言实现的）：

* 结构体 ``dueros_service_t`` ，是内部私有数据。

    * ``.duer_que``: 与内部任务 ``dueros_task`` 之间通信的消息队列。
    * ``.duer_state``: DuerOS Service 的状态。

* **private method** 部分，是 DuerOS Servcie 内部函数。这些函数的实现大部分都很简单:

    * ``dueros_task()``:  内部任务。
    * ``dueros_start()``: 启动 DuerOS Service。
    * ``dueros_stop()``: 停止 DuerOS Service。
    * ``dueros_connect()``: 连接 DuerOS Service。
    * ``dueros_disconnect()``: 断开 DuerOS Service。
    * ``dueros_destroy()``: 销毁 DuerOS Service。

* **public method** 部分，是 DuerOS Servcie 的提供的 API 函数。这些函数的实现大部分都很简单:

    * ``dueros_service_create()`` 的实现略微复杂一点，看这里  `4. API`_ 。
    * ``dueros_service_state_get()`` 获得 DuerOS Service 状态。

3. 序列图
=============

.. uml::

    caption DuerOS Servcie 序列图

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher\n_dueros_app.c" as adf_app  order 10
    end box

    box "esp_dispatcher"
    participant "esp_dispatcher.c"  as esp_dispatcher  order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    participant "audio_service.c"  as audio_service  order 40
    end box

    box "esp_actions"
    participant "dueros_action.c"  as dueros_action  order 50
    end box

    box "dueros_service" #LightBlue 
    participant "dueros_service.c" as dueros_service order 60
    participant "dueros_task()"    as service_task order 70
    end box

    box "dueros"
    participant "lightduer" as lightduer order 80
    end box
      
    == Create dueros service & set callback ==
    autonumber 1 "<b>(<u>##</u>)"
    adf_app        -> dueros_service : audio_serv = \n dueros_service_create()
    audio_service  <- dueros_service : audio_service_create({\n .service_destroy = dueros_destroy, \n .service_start = dueros_start, \n .service_stop = dueros_stop, \n .service_connect = dueros_connect, \n .service_disconnect = dueros_disconnect, \n .task_func  = dueros_task, \n .user_data = (void *)serv})

    alt .task_func!=NULL (实际上是 .task_stack > 0)
    audio_service -> service_task : xTaskCreatePinnedToCore({.task_func})
    activate service_task

    service_task -> lightduer : duer_initialize()
    service_task -> lightduer : duer_set_event_callback \n (**duer_event_hook**)
    service_task -> service_task : duer_init_device_info()
    service_task -> lightduer : duer_register_device_info_ops()
    service_task -> service_task : duer_state = \n SERVICE_STATE_IDLE
    end

    == Register dueros service execution type ==
    autonumber 10 "<b>(<u>##</u>)"
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE \n _DUER_CONNECT, \n dueros_action_connect)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE \n _DUER_DISCONNECT, \n dueros_action_disconnect)

    == Set event callback ==
    adf_app        -> audio_service  : audio_service_set_callback \n ({.callback_func=duer_callback})


    == Connect dueros service ==
    autonumber 20 "<b>(<u>##</u>)"
    adf_app        <-]              : wifi_service_cb \n (WIFI_SERV_EVENT_CONNECTED)
    adf_app        -> esp_dispatcher : esp_dispatcher_execute \n (ACTION_EXE_TYPE \n _DUER_CONNECT)
    esp_dispatcher -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
    dispatcher_task -> dueros_action : exe_item->exe_func \n (instance, arg, result) \n ==>　dueros_action_connect()
    dueros_action  -> audio_service : audio_service_connect()
    alt .service_connect != NULL
    audio_service -> dueros_service : .service_connect() \n ==> dueros_connect()
    dueros_service -> service_task  : duer_que_send \n (DUER_CMD_LOGIN)
    service_task   -> service_task  : duer_login()
    service_task   -> lightduer     : duer_start()
    service_task   -> service_task  : duer_state = \n SERVICE_STATE_CONNECTING
    audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_CONNECTING)
    alt .callback_func != NULL
    adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
    end
    end


    == Connected dueros service ==
    autonumber 35 "<b>(<u>##</u>)"
    dueros_service <- lightduer    : duer_event_hook(DUER_EVENT_STARTED)
    dueros_service -> service_task : duer_que_send \n (DUER_CMD_CONNECTED)
    service_task   -> service_task  : duer_state = \n SERVICE_STATE_CONNECTED
    audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_CONNECTED)
    alt .callback_func != NULL
    adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
    end


    == Start dueros service ==
    autonumber 45 "<b>(<u>##</u>)"
    adf_app       <-]              : rec_engine_cb \n (REC_EVENT_VAD_START)
    adf_app       -> audio_service : audio_service_start()
    alt .service_start != NULL
    audio_service -> dueros_service : .service_start() \n ==> dueros_start()
    dueros_service -> service_task : duer_que_send \n (DUER_CMD_START)
    service_task   -> lightduer    : duer_voice_start \n (RECORD_SAMPLE_RATE)
    service_task   -> lightduer    : duer_dcs_on \n _listen_started()
    service_task   -> service_task  : duer_state = \n SERVICE_STATE_RUNNING
    audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_RUNNING)
    alt .callback_func != NULL
    adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
    end
    loop 从 rec_engine 读完所有数据
    service_task   ->]              : rec_engine_data_read()
    service_task   -> lightduer     : duer_voice_send()
    end
    end


    == Stop dueros service ==
    autonumber 60 "<b>(<u>##</u>)"
    adf_app         <-]              : rec_engine_cb \n (REC_EVENT_VAD_STOP \n | REC_EVENT_WAKEUP_END)
    adf_app         -> audio_service : audio_service_stop()
    alt .service_stop != NULL
    audio_service -> dueros_service : .service_stop() \n ==> dueros_stop()
    dueros_service -> service_task : duer_que_send \n (DUER_CMD_STOP)
    service_task   -> lightduer    : duer_voice_stop()
    service_task   -> service_task  : duer_state = \n SERVICE_STATE_STOPPED
    audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_STOPPED)
    alt .callback_func != NULL
    adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
    end
    end


    == Disconnect dueros service ==
    autonumber 70 "<b>(<u>##</u>)"
    alt Wi-Fi 事件
    adf_app        <-]              : wifi_service_cb \n (WIFI_SERV_EVENT_DISCONNECTED)
    adf_app        -> esp_dispatcher : esp_dispatcher_execute \n (ACTION_EXE_TYPE \n _DUER_DISCONNECT)
    esp_dispatcher -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
    dispatcher_task -> dueros_action : exe_item->exe_func \n (instance, arg, result) \n ==>　dueros_action_disconnect()
    dueros_action  -> audio_service : audio_service_disconnect()
    else duer 事件
    dueros_service -> lightduer      : duer_event_hook(DUER_EVENT_STOPPED)
    dueros_service -> audio_service  : audio_service_disconnect()
    end

    alt .service_disconnect != NULL
    audio_service -> dueros_service   : .service_disconnect() \n ==> dueros_disconnect()
    dueros_service -> service_task  : duer_que_send \n (DUER_CMD_QUIT)
    service_task   -> lightduer     : duer_stop()
    service_task   -> service_task  : duer_state = \n SERVICE_STATE_IDLE
    audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_IDLE)
    alt .callback_func != NULL
    adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
    end
    end


    == --Destory dueros service-- ==
    autonumber 90 "<b>(<u>##</u>)"
    [-> audio_service                 : audio_service_destroy()
    alt .service_desotry != NULL
    audio_service  -> dueros_service  : .service_desotry() \n ==> dueros_destory()
    dueros_service -> service_task    : duer_que_send \n (DUER_CMD_DESTROY)
    service_task   -> lightduer     : duer_voice_stop()
    service_task   -> service_task  : duer_state = \n SERVICE_STATE_IDLE
    dueros_service -> service_task    : (destory task)
    deactivate service_task 
    end

**对像说明：**

* **esp_dispatcher_dueros_app.c**: 某个应用程序
* **dueros_service.c**: 小度助手音频子服务
* **dueros_task()**: 音频子服务的内部任务
* **audio_service.c**：音频服务

**流程说明：**

1. 应用程序 esp_dispatcher_dueros_app.c 调用 DuerOS Service 音频子服务 ``dueros_service_create()``。
2. dueros_service.c 调用 ``audio_service_create()``, 并会将 ``.service_destroy`` ， ``.service_start`` , ``.service_stop`` , ``.service_connect`` , ``.service_disconnect`` ,  等回调函数作为参数的字段传入。 同时也会将自已的地址，作为 ``.user_data`` 参数字段传入。 因 DuerOS Service 音频子服务需要创建内部任务，也会将内部任务函数 dueros_task() 作为 ``.task_func`` 参数字段传入。
3. audio_service.c 将上述回调函数和 ``.user_data`` 保存下来。因为 ``.task_func`` 不为空(实际上是 ``.task_stack > 0``)，故同时创建内部任务。
4. dueros_task() 调用 ``duer_initialize()`` 初始化 lightduer 。
5. dueros_task() 注册事件回调函数 ``duer_set_event_callback(duer_event_hook)`` 。
6. ``duer_init_device_info()`` 。
7. ``duer_register_device_info_ops()`` 。
8. dueros_task() 设置状态为 ``SERVICE_STATE_IDLE`` 。

10. esp_dispatcher_dueros_app.c 注册分发器 esp_dispather 功能
``esp_dispatcher_reg_exe_func( ACTION_EXE_TYPE_DUER_CONNECT, dueros_action_connect)``。 当收到 ``ACTION_EXE_TYPE_DUER_CONNECT`` 指令时，执行 ``dueros_action_connect()`` 。 
11. esp_dispatcher_dueros_app.c 注册分发器 esp_dispather 功能
``esp_dispatcher_reg_exe_func( ACTION_EXE_TYPE_DUER_DISCONNECT, dueros_action_disconnect)``。 当收到 ``ACTION_EXE_TYPE_DUER_DISCONNECT`` 指令时，执行 ``dueros_action_disconnect()`` 。

12. esp_dispatcher_dueros_app.c 调用 ``audio_service_set_callback()`` 设置事件回调函数 ``.callback_func`` 。

20. esp_dispatcher_dueros_app.c 接收到事件 ``wifi_service_cb(WIFI_SERV_EVENT_CONNECTED)``。
21. esp_dispatcher_dueros_app.c 调用执行分发器功能 ``esp_dispatcher_execute(ACTION_EXE_TYPE_DUER_CONNECT)``。
22. esp_dispatcher 发送消息 ``ESP_DISPCH_EVENT_TYPE_EXE`` 给其内部任务。
23. esp_dispatcher 内部任务执行动作, 实际上是调用 ``dueros_action_connect()`` 。
24. dueros_action 调用 ``audio_service_connect()`` 。
25. 因 ``.service_connect`` 不为空，audio_service.c 会执行 ``.service_connect``, 实际上是执行 ``dueros_connect()`` 。
26. dueros_server.c 发送 ``DUER_CMD_LOGIN`` 消息给内部任务。
27. dueros_task() 调用 ``duer_login()`` 。
28. dueros_task() 调用 ``duer_start()`` 启动 lightduer 。
29. dueros_task() 设置状态为 ``SERVICE_STATE_IDLE`` 。
30. dueros_task() 调用音频服务事件回调函数 ``audio_service_callback(SERVICE_STATE_CONNECTING)`` 。
31. audio_service.c 调用应用的事件回调函数 ``duer_callback()`` 。

35. audio_service.c 接收到 lightduer 的开始事件 ``DUER_EVENT_START`` 。
36. audio_service.c 通知内部任务已连接 ``DUER_CMD_CONNECTED`` 。
37. dueros_task() 设置状态为 ``SERVICE_STATE_CONNECTED`` 。
38. dueros_task() 调用音频服务事件回调函数 ``audio_service_callback(SERVICE_STATE_CONNECTED)`` 。
39. audio_service.c 调用应用的事件回调函数 ``duer_callback()`` 。

45. esp_dispatcher_dueros_app.c 接收到事件 ``rec_engine_cb (REC_EVENT_VAD_START)``。
46. esp_dispatcher_dueros_app.c 调用 ``audio_service_start()``。
47. ``.service_start`` 不为空，被执行（ 实际上是执行 ``dueros_start()`` ）。
48. audio_service.c 发送消息 ``DUER_CMD_START`` 给内部任务。
49. 内部任务 dueros_task()  调用 ``duer_voice_start(RECORD_SAMPLE_RATE)`` 。
50. 内部任务 dueros_task()  调用 ``duer_dcs_on_listen_started()`` 。
51. dueros_task() 设置状态为 ``SERVICE_STATE_RUNNING`` 。
52. dueros_task() 调用音频服务事件回调函数 ``audio_service_callback(SERVICE_STATE_CONNECTED)`` 。
53. audio_service.c 调用应用的事件回调函数 ``duer_callback()`` 
54. dueros_task() 从 rec_engine 读数据 ``rec_engine_data_read()`` 。
55. dueros_task() 调用 ``duer_voice_send()`` 发送语音 。

60. esp_dispatcher_dueros_app.c 接收到事件 ``rec_engine_cb()``, 接收到 ``REC_EVENT_VAD_STOP`` 或 ``REC_EVENT_WAKEUP_END`` 任一事件。
61. esp_dispatcher_dueros_app.c 调用 ``audio_service_stop()``。
62. ``.service_stop`` 不为空，被执行（ 实际上是执行 ``dueros_stop()`` ）。
63. audio_service.c 发送消息 ``DUER_CMD_STOP`` 给内部任务。
64. 内部任务 dueros_task()  调用 ``duer_voice_stop()`` 。
65. dueros_task() 设置状态为 ``SERVICE_STATE_STOPPED`` 。
66. dueros_task() 调用音频服务事件回调函数 ``audio_service_callback(SERVICE_STATE_STOPPED)`` 。
67. audio_service.c 调用应用的事件回调函数 ``duer_callback()`` 

70. esp_dispatcher_dueros_app.c 接收到事件 ``wifi_service_cb(WIFI_SERV_EVENT_DISCONNECTED)``。
71. esp_dispatcher_dueros_app.c 调用执行分发器功能 ``esp_dispatcher_execute(ACTION_EXE_TYPE_DUER_CONNECT)``。
72. esp_dispatcher 发送消息 ``ESP_DISPCH_EVENT_TYPE_EXE`` 给其内部任务。
73. esp_dispatcher 内部任务执行动作, 实际上是调用 ``dueros_action_disconnect()`` 。
74. dueros_action 调用 ``audio_service_disconnect()`` 。

75. audio_service.c 接收到 lightduer 的停止事件 ``DUER_EVENT_STOPPED`` 。
76. dueros_service.c 调用 ``audio_service_disconnect()`` 。

    70~74 或 75~76 二者选其一。

77. 因 ``.service_connect`` 不为空，audio_service.c 会执行 ``.service_connect``, 实际上是执行 ``dueros_connect()`` 。
78. dueros_server.c 发送 ``DUER_CMD_QUIT`` 消息给内部任务。
79. dueros_task() 调用 ``duer_stop()`` 停止 lightduer 。
80. dueros_task() 设置状态为 ``SERVICE_STATE_IDLE`` 。
81. dueros_task() 调用音频服务事件回调函数 ``audio_service_callback(SERVICE_STATE_CONNECTING)`` 。
82. audio_service.c 调用应用的事件回调函数 ``duer_callback()`` 。

90. (实际上没有代码)调用 ``audio_service_destroy()``, 销毁 DuerOS Service音频子服务。
91. 因 ``.service_destroy`` 不为空， 其被 audio_service.c 调用（ 实际上是执行 ``dueros_destory()`` ）。
92. dueros_server.c 发送 ``DUER_CMD_DESTROY`` 消息给内部任务。
93. dueros_task() 调用 ``duer_voice_stop()`` 。
94. dueros_task() 设置状态为 ``SERVICE_STATE_IDLE`` 。
95. dueros_service.c 中止内部任务 dueros_task() 。


4. API
=========

* dueros_service_create()
* audio_service_create
* audio_service_set_callback()

    .. uml::

        caption Create dueros service & set callback

        box "esp_dispatcher_dueros"
        participant "esp_dispatcher\n_dueros_app.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "esp_dispatcher.c"  as esp_dispatcher  order 20
        participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
        participant "audio_service.c"  as audio_service  order 40
        end box

        box "esp_actions"
        participant "dueros_action.c"  as dueros_action  order 50
        end box

        box "dueros_service" #LightBlue 
        participant "dueros_service.c" as dueros_service order 60
        participant "dueros_task()"    as service_task order 70
        end box

        box "dueros"
        participant "lightduer" as lightduer order 80
        end box
        
        == Create dueros service & set callback ==
        autonumber 1 "<b>(<u>##</u>)"
        adf_app        -> dueros_service : audio_serv = \n dueros_service_create()
        audio_service  <- dueros_service : audio_service_create({\n .service_destroy = dueros_destroy, \n .service_start = dueros_start, \n .service_stop = dueros_stop, \n .service_connect = dueros_connect, \n .service_disconnect = dueros_disconnect, \n .task_func  = dueros_task, \n .user_data = (void *)serv})

        alt .task_func!=NULL (实际上是 .task_stack > 0)
        audio_service -> service_task : xTaskCreatePinnedToCore({.task_func})
        activate service_task

        service_task -> lightduer : duer_initialize()
        service_task -> lightduer : duer_set_event_callback \n (**duer_event_hook**)
        service_task -> service_task : duer_init_device_info()
        service_task -> lightduer : duer_register_device_info_ops()
        service_task -> service_task : duer_state = \n SERVICE_STATE_IDLE
        end

        == Register dueros service execution type ==
        autonumber 10 "<b>(<u>##</u>)"
        adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE \n _DUER_CONNECT, \n dueros_action_connect)
        adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE \n _DUER_DISCONNECT, \n dueros_action_disconnect)

        == Set event callback ==
        adf_app        -> audio_service  : audio_service_set_callback \n ({.callback_func=duer_callback})


* audio_service_connect()
* dueros_connect()

    .. uml::

        caption Connect dueros service

        box "esp_dispatcher_dueros"
        participant "esp_dispatcher\n_dueros_app.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "esp_dispatcher.c"  as esp_dispatcher  order 20
        participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
        participant "audio_service.c"  as audio_service  order 40
        end box

        box "esp_actions"
        participant "dueros_action.c"  as dueros_action  order 50
        end box

        box "dueros_service" #LightBlue 
        participant "dueros_service.c" as dueros_service order 60
        participant "dueros_task()"    as service_task order 70
        end box

        box "dueros"
        participant "lightduer" as lightduer order 80
        end box

        == Connect dueros service ==
        autonumber 20 "<b>(<u>##</u>)"
        adf_app        <-]              : wifi_service_cb \n (WIFI_SERV_EVENT_CONNECTED)
        adf_app        -> esp_dispatcher : esp_dispatcher_execute \n (ACTION_EXE_TYPE \n _DUER_CONNECT)
        esp_dispatcher -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
        dispatcher_task -> dueros_action : exe_item->exe_func \n (instance, arg, result) \n ==>　dueros_action_connect()
        dueros_action  -> audio_service : audio_service_connect()
        alt .service_connect != NULL
        audio_service -> dueros_service : .service_connect() \n ==> dueros_connect()
        dueros_service -> service_task  : duer_que_send \n (DUER_CMD_LOGIN)
        service_task   -> service_task  : duer_login()
        service_task   -> lightduer     : duer_start()
        service_task   -> service_task  : duer_state = \n SERVICE_STATE_CONNECTING
        audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_CONNECTING)
        alt .callback_func != NULL
        adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
        end
        end


        == Connected dueros service ==
        autonumber 35 "<b>(<u>##</u>)"
        dueros_service <- lightduer    : duer_event_hook(DUER_EVENT_STARTED)
        dueros_service -> service_task : duer_que_send \n (DUER_CMD_CONNECTED)
        service_task   -> service_task  : duer_state = \n SERVICE_STATE_CONNECTED
        audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_CONNECTED)
        alt .callback_func != NULL
        adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
        end


* audio_service_start()
* dueros_start()

    .. uml::

        caption Start dueros service

        box "esp_dispatcher_dueros"
        participant "esp_dispatcher\n_dueros_app.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "esp_dispatcher.c"  as esp_dispatcher  order 20
        participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
        participant "audio_service.c"  as audio_service  order 40
        end box

        box "esp_actions"
        participant "dueros_action.c"  as dueros_action  order 50
        end box

        box "dueros_service" #LightBlue 
        participant "dueros_service.c" as dueros_service order 60
        participant "dueros_task()"    as service_task order 70
        end box

        box "dueros"
        participant "lightduer" as lightduer order 80
        end box

        == Start dueros service ==
        autonumber 45 "<b>(<u>##</u>)"
        adf_app       <-]              : rec_engine_cb \n (REC_EVENT_VAD_START)
        adf_app       -> audio_service : audio_service_start()
        alt .service_start != NULL
        audio_service -> dueros_service : .service_start() \n ==> dueros_start()
        dueros_service -> service_task : duer_que_send \n (DUER_CMD_START)
        service_task   -> lightduer    : duer_voice_start \n (RECORD_SAMPLE_RATE)
        service_task   -> lightduer    : duer_dcs_on \n _listen_started()
        service_task   -> service_task  : duer_state = \n SERVICE_STATE_RUNNING
        audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_RUNNING)
        alt .callback_func != NULL
        adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
        end
        loop 从 rec_engine 读完所有数据
        service_task   ->]              : rec_engine_data_read()
        service_task   -> lightduer     : duer_voice_send()
        end
        end

* audio_service_stop()
* dueros_stop()

    .. uml::

        caption Stop dueros service

        box "esp_dispatcher_dueros"
        participant "esp_dispatcher\n_dueros_app.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "esp_dispatcher.c"  as esp_dispatcher  order 20
        participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
        participant "audio_service.c"  as audio_service  order 40
        end box

        box "esp_actions"
        participant "dueros_action.c"  as dueros_action  order 50
        end box

        box "dueros_service" #LightBlue 
        participant "dueros_service.c" as dueros_service order 60
        participant "dueros_task()"    as service_task order 70
        end box

        box "dueros"
        participant "lightduer" as lightduer order 80
        end box

        == Stop dueros service ==
        autonumber 60 "<b>(<u>##</u>)"
        adf_app         <-]              : rec_engine_cb \n (REC_EVENT_VAD_STOP \n | REC_EVENT_WAKEUP_END)
        adf_app         -> audio_service : audio_service_stop()
        alt .service_stop != NULL
        audio_service -> dueros_service : .service_stop() \n ==> dueros_stop()
        dueros_service -> service_task : duer_que_send \n (DUER_CMD_STOP)
        service_task   -> lightduer    : duer_voice_stop()
        service_task   -> service_task  : duer_state = \n SERVICE_STATE_STOPPED
        audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_STOPPED)
        alt .callback_func != NULL
        adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
        end
        end

* audio_service_disconnect()
* dueros_disconnect()

    .. uml::

        caption Disconnect dueros service

        box "esp_dispatcher_dueros"
        participant "esp_dispatcher\n_dueros_app.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "esp_dispatcher.c"  as esp_dispatcher  order 20
        participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
        participant "audio_service.c"  as audio_service  order 40
        end box

        box "esp_actions"
        participant "dueros_action.c"  as dueros_action  order 50
        end box

        box "dueros_service" #LightBlue 
        participant "dueros_service.c" as dueros_service order 60
        participant "dueros_task()"    as service_task order 70
        end box

        box "dueros"
        participant "lightduer" as lightduer order 80
        end box

        == Disconnect dueros service ==
        autonumber 70 "<b>(<u>##</u>)"
        alt Wi-Fi 事件
        adf_app        <-]              : wifi_service_cb \n (WIFI_SERV_EVENT_DISCONNECTED)
        adf_app        -> esp_dispatcher : esp_dispatcher_execute \n (ACTION_EXE_TYPE \n _DUER_DISCONNECT)
        esp_dispatcher -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
        dispatcher_task -> dueros_action : exe_item->exe_func \n (instance, arg, result) \n ==>　dueros_action_disconnect()
        dueros_action  -> audio_service : audio_service_disconnect()
        else duer 事件
        dueros_service -> lightduer      : duer_event_hook(DUER_EVENT_STOPPED)
        dueros_service -> audio_service  : audio_service_disconnect()
        end

        alt .service_disconnect != NULL
        audio_service -> dueros_service   : .service_disconnect() \n ==> dueros_disconnect()
        dueros_service -> service_task  : duer_que_send \n (DUER_CMD_QUIT)
        service_task   -> lightduer     : duer_stop()
        service_task   -> service_task  : duer_state = \n SERVICE_STATE_IDLE
        audio_service  <- service_task  : audio_service_callback \n (SERVICE_STATE_IDLE)
        alt .callback_func != NULL
        adf_app       <- audio_service  : .callback_func() \n ==> duer_callback()
        end
        end

* audio_service_destroy()
* dueros_destory()

    .. uml::

        caption Destory dueros service

        box "esp_dispatcher_dueros"
        participant "esp_dispatcher\n_dueros_app.c" as adf_app  order 10
        end box

        box "esp_dispatcher"
        participant "esp_dispatcher.c"  as esp_dispatcher  order 20
        participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
        participant "audio_service.c"  as audio_service  order 40
        end box

        box "esp_actions"
        participant "dueros_action.c"  as dueros_action  order 50
        end box

        box "dueros_service" #LightBlue 
        participant "dueros_service.c" as dueros_service order 60
        participant "dueros_task()"    as service_task order 70
        end box

        box "dueros"
        participant "lightduer" as lightduer order 80
        end box
        
        == --Destory dueros service-- ==
        autonumber 90 "<b>(<u>##</u>)"
        [-> audio_service                 : audio_service_destroy()
        alt .service_desotry != NULL
        audio_service  -> dueros_service  : .service_desotry() \n ==> dueros_destory()
        dueros_service -> service_task    : duer_que_send \n (DUER_CMD_DESTROY)
        service_task   -> lightduer     : duer_voice_stop()
        service_task   -> service_task  : duer_state = \n SERVICE_STATE_IDLE
        dueros_service -> service_task    : (destory task)
        deactivate service_task 
        end
