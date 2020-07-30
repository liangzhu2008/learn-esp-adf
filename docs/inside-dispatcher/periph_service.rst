periph_service.c
#############################


概述
======

**外设服务** Periph_Servcie 的出发点，应该是用于 OTA_Service、Input_Key_Servcie 等 **具体服务** 的基础类。但是当前的代码实现，后者实现时并没有完全继承（或覆盖）前者————有些 API 需要调用后者的，有些仍然需要调用前者的。大致的原则：优先调用具体服务的API；否则调用外设服务的API（这时需要注意具体服务是如何实现的）。


.. uml::

    class periph_service_handle_t {
        __ private data __
        -periph_service_ctrl **service_start**;
        -periph_service_ctrl **service_stop**;
        -periph_service_ctrl **service_destroy**;
        -periph_service_io   **service_ioctl**;
        -periph_service_cb   **callback_func**;
        -void*               user_cb_ctx;
        -char*               service_name;
        -TaskHandle_t        task_handle;
        -audio_thread_t      audio_thread;
        -void*               user_data;

        __ public method __
        +periph_service_create()
        +periph_service_destroy()
        +periph_service_start()
        +periph_service_stop()
        +periph_service_set_callback()
        +periph_service_callback()
        +periph_service_set_data()
        +periph_service_get_data()
        +periph_service_ioctl()
    }


.. uml::

    caption periph_service.c

    box "xxx_app"
    participant "xxx_app.c"         as adf_app  order 10
    end box

    box "esp_dispatcher" #LightBlue
    participant "periph_service.c"  as periph_service  order 10
    end box

    box "xxx_service" 
    participant "xxx_service.c"   as xxx_service  order 30
    participant "xxx_service_task()" as service_task  order 40
    end box
      
    == Create peripheral service & set callback ==
    autonumber 1 "<b>(<u>##</u>)"
    adf_app        -> xxx_service : //xxx_service_create(--{.callback=app_event_cb}--)//
    periph_service <- xxx_service : periph_service_create({.task_stack = task_stack, \n .task_prio  = task_prio, \n .task_core  = task_core, \n .task_func  = xxx_service_task, \n .service_start = xxx_service_start, \n .service_stop = xxx_service_stop, \n .service_ioctl = xxx_service_ioctl, \n .service_destroy = xxx_service_destroy, \n .user_data = (void *)serv})

    alt task_stack > 0
    periph_service -> service_task : audio_thread_create({.task_func})
    activate service_task
    end

    periph_service <- xxx_service : --periph_service_set_data(data)--
    periph_service <- xxx_service : periph_service_set_callback({.callback_func})

    == Start peripheral service ==
    autonumber 10 "<b>(<u>##</u>)"
    adf_app         -> periph_service : periph_service_start()
    alt .service_start != NULL
    periph_service -> xxx_service : .service_start() ==> xxx_service_start()
    ...
    end

    == Execute callback ==
    autonumber 20 "<b>(<u>##</u>)"
    service_task    <-] 
    periph_service  <- service_task : periph_service_callback()
    alt .callback_func != NULL
    adf_app         <- periph_service : .callback_func() ==> //app_event_cb()//
    ...
    end

    == Stop peripheral service ==
    autonumber 30 "<b>(<u>##</u>)"
    adf_app         -> periph_service : periph_service_stop()
    alt .service_stop != NULL
    periph_service -> xxx_service : .service_stop() ==> xxx_service_stop()
    ...
    end

    == Destory peripheral service ==
    autonumber 40 "<b>(<u>##</u>)"
    adf_app        -> xxx_service : //xxx_service_destroy()//
    xxx_service    -> service_task : (destory task)
    deactivate service_task 
    periph_service <- xxx_service : periph_service_destroy()


.. note::

    xxxx.


periph_service_create()
========================



periph_service_destroy()
========================



periph_service_start()
========================


periph_service_stop()
========================



periph_service_set_callback()
==============================


periph_service_callback()
===========================


periph_service_set_data()
===========================

periph_service_get_data()
==========================

periph_service_ioctl()
========================


