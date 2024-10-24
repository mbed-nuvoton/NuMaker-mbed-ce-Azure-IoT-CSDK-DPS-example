# Example for provisioning with Azure DPS on Nuvoton's Mbed CE enabled boards

This is an example to show provisioning with [Azure IoT Hub Device Provisioning Service](https://docs.microsoft.com/en-us/azure/iot-dps/) on Nuvoton's Mbed CE enabled boards.
It relies on the following modules:

-   [Mbed OS Community Edition](https://github.com/mbed-ce/mbed-os)
-   [Azure IoT Device SDK port for Mbed OS](https://github.com/ARMmbed/mbed-client-for-azure):
    -   [Azure IoT C SDKs and Libraries](https://github.com/Azure/azure-iot-sdk-c)
    -   [Adapters for Mbed OS](https://github.com/ARMmbed/mbed-client-for-azure/tree/master/mbed/adapters)
    -   Other dependency libraries
-   [NTP client library](https://github.com/ARMmbed/ntp-client)

## Support targets

Platform                        |  Connectivity     | Notes
--------------------------------|-------------------|---------------
Nuvoton NUMAKER_PFM_NUC472      | Ethernet          |
Nuvoton NUMAKER_PFM_M487        | Ethernet          |
Nuvoton NUMAKER_IOT_M487        | Wi-Fi ESP8266     |
Nuvoton NUMAKER_IOT_M467        | Wi-Fi ESP8266     |
Nuvoton NUMAKER_IOT_M263A       | Wi-Fi ESP8266     |

## Support development tools

Use cmake-based build system.
Check out [hello world example](https://github.com/mbed-ce/mbed-ce-hello-world) for getting started.

**_NOTE:_** Legacy development tools below are not supported anymore.
-   [Arm's Mbed Studio](https://os.mbed.com/docs/mbed-os/v6.15/build-tools/mbed-studio.html)
-   [Arm's Mbed CLI 2](https://os.mbed.com/docs/mbed-os/v6.15/build-tools/mbed-cli-2.html)
-   [Arm's Mbed CLI 1](https://os.mbed.com/docs/mbed-os/v6.15/tools/developing-mbed-cli.html)

For [VS Code development](https://github.com/mbed-ce/mbed-os/wiki/Project-Setup:-VS-Code)
or [OpenOCD as upload method](https://github.com/mbed-ce/mbed-os/wiki/Upload-Methods#openocd),
install below additionally:

-   [NuEclipse](https://github.com/OpenNuvoton/Nuvoton_Tools#numicro-software-development-tools): Nuvoton's fork of Eclipse
-   Nuvoton forked OpenOCD: Shipped with NuEclipse installation package above.
    Checking openocd version `openocd --version`, it should fix to `0.10.022`.

## Developer guide

This section is intended for developers to get started, import the example application, build, and get it running and provisioning with Azure DPS.

### Hardware requirements

-   Nuvoton's Mbed CE enabled board

### Software requirements

-   [Azure account](https://portal.azure.com/)

### Hardware setup

-   Switch target board
    -   NuMaker-IoT-M467's Nu-Link2: TX/RX/VCOM to ON, MSG to non-ON
-   Connect target board to host through USB
    -   NuMaker-IoT-M467: Mbed USB drive shows up in File Browser

### Operations on Azure portal

Follow the [doc](https://docs.microsoft.com/en-us/azure/iot-dps/quick-setup-auto-provision) to set up DPS on Azure portal.

For easy, choose [individual enrollment](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#individual-enrollment) using [symmetric key](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#attestation-mechanism).
Take note of the following items.

-   [Device provisioning endpoint](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#device-provisioning-endpoint):
    Service endpoint or global device endpoint from Provisioning service overview page
    
-   [ID scope](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#id-scope):
    ID Scope value from Provisioning service overview page

-   [Registration ID](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#registration-id):
    Registration ID provided when doing individual registration
    
-   [Symmetric key](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#attestation-mechanism):
    Symmetric key from individual registration detail page

### Build the example

In the following, we take [NuMaker-IoT-M467](https://www.nuvoton.com/board/numaker-iot-m467/) as example board to show this example.

1.  Clone the example and navigate into it
    ```sh
    $ git clone https://github.com/mbed-nuvoton/NuMaker-mbed-ce-Azure-IoT-CSDK-DPS-example
    $ cd NuMaker-mbed-ce-Azure-IoT-CSDK-DPS-example
    $ git checkout -f master
    ```

1.  Deploy necessary libraries
    ```
    $ git submodule update --init
    ```
    Or for fast install:
    ```
    $ git submodule update --init --filter=blob:none
    ```

    Deploy further for `mbed-ce-client-for-azure` library:
    ```
    $ cd mbed-ce-client-for-azure; \
    git submodule update --init; \
    cd ..
    ```
    Or for fast install:
    ```
    $ cd mbed-ce-client-for-azure; \
    git submodule update --init --filter=blob:none; \
    cd ..
    ```

1.  Configure connecting with IoT Hub via DPS or straight. To via DPS, set value of `use_dps` to `true`; otherwise, `null`.

    **mbed_app.json5**:
    ```json5
    "use_dps": {
        "help": "Enable connecting with IoT Hub via DPS",
        "options": [null, true],
        "value": true
    },
    ```

1.  On connecting with IoT Hub via DPS,

    1.  Configure HSM type. Set `hsm_type` to `HSM_TYPE_SYMM_KEY` to match [symmetric key](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#attestation-mechanism) attestation type.

        **configs/aziot_user_config.h**:
        ```C++
        #define AZIOT_CONF_APP_HSM_TYPE HSM_TYPE_SYMM_KEY
        ```

    1.  Configure DPS parameters. They should have been noted in [above](#operations-on-azure-portal).

        **configs/aziot_user_config.h**:
        ```C++
        #define AZIOT_CONF_APP_PROVISION_REGISTRATION_ID "<REGISTRATION_ID>"
        #define AZIOT_CONF_APP_PROVISION_SYMMETRIC_KEY "<SYMMETRIC_KEY>"
        #define AZIOT_CONF_APP_PROVISION_ENDPOINT "global.azure-devices-provisioning.net"
        #define AZIOT_CONF_APP_PROVISION_ID_SCOPE "<ID_SCOPE>"
        ```

        **NOTE**: For non-symmetric key attestation type, `AZIOT_CONF_APP_PROVISION_SYMMETRIC_KEY`
        is unnecessary and `AZIOT_CONF_APP_PROVISION_REGISTRATION_ID` is acquired through other means.

1.  On connecting with IoT Hub straight, 

    1.  Configure connection string.

        **configs/aziot_user_config.h**:
        ```C++
        #define AZIOT_CONF_APP_DEVICE_CONNECTION_STRING "<DEVICE_CONNECTION_STRING>"
        ```

1.  Configure network interface
    -   Ethernet: Need no further configuration.

        **mbed_app.json5**:
        ```json5
        "target.network-default-interface-type" : "Ethernet",
        ```

    -   WiFi: Configure WiFi `SSID`/`PASSWORD`.

        **mbed_app.json5**:
        ```json5
        "target.network-default-interface-type" : "WIFI",
        "nsapi.default-wifi-security"           : "WPA_WPA2",
        "nsapi.default-wifi-ssid"               : "\"SSID\"",
        "nsapi.default-wifi-password"           : "\"PASSWORD\"",
        ```

1.  Compile with cmake/ninja
    ```
    $ mkdir build; cd build
    $ cmake .. -GNinja -DCMAKE_BUILD_TYPE=Develop -DMBED_TARGET=NUMAKER_IOT_M467
    $ ninja
    $ cd ..
    ```

1.  Flash by drag-n-drop'ing the built image file below onto **NuMaker-IoT-M467** board

    `build/NuMaker-mbed-ce-Azure-IoT-CSDK-DPS-example.bin`

### Monitor the application through host console

Configure host terminal program with **115200/8-N-1**, and you should see log similar to below:

```
Info: Connecting to the network
Info: Connection success, MAC: a4:cf:12:b7:82:3b
Info: Getting time from the NTP server
Info: Time: Tue Oct27 3:13:29 2020

Info: RTC reports Tue Oct27 3:13:29 2020

Info: Provisioning API Version: 1.3.9

Info: Iothub API Version: 1.3.9

Info: Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED

Info: Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

Info: Registration Information received from service: nuvoton-test-001.azure-devices.net!

Info: Creating IoTHub Device handle

Info: Sending 1 messages to IoTHub every 2 seconds for 2 messages (Send any message to stop)

Info: IoTHubClient_LL_SendEventAsync accepted message [1] for transmission to IoT Hub.

Info: IoTHubClient_LL_SendEventAsync accepted message [2] for transmission to IoT Hub.

Info: Press any enter to continue:

```

### Walk through source code

#### Custom HSM (`hsm_custom/`)

[Azure C-SDK Provisioning Client](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_provisioning_client.md) requires [HSM](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service#hardware-security-module).
This directory provides one custom HSM library for development.
It is adapted from [Azure C-SDK custom hsm example](https://github.com/Azure/azure-iot-sdk-c/tree/master/provisioning_client/samples/custom_hsm_example).

##### Using DPS with symmetric key

If you run provisioning process like this example, provide `provision_registration_id` and `provision_symmetric_key` as [above](#compile-with-mbed-cli).
During provisioning process, `SYMMETRIC_KEY` and `REGISTRATION_NAME` will be overridden through `custom_hsm_set_key_info`.
So you needn't override `SYMMETRIC_KEY` and `REGISTRATION_NAME` below.

If you don't run provisioning process and connect straight to IoT Hub instead, override `SYMMETRIC_KEY` and `REGISTRATION_NAME` below.

```C
// Provided for sample only
static const char* const SYMMETRIC_KEY = "Symmetric Key value";
static const char* const REGISTRATION_NAME = "Registration Name";
```

##### Using DPS with X.509 certificate

First, use the same [step](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-security-x509-get-started) to set up X.509 security on Azure portal.

To use DPS with X.509 certificate, override `COMMON_NAME`, `CERTIFICATE`, and `PRIVATE_KEY` below.
`COMMON_NAME` is the registration ID.
`CERTIFICATE` and `PRIVATE_KEY` are your self-signed or CA-signed certificate and private key in PEM format.

```C
// This sample is provided for development
// For more information please see the devdoc using_custom_hsm.md
static const char* const COMMON_NAME = "custom-hsm-example";
static const char* const CERTIFICATE = "-----BEGIN CERTIFICATE-----""\n"
"BASE64 Encoded certificate Here""\n"
"-----END CERTIFICATE-----";
static const char* const PRIVATE_KEY = "-----BEGIN PRIVATE KEY-----""\n"
"BASE64 Encoded certificate Here""\n"
"-----END PRIVATE KEY-----";
```

#### Platform entropy source (`targets/TARGET_NUVOTON/platform_entropy.cpp`)

Mbedtls requires [entropy source](https://os.mbed.com/docs/mbed-os/v6.4/porting/entropy-sources.html).
On targets with `TRNG` hardware, Mbed OS has supported it.
On targets without `TRNG` hardware, substitute platform entropy source must be provided.
This directory provides one platform entropy source implementation for Nuvoton's targets without `TRNG` hardware.
