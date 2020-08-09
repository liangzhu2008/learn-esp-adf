分发器 esp_dispatcher.c
###########################

`esp_dispatcher.c`__, `esp_dispatcher_dueros_app.c`__.

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_dispatcher/esp_dispatcher.c
.. __: https://github.com/espressif/esp-adf/blob/master/examples/advanced_examples/esp_dispatcher_dueros/main/esp_dispatcher_dueros_app.c


序列图
============

.. uml::

    caption esp_dispatcher.c

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher\n_dueros_app.c"         as adf_app           order 10
    end box

    box "esp_dispatcher" #LightBlue
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    end box
      
    == Create dispatcher ==
    adf_app -> dispatcher_comp : dispatcher = esp_dispatcher_create(&d_cfg)

    dispatcher_comp ->] : impl->result_que = xQueueCreate()
    dispatcher_comp ->] : impl->exe_que = xQueueCreate()
    dispatcher_comp ->] : impl->mutex = mutex_create()
    dispatcher_comp ->] : STAILQ_INIT(&impl->exe_list)
    dispatcher_comp -> dispatcher_task : xTaskCreatePinnedToCore()
    activate dispatcher_task 

    == Register execution function ==
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_CONNECT, \nwifi_action_connect)
    dispatcher_comp ->]         : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->player, \nACTION_EXE_TYPE_AUDIO_PLAY, \nplayer_action_play)
    dispatcher_comp ->]         : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    ...  ...
    
    == Execution function ==
    adf_app -> dispatcher_comp  : esp_dispatcher_execute(d->dispatcher, \nACTION_EXE_TYPE_AUDIO_PLAY, \nNULL, NULL)
    dispatcher_comp ->]         : **mutex_lock(impl->mutex)**
    dispatcher_comp -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
    dispatcher_task ->]         : exe_item->exe_func \n (instance, arg, result)
    dispatcher_task -> dispatcher_comp : xQueueSend(dispch->result_que, \n**ESP_OK** or **ESP_ERR_ADF_NOT_SUPPORT**)
    dispatcher_comp ->]         : xQueueReceive(impl->result_que)
    dispatcher_comp ->]         : **mutex_unlock(impl->mutex)**

    == Destory dispatcher ==
    [-> dispatcher_comp : esp_dispatcher_destroy(dispatcher)

    dispatcher_comp -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_CMD**)
    dispatcher_task -> dispatcher_comp : xQueueSend(dispch->result_que, **ESP_OK**)
    dispatcher_task ->] : vTaskDelete(NULL)
    deactivate dispatcher_task 

    dispatcher_comp ->] : xQueueReceive(impl->result_que)
    dispatcher_comp ->] : STAILQ_FOREACH(item, &impl->exe_list, ...) \n { STAILQ_REMOVE(&impl->exe_list, item, ...) }
    dispatcher_comp ->] : vQueueDelete(impl->result_que)
    dispatcher_comp ->] : vQueueDelete(impl->exe_que)
    dispatcher_comp ->] : mutex_destroy(impl->mutex)
    

.. note::

    函数 esp_dispatcher_execute() 内部，有 **加锁** 机制 ——— 调用了 mutex_lock(impl->mutex) 和 mutex_unlock(impl->mutex)。
    
    软件设计的时候要小心，不要因此形成 **死锁** 的隐患。


esp_dispatcher_create()
=================================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher\n_dueros_app.c"         as adf_app           order 10
    end box

    box "esp_dispatcher" #LightBlue
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    end box

    == Create dispatcher ==
    adf_app -> dispatcher_comp : dispatcher = esp_dispatcher_create(&d_cfg)

    dispatcher_comp ->] : impl->result_que = xQueueCreate()
    dispatcher_comp ->] : impl->exe_que = xQueueCreate()
    dispatcher_comp ->] : impl->mutex = mutex_create()
    dispatcher_comp ->] : STAILQ_INIT(&impl->exe_list)
    dispatcher_comp -> dispatcher_task : xTaskCreatePinnedToCore()
    activate dispatcher_task 


esp_dispatcher_reg_exe_func()
=====================================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher\n_dueros_app.c"         as adf_app           order 10
    end box

    box "esp_dispatcher" #LightBlue
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    end box

    == Register execution function ==
    adf_app -> dispatcher_comp : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_CONNECT, \nwifi_action_connect)
    dispatcher_comp ->] : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    adf_app -> dispatcher_comp : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->player, \nACTION_EXE_TYPE_AUDIO_PLAY, \nplayer_action_play)
    dispatcher_comp ->] : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    ...  ...
    


esp_dispatcher_execute()
================================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher\n_dueros_app.c"         as adf_app           order 10
    end box

    box "esp_dispatcher" #LightBlue
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    end box
    
    == Execution function ==
    adf_app -> dispatcher_comp : esp_dispatcher_execute(d->dispatcher, \nACTION_EXE_TYPE_AUDIO_PLAY, \nNULL, NULL)
    dispatcher_comp ->] : **mutex_lock(impl->mutex)**
    dispatcher_comp -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
    dispatcher_task ->]                : exe_item->exe_func \n (instance, arg, result)
    dispatcher_task -> dispatcher_comp : xQueueSend(dispch->result_que, \n**ESP_OK** or **ESP_ERR_ADF_NOT_SUPPORT**)
    dispatcher_comp ->] : xQueueReceive(impl->result_que)
    dispatcher_comp ->] : **mutex_unlock(impl->mutex)**   

.. note::

    函数 esp_dispatcher_execute() 内部，有 **加锁** 机制 ——— 调用了 mutex_lock(impl->mutex) 和 mutex_unlock(impl->mutex)。

    软件设计的时候要小心，不要因此形成 **死锁** 的隐患。


esp_dispatcher_destroy()
====================================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher\n_dueros_app.c"         as adf_app           order 10
    end box

    box "esp_dispatcher" #LightBlue
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    end box

    == Destory dispatcher ==
    [-> dispatcher_comp : esp_dispatcher_destroy(dispatcher)

    dispatcher_comp -> dispatcher_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_CMD**)
    dispatcher_task -> dispatcher_comp : xQueueSend(dispch->result_que, **ESP_OK**)
    dispatcher_task ->] : vTaskDelete(NULL)
    deactivate dispatcher_task 

    dispatcher_comp ->] : xQueueReceive(impl->result_que)
    dispatcher_comp ->] : STAILQ_FOREACH(item, &impl->exe_list, ...) \n { STAILQ_REMOVE(&impl->exe_list, item, ...) }
    dispatcher_comp ->] : vQueueDelete(impl->result_que)
    dispatcher_comp ->] : vQueueDelete(impl->exe_que)
    dispatcher_comp ->] : mutex_destroy(impl->mutex)
