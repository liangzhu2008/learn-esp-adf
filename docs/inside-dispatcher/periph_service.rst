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
