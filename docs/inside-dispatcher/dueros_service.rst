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
    participant "audio_service.c"  as audio_service  order 30
    end box

    box "esp_actions"
    participant "dueros_action.c"  as dueros_action  order 40
    end box

    box "dueros_service" #LightBlue 
    participant "dueros_service.c" as dueros_service order 60
    participant "dueros_task()"    as service_task order 70
    end box
      
    == Create audio service & set callback ==
    autonumber 1 "<b>(<u>##</u>)"
    adf_app        -> dueros_service : audio_serv = \n dueros_service_create()
    audio_service  <- dueros_service : audio_service_create({\n .service_destroy = dueros_destroy, \n .service_start = dueros_start, \n .service_stop = dueros_stop, \n .service_connect = dueros_connect, \n .service_disconnect = dueros_disconnect, \n .task_func  = dueros_task, \n .user_data = (void *)serv})

    alt .task_func!=NULL (实际上是 .task_stack > 0)
    audio_service -> service_task : xTaskCreatePinnedToCore({.task_func})
    activate service_task
    end

    == Register dueros service execution type ==
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_DUER_VOLUME_ADJ, \n duer_dcs_action_vol_adj)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_DUER_AUDIO, \n duer_dcs_action_audio_play)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_DUER_SPEAK, \n duer_dcs_action_speak)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_AUDIO_GET_PROGRESS_BYTE, \n duer_dcs_action_get_progress)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_AUDIO_PAUSE, \n duer_dcs_action_audio_pause)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_AUDIO_RESUME, \n duer_dcs_action_audio_resume)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_AUDIO_VOLUME_GET, \n duer_dcs_action_get_state)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_AUDIO_VOLUME_SET, \n duer_dcs_action_vol_set)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_AUDIO_STOP, \n duer_dcs_action_audio_stop)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_DUER_CONNECT, \n dueros_action_connect)
    adf_app        -> esp_dispatcher  :  esp_dispatcher_reg_exe_func \n (ACTION_EXE_TYPE_DUER_DISCONNECT, \n dueros_action_disconnect)

    == Set event callback ==
    adf_app        -> audio_service  : audio_service_set_callback \n ({.callback_func=duer_callback})

    == Start audio service ==
    autonumber 20 "<b>(<u>##</u>)"
    adf_app       <-]              : rec_engine_cb \n (REC_EVENT_VAD_START)
    adf_app       -> audio_service : audio_service_start()
    alt .service_start != NULL
    audio_service -> dueros_service : .service_start() \n ==> dueros_start()
    end

    == Connect audio service ==
    autonumber 30 "<b>(<u>##</u>)"
    adf_app        <-]              : wifi_service_cb \n (WIFI_SERV_EVENT_CONNECTED)
    adf_app        -> esp_dispatcher : esp_dispatcher_execute \n (ACTION_EXE_TYPE_DUER_CONNECT)
    esp_dispatcher -> dueros_action : **send msg to new task \n dueros_action_connect()**
    dueros_action  -> audio_service : audio_service_connect()
    alt .service_connect != NULL
    audio_service -> dueros_service : .service_connect() \n ==> dueros_connect()
    end

    == Execute callback ==
    autonumber 40 "<b>(<u>##</u>)"
    service_task    <-] 
    audio_service  <- service_task : audio_service_callback()
    alt .callback_func != NULL
    adf_app       <- audio_service : .callback_func() \n ==> duer_callback()
    end

    == Disconnect audio service ==
    autonumber 50 "<b>(<u>##</u>)"
    adf_app        <-]              : wifi_service_cb \n (WIFI_SERV_EVENT_DISCONNECTED)
    adf_app        -> esp_dispatcher : esp_dispatcher_execute \n (ACTION_EXE_TYPE_DUER_DISCONNECT)
    esp_dispatcher -> dueros_action : **send msg to new task \n dueros_action_disconnect()**
    dueros_action  -> audio_service : audio_service_disconnect()
    alt .service_disconnect != NULL
    audio_service -> dueros_service   : .service_disconnect() \n ==> dueros_disconnect()
    end

    == Stop audio service ==
    autonumber 60 "<b>(<u>##</u>)"
    adf_app         <-]              : rec_engine_cb \n (REC_EVENT_VAD_STOP \n | REC_EVENT_WAKEUP_END)
    adf_app         -> audio_service : audio_service_stop()
    alt .service_stop != NULL
    audio_service   -> dueros_service   : .service_stop() \n ==> dueros_stop()
    end

    == --Destory audio service-- ==
    autonumber 70 "<b>(<u>##</u>)"
    [-> audio_service                 : audio_service_destroy()
    alt .service_desotry != NULL
    audio_service  -> dueros_service  : .service_desotry() \n ==> dueros_destory()    
    dueros_service    -> service_task : (destory task)
    deactivate service_task 
    end

.. **对像说明：**

.. * **xxx_app.c**: 某个用户程序
.. * **dueros_service.c**: 某个音频子服务
.. * **dueros_task()**: 音频子服务的内部任务
.. * **audio_service.c**：音频服务

.. **流程说明：**

.. 1. xxx_app.c 调用某个音频子服务 ``dueros_service_create()``。

.. 2. dueros_service.c 调用 ``audio_service_create()``, 并会将 ``.service_destroy`` ， ``.service_start`` , ``.service_stop`` , ``.service_connect`` , ``.service_disconnect`` ,  等回调函数作为参数的字段传入。 同时也会将自已的地址，作为 ``.user_data`` 参数字段传入。 若音频子服务需要创建内部任务，则会将内部任务函数作为 ``.task_func`` 参数字段传。

.. 3. audio_service.c 将上述回调函数和 ``.user_data`` 保存下来。若 ``.task_func`` 不为空(实际上是 ``.task_stack > 0``)，则创建内部任务。

.. 4. 若有需要，xxx_app.c 调用 ``audio_service_set_callback()`` 设置事件回调函数 ``.callback_func`` 。

.. 5. ``audio_service_set_data()`` 此函数有缺陷，且实际上没有调用过。


.. 10. xxx_app.c 调用 ``audio_service_start()``。
.. 11. 若 ``.service_start`` 不为空，则会被执行。

.. 20. xxx_app.c 调用 ``audio_service_connect()``。
.. 21. 若 ``.service_connect`` 不为空，则会被执行。

.. 30. 内部任务 dueros_task() 收到外部事件。
.. 31. 内部任务 dueros_task() 调用 ``audio_service_callback()`` 。
.. 32. 若 ``.callback_func`` 不为空， 则会被执行。

.. 40. xxx_app.c 调用 ``audio_service_discconect()``。
.. 41. 若 ``.service_discconect`` 不为空，则会被执行。

.. 50. xxx_app.c 调用 ``audio_service_stop()``。
.. 51. 若 ``.service_stop`` 不为空，则会被执行。

.. 60. xxx_app.c 调用 ``audio_service_destroy()``, 销毁某个音频子服务。
.. 61. 若 ``.service_destroy`` 不为空， 则会被 audio_service.c 调用。
.. 62. dueros_service.c 中止内部任务 dueros_task() 。

..     *DuerOS Service 是 60, 61, 62 流程。*


.. .. note::

..     上述流程只表示一般做法。各音频子服务的实现，可能与上述流程并不完全一致。


.. 4. API
.. =========


.. * audio_service_create()

..     .. uml::

..         box "xxx_app"
..         participant "xxx_app.c"         as adf_app  order 10
..         end box

..         box "esp_dispatcher" #LightBlue
..         participant "audio_service.c"  as audio_service  order 20
..         end box

..         box "dueros_service" 
..         participant "dueros_service.c"   as dueros_service  order 30
..         participant "dueros_task()" as service_task  order 40
..         end box
        
..         == Create audio service & set callback ==
..         autonumber 1 "<b>(<u>##</u>)"
..         adf_app        -> dueros_service : dueros_service_create()
..         audio_service  <- dueros_service : audio_service_create({\n .service_destroy = dueros_service_destroy, \n .service_start = dueros_start, \n .service_stop = dueros_stop, \n .service_connect = dueros_connect, \n .service_disconnect = dueros_disconnect, \n .task_func  = dueros_task, \n .user_data = (void *)serv})

..         alt .task_func!=NULL (实际上是 .task_stack > 0)
..         audio_service -> service_task : xTaskCreatePinnedToCore({.task_func})
..         activate service_task
..         end

..         adf_app      -> audio_service : audio_service_set_callback \n ({.callback_func=app_event_cb})
..         audio_service  <- dueros_service : (--audio_service_set_data(data)--)

.. * audio_service_destroy()
.. * audio_service_start()
.. * audio_service_stop()
.. * audio_service_connect()
.. * audio_service_disconnect()
.. * audio_service_set_callback()
.. * audio_service_callback()
.. * audio_service_set_data()
.. * audio_service_get_data()


.. 5. 与音频子服务的映射
.. =================================================

.. 5.1 完全映射
.. ----------------------------

.. 同时包括了 **回调函数映射** 与 **API映射** 。

.. .. figure:: ../_static/inside-dispatcher/audio_service_full_map.png
..    :alt: audio service full map
..    :align: center

..    Audio  Service 与 各音频子服务的映射


.. 说明：

.. * BlueTooth Service 不是基于 Audio Servcie 实现的，与相差甚远，**无法列出对应关系** 。 

.. * **黑色粗体与紫色粗体文字** ：用户可调用的 API 函数。
.. * **无调用** ：提供了API, 但在 ESP_ADF 中没有调用过。
.. * **内部API,用户不可调用** ：供音频子服务调用的API。
.. * **空函数**：内部实现为空，或基本为空。
.. * ``.task_func`` ： 这不是 callback, 只是 ``audio_service_create()`` 的参数的一个字段。若这个字段非空，则会创建一个音频子服务的内部任务。
.. * ``audio_service_set_data()`` ：没有任何地方调用。实际上也 **不能被调用** ，该函数修改的 ``.user_data`` 字段，在 ``audio_service_create()`` 中已经被赋值了。

.. * ``audio_service_get_data()`` ：为各音频子服务提供的内部 API，用户不应该调用。	


.. 5.2 回调函数映射
.. -----------------------------

.. .. figure:: ../_static/inside-dispatcher/audio_service_callback_map.png
..    :alt: audio service callback map
..    :align: center

..    Audio  Service 与 各音频子服务的回调函数映射


.. 5.3 API 映射
.. ----------------------------

.. .. figure:: ../_static/inside-dispatcher/audio_service_api_map.png
..    :alt: audio service api map
..    :align: center

..    Audio  Service 与 各音频子服务的 API 映射


.. 上表进一步说明了如下原则： **除了创建 Create 与特殊 API 要调用音频子服务的API；一般都调用音频服务的 API** 。


