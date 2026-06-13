# esphome-p1reader
This is a fork from [psvanstrom's project](https://github.com/psvanstrom/esphome-p1reader) with some small customisations to suit it to my needs, focused around compatibility with Belgian/Fluvius meters.

## ESPHome version
The current version in main is tested with ESPHome version `2026.6.1` and `2026.6.3`.

## Hardware
I have used an ESP-12 based NodeMCU for my circuit.

### Parts
- 1 NodeMCU
- 1 BC547 / 2N3904 NPN transistor
- 1 4.7kOhm Resistor
- 1 10kOhm Resistor
- 1 RJ12 6P6C port
- 1 RJ12 to RJ12 cable (6 wires)

### Wiring
The circuit is very simple, basically the 5V TX output on the P1 connector is converted to 3.3V and inverted by the transistor and connected to the UART0 RX pin on the microcontroller. The RTS (request to send) pin is pulled high so that data is sent continuously and GND and 5V are taken from the P1 connector to drive the microcontroller.

#### Wiring NodeMCU ESP-12
![Wiring Diagram](images/wiring.png)

## Installation

Clone the repository and create a companion `secrets.yaml` file with the following fields:

Create a companion `secrets.yaml` file with the following fields:
```
wifi_ssid: <your wifi SSID>
wifi_password: <your wifi password>
fallback_password: <fallback AP password>
encryption_key: the encryption key shared with HA
ota_password: <The OTA password>
```
Check [the Native API Component chapter of the ESPHome documentation](https://esphome.io/components/api.html#configuration-variables) for more info on the encryption key, this page also let you easily generate an encryption key.

Make sure to place the `secrets.yaml` file in the root path of the cloned project. The `fallback_password` and `ota_password` fields can be set to any password before doing the initial upload of the firmware.

Prepare the microcontroller with ESPHome before you connect it to the circuit:
- Install the `esphome` [command line tool](https://esphome.io/guides/getting_started_command_line.html)
- Plug in the microcontroller to your USB port and run `esphome run p1reader.yaml` to flash the firmware
- Remove the USB connection and connect the microcontroller to the rest of the circuit and plug it into the P1 port.
- If everything works, your Home Assistant will now auto detect your new ESPHome integration.

You can check the logs by issuing `esphome p1reader.yaml logs` (or use the super awesome ESPHome dashboard available as a Hass.io add-on or standalone). The logs should output data similar to this every 10 seconds when using `DEBUG` loglevel:
```
[18:40:01][D][data:264]: /ELL5\253833635_A
[18:40:01][D][data:264]:
[18:40:01][D][data:264]: 0-0:1.0.0(210217184019W)
[18:40:01][D][data:264]: 1-0:1.8.0(00006678.394*kWh)
[18:40:01][D][data:264]: 1-0:2.8.0(00000000.000*kWh)
[18:40:01][D][data:264]: 1-0:3.8.0(00000021.988*kvarh)
[18:40:01][D][data:264]: 1-0:4.8.0(00001020.971*kvarh)
[18:40:01][D][data:264]: 1-0:1.7.0(0001.727*kW)
[18:40:01][D][data:264]: 1-0:2.7.0(0000.000*kW)
[18:40:01][D][data:264]: 1-0:3.7.0(0000.000*kvar)
[18:40:01][D][data:264]: 1-0:4.7.0(0000.309*kvar)
[18:40:01][D][data:264]: 1-0:21.7.0(0001.023*kW)
[18:40:01][D][data:264]: 1-0:41.7.0(0000.350*kW)
[18:40:01][D][data:264]: 1-0:61.7.0(0000.353*kW)
[18:40:01][D][data:264]: 1-0:22.7.0(0000.000*kW)
[18:40:01][D][data:264]: 1-0:42.7.0(0000.000*kW)
[18:40:01][D][data:264]: 1-0:62.7.0(0000.000*kW)
[18:40:01][D][data:264]: 1-0:23.7.0(0000.000*kvar)
[18:40:01][D][data:264]: 1-0:43.7.0(0000.000*kvar)
[18:40:01][D][data:264]: 1-0:63.7.0(0000.000*kvar)
[18:40:01][D][data:264]: 1-0:24.7.0(0000.009*kvar)
[18:40:01][D][data:264]: 1-0:44.7.0(0000.161*kvar)
[18:40:01][D][data:264]: 1-0:64.7.0(0000.138*kvar)
[18:40:01][D][data:264]: 1-0:32.7.0(240.3*V)
[18:40:01][D][data:264]: 1-0:52.7.0(240.1*V)
[18:40:01][D][data:264]: 1-0:72.7.0(241.3*V)
[18:40:01][D][data:264]: 1-0:31.7.0(004.2*A)
[18:40:01][D][data:264]: 1-0:51.7.0(001.6*A)
[18:40:01][D][data:264]: 1-0:71.7.0(001.7*A)
[18:40:01][D][data:264]: !7945
[18:40:01][I][crc:275]: Telegram read. CRC: 7945 = 7945. PASS = YES
```

Note that the default is the `INFO` loglevel since logging affects performance.

The last row contains the CRC check. If you constantly get invalid CRC there might be something wrong with the serial communication.

## Technical documentation
Specification overview:
https://www.tekniskaverken.se/siteassets/tekniska-verken/elnat/elmatare-och-elanvandning/aidon-rj12-han-interface-v17a.pdf

- OBIS codes:
https://tech.enectiva.cz/en/installation-instructions/others/obis-codes-meaning/

- P1 hardware info (in Dutch):
http://domoticx.com/p1-poort-slimme-meter-hardware/

- Original Dutch specification (P1 Companion Standard - DSMR 5.0.2).
https://www.netbeheernederland.nl/_upload/Files/Slimme_meter_15_a727fce1f1.pdf

- Luxembourg specification (E-Meter P1 Specification 1.1.2)
https://www.luxmetering.lu/pdf/SPEC%20-%20E-Meter_P1_specification_20210308.pdf

- Belgian specification
https://www.fluvius.be/sites/fluvius/files/2020-03/1901-fluvius-technical-specification-user-ports-digital-meter.pdf
https://www.fluvius.be/sites/fluvius/files/2019-12/e-mucs_h_ed_1_3.pdf

- Swedish specification (Branschrekommendation för lokalt kundgränssnitt för elmätare 2.0)
https://www.energiforetagen.se/globalassets/energiforetagen/det-erbjuder-vi/kurser-och-konferenser/elnat/branschrekommendation-lokalt-granssnitt-v2_0-201912.pdf

- In Lithuania Dutch specification is used (in Lithuanian):
  https://ismaniejiskaitikliai.lt/dazniausiai-uzduodami-klausimai/1#c-8/t-74
 



