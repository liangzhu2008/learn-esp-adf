WiFi 服务 WiFi Service
##########################

`wifi_service.c`__, `esp_dispatcher_dueros_app.c`__.

.. __: https://github.com/espressif/esp-adf/blob/master/components/wifi_service/src/wifi_service.c
.. __: https://github.com/espressif/esp-adf/blob/master/examples/advanced_examples/esp_dispatcher_dueros/main/esp_dispatcher_dueros_app.c


概述
============

.. role:: strike
   :class: strike

.. uml::

    caption wifi_service.c

    box "esp_dispatcher_dueros"
    participant "esp_dispatcher \n _dueros_app.c" as adf_app order 10
    end box

    box "esp_dispatcher" 
    participant "esp_dispatcher.c\n"    as dispatcher_comp  order 20
    participant "esp_dispatcher.c @ \ndispatcher_event_task()" as dispatcher_task   order 30
    participant "periph_service.c \n"   as periph_service   order 40
    end box

    box "esp_actions"
    participant "display_action.c \n" as display_action  order 50
    end box

    box "wifi_service" #LightBlue
    participant "wifi_service.c \n"      as wifi_service  order 60
    participant "wifi_task() \n"         as wifi_task     order 61
    participant "wifi_event_cb() \n"     as wifi_event_cb order 62
    
    participant "esp_wifi_setting.c \n"  as wifi_setting  order 70
    participant "smart_config.c \n"      as smartconfig   order 71
    participant "smartconfig_cb() \n"    as smartconfig_cb order 72

    participant "wifi_ssid_manager.c \n" as ssid_manager  order 80
    end box

    box "ESP_IDF"
    participant "esp_wifi.h \n"         as idf_wifi         order 90
    participant "smartconfig.c \n"      as idf_smartconfig  order 91
    end box

    == Create dispatcher ==
    adf_app -> dispatcher_comp : dispatcher = esp_dispatcher_create(&d_cfg)
    dispatcher_comp -> dispatcher_task : xTaskCreatePinnedToCore()
    activate dispatcher_task 

    == <color:red>Create Wi-Fi servcie</color> ==
    adf_app         -> wifi_service    : wifi_serv = wifi_service_create({.evt_cb = **wifi_service_cb**})
    wifi_service    -> ssid_manager    : ssid_manager = wifi_ssid_manager_create()
    wifi_service    ->]                : wifi_serv_que = xQueueCreate()
    wifi_service    ->]                : sync_evt = xEventGroupCreate()
    periph_service  <- wifi_service    : periph_service_create \n ({.task_func = **wifi_task**, \n .service_start = **_wifi_start**, \n .service_stop = **_wifi_stop**, \n .service_destroy = **wifi_service_destroy**})
    periph_service  -> wifi_task       : audio_thread_create()
    activate wifi_task

    wifi_task       -> wifi_task       : wifi_sta_setup()
    wifi_task       -> wifi_event_cb   : esp_event_loop_create_default()
    wifi_task       -> wifi_event_cb   : esp_event_handler_register(wifi_event_cb)
    activate wifi_event_cb

    wifi_task       -> idf_wifi   : esp_wifi_init(); esp_wifi_set_storage(WIFI_STORAGE_FLASH)
    wifi_task       -> idf_wifi   : esp_wifi_set_mode(WIFI_MODE_STA); esp_wifi_set_config(WIFI_IF_STA); esp_wifi_start()

    periph_service  <- wifi_service    : periph_service_set_callback(cb : {.evt_cb})
    periph_service  <- periph_service  : impl->callback_func = cb


    == Register Wi-Fi service execution function ==
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_CONNECT, \nwifi_action_connect)
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_DISCONNECT, \nwifi_action_disconnect)
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_SETTING_STOP, \nwifi_action_setting_stop)
    adf_app -> dispatcher_comp  : esp_dispatcher_reg_exe_func(dispatcher, \ndueros_speaker->wifi_serv, \nACTION_EXE_TYPE_WIFI_SETTING_START, \nwifi_action_setting_start)

    == <color:red>Initialize Wi-Fi provisioning type(AIRKISS or SMARTCONFIG)</color>  ==
    adf_app      -> smartconfig     : h = smart_config_create(&info)
    wifi_setting <- smartconfig     : esp_wifi_setting_create("esp_smart_config"), \nesp_wifi_setting_set_data(info), \nesp_wifi_setting_register_function \n(_smart_config_start, \n_smart_config_stop)
    adf_app      -> wifi_setting    : esp_wifi_setting_regitster_notify_handle(wifi_serv)
    adf_app      -> wifi_service    : wifi_service_register_setting_handle(wifi_serv, h)
    periph_service <- wifi_service  : periph_service_get_data()
    adf_app      -> wifi_service    : wifi_service_set_sta_info(wifi_serv, &sta_cfg)
    periph_service <- wifi_service  : periph_service_get_data()

    == <color:red>Connect Wi-Fi Service </color>  ==
    adf_app -> wifi_service     : wifi_service_connect(wifi_serv)
    wifi_service -> wifi_task   : wifi_serv_cmd_send(WIFI_SERV_CMD_CONNECT)
    wifi_task   -> ssid_manager : wifi_ssid_manager_get_latest_config()
    wifi_task   -> idf_wifi     : esp_wifi_set_mode(WIFI_MODE_STA), esp_wifi_set_config(WIFI_IF_STA, wifi_cfg), esp_wifi_connect()
    
    == <color:red> Start Wi-Fi Service setting </color>  ==
    adf_app -> wifi_service     : wifi_service_setting_start(wifi_serv)
    wifi_service -> wifi_task   : wifi_serv_cmd_send(WIFI_SERV_CMD_SETTING_START)
    wifi_task    -> idf_wifi    : esp_wifi_disconnect()
    wifi_task  -> wifi_setting  : esp_wifi_setting_start()
    wifi_setting -> smartconfig : _smart_config_start()
    smartconfig -> smartconfig_cb : esp_event_handler_register \n (smartconfig_cb)
    activate smartconfig_cb
    smartconfig -> idf_smartconfig : esp_smartconfig_start()
    smartconfig_cb <- idf_smartconfig : smartconfig_cb(SC_EVENT_GOT_SSID_PSWD)
    wifi_setting   <- smartconfig_cb  : esp_wifi_setting_info_notify(sta_conf)
    wifi_service   -> wifi_setting  : wifi_service_update_sta_info()
    wifi_service   -> wifi_task  : wifi_serv_cmd_send \n (WIFI_SERV_CMD_UPDATE)
    wifi_task -> idf_wifi : esp_wifi_set_mode(WIFI_MODE_STA); esp_wifi_set_config(WIFI_IF_STA, wifi_cfg); esp_wifi_connect()


    == <color:red> Stop Wi-Fi Service setting </color>  ==
    adf_app -> wifi_service     : wifi_service_setting_stop(wifi_serv)
    wifi_service -> wifi_task   : wifi_serv_cmd_send(WIFI_SERV_CMD_SETTING_STOP)
    wifi_task    -> idf_wifi    : esp_wifi_disconnect()
    wifi_task  -> wifi_setting  : esp_wifi_setting_stop()
    wifi_setting -> smartconfig : _smart_config_stop()
    smartconfig -> idf_smartconfig : esp_smartconfig_stop()


    == <color:red> Wi-Fi Event callback </color>  ==
    alt   IP_EVENT_STA_GOT_IP
    wifi_event_cb <- idf_wifi   :  wifi_event_cb(IP_EVENT_STA_GOT_IP)
    wifi_service -> wifi_event_cb : wifi_serv_state_send(WIFI_SERV_EVENT_CONNECTED)
    wifi_service -> wifi_task : wifi_serv_cmd_send \n (WIFI_SERV_EVENT_CONNECTED)
    wifi_task -> ssid_manager: wifi_ssid_manager_save(ssid,password)
    else WIFI_EVENT_STA_DISCONNECTED
    wifi_event_cb <- idf_wifi   :  wifi_event_cb(WIFI_EVENT_STA_DISCONNECTED)
    wifi_service -> wifi_event_cb : wifi_serv_state_send(WIFI_SERV_EVENT_DISCONNECTED)
    wifi_service -> wifi_task : wifi_serv_cmd_send \n (WIFI_SERV_EVENT_DISCONNECTED)
    wifi_task -> ssid_manager: wifi_ssid_manager_get_ssid_num(), wifi_ssid_manager_get_best_config()
    end 
    periph_service <- wifi_task : periph_service_callback()
    adf_app <- periph_service   : wifi_service_cb()
    ...

    == <color:red> Disconnect Wi-Fi servcie </color> ==
    adf_app -> wifi_service   : wifi_service_disconnect(wifi_serv)
    wifi_service -> wifi_task : wifi_serv_cmd_send \n (WIFI_SERV_CMD_DISCONNECT)
    wifi_task    -> idf_wifi  : esp_wifi_disconnect()

    == <color:red>Destory display servcie</color> ==
    adf_app -> wifi_service : wifi_service_destroy(wifi_serv)
    wifi_service -> wifi_task : wifi_serv_cmd_send \n (WIFI_SERV_CMD_DESTROY)
    deactivate wifi_task

    
    note over adf_app, wifi_setting
    1. "ota_service_create({.evt_cb=**ota_service_cb**})" 表示调用函数时传入一个参数，该参数的 evt_cb 字段的值为 ota_service_cb 。
    2. "periph_service_set_callback(cb : {.evt_cb})" 表示调用函数时，参数 cb 的值为 某个变量的 evt_cb 字段。
    3. "impl->callback_func() ==> ota_service_cb()" 表示执行的代码 impl->callback_func()  最终调用了 ota_service_cb() 这个回调函数。
    end note


.. note::

    Wi-Fi 服务 Wi-Fi Service 既有回调函数 Callback，也有内部的任务 Task。
