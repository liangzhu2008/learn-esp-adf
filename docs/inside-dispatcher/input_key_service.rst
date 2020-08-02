按键服务 Input Key Service
####################################

.. tip:: 

    如果图片看不清楚。你可以 **在图片上点击鼠标右键** --> **在新标签页中打开图片** ，然后你可以放大、缩小、移动图片。

`input_key_service.c`__, `esp_dispatcher_dueros_app.c`__.

.. __: https://github.com/espressif/esp-adf/blob/master/components/input_key_service/input_key_service.c
.. __: https://github.com/espressif/esp-adf/blob/master/examples/advanced_examples/esp_dispatcher_dueros/main/esp_dispatcher_dueros_app.c


概述
============

.. role:: strike
   :class: strike

.. uml::

    caption esp_dispatcher.c

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box
      
    == Create servcie ==
    adf_app        -> input_service : input_serv = input_key_service_create(&input_cfg)
    periph_service <- input_service : periph_service_create()
    periph_service -> input_task    : audio_thread_create()
    activate input_task 
    periph_service <- input_task    : periph_service_get_data()

    == Add key ==
    adf_app        -> input_service : input_key_service_add_key( \n input_serv, input_key_info, ...)
    input_service   ->]             : STAILQ_INSERT_TAIL( \n ..., input_key_node, ...)

    == Set callback ==
    adf_app        -> periph_service : periph_service_set_callback( \n input_serv, input_key_service_cb)
    periph_service -> periph_service : impl->callback_func \n = input_key_service_cb

    == Get service state ==
                  [-> input_service   : get_input_key_service_state(input_serv)

    == Receive key event ==
    input_task     <-]               : (key)
    input_task     -> input_task     : input_key_service_event_receive()
    input_task     ->]               : esp_periph_set_get_queue()
    input_task     ->]               : xQueueReceive()
    periph_service <- input_task     : periph_service_callback()
    adf_app        <- periph_service : input_key_service_cb()

    == --Start servcie-- ==
                  [-> periph_service : --periph_service_start(input_serv)--
    periph_service ->x input_service  : --input_key_service_start()--

    == --Stop servcie-- ==
                  [-> periph_service : --periph_service_stop(input_serv)--
    periph_service ->x input_service  : --input_key_service_stop()--

    == Destory servcie ==
                  [-> periph_service : periph_service_destory(input_serv)
    periph_service -> input_service   : input_key_service_destroy()
    input_service   -> input_service   : input_key_service_event_send( \n input_key_handle,  \n **PERIPH_SERVICE_STATE_STOPPED**)
    input_service   -> input_task     : xQueueSend(input_ser_queue,  \n **PERIPH_SERVICE_STATE_STOPPED**)
    input_task     ->]               : STAILQ_REMOVE(input_info_list)
    input_task     ->]               : vTaskDelete(NULL)
    deactivate input_task 
    
.. note::

    函数 input_key_service_start() 与 input_key_service_stop() 实际上没有被调用过，函数也没有功能。

    函数 input_key_service_destroy() 实际上也没有被调用过，不过函数有功能可以被调用。

    此处的回调函数，例如 input_key_service_cb(), 实际上是运行在 input_key_service_task() 这个任务里的。


input_key_service_create()
============================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

      
    == Create servcie ==
    adf_app        -> input_service : input_serv = input_key_service_create(&input_cfg)
    periph_service <- input_service : periph_service_create()
    periph_service -> input_task    : audio_thread_create()
    activate input_task 
    periph_service <- input_task    : periph_service_get_data()


input_key_service_add_key()
============================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

    == Add key ==
    adf_app        -> input_service : input_key_service_add_key( \n input_serv, input_key_info, ...)
    input_service   ->]             : STAILQ_INSERT_TAIL( \n ..., input_key_node, ...)


periph_service_set_callback()
===============================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

    == Set callback ==
    adf_app        -> periph_service : periph_service_set_callback( \n input_serv, input_key_service_cb)
    periph_service -> periph_service : impl->callback_func \n = input_key_service_cb


get_input_key_service_state()
=============================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

    == Get service state ==
                  [-> input_service   : get_input_key_service_state(input_serv)


receive key event
========================================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

    == Receive key event ==
    input_task     <-]               : (key)
    input_task     -> input_task     : input_key_service_event_receive()
    input_task     ->]               : esp_periph_set_get_queue()
    input_task     ->]               : xQueueReceive()
    periph_service <- input_task     : periph_service_callback()
    adf_app        <- periph_service : input_key_service_cb()
    
.. note::

    此处的回调函数，例如 input_key_service_cb(), 实际上是运行在 input_key_service_task() 这个任务里的。


--input_key_service_start()--
========================================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

    == --Start servcie-- ==
                  [-> periph_service : --periph_service_start(input_serv)--
    periph_service ->x input_service  : --input_key_service_start()--


.. note::

    函数 input_key_service_start() 实际上没有被调用过。


--input_key_service_stop()--
========================================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

    == --Stop servcie-- ==
                  [-> periph_service : --periph_service_stop(input_serv)--
    periph_service ->x input_service  : --input_key_service_stop()--
 
.. note::

    函数 input_key_service_stop() 实际上没有被调用过，函数也没有功能。


input_key_service_destroy()
==============================

.. uml::

    hide footbox

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "periph_service.c \n"               as periph_service order 20
    end box

    box "input_key_service" #LightBlue
    participant "input_key_service.c \n"                as input_service order 50
    participant "input_key_service.c @\n input_key_service_task()" as input_task order 60
    end box

    == Destory servcie ==
                  [-> periph_service : periph_service_destory(input_serv)
    periph_service -> input_service   : input_key_service_destroy()
    input_service   -> input_service   : input_key_service_event_send( \n input_key_handle,  \n **PERIPH_SERVICE_STATE_STOPPED**)
    input_service   -> input_task     : xQueueSend(input_ser_queue,  \n **PERIPH_SERVICE_STATE_STOPPED**)
    input_task     ->]               : STAILQ_REMOVE(input_info_list)
    input_task     ->]               : vTaskDelete(NULL)
    deactivate input_task 

    
.. note::

    函数 input_key_service_destroy() 实际上也没有被调用过，不过函数有功能可以被调用。

