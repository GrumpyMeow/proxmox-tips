# Introduction
For a few years i've been using Zigbee2MQTT which did work fine. But during some refactoring of my homeserver i wanted to switch to ZHA as it's integrated in Home Assistant.
Unfortunatly devices are becomming unavailable and seem to be dropping of the network. So now i'll be switching back to Zigbee2MQTT.

# Zigbee2MQTT

1. I added a new Home Assistant addon repository: https://github.com/zigbee2mqtt/hassio-zigbee2mqtt
2. I installed the Zigbee2MQTT addon
3. Copied the serial port ZHA is using: `/dev/serial/by-id/usb-Silicon_Labs_CP2102N_USB_to_UART_Bridge_Controller_46541a0ee993eb1198edb15b3d98b6d1-if00-port0` and `/dev/ttyUSB1`
4. Disabled the ZHA-integration
5. In the field "serial" i put in: `port: /dev/serial/by-id/usb-Silicon_Labs_CP2102N_USB_to_UART_Bridge_Controller_46541a0ee993eb1198edb15b3d98b6d1-if00-port0`
6. I started the addon
7. I then removed the ZHA-integration
