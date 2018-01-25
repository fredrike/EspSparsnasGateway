# EspSparsnasGateway

This is a Mqtt Gateway for Ikeas energy monitor Sparsnas. The monitor
sends encoded data by radio to a control panel. This device collects the data
and sends it to Mqtt-enabled receivers in Json-format. Thus you need a Mqtt broker
in your network and adjust the settings  in the ino-file:

```c
// Settings for the Mqtt broker:
#define MQTT_USERNAME "<username>"    // If used by the broker
#define MQTT_PASSWORD "<password>"     //    -   *   -
const char* mqtt_server = "192.168.1.79";  // Mqtt brokers IP
```

The data is also printed to the seriaĺ port. If the reception is bad, the received data can be bad.
This gives a CRC-error, the data is in this case not sent via Mqtt but printed via the serial port.

The data sent via Mqtt is in Json format and looks like this:

```json
{"seq":13688,"power":2024,"total":1821.344,"battery":100}
```

## Home Assistant
The Mqtt data can be used anywhere, here's an example for the Home Automation software Home Assistant.
In Home Assistant the sensors can look like this:

### Sparnas energy monitor
```yaml
  - platform: mqtt
    state_topic: "EspSparsnasGateway/values"
    name: "House energy usage"
    unit_of_measurement: "W"
    value_template: '{{ float(value_json.power) | round(0)  }}'

  - platform: mqtt
    state_topic: "EspSparsnasGateway/values"
    name: "House energy meter batt"
    unit_of_measurement: "%"
    value_template: '{{ float(value_json.battery) }}'
```
We then get these sensors:

```
-sensor.house_energy_meter_batt
-sensor.house_energy_usage
```

The result can be seen below 

<img src=https://github.com/bphermansson/EspSparsnasGateway/raw/master/SparsnasHass.png width="25%"/>

# Hardware
The hardware used is a Esp8266-based wifi-enabled Mcu. You can use different devices like a Wemos Mini or a Nodemcu but take care of the Gpio labels that can differ. The receiver is a RFM69B radio transciever. I use an 868MHz device, but a 900MHz should work as well. To this a simple antenna is connected, I use a straight wire, 86 millimeters long connected to the RFM's Ant-connection. The wire shall be vertical, standing up. 

The connection for the RFM69 is hardcoded. This is standard Spi connections set in the spi-library that can't be changed. See https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/using-the-arduino-addon. 

The schematic below shows a Nodemcu, but you can use another ESP8266-based device if you want (except the Esp-01). Use these pin mappings:

```
NodeMcu - Esp12
D1      - Gpio5
D5      - Gpio14
D6      - Gpio12
D7      - Gpio13
D8      - Gpio15
```

![Schematic](https://github.com/bphermansson/EspSparsnasGateway/raw/master/EspSparsnasGateway_schem_Nodemcu.png)

## Hardware hacks to ensure good RF performance.
Also, add two capacitors, 330-470uF and 100nF, to Vin in and Gnd  respectively for stability.
Keep the wires between the RFM69 module and the NodeMCU as short as possible and DO NOT make them 8 cm long hence that calculates into 1/4 wavelenth of 868 MHz.
You will experiance interference and very poor performance if the above is not applied and followed.

The code is based on Sommarlovs version of Ludvig Strigeus code. 
http://elektronikforumet.com/forum/viewtopic.php?f=2&t=85006&start=255
https://github.com/strigeus/sparsnas_decoder
