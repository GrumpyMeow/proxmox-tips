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
        - bai/RemainingBoilerblocktime
        - bai/ReturnTemp
        - bai/PumpPowerDesired
    alias: >-
      Definieer lijst van topics waarvan de waarde regelmatig bijgewerkt dient
      te worden
  - alias: >-
      Publiceer "get?1" commando's voor het regelmatig bijwerken van de waarden
      van topics
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
  - platform: state
    entity_id:
      - sensor.ebusd_scan
    to: "\"finished\""
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


Because of the above script i can see an accurate representation of my boiler:
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/914762cc-80db-4694-9353-554484a47544)

![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/d7a36f2a-e878-456a-92f5-c23607af984f)


![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/b2f9c4bd-b96a-4ad7-b7f9-e4220140444b)
