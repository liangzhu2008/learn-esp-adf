esp_dispatcher
--------------


.. uml::

    title esp_dispatcher

    participant "esp_dispatcher\n_dueros_app.c" as esp_app order 10
    participant "esp_dispatcher.c\n" as esp_component order 50
    participant "esp_dispatcher.c \ndispatcher_event_task" as component_task order 60
   
    esp_app -> esp_component : dispatcher = esp_dispatcher_create(&d_cfg)

    esp_component ->] : result_que = xQueueCreate()
    esp_component ->] : exe_que = xQueueCreate()
    esp_component ->] : mutex = mutex_create()
    esp_component -> component_task : xTaskCreatePinnedToCore()
    activate component_task 

