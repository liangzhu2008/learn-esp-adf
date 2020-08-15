动作 ESP Action
#############################

`esp_action_def.h`__ ,  `esp_action_exe_type.h`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_dispatcher/include/esp_action_def.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_dispatcher/include/esp_action_exe_type.h


Action 原型
====================

`esp_action_def.h`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_dispatcher/include/esp_action_def.h

Action 的 **函数原型** 如下：

.. code:: c

    typedef esp_err_t (*esp_action_exe) (void *instance, action_arg_t *arg, action_result_t *result);


``arg`` 是传入的参数，定义如下：

.. code:: c

    /**
    * @brief The action arguments
    */
    typedef struct {
        void                                *data;      /*!< Pointer to arguments data */
        int                                 len;        /*!< Length of the data */
    } action_arg_t;


``result`` 是传出的结果，定义如下：

.. code:: c

    /**
    * @brief The result of action
    */
    typedef struct {
        esp_err_t                           err;        /*!< Error code */
        void                                *data;      /*!< Pointer to result data */
        int                                 len;        /*!< Length of the data */
    } action_result_t;


Action 类型
======================

`esp_action_exe_type.h`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_dispatcher/include/esp_action_exe_type.h


ESP_ADF 定义了一些默认的 **动作执行类型 Action Type** ：

.. code:: c

    typedef enum {
        ACTION_EXE_TYPE_UNKNOWN                                = - 1,

        // Wi-Fi 相关的 Action Type
        ACTION_EXE_TYPE_CONNECTIVITY_BASE                      = 0x0,
        ACTION_EXE_TYPE_WIFI_DISCONNECT                        = 0x1,
        ACTION_EXE_TYPE_WIFI_CONNECT                           = 0x2,
        ACTION_EXE_TYPE_WIFI_SETTING_START                     = 0x3,
        ACTION_EXE_TYPE_WIFI_SETTING_STOP                      = 0x4,
        ACTION_EXE_TYPE_CONNECTIVITY_MAX                       = 0xFFF,

        // Audio 相关的 Action Type
        ACTION_EXE_TYPE_AUDIO_BASE                             = 0x1000,
        ACTION_EXE_TYPE_AUDIO_PLAY                             = 0x1001,
        ACTION_EXE_TYPE_AUDIO_PAUSE                            = 0x1002,
        ACTION_EXE_TYPE_AUDIO_RESUME                           = 0x1003,
        ACTION_EXE_TYPE_AUDIO_STOP                             = 0x1004,
        ACTION_EXE_TYPE_AUDIO_GET_PROGRESS_BYTE                = 0x1005,
        ACTION_EXE_TYPE_AUDIO_GET_PROGRESS_TIME                = 0x1006,
        ACTION_EXE_TYPE_AUDIO_VOLUME_SET                       = 0x1007,
        ACTION_EXE_TYPE_AUDIO_VOLUME_GET                       = 0x1008,
        ACTION_EXE_TYPE_AUDIO_VOLUME_UP                        = 0x1009,
        ACTION_EXE_TYPE_AUDIO_VOLUME_DOWN                      = 0x100A,
        ACTION_EXE_TYPE_AUDIO_MUTE_ON                          = 0x100B,
        ACTION_EXE_TYPE_AUDIO_MUTE_OFF                         = 0x100C,
        ACTION_EXE_TYPE_AUDIO_MAX                              = 0x1FFF,

        // Recorder 相关的 Action Type
        ACTION_EXE_TYPE_RECORDER_BASE                          = 0x2000,
        ACTION_EXE_TYPE_REC_WAV_TURN_ON                        = 0x2001,
        ACTION_EXE_TYPE_REC_WAV_TURN_OFF                       = 0x2002,
        ACTION_EXE_TYPE_REC_AMR_TURN_ON                        = 0x2003,
        ACTION_EXE_TYPE_REC_AMR_TURN_OFF                       = 0x2004,
        ACTION_EXE_TYPE_RECORDER_MAX                           = 0x2FFF,

        // Display 相关的 Action Type
        ACTION_EXE_TYPE_DISPLAY_BASE                           = 0x3000,
        ACTION_EXE_TYPE_DISPLAY_TURN_ON                        = 0x3001,
        ACTION_EXE_TYPE_DISPLAY_TURN_OFF                       = 0x3002,
        ACTION_EXE_TYPE_DISPLAY_WIFI_SETTING                   = 0x3003,
        ACTION_EXE_TYPE_DISPLAY_WIFI_DISCONNECTED              = 0x3004,
        ACTION_EXE_TYPE_DISPLAY_WIFI_CONNECTED                 = 0x3005,
        ACTION_EXE_TYPE_DISPLAY_SETTING_TIMEOUT                = 0x3006,
        ACTION_EXE_TYPE_DISPLAY_BATTERY_CHARGING               = 0x3007,
        ACTION_EXE_TYPE_DISPLAY_BATTERY_FULL                   = 0x3008,
        ACTION_EXE_TYPE_DISPLAY_BATTERY_DISCHARGING            = 0x3009,
        ACTION_EXE_TYPE_DISPLAY_MAX                            = 0x3FFF,

        // Duer 相关的 Action Type
        ACTION_EXE_TYPE_DUER_BASE                              = 0x4000,
        ACTION_EXE_TYPE_DUER_AUDIO                             = 0x4001,
        ACTION_EXE_TYPE_DUER_SPEAK                             = 0x4002,
        ACTION_EXE_TYPE_DUER_VOLUME_ADJ                        = 0x4003,
        ACTION_EXE_TYPE_DUER_PAUSE                             = 0x4004,
        ACTION_EXE_TYPE_DUER_RESUME                            = 0x4005,
        ACTION_EXE_TYPE_DUER_STOP                              = 0x4006,
        ACTION_EXE_TYPE_DUER_DISCONNECT                        = 0x4007,
        ACTION_EXE_TYPE_DUER_CONNECT                           = 0x4008,
        ACTION_EXE_TYPE_DUER_MAX                               = 0x4FFF,

        // 客户自定义的 Action Type 从这里开始
        ACTION_EXE_TYPE_CUSTOMER_BASE                          = 0x80000,
    } action_exe_type_t;

每一大类的 Action Type, 都有一个起始值 ``ACTION_EXE_TYPE_XXX_BASE`` ， 还有一个最大值 ``ACTION_EXE_TYPE_XXX_MAX`` 。

客户自定义的 Action Type 从 ``ACTION_EXE_TYPE_CUSTOMER_BASE`` 开始。


Action 实现
======================

Action 的实现都很简单，其本上都是对其它函数的简单封装。不讲解，直接上代码。

Wi-Fi Action
--------------------

`wifi_action.h`__ ， `wifi_action.c`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/include/wifi_action.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/wifi_action.c

.. code:: c

    /**
    * brief       Wi-Fi provides service of connection
    *
    * @param instance          The Wi-Fi service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t wifi_action_connect(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ...
        wifi_service_connect(wifi_serv);
        ...
    }

    /**
    * brief      Wi-Fi provides service of disconnection
    *
    *
    * @param instance          The Wi-Fi service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t wifi_action_disconnect(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ...
        wifi_service_disconnect(wifi_serv);
        ...
    }

    /**
    * brief      Wi-Fi provides service of start Wi-Fi setting
    *
    * @param instance          The Wi-Fi service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t wifi_action_setting_start(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ...
        wifi_service_setting_start(wifi_serv, 0);
        ...
    }

    /**
    * brief      Wi-Fi provides service of stop Wi-Fi setting
    *
    * @param instance          The Wi-Fi service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t wifi_action_setting_stop(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ...
        wifi_service_setting_stop(wifi_serv, 0);
        ...
    }


Player Action (Audio Action)
-----------------------------------

`player_action.h`__ ， `player_action.c`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/include/player_action.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/player_action.c

目前有四个 Action 是没有实现的，你需要自己实现：player_action_next(), player_action_prev(), player_action_mute_on(), player_action_mute_off() 。

.. code:: c

    /**
    * brief      Player provides service of playing music
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_play(void *instance, action_arg_t *arg, action_result_t *result)
    {
        esp_audio_handle_t handle = (esp_audio_handle_t)instance;
        ESP_LOGI(TAG, "%s", __func__);
        int ret = esp_audio_play(handle, AUDIO_CODEC_TYPE_DECODER, NULL, 0);
        return ret;
    }

    /**
    * brief      Player provides service of pausing music playing
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_pause(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        esp_audio_handle_t handle = (esp_audio_handle_t)instance;
        int ret = esp_audio_pause(handle);
        return ret;
    }

    /**
    * brief      Player provides service of playing the next audio file
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_next(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);

        return ESP_OK;
    }

    /**
    * brief      Player provides service of playing the previous audio file
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_prev(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);

        return ESP_OK;
    }

    /**
    * brief      Player provides service of increasing volume
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_vol_up(void *instance, action_arg_t *arg, action_result_t *result)
    {
        int player_volume = 0;
        esp_audio_handle_t handle = (esp_audio_handle_t)instance;
        esp_audio_vol_get(handle, &player_volume);
        player_volume += 10;
        if (player_volume > 100) {
            player_volume = 100;
        }
        esp_audio_vol_set(handle, player_volume);
        ESP_LOGI(TAG, "%s, vol:[%d]", __func__, player_volume);
        return ESP_OK;
    }

    /**
    * brief      Player provides service of decreasing volume
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_vol_down(void *instance, action_arg_t *arg, action_result_t *result)
    {
        int player_volume = 0;
        esp_audio_handle_t handle = (esp_audio_handle_t)instance;
        esp_audio_vol_get(handle, &player_volume);
        player_volume -= 10;
        if (player_volume < 0) {
            player_volume = 0;
        }
        esp_audio_vol_set(handle, player_volume);
        ESP_LOGI(TAG, "%s, vol:[%d]", __func__, player_volume);
        return ESP_OK;
    }

    /**
    * brief      Player provides service of mute on
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_mute_on(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return ESP_OK;
    }

    /**
    * brief      Player provides service of mute off
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t player_action_mute_off(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return ESP_OK;
    }


Recorder Action
------------------

`recorder_action.h`__ ， `recorder_action.c`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/include/recorder_action.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/recorder_action.c

目前有两个 Action 是没有实现的，你需要自己实现： recorder_action_rec_amr_turn_on(), recorder_action_rec_amr_turn_off() 。

.. code:: c

    /**
    * brief       Recorder provides service of turn on WAV recoding
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t recorder_action_rec_wav_turn_on(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        int ret = rec_engine_trigger_start();
        return ret;
    }

    /**
    * brief      Recorder provides service of turn off WAV recoding
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t recorder_action_rec_wav_turn_off(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        int ret = rec_engine_trigger_stop();
        return ret;
    }

    /**
    * brief      Recorder provides service of turn on AMR recoding
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t recorder_action_rec_amr_turn_on(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return ESP_OK;
    }

    /**
    * brief      Recorder provides service of turn off AMR recoding
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t recorder_action_rec_amr_turn_off(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return ESP_OK;
    }


Display Action
---------------

`display_action.h`__ ， `display_action.c`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/include/display_action.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/display_action.c


.. code:: c

    /**
    * brief      Display provides pattern of Wi-Fi disconnection
    *
    * @param instance          The display service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, errors
    */
    esp_err_t display_action_wifi_disconnected(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        display_service_handle_t dis = (display_service_handle_t)instance;
        int ret = display_service_set_pattern(dis, DISPLAY_PATTERN_WIFI_DISCONNECTED, 0);
        return ret;
    }

    /**
    * brief      Display provides pattern of Wi-Fi connection
    *
    * @param instance          The display service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, errors
    */
    esp_err_t display_action_wifi_connected(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        display_service_handle_t dis = (display_service_handle_t)instance;
        int ret = display_service_set_pattern(dis, DISPLAY_PATTERN_WIFI_CONNECTED, 0);
        return ret;
    }

    /**
    * brief      Display provides pattern of wifi setting
    *
    * @param instance          The display service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, errors
    */
    esp_err_t display_action_wifi_setting(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        display_service_handle_t dis = (display_service_handle_t)instance;
        int ret = display_service_set_pattern(dis, DISPLAY_PATTERN_WIFI_SETTING, 0);
        return ret;
    }

    /**
    * brief      Display provides pattern of turn off
    *
    * @param instance          The display service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, errors
    */
    esp_err_t display_action_turn_off(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        display_service_handle_t dis = (display_service_handle_t)instance;
        int ret = display_service_set_pattern(dis, DISPLAY_PATTERN_TURN_OFF, 0);
        return ret;
    }

    /**
    * brief      Display provides pattern of turn on
    *
    * @param instance          The display service instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, errors
    */
    esp_err_t display_action_turn_on(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        display_service_handle_t dis = (display_service_handle_t)instance;
        int ret = display_service_set_pattern(dis, DISPLAY_PATTERN_TURN_ON, 0);
        return ret;
    }


Dueros Action
--------------------

`dueros_action.h`__ ， `dueros_action.c`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/include/dueros_action.h

.. __: https://github.com/espressif/esp-adf/blob/master/components/esp_actions/dueros_action.c

目前 dueros_action_disconnect() 是没有实现的，你需要自己实现。

.. code:: c

    /**
    * brief      Dueros provides service of disconnection
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t dueros_action_disconnect(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return ESP_OK;
    }

    /**
    * brief      DuerOS provides service of connection
    *
    * @param instance          The player instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t dueros_action_connect(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        audio_service_handle_t handle = (audio_service_handle_t)instance;
        int ret = audio_service_connect(handle);
        return ret;
    }


Dueros Audio Action
--------------------

`duer_audio_action.h`__ ， `duer_audio_action.c`__ 。

.. __: https://github.com/espressif/esp-adf/blob/master/examples/advanced_examples/esp_dispatcher_dueros/main/duer_audio_action.h

.. __: https://github.com/espressif/esp-adf/blob/master/examples/advanced_examples/esp_dispatcher_dueros/main/duer_audio_action.c

示例 esp_dispatcher_dueros 中， 有一个 Audio Action 的实现，也可以学习一下。这些代码是基于一个 `esp_audio`__ 的库实现的，很遗憾，这个库没有源码。

.. __: https://github.com/espressif/esp-adf-libs/tree/747dc66268129fc20da3149d79d2e933ee397ed9/esp_audio

.. code:: c

    /**
    * brief      DuerOS provides service of setting volume
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_vol_set(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s,vol:%d", __func__, (int)arg->data);
        esp_player_vol_set((int)arg->data);
        return ESP_OK;
    }

    /**
    * brief      DuerOS provides service of adjust volume
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_vol_adj(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s, adj_volume:%d", __func__, (int)arg->data);
        esp_player_vol_set((int)arg->data);
        return ESP_OK;
    }

    /**
    * brief      DuerOS provides service of turn on the mute
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_mute_on(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return ESP_OK;
    }

    /**
    * brief      DuerOS provides service of turn off the mute
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_mute_off(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return ESP_OK;
    }

    /**
    * brief      DuerOS provides service of get player state
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_get_state(void *instance, action_arg_t *arg, action_result_t *result)
    {
        int *vol = audio_calloc(1, sizeof(int));
        AUDIO_MEM_CHECK(TAG, vol, {
            result->data = 0;
            result->len = 0;
            result->err = ESP_FAIL;
            return ESP_FAIL;
        });
        esp_audio_vol_get((esp_audio_handle_t)instance, vol);
        result->data = vol;
        result->len = sizeof(int);
        result->err = ESP_OK;
        ESP_LOGI(TAG, "%s, vol:%d, %p ", __func__, *vol, result->data);
        return ESP_OK;
    }

    /**
    * brief      DuerOS provides service of playing speak
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_speak(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s, %p, %s, %d", __func__, arg->data, (char *)arg->data, arg->len);
        esp_player_music_play((char *)arg->data, 0, MEDIA_SRC_TYPE_DUER_SPEAK);
        return ESP_OK;
    }

    /**
    * brief      DuerOS provides service of playing music
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_audio_play(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s, %p, %s, %d", __func__, arg->data, (char *)arg->data, arg->len);
        duer_dcs_audio_info_t *info = (duer_dcs_audio_info_t *)arg->data;
        int ret = esp_player_music_play((char *)info->url, info->offset, MEDIA_SRC_TYPE_DUER_MUSIC);
        return ret;
    }

    /**
    * brief      DuerOS provides service of stop music playing
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_audio_stop(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s", __func__);
        return esp_player_music_stop();
    }

    /**
    * brief      DuerOS provides service of pause music playing
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_audio_pause(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGE(TAG, "%s", __func__);
        return esp_player_music_pause();
    }

    /**
    * brief      DuerOS provides service of resume music playing
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_audio_resume(void *instance, action_arg_t *arg, action_result_t *result)
    {
        ESP_LOGI(TAG, "%s, %p, %s, %d", __func__, arg->data, (char *)arg->data, arg->len);
        duer_dcs_audio_info_t *info = (duer_dcs_audio_info_t *)arg->data;
        int ret = esp_player_music_play((char *)info->url, info->offset, MEDIA_SRC_TYPE_DUER_MUSIC);
        return ret;
    }

    /**
    * brief      DuerOS provides service of get the playing progress
    *
    * @param instance          The DuerOS instance
    * @param arg               The arguments of execution function
    * @param result            The result of execution function
    *
    * @return
    *     - ESP_OK, success
    *     - Others, error
    */
    esp_err_t duer_dcs_action_get_progress(void *instance, action_arg_t *arg, action_result_t *result)
    {
        int *pos = audio_calloc(1, sizeof(int));
        AUDIO_MEM_CHECK(TAG, pos, {
            result->data = 0;
            result->len = 0;
            result->err = ESP_FAIL;
            return ESP_FAIL;
        });
        esp_player_pos_get(pos);
        result->data = pos;
        result->len = sizeof(int);
        result->err = ESP_OK;
        ESP_LOGI(TAG, "%s, pos:%d, %p ", __func__, *pos, result->data);
        return ESP_OK;
    }

