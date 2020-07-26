在线升级服务 OTA Service
####################################

`ota_service.c`__, `ota_example.c`__.

.. __: https://github.com/espressif/esp-adf/blob/master/components/ota_service/ota_service.c
.. __: https://github.com/espressif/esp-adf/blob/master/examples/ota/main/ota_example.c

.. tip:: 

    如果图片太大，看不清楚。你可以 **在图片上点击鼠标右键** --> **在新窗口中打开图片** ，然后你可以放大、缩小、移动图片。



概述
============

.. role:: strike
   :class: strike

.. uml::

    caption ota_service.c

    box "ota"
    participant "ota_example.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n" as periph_service order 20
    end box

    box "ota_service" #LightBlue
    participant "ota_service.c \n" as ota_service order 50
    participant "ota_task()"       as ota_task    order 60
    end box
      

    == Create OTA Servcie ==
    autonumber 1 "<b>(<u>##</u>)"
    adf_app        -> ota_service   : ota_service_create({.evt_cb=**ota_service_cb**})
    ota_service    ->]              : srv_q = xQueueCreate()
    periph_service <- ota_service   : periph_service_create \n ({.task_func = **ota_task**, \n .service_start = **_ota_start**, \n .service_destroy = **_ota_destroy**})
    periph_service -> ota_task      : audio_thread_create(**ota_task**)
    activate ota_task 
    periph_service <- ota_service   : periph_service_set_callback(cb : {.evt_cb})
    periph_service -> periph_service : impl->callback_func = cb
    ota_task       -> ota_task      : ota->state = OTA_IDLE

    == Set OTA upgrade paramter ==
    autonumber 11 "<b>(<u>##</u>)"
    adf_app        -> ota_service   : ota_data_get_default_proc(&upgrade_list[0])
    adf_app        -> ota_service   : ota_data_get_default_proc(&upgrade_list[1])
    adf_app        -> ota_service   : ota_service_set_upgrade_param(upgrade_list)

    == Start OTA Servcie ==
    autonumber 21 "<b>(<u>##</u>)"
    adf_app        -> periph_service : periph_service_start()
    periph_service -> ota_service   : impl->service_start() ==> _ota_start()
    ota_service    -> ota_task      : ota_service_cmd_send \n (OTA_SERVICE_CMD_START)    
    ota_task       -> ota_task      : ota->state = OTA_START

    autonumber 31 "<b>(<u>##</u>)"
    loop "upgrade_list.length()"
        ota_service    <- ota_task  : ota_service_process()
        ota_service -> ota_service  : upgrade_info->prepare(), \n upgrade_info->need_upgrade(), \n upgrade_info->execute_upgrade(), \n upgrade_info->finished_check()
        periph_service <- ota_task  : periph_service_callback({.type = OTA_SERV_EVENT_TYPE_RESULT})
        adf_app  <- periph_service  : impl->callback_func() ==> \n ota_service_cb \n ({.type = OTA_SERV_EVENT_TYPE_RESULT})
        ...
    end
    ota_task       -> ota_task      : ota->state = OTA_END

    autonumber 41 "<b>(<u>##</u>)"
    alt "ota->upgrade_list[i].reboot_flag == true, 0<=i<upgrade_list.length()"
        ota_task ->]        : esp_restart()
    else 
        periph_service <- ota_task  : periph_service_callback \n ({.type = OTA_SERV_EVENT_TYPE_FINISH})
        adf_app  <- periph_service  : impl->callback_func() ==> \n ota_service_cb \n ({.type = OTA_SERV_EVENT_TYPE_FINISH})
        ...
        ota_task -> ota_task        : ota->state = OTA_IDLE
    end

    == Destory OTA Service ==
    autonumber 51 "<b>(<u>##</u>)"
    adf_app    -> periph_service    : periph_service_destroy()
    periph_service -> ota_service   : impl->service_destroy() ==> _ota_destroy()
    ota_service    -> ota_task      : ota_service_cmd_send \n (OTA_SERVICE_CMD_DESTROY)
    ota_task       -> ota_task      : ota->state =  OTA_DESTROY

    note over adf_app, ota_task
    1. "ota_service_create({.evt_cb=**ota_service_cb**})" 表示调用函数时传入一个参数，该参数的 evt_cb 字段的值为 ota_service_cb 。
    2. "periph_service_set_callback(cb : {.evt_cb})" 表示调用函数时，参数 cb 的值为 某个变量的 evt_cb 字段。
    3. "impl->callback_func() ==> ota_service_cb()" 表示执行的代码 impl->callback_func()  最终调用了 ota_service_cb() 这个回调函数。
    end note


.. note::

    OTA服务 OTA Service 既有回调函数 Callback，也有内部的任务 Task。


ota_service_create()
==========================

.. uml::

    caption ota_service.c

    box "ota"
    participant "ota_example.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n" as periph_service order 20
    end box

    box "ota_service" #LightBlue
    participant "ota_service.c \n" as ota_service order 50
    participant "ota_task()"       as ota_task    order 60
    end box
      

    == Create OTA Servcie ==
    autonumber 1 "<b>(<u>##</u>)"
    adf_app        -> ota_service   : ota_service_create({.evt_cb=**ota_service_cb**})
    ota_service    ->]              : srv_q = xQueueCreate()
    periph_service <- ota_service   : periph_service_create \n ({.task_func = **ota_task**, \n .service_start = **_ota_start**, \n .service_destroy = **_ota_destroy**})
    periph_service -> ota_task      : audio_thread_create(**ota_task**)
    activate ota_task 
    periph_service <- ota_service   : periph_service_set_callback(cb : {.evt_cb})
    periph_service -> periph_service : impl->callback_func = cb
    ota_task       -> ota_task      : ota->state = OTA_IDLE



ota_data_get_default_proc()
===============================

ota_service_set_upgrade_param()
====================================

.. uml::

    caption ota_service.c

    box "ota"
    participant "ota_example.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n" as periph_service order 20
    end box

    box "ota_service" #LightBlue
    participant "ota_service.c \n" as ota_service order 50
    participant "ota_task()"       as ota_task    order 60
    end box
      
    == Set OTA upgrade paramter ==
    autonumber 11 "<b>(<u>##</u>)"
    adf_app        -> ota_service   : ota_data_get_default_proc(&upgrade_list[0])
    adf_app        -> ota_service   : ota_data_get_default_proc(&upgrade_list[1])
    adf_app        -> ota_service   : ota_service_set_upgrade_param(upgrade_list)



periph_service_start() / _ota_start()
==========================================

.. uml::

    caption ota_service.c

    box "ota"
    participant "ota_example.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n" as periph_service order 20
    end box

    box "ota_service" #LightBlue
    participant "ota_service.c \n" as ota_service order 50
    participant "ota_task()"       as ota_task    order 60
    end box
      
    == Start OTA Servcie ==
    autonumber 21 "<b>(<u>##</u>)"
    adf_app        -> periph_service : periph_service_start()
    periph_service -> ota_service   : impl->service_start() ==> _ota_start()
    ota_service    -> ota_task      : ota_service_cmd_send \n (OTA_SERVICE_CMD_START)    
    ota_task       -> ota_task      : ota->state = OTA_START

    autonumber 31 "<b>(<u>##</u>)"
    loop "upgrade_list.length()"
        ota_service    <- ota_task  : ota_service_process()
        ota_service -> ota_service  : upgrade_info->prepare(), \n upgrade_info->need_upgrade(), \n upgrade_info->execute_upgrade(), \n upgrade_info->finished_check()
        periph_service <- ota_task  : periph_service_callback({.type = OTA_SERV_EVENT_TYPE_RESULT})
        adf_app  <- periph_service  : impl->callback_func() ==> \n ota_service_cb \n ({.type = OTA_SERV_EVENT_TYPE_RESULT})
        ...
    end
    ota_task       -> ota_task      : ota->state = OTA_END

    autonumber 41 "<b>(<u>##</u>)"
    alt "ota->upgrade_list[i].reboot_flag == true, 0<=i<upgrade_list.length()"
        ota_task ->]        : esp_restart()
    else 
        periph_service <- ota_task  : periph_service_callback \n ({.type = OTA_SERV_EVENT_TYPE_FINISH})
        adf_app  <- periph_service  : impl->callback_func() ==> \n ota_service_cb \n ({.type = OTA_SERV_EVENT_TYPE_FINISH})
        ...
        ota_task -> ota_task        : ota->state = OTA_IDLE
    end
    

periph_service_destroy() / _ota_destroy()
===========================================

.. uml::

    caption ota_service.c

    box "ota"
    participant "ota_example.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n" as periph_service order 20
    end box

    box "ota_service" #LightBlue
    participant "ota_service.c \n" as ota_service order 50
    participant "ota_task()"       as ota_task    order 60
    end box
      

    == Destory OTA Service ==
    autonumber 51 "<b>(<u>##</u>)"
    adf_app    -> periph_service    : periph_service_destroy()
    periph_service -> ota_service   : impl->service_destroy() ==> _ota_destroy()
    ota_service    -> ota_task      : ota_service_cmd_send \n (OTA_SERVICE_CMD_DESTROY)
    ota_task       -> ota_task      : ota->state =  OTA_DESTROY
