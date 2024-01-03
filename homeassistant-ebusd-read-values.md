The following script will send an "ebusd/list"- and some "get"-commands via MQTT. This to populate Home Assistant entities as early as possible with values.
This script also registers high-priority updating for some values.

```
alias: Alle ebusd entiteiten voorzien van de ontvangen waarden of null
description: ""
sequence:
  - service: mqtt.publish
    data:
      topic: ebusd/list
      payload: " "
    alias: Publiceer "ebusd/list" commando voor ophalen alle bekende waarden
  - delay:
      seconds: 5
  - variables:
      topics:
        - 350/DisplayedHc1RoomTempDesired
        - 350/DisplayedRoomTemp
        - 350/HwcOPMode
        - 350/Hc1DayTemp
        - 350/Hc1HolidayRoomTemp
        - 350/Hc1NightTemp
        - bai/PrEnergySumHc1
        - bai/PrEnergySumHwc1
        - bai/PartloadHcKW
        - bai/PartloadHwcKW
    alias: Definieer lijst van topics waarvan de waarde opgevraagd dient te worden
  - alias: Publiceer "get" commando's voor ophalen van waarden van topics
    repeat:
      count: "{{ topics | count }}"
      sequence:
        - variables:
            topic: ebusd/{{ topics[repeat.index - 1] }}/get
        - service: mqtt.publish
          data:
            topic: "{{topic}}"
  - variables:
      topics:
        - bai/Flame
        - bai/CirPump
        - bai/HwcWaterflow
        - bai/PrEnergySumHc1
        - bai/PrEnergySumHwc1
        - bai/Statenumber
        - bai/HwcDemand
    alias: >-
      Definieer lijst van topics waarvan de waarde regelmatig bijgewerkt dient te worden
  - alias: >-
      Publiceer "get?1" commando's voor het regelmatig bijwerken van de waarden van topics
    repeat:
      count: "{{ topics | count }}"
      sequence:
        - variables:
            topic: ebusd/{{ topics[repeat.index - 1] }}/get
        - service: mqtt.publish
          data:
            topic: "{{topic}}"
            payload: "?1"


```

I combine the script above with the two following automations:
```
alias: Alle ebusd waarden opvragen bij starten Home Assistant
description: ""
trigger:
  - platform: homeassistant
    event: start
action:
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - service: script.turn_on
    metadata: {}
    data: {}
    target:
      entity_id: script.opvragen_ebusd_waarden
```

```
alias: Alle ebusd waarden opvragen na opstarten ebusd/mqtt
description: ""
trigger:
  - platform: mqtt
    topic: ebusd/global/scan
    payload: finished
condition: []
action:
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - service: script.turn_on
    metadata: {}
    data: {}
    target:
      entity_id: script.opvragen_ebusd_waarden
mode: single
```
