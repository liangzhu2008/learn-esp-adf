显示服务 Display Servcie
#################################

`display_service.c`__, `esp_dispatcher_dueros_app.c`__.

.. __: https://github.com/espressif/esp-adf/blob/master/components/display_service/display_service.c

.. __: https://github.com/espressif/esp-adf/blob/master/examples/advanced_examples/esp_dispatcher_dueros/main/esp_dispatcher_dueros_app.c

.. tip:: 

    如果图片太大，看不清楚。你可以 **在图片上点击鼠标右键** --> **在新窗口中打开图片** ，然后你可以放大、缩小、移动图片。



概述
============

.. role:: strike
   :class: strike

.. uml::

    caption display_service.c

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box
    
    box "audio_board"
    participant "lyrat_v4.3\\\n board.c"  as adf_board order 15
    end box

    box "esp_dispatcher" 
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    participant "periph_service.c \n"  as periph_service order 40
    end box

    box "esp_actions"
    participant "display_action.c \n" as display_action  order 45
    end box

    box "display_service" #LightBlue
    participant "display_service.c \n" as display_service  order 50
    participant "led_indicator.c \n"   as led_indicator order 60
    end box

    box "esp_peripherals" 
    participant "esp_peripherals.c \n" as esp_peripherals order 70
    participant "periph_led.c \n"      as periph_led      order 80
    end box

    == Create dispatcher ==
    adf_app -> dispatcher_comp : dispatcher = esp_dispatcher_create(&d_cfg)
    dispatcher_comp -> dispatcher_task : xTaskCreatePinnedToCore()
    activate dispatcher_task 

    == <color:red>Create display servcie</color> ==
    adf_app         -> adf_board     : disp_serv = audio_board_led_init()

    adf_board       -> led_indicator : led_indicator_init()
    led_indicator   -> periph_led   : periph_handle = periph_led_init(&led_cfg)
    esp_peripherals <- periph_led   : periph = \n esp_periph_create \n (PERIPH_ID_LED)
    esp_peripherals <- periph_led   : esp_periph_set_data \n (periph)
    esp_peripherals <- periph_led   : esp_periph_set_function \n (periph, _led_init, \n _led_run, _led_destroy)
    led_indicator -> esp_peripherals : esp_periph_init \n (periph_handle)
    esp_peripherals -> periph_led     : periph->init(periph) \n ==> _led_init()

    adf_board        -> display_service : display_service_create \n ({.service_ioctl = **led_indicator_pattern**})
    periph_service   <- display_service : periph_service_create()


    == Register display service execution function ==
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->disp_serv, \n ACTION_EXE_TYPE_DISPLAY_TURN_OFF, \n display_action_turn_off);
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->disp_serv, \n ACTION_EXE_TYPE_DISPLAY_TURN_ON, \n display_action_turn_on);
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->disp_serv, \n ACTION_EXE_TYPE_DISPLAY_WIFI_SETTING, \n display_action_wifi_setting);
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->disp_serv, \n **ACTION_EXE_TYPE_DISPLAY_WIFI_CONNECTED**, \n display_action_wifi_connected);
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->disp_serv, \n ACTION_EXE_TYPE_DISPLAY_WIFI_DISCONNECTED, \n display_action_wifi_disconnected);


    == <color:red>Set service display pattern</color> (Execution function) ==
    adf_app <-] : wifi_service_cb(WIFI_SERV_EVENT_CONNECTED)
    adf_app ->  dispatcher_comp : esp_dispatcher_execute(d->dispatcher, \n ACTION_EXE_TYPE_DISPLAY_WIFI_CONNECTED)
    dispatcher_comp -> dispatcher_task
    dispatcher_task -> display_action  : exe_item->exe_func ==> \n display_action_wifi_connected()
    display_action  -> display_service : display_service_set_pattern \n (DISPLAY_PATTERN_WIFI_CONNECTED)
    periph_service  <- display_service : periph_service_ioctl()
    periph_service  -> led_indicator   : impl->service_ioctl() ==> led_indicator_pattern()
    led_indicator   -> periph_led      : periph_led_blink()


    == <color:red>--Destory display servcie--</color> ==
    [-> display_service : display_destroy()
    periph_service <- display_service : periph_service_destory(disp_serv)
    periph_service ->]  : --impl->service_destroy()--

    
    note over dispatcher_comp, led_indicator
    红色字体说明的消息序列段，才是与显示服务 Display Service 直接相关的。   其它的消息序列段，是与 esp_dispather 相关的。
    "periph->init(periph) ==> _led_init()" 表示执行的代码 periph->init(periph) 最终调用了 _led_init() 这个回调函数。
    end note


.. note::

    函数 display_destroy() 实际上也没有被调用过，不过函数有功能可以被调用。

    显示服务 Display Service 没有回调函数 Callback，也无内部的任务 Task。


audio_board_led_init()
=======================

display_service_create()
============================

.. uml::
    
    caption display_service_create()

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box
    
    box "audio_board"
    participant "lyrat_v4.3\\\n board.c"  as adf_board order 15
    end box

    box "esp_dispatcher" 
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    participant "periph_service.c \n"  as periph_service order 40
    end box

    box "esp_actions"
    participant "display_action.c \n" as display_action  order 45
    end box

    box "display_service" #LightBlue
    participant "display_service.c \n" as display_service  order 50
    participant "led_indicator.c \n"   as led_indicator order 60
    end box

    box "esp_peripherals" 
    participant "esp_peripherals.c \n" as esp_peripherals order 70
    participant "periph_led.c \n"      as periph_led      order 80
    end box


    == <color:red>Create display servcie</color> ==
    adf_app         -> adf_board     : disp_serv = audio_board_led_init()

    adf_board       -> led_indicator : led_indicator_init()
    led_indicator   -> periph_led   : periph_handle = periph_led_init(&led_cfg)
    esp_peripherals <- periph_led   : periph = \n esp_periph_create \n (PERIPH_ID_LED)
    esp_peripherals <- periph_led   : esp_periph_set_data \n (periph)
    esp_peripherals <- periph_led   : esp_periph_set_function \n (periph, _led_init, \n _led_run, _led_destroy)
    led_indicator -> esp_peripherals : esp_periph_init \n (periph_handle)
    esp_peripherals -> periph_led     : periph->init(periph) \n ==> _led_init()

    adf_board        -> display_service : display_service_create \n ({.service_ioctl = **led_indicator_pattern**})
    periph_service   <- display_service : periph_service_create()


.. note::
    
    显示服务 Display Service 没有回调函数 Callback，也无内部的任务 Task。


display_service_set_pattern()
=================================

.. uml::

    caption display_service.c

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "audio_board"
    participant "lyrat_v4.3\\\n board.c"  as adf_board order 15
    end box

    box "esp_dispatcher" 
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    participant "periph_service.c \n"  as periph_service order 40
    end box

    box "esp_actions"
    participant "display_action.c \n" as display_action  order 45
    end box

    box "display_service" #LightBlue
    participant "display_service.c \n" as display_service  order 50
    participant "led_indicator.c \n"   as led_indicator order 60
    end box

    box "esp_peripherals" 
    participant "esp_peripherals.c \n" as esp_peripherals order 70
    participant "periph_led.c \n"      as periph_led      order 80
    end box

    == <color:red>Set service display pattern</color> (Execution function) ==
    adf_app <-] : wifi_service_cb(WIFI_SERV_EVENT_CONNECTED)
    adf_app ->  dispatcher_comp : esp_dispatcher_execute(d->dispatcher, \n ACTION_EXE_TYPE_DISPLAY_WIFI_CONNECTED)
    dispatcher_comp -> dispatcher_task
    dispatcher_task -> display_action  : exe_item->exe_func ==> \n display_action_wifi_connected()
    display_action  -> display_service : display_service_set_pattern \n (DISPLAY_PATTERN_WIFI_CONNECTED)
    periph_service  <- display_service : periph_service_ioctl()
    periph_service  -> led_indicator   : impl->service_ioctl() ==> led_indicator_pattern()
    led_indicator   -> periph_led      : periph_led_blink()


display_destroy()
=========================

.. uml::

    caption display_service.c

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "audio_board"
    participant "lyrat_v4.3\\\n board.c"  as adf_board order 15
    end box

    box "esp_dispatcher" 
    participant "esp_dispatcher.c\n"                    as dispatcher_comp   order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    participant "periph_service.c \n"  as periph_service order 40
    end box

    box "esp_actions"
    participant "display_action.c \n" as display_action  order 45
    end box

    box "display_service" #LightBlue
    participant "display_service.c \n" as display_service  order 50
    participant "led_indicator.c \n"   as led_indicator order 60
    end box

    box "esp_peripherals" 
    participant "esp_peripherals.c \n" as esp_peripherals order 70
    participant "periph_led.c \n"      as periph_led      order 80
    end box

    == <color:red>--Destory display servcie--</color> ==
    [-> display_service : display_destroy()
    periph_service <- display_service : periph_service_destory(disp_serv)
    periph_service ->]  : --impl->service_destroy()--

.. note::

    函数 display_destroy() 实际上也没有被调用过，不过函数有功能可以被调用。
