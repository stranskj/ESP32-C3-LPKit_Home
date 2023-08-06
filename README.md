# ESP32-C3-LPKit with ESPHome
The development board ESP32-C3-LPKit from LaskaKit makes it easy for rapid development of... stuff :-) And ESPhome makes it easy to create a firmware for the stuff. So here is how you can get started with this combination. The board HW description can be found [here](https://github.com/LaskaKit/ESP32-C3-LPKit).

The following description is tested with board version 2.1

## Adding to ESPhome and basic configuration
This are minimal steps you need to do to get your board running under the ESPhome:
1. Click on "New Device"
2. Name the configuration
3. Select device type "ESP-C2
4. Enter your WiFi credentials, unless  you did it for other board.

ESPhome now created basic configuration, which needs following changes:
```
esphome:
  platformio_options:
    board_build.flash_mode: dio
    board_build.f_flash: 40000000L
    board_build.flash_size: 4MB
    board_build.mcu: esp32c3

esp32:
  board: esp32-c3-devkitm-1
  variant: ESP32C3
  framework:
    type: arduino
```
5. Now you can plugin the board to the PC running ESPhome (with USBC cable). In my case, the board shows as `/dev/ttyACM0` (running Debian 11).
6. Hit `Install` to build and upload the firmware.

Now the board should connect to your WiFi, and further firmware updates and log readings work OTA.

[Full working code](examples/esp32c3_lpkit_basic_conf.yaml)

## Configuring sensors with I2C on μŠup connector
The board features a μŠup conector, which makes live of solder-noob so much easier :-) There is plenty sensors and other stuff using the connector.

> ### Note on the conecting cables
> Make sure, you use the correct one :-) The μŠup system uses JST-SH-4 1.0mm 4pin connectors, but make sure that the cable is "crossed". E.g. if you orient both connectors same way, the order of wire colors is the same too.

The μŠup connector is wired to the GPIO8, GPIO10, 3.3V and GND. Following configuration enables I2C on these pins:
```
i2c:
  sda: 8
  scl: 10
  scan: False
  id: bus_a
```
The `scan` parameter is important, when it is set to `True`, the board won't boot.

And to connect for example [BME280](https://github.com/LaskaKit/BMP280-Sensor), add this:
```
sensor:
  - platform: bme280
    i2c_id: bus_a
    temperature:
      name: "BME280 Temperature"
      oversampling: 16x
    pressure:
      name: "BME280 Pressure"
    humidity:
      name: "BME280 Humidity"
    address: 0x77
    update_interval: 15s
```
(sensor details at [ESPhome](https://esphome.io/components/sensor/bme280.html))

[Full working code](examples/esp32c3_lpkit_bme280.yaml)

Or another example with [SCD41](https://github.com/LaskaKit/SCD41-CO2-Sensor)
```
sensor:
  - platform: scd4x
    i2c_id: bus_a
    address: 0x62
    co2:
      name: "CO2"
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
```
(sensor details at [ESPhome](https://esphome.io/components/sensor/scd4x.html))

[Full working code](examples/esp32c3_lpkit_scd41.yaml)

With some sensors, a warning is triggered about the sensor taking too much time to operate. Adding `frequency: 400kHz` to `i2c` section resolves the problem for some of the sensors (in my case, it did for SCD41, but didn't BME280)
