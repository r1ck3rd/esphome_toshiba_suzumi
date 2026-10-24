# ESPHome Toshiba / Carrier HVAC Controller over UART / WIFI Interface
This repository contains both the hardware (kicad & production files) for an ESP32 module as well as an [ESPHome](https://esphome.io/) component to control Toshiba / Carrier RAS HVAC units independently from the Toshiba Cloud.

ESP8266 is also supported but requires custom wiring.

The device can be used as a local, low cost drop in replacement to control the AC from [Home Assistant](https://www.home-assistant.io/) or others.

Features of the component:
* All operation modes & fan modes supported
* Does not depend on external vendor cloud services
* External temperature sensor support for accurate temperature regulation (external sensor entity configurable)
* Wired and IR remotes still work, even with external temperature sensor
* Supports most special modes / merit modes like silent, fireplace, extended heating temperature range
* Support for power modes on single split units and ionizer toggle
* Additional information like fan speeds, compressor load, refrigerant / pipe temperatures
* WiFi LED toggle (on/off via UART registers 0xDE/0xDF)

Missing functionality:
* Manual defrost (probably unsupported over UART WIFI interface)
* ODU error messages (not decoded yet)
* Timer / schedules (had no interest in this functionality)
* Capability detection (all fan & merit modes will be shown, but not all might have an effect depending on the IDU)


Requirements:
* Toshiba/Carrier RAS single- or multisplit unit with built-in WIFI interface or WIFI module support (others might work as well, but untested)
* ESP32 DevKit 38 Pins (others like NodeMCU 8266 work as well but might require modifications)
* Toshiba HVAC module (see `hardware/` directory)
* ESPhome installed to flash the ESP32 initially


<img src="images/entities.png" width="100%" alt="Home Assistant Entities">

# UART Protocol
The IDU communicates over 5V `UART 9600E1`.
Messages include a length and a distinct header. Initial synchronization is ensured with a 200ms message timeout.

The protocol is relatively simple and can be observed in the `process_uart_rx` and `handle_message` functions, where the message type is identified by its length.
I've decoded most of the protocol by reading the communication with stock WRE-T00BJ10 modules and analyzing the messages against known responses from the Toshiba Cloud.

We can request registers from the IDU, we can write certain registers and also the IDU will send registers if updated externally (IDU / ODU status and changes via IR remote).


# Compatibility
Probably all Toshiba RAS units with wifi capabilities (built-in or with additional module) are supported.

The software has been tested with the following outdoor units:
* RAS-3M18U2AVG-E (Tripe-split)
* RAS-2M14U2AVG-E (Dual-split)
* RAS-25G2KVP-ND (Single-split)
And these indoor units:
* RAS-B10N4KVRG-E (Haori)
* RAS-25G2AVP-ND (Super DAISEIKAI 8)

Other units (with any compatible outdoor unit) that are almost certainly supported:
* Toshiba Seiya
* Toshiba Shorai
* Rebranded models like *Carrier X Inverter*


# Hardware

<img src="images/schematic.png" width="75%" height="75%">
<img src="images/back.jpg" width="320px" alt="PCB Headers">
<img src="images/top.jpg" width="320px" alt="PCB Headers">

The hardware needs three main components:
1) **Microcontroller**. Any ESP32-DevKit with 38 Pins should work. ESP8266 is also supported, see "ESP8266" below.
2) **Bidirectional Voltage Level Translator**. This is needed to connect to the UART interface and shift between 5V from the AC to 3.3V of the ESP32. I used a `Texas Instruments TXS0108EPWR`.
3) **Connector** Toshiba uses as JST PASK connector for the WIFI modules cable, `JST S05B-PASK-2(LF)(SN)` is the correct connector. 

External voltage regulators are not required, as the UART inteface in the indoor units works at 5V and can be directly fed into the ESP32 DevKit.

The Kicad 7 project files and ready-to-use production files (for JLCPCB or PcbWay manufacturing) are available in `hardware/` and `hardware/production` respectively.

*Attention: The Haori units with built-in WIFI modules lack height for the 2x 19p 2.54mm pin sockets. I disabled placement for the pin sockets and soldered the ESP32 on the modules:* 

<img src="images/headers.jpg" width="320px" alt="PCB Headers">

# Installation
This depends on the indoor unit.
The following photos from a Toshiba Haori show how the built-in module can be replaced in a few minutes:

<img src="images/install1.jpg" width="320px" alt="Installation 1">
<img src="images/install2.jpg" width="320px" alt="Installation 2">
<img src="images/install3.jpg" width="320px" alt="Installation 3">
<img src="images/install4.jpg" width="320px" alt="Installation 4">
<img src="images/install5.jpg" width="320px" alt="Installation 5">


# License

This project is released under the MIT License.
