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

1. The bridge connects to the first HR and Cadence sensor it finds. It is not build for environments where multiple sensors of the same type are active, this will need hard sensor pairing to be implemented of the use of `HR_SENSOR_NAME` and `CADENCE_SENSOR_NAME` with the partial or full BLE sensor name.
2. In testing the bridge ignores the cadence broadcast of my trainer (Elite Justo 2). YMMV here depending on how the trainer has implemented cadence broadcast (as part of the power service or as separate cadence service).
3. The bridge has been tested with only the following components: Rouvy, Polar Verity Sense HR sensor, Garmin Dual HR sensor, Garmin Cadence sensor
4. The bridge can only support one client (Rouvy) at the time
5. The bridge broadcasts the mDNS service even when no BLE sensor is connected
6. The startup sequence of sensors, bridge and Rouvy does not matter, as soon as the bridge starts it searches for BLE sensors
7. If a BLE sensor disconnects the search for a sensor is restarted
8. The BLE sensor name is copied to the model # of the DirCon equivalent so one can see which BLE sensor is actually connected
9. When the firmware is compiled to have 1 sensor group then no battery value is bridged. For Zwift this is no issue as it is not shown anyway.
10. Make sure your switch allows for mDNS/Bonjour traffic.
11. The bridge can only connect to 2.4Ghz networks, 2.4Ghz and 5GHz networks that share the same SSID must be split using different SSID for 2.4 and 5Ghz

## Planned but not yet implemented

1. Likely a small carrier PCB to be created with a PCB power supply and preparation for signaling status via LED's connected to the GPIO's

## Out of scope

1. An adhoc Wireless Access point to do initial configuration (connecting to the network)
2. No web interface for BLE sensor management (pairing)
3. No 'companion' app to setup the bridge
4. no OTA firmware updates
5. No config for fixed IP, it will do DHCP, if fixed IP is needed: use your router to assign a fixed IP
6. No TCP/IP v6 support
7. No bridging of trainers (as the newer ones already have DirCon support)
8. No Wifi 5Ghz support by limitation of the ESP32S3 uBLOX radio
9. No band steering by limitation of the ESP32S3 uBLOX radio

If you want to implement one or more other features: clone this repo. I'll keep this version as-is. 


