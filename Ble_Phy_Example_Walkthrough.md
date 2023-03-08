# PHY Central Example Walkthrough

## Introduction

In this tutorial the ble_phy central example code for the ESP32 is reviewed.The code implements a BLE Cental PHY , which establishes connection on LE 1M PHY and switch to LE 2M PHY once connection is established.The Central then pefrom GATT read operation against specified peer and disconnect once this is completed.

## Includes

This example is located in the examples folder of the ESP-IDF under the [bluetooth/nimble/ble_phy/phy_cent/main](../main). The [main.c](../main/main.c) file located in the main folder contains all the functionality that we are going to review. The header files contained in [main.c](../main/main.c) are:

```c
#include "esp_log.h"
#include "nvs_flash.h"

/* BLE */
#include "nimble/nimble_port.h"
#include "nimble/nimble_port_freertos.h"
#include "host/ble_hs.h"
#include "host/util/util.h"
#include "console/console.h"
#include "services/gap/ble_svc_gap.h"
#include "phy_cent.h"

```

These `includes` are required for the FreeRTOS and underlaying system components to run, including the logging functionality and a library to store data in non-volatile flash memory. We are interested in `“nimble_port.h”`, `“nimble_port_freertos.h”`, `"ble_hs.h"` and `“ble_svc_gap.h”`, `“phy_cent.h”` which expose the BLE APIs required to implement this example.

* `nimble_port.h`: Includes the declaration of functions reuired for intialization of nimble stack. 
* `nimble_port_freertos.h`: initializes and enable NimBLE host task..
* `ble_hs.h`: Defines the functionalities to hande the host event 
* `ble_svc_gap.h`:
* `phy_cenr.h`: 

## Main Entry Point

The program’s entry point is the app_main() function:

```c
void
app_main(void)
{
    int rc;
    /* Initialize NVS — it is used to store PHY calibration data */
    esp_err_t ret = nvs_flash_init();
    if  (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
        ESP_ERROR_CHECK(nvs_flash_erase());
        ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);

    nimble_port_init();
    /* Configure the host. */
    ble_hs_cfg.reset_cb = blecent_on_reset;
    ble_hs_cfg.sync_cb = blecent_on_sync;
    ble_hs_cfg.store_status_cb = ble_store_util_status_rr;

    /* Initialize data structures to track connected peers. */
    rc = peer_init(MYNEWT_VAL(BLE_MAX_CONNECTIONS), 64, 64, 64);
    assert(rc == 0);

    /* Set the default device name. */
    rc = ble_svc_gap_device_name_set("blecent-phy");
    assert(rc == 0);

    /* XXX Need to have template for store */
    ble_store_config_init();

    nimble_port_freertos_init(blecent_host_task);

}

```

The main function starts by initializing the non-volatile storage library. This library allows to save key-value pairs in flash memory and is used by some components such as the Wi-Fi library to save the SSID and password:

```c
esp_err_t ret = nvs_flash_init();
if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
    ESP_ERROR_CHECK(nvs_flash_erase());
    ret = nvs_flash_init();
}
ESP_ERROR_CHECK( ret );
```

## BT Controller and Stack Initialization


