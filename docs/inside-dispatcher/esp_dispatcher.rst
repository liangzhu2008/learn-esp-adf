esp_dispatcher.c
##################

`esp_dispatcher.c`__, `esp_dispatcher_dueros_app.c`__.

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_dispatcher/esp_dispatcher.c
.. __: https://github.com/espressif/esp-adf/blob/master/examples/advanced_examples/esp_dispatcher_dueros/main/esp_dispatcher_dueros_app.c


.. uml::

    caption esp_dispatcher.c

    participant "esp_dispatcher\n_dueros_app.c" as esp_app order 10
    participant "esp_dispatcher.c\n" as esp_component order 50
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as component_task order 60
      
    == Create ==
    esp_app -> esp_component : dispatcher = esp_dispatcher_create(&d_cfg)

    esp_component ->] : impl->result_que = xQueueCreate()
    esp_component ->] : impl->exe_que = xQueueCreate()
    esp_component ->] : impl->mutex = mutex_create()
    esp_component ->] : STAILQ_INIT(&impl->exe_list)
    esp_component -> component_task : xTaskCreatePinnedToCore()
    activate component_task 

    == Register execution function ==
    esp_app -> esp_component : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_CONNECT, \nwifi_action_connect)
    esp_component ->] : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    esp_app -> esp_component : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->player, \nACTION_EXE_TYPE_AUDIO_PLAY, \nplayer_action_play)
    esp_component ->] : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    ...  ...
    
    == Execution function ==
    esp_app -> esp_component : esp_dispatcher_execute(d->dispatcher, \nACTION_EXE_TYPE_AUDIO_PLAY, \nNULL, NULL)
    esp_component ->] : **mutex_lock(impl->mutex)**
    esp_component -> component_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
    component_task -> esp_component : xQueueSend(dispch->result_que, \n**ESP_OK** or **ESP_ERR_ADF_NOT_SUPPORT**)
    esp_component ->] : xQueueReceive(impl->result_que)
    esp_component ->] : **mutex_unlock(impl->mutex)**

    == Destory ==
    [-> esp_component : esp_dispatcher_destroy(dispatcher)

    esp_component -> component_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_CMD**)
    component_task -> esp_component : xQueueSend(dispch->result_que, **ESP_OK**)
    component_task ->] : vTaskDelete(NULL)
    deactivate component_task 

    esp_component ->] : xQueueReceive(impl->result_que)
    esp_component ->] : STAILQ_FOREACH(item, &impl->exe_list, ...) \n { STAILQ_REMOVE(&impl->exe_list, item, ...) }
    esp_component ->] : vQueueDelete(impl->result_que)
    esp_component ->] : vQueueDelete(impl->exe_que)
    esp_component ->] : mutex_destroy(impl->mutex)
    

.. note::

    函数 esp_dispatcher_execute() 内部，有 **加锁** 机制 ——— 调用了 mutex_lock(impl->mutex) 和 mutex_unlock(impl->mutex)。
    
    软件设计的时候要小心，不要因此形成 **死锁** 的隐患。


esp_dispatcher_create()
========================

.. uml::

    hide footbox

    participant "esp_dispatcher\n_dueros_app.c" as esp_app order 10
    participant "esp_dispatcher.c\n" as esp_component order 50
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as component_task order 60

    == Create ==
    esp_app -> esp_component : dispatcher = esp_dispatcher_create(&d_cfg)

    esp_component ->] : impl->result_que = xQueueCreate()
    esp_component ->] : impl->exe_que = xQueueCreate()
    esp_component ->] : impl->mutex = mutex_create()
    esp_component ->] : STAILQ_INIT(&impl->exe_list)
    esp_component -> component_task : xTaskCreatePinnedToCore()
    activate component_task 


esp_dispatcher_reg_exe_func()
=============================

.. uml::

    hide footbox

    participant "esp_dispatcher\n_dueros_app.c" as esp_app order 10
    participant "esp_dispatcher.c\n" as esp_component order 50
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as component_task order 60

    == Register execution function ==
    esp_app -> esp_component : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_CONNECT, \nwifi_action_connect)
    esp_component ->] : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    esp_app -> esp_component : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->player, \nACTION_EXE_TYPE_AUDIO_PLAY, \nplayer_action_play)
    esp_component ->] : STAILQ_INSERT_TAIL(&impl->exe_list, item, ...)
    ...  ...
    


esp_dispatcher_execute()
=========================

.. uml::

    hide footbox

    participant "esp_dispatcher\n_dueros_app.c" as esp_app order 10
    participant "esp_dispatcher.c\n" as esp_component order 50
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as component_task order 60
    
    == Execution function ==
    esp_app -> esp_component : esp_dispatcher_execute(d->dispatcher, \nACTION_EXE_TYPE_AUDIO_PLAY, \nNULL, NULL)
    esp_component ->] : **mutex_lock(impl->mutex)**
    esp_component -> component_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_EXE**)
    component_task -> esp_component : xQueueSend(dispch->result_que, \n**ESP_OK** or **ESP_ERR_ADF_NOT_SUPPORT**)
    esp_component ->] : xQueueReceive(impl->result_que)
    esp_component ->] : **mutex_unlock(impl->mutex)**   

.. note::

    函数 esp_dispatcher_execute() 内部，有 **加锁** 机制 ——— 调用了 mutex_lock(impl->mutex) 和 mutex_unlock(impl->mutex)。

    软件设计的时候要小心，不要因此形成 **死锁** 的隐患。


esp_dispatcher_destroy()
========================

.. uml::

    hide footbox

    participant "esp_dispatcher\n_dueros_app.c" as esp_app order 10
    participant "esp_dispatcher.c\n" as esp_component order 50
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as component_task order 60

    == Destory ==
    [-> esp_component : esp_dispatcher_destroy(dispatcher)

    esp_component -> component_task : xQueueSend(impl->exe_que, \n**ESP_DISPCH_EVENT_TYPE_CMD**)
    component_task -> esp_component : xQueueSend(dispch->result_que, **ESP_OK**)
    component_task ->] : vTaskDelete(NULL)
    deactivate component_task 

    esp_component ->] : xQueueReceive(impl->result_que)
    esp_component ->] : STAILQ_FOREACH(item, &impl->exe_list, ...) \n { STAILQ_REMOVE(&impl->exe_list, item, ...) }
    esp_component ->] : vQueueDelete(impl->result_que)
    esp_component ->] : vQueueDelete(impl->exe_que)
    esp_component ->] : mutex_destroy(impl->mutex)
