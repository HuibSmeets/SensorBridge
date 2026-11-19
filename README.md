# SensorBridge
Is a BLE to DirCon sensor bridge for HR and cadence sensors[^1] using and Espressif ESP32S3. This piece of work is mainly targeted at Rouvy users that run Rouvy on an Apple TV, run into the 2 bluetooth connection limitation and do not have a trainer that can do the sensor bridging (reliably). Until this moment, unlike Zwift, MyWoosh and others Rouvy has not added a bridge function into their Companion App.

DirCon uses mDNS/Bonjour service discovery and basically uses the same GATT structure as BLE does. 

[^1]: It ignores cadence data from smart trainers, only actual cadence sensor is bridged.

## Hardware
The bridge uses a XIAO ESP32S3 from Seedstudio. Not tested but the code should also run on an Arduino ESP32 as it uses the same BLE/Wifi interface (uBLOX) that allows for simultaneous use of the radio for Wifi and BLE. No further hardware than a USB power supply and a case for the ESP is needed.

## Firmware
Obviously, I must say, a larger part of the firmware is vibe coded using Claude.ai. It is still readable and maintainable code. When desired the code can be extended with, example, connection signaling via LED, explicit sensor pairing, etc

# Prerequisites

The code can be compiled and uploaded to the ESP via the Arduino IDE. For this is needed:
1. An __XIAO ESP32S3__
2. Installing the __ARDUINO IDE__ if not yet done
3. via boards manager installl the __'esp32 by Espressif Systems'__ cores and select the XIAO_ESP32S3 as board
4. and via library manager install the __'NimBLE-Arduino by h2zero'__,
5. the __XIAO ESP32S3__ can only connect to 2.4GHz network, if your 2.4 and 5Ghz network share the same __SSID__ the 2.4Ghz must be given its own __SSID__

The default settings(as of November 2026) for the ESP cores in the IDE need no change.

# Compiling time configuration

For reasons of simplicity and an unlikely need for constant configuration changes various option need to be set in the source code before compiling and uploading

## Wifi connectivity 

Update `WIFI_SSID` with the __SSID__ of your 2.4GHz network.

Update `WIFI_PASS` with the password of your 2.4GHz network.

The Wifi stack should handle the various security/encryption challenges itself, it connected without issues to my WPA2-PSK network.

## Sensor filtering

No explicit BLE pairing is implemented, the bridge connects to the first sensor it finds that broadcasts the HR and the Cadence GATT service characteristic. It ignores the cadence characteristic from my smart trainer. Please verify that it connects to the correct cadence.

To force a specific sensor to be connected a prefix filter can be defined for both sensors

`HR_SENSOR_NAME` and `CADENCE_SENSOR_NAME` can be updated to only connect to sensors which names __starts__ with that string. 

## mDNS sensor grouping

Although official out of scope this bridge it also works with Zwift (tests with MyWhoosh were not successful) but to work with Zwift the 2 sensors must be in 1 group. For use with Zwift set 'CADENCE_GROUP` to 0. 

Rouvy accepts the sensors in separate and combined groups however when using the combined sensor group the battery level of both sensors are not shown as there can only be 1 battery level per sensor group. I've chosen to suppress the battery level in that case.

## Various other compile time configurations

There are various other compile time option. Notably `MANUFACTURER_NAME` and `DEVICE_NAME_PREFIX`. The first speaks for itself, the latter is the name of the sensor as exposed by the bridge concatenated with " HR" and " CAD". In case of sensor grouping " HR CAD" will be concatenated.

# Good to know

## what it does, what it does not and other things to know about

1. it connects to the first HR and Cadence sensor it finds. It is not build for environments where multiple sensors of the same type are active, this needed hard sensor pairing to be implemented of the use of `HR_SENSOR_NAME` and `CADENCE_SENSOR_NAME` with the partial or full BLE sensor name.
2. 
