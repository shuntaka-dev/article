---
title: "esp-aws-iot Deep Dive 其の一"
type: "tech"
category: []
description: "非公開記事"
publish: false
---

# test

ESP32モジュールにあh、nvs(non volatile storage)という領域があるようだ。
```c
  esp_err_t err = nvs_flash_init();
  if (err == ESP_ERR_NVS_NO_FREE_PAGES ||
      err == ESP_ERR_NVS_NEW_VERSION_FOUND) {
    ESP_ERROR_CHECK(nvs_flash_erase());
    err = nvs_flash_init();
  }
  ESP_ERROR_CHECK(err);
```

M5Stack Core2 for AWSのは、`ESP32-D0WDQ6-V3`マイコンを搭載している。
[ESP32 Series Datasheet](https://www.mouser.jp/datasheet/2/891/Espressif_Systems_01292021_esp32-1991551.pdf)には記載はなさそう..(?)


# wifi初期化処理

`#define`でmain.cで利用する変数に、`sdkconfig`の変数名を代入している

```c:src/main.c
#define EXAMPLE_WIFI_SSID CONFIG_WIFI_SSID
#define EXAMPLE_WIFI_PASS CONFIG_WIFI_PASSWORD

// (中略)

static void initialise_wifi(void) {
  tcpip_adapter_init();
  wifi_event_group = xEventGroupCreate();
  ESP_ERROR_CHECK(esp_event_loop_init(event_handler, NULL));
```

* tcpip_adapter_init: framework-espidf/components/tcpip_adapter
* xEventGroupCreate: framework-espidf/components/freertos

```c
  wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
  ESP_ERROR_CHECK(esp_wifi_init(&cfg));
  ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
  wifi_config_t wifi_config = {
      .sta =
          {
              .ssid = EXAMPLE_WIFI_SSID,
              .password = EXAMPLE_WIFI_PASS,
          },
  };
  ESP_LOGI(TAG, "Setting WiFi configuration SSID %s...", wifi_config.sta.ssid);
  ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
  ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_config)); ESP_ERROR_CHECK(esp_wifi_start());
}
```

* framework-espidf/components/esp_wifi系統
  * WIFI_INIT_CONFIGDEFAULT
  * esp_wifi_init
  * esp_wifi_set_storage
  * esp_wifi_set_mode
  * esp_wifi_set_config
  * esp_wifi_start


# aws_iot_taskコードリーディング

## MQTTクライアントとパラメータ定義の初期化
`esp-aws-iot`を使ったMQTTクライアントの作成している

```c:src/main.c
void aws_iot_task(void *param) {
  char cPayload[100];

  int32_t i = 0;

  IoT_Error_t rc = FAILURE;

  AWS_IoT_Client client;
  IoT_Client_Init_Params mqttInitParams = iotClientInitParamsDefault;
  IoT_Client_Connect_Params connectParams = iotClientConnectParamsDefault;
```


## AWS_IOTへホスト名、ポート名定義
```c:src/main.c
char HostAddress[255] = AWS_IOT_MQTT_HOST;
uint32_t port = AWS_IOT_MQTT_PORT;
// 中略

void aws_iot_task(void *param) {
  // 中略
  IoT_Publish_Message_Params paramsQOS0;
  IoT_Publish_Message_Params paramsQOS1;

  ESP_LOGI(TAG, "AWS IoT SDK Version %d.%d.%d-%s", VERSION_MAJOR, VERSION_MINOR,
           VERSION_PATCH, VERSION_TAG);

  mqttInitParams.enableAutoReconnect = false; // We enable this later below
  mqttInitParams.pHostURL = HostAddress;
  mqttInitParams.port = port;
```

menuconfigで設定する値、下記のヘッダーファイルにCONFIGとのマッピングがあるため、変数名`AWS_IOT_MQTT_HOST`でHOST名が取得できる
```c:components/esp-aws-iot/port/include/aws_iot_config.h
#define AWS_IOT_MQTT_HOST              CONFIG_AWS_IOT_MQTT_HOST ///< Customer specific MQTT HOST. The same will be used for Thing Shadow
#define AWS_IOT_MQTT_PORT              CONFIG_AWS_IOT_MQTT_PORT ///< default port for MQTT/S
```


## 証明書類の読み出し実装部

```c:src/main.c
#if defined(CONFIG_EXAMPLE_EMBEDDED_CERTS)

extern const uint8_t
    aws_root_ca_pem_start[] asm("_binary_aws_root_ca_pem_start");
extern const uint8_t aws_root_ca_pem_end[] asm("_binary_aws_root_ca_pem_end");
extern const uint8_t
    certificate_pem_crt_start[] asm("_binary_certificate_pem_crt_start");
extern const uint8_t
    certificate_pem_crt_end[] asm("_binary_certificate_pem_crt_end");
extern const uint8_t
    private_pem_key_start[] asm("_binary_private_pem_key_start");
extern const uint8_t private_pem_key_end[] asm("_binary_private_pem_key_end");

#elif defined(CONFIG_EXAMPLE_FILESYSTEM_CERTS)

static const char *DEVICE_CERTIFICATE_PATH = CONFIG_EXAMPLE_CERTIFICATE_PATH;
static const char *DEVICE_PRIVATE_KEY_PATH = CONFIG_EXAMPLE_PRIVATE_KEY_PATH;
static const char *ROOT_CA_PATH = CONFIG_EXAMPLE_ROOT_CA_PATH;

#else
#error "Invalid method for loading certs"
#endif
// 中略

void aws_iot_task(void *param) {
// 中略
#if defined(CONFIG_EXAMPLE_EMBEDDED_CERTS)
    mqttInitParams.pRootCALocation = (const char *)aws_root_ca_pem_start;
    mqttInitParams.pDeviceCertLocation = (const char *)certificate_pem_crt_start;
    mqttInitParams.pDevicePrivateKeyLocation = (const char *)private_pem_key_start;

#elif defined(CONFIG_EXAMPLE_FILESYSTEM_CERTS)
    mqttInitParams.pRootCALocation = ROOT_CA_PATH;
    mqttInitParams.pDeviceCertLocation = DEVICE_CERTIFICATE_PATH;
    mqttInitParams.pDevicePrivateKeyLocation = DEVICE_PRIVATE_KEY_PATH;
#endif
```


```src/CMakeList.txt
# This file was automatically generated for projects
# without default 'CMakeLists.txt' file.

FILE(GLOB_RECURSE app_sources ${CMAKE_SOURCE_DIR}/src/*.*)

set(COMPONENT_SRCS "main.c")
set(COMPONENT_ADD_INCLUDEDIRS ".")

register_component()

if(CONFIG_EXAMPLE_EMBEDDED_CERTS)
target_add_binary_data(${COMPONENT_TARGET} "certs/aws-root-ca.pem" TEXT)
target_add_binary_data(${COMPONENT_TARGET} "certs/certificate.pem.crt" TEXT)
target_add_binary_data(${COMPONENT_TARGET} "certs/private.pem.key" TEXT)
endif()
```


## その他、MQTTクライアント初期化に必要なパラメーターを初期化
```c:src/main.c
    mqttInitParams.mqttCommandTimeout_ms = 20000;
    mqttInitParams.tlsHandshakeTimeout_ms = 5000;
    mqttInitParams.isSSLHostnameVerify = true;
    mqttInitParams.disconnectHandler = disconnectCallbackHandler;
    mqttInitParams.disconnectHandlerData = NULL;
```
