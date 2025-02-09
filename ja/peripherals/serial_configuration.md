# Serial Port Configuration

Most serial (UART) ports on a Pixhawk board can be fully configured via parameters (exceptions are ports that are used for a very specific purpose like RC input, or which are not configurable like `SERIAL 5`).

The configuration makes it easy to (for example):

* change the baudrate on a port.
* run MAVLink on a different port, or change the streamed messages.
* setup dual GPS.
* enable sensors that run on a serial port, such as some [distance sensors](../sensor/rangefinders.md).

## Pre-configured Ports {#default_port_mapping}

The following functions are typically mapped to the same specific serial ports on all boards, and are hence mapped by default:

* MAVLink is mapped to the `TELEM 1` port with baudrate 57600 (for a [telemetry module](../telemetry/README.md)).
* GPS 1 ([gps driver](https://dev.px4.io/en/middleware/modules_driver.html#gps)) is mapped to the `GPS 1` port with a baudrate *Auto* (with this setting a GPS will automatically detect the baudrate - except for the Trimble MB-Two, which requires 115200 baudrate).

All other ports have no assigned functions by default (are disabled).

> **Tip** The ports mappings above can be disabled by setting [MAV_0_CONFIG](../advanced_config/parameter_reference.md#MAV_0_CONFIG) and [GPS_1_CONFIG](../advanced_config/parameter_reference.md#GPS_1_CONFIG) to *Disabled*, respectively.

## How to Configure a Port

All the serial drivers/ports are configured in the same way:

1. Set the configuration parameter for the service/peripheral to the port it will use. > **Note** Configuration parameter names follow the pattern `\*\_CONFIG` or `\*\_CFG` (*QGroundControl* only displays the parameters for services/drivers that are present in firmware). At time of writing the current set is: [GPS_1_CONFIG](../advanced_config/parameter_reference.md#GPS_1_CONFIG), [GPS_2_CONFIG](../advanced_config/parameter_reference.md#GPS_2_CONFIG), [ISBD_CONFIG](../advanced_config/parameter_reference.md#ISBD_CONFIG), [MAV_0_CONFIG](../advanced_config/parameter_reference.md#MAV_0_CONFIG), [MAV_1_CONFIG](../advanced_config/parameter_reference.md#MAV_1_CONFIG), [MAV_2_CONFIG](../advanced_config/parameter_reference.md#MAV_2_CONFIG), [RTPS_CONFIG](../advanced_config/parameter_reference.md#RTPS_CONFIG), [RTPS_MAV_CONFIG](../advanced_config/parameter_reference.md#RTPS_MAV_CONFIG), [TEL_FRSKY_CONFIG](../advanced_config/parameter_reference.md#TEL_FRSKY_CONFIG), [TEL_HOTT_CONFIG](../advanced_config/parameter_reference.md#TEL_HOTT_CONFIG), [SENS_LEDDAR1_CFG](../advanced_config/parameter_reference.md#SENS_LEDDAR1_CFG), [SENS_SF0X_CFG](../advanced_config/parameter_reference.md#SENS_SF0X_CFG), [SENS_TFMINI_CFG](../advanced_config/parameter_reference.md#SENS_TFMINI_CFG), [SENS_ULAND_CFG](../advanced_config/parameter_reference.md#SENS_ULAND_CFG). 
2. Reboot the vehicle in order to make the additional configuration parameters visible.
3. Set the baud rate parameter for the selected port to the desired value.
4. Configure module-specific parameters (i.e. MAVLink streams and data rate configuration).

The [GPS/Compass > Secondary GPS](../gps_compass/README.md#dual_gps) section provides a practical example of how to configure a port in *QGroundControl* (it shows how to use `GPS_2_CONFIG` to run a secondary GPS on the `TELEM 2` port).

## Deconficting Ports

Port conflicts are handled by system startup, which ensures that at most one service is run on a specific port.

> **Caution** At time of writing there is no user feedback about conflicting ports.

## Troubleshooting

### Configuration Parameter Missing from *QGroundControl* {#parameter_not_in_firmware}

*QGroundControl* only displays the parameters for services/drivers that are present in firmware. If a parameter is missing, then you may need to add it in firmware.

> **Note** PX4 firmware includes most drivers by default on [Pixhawk-series](../flight_controller/pixhawk_series.md) boards. Flash-limited boards may comment out/omit the driver (at time of writing this only affects boards based on FMUv2).

You can include the missing driver in firmware by uncommenting (or adding) the driver in the **default.cmake** config file that corresponds to the [board](https://github.com/PX4/Firmware/tree/master/boards/px4) you want to build for. For example, to enable the sf0x driver, you would remove the `#` at the beginning of the line below.

    #distance_sensor/sf0x
    

You will then need to build the firmware for your platform, as described in [Building PX4 Software](https://dev.px4.io/en/setup/building_px4.html) (PX4 Development Guide).

## Further Information

* [MAVLink Peripherals (OSD/GCS/Companion Computers/etc.)](../peripherals/mavlink_peripherals.md)