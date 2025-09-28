# Central heating using Home Assistant and eBus
I am using a "2 presets" configuration with a high-temperature and low-temperature preset.

I am using a setup which is using a [Schedule helper](https://www.home-assistant.io/integrations/schedule/) to switch the heating-presets (high or low) at which times.

I am also using a [Boolean helper](https://www.home-assistant.io/integrations/input_boolean/) to temporarely pause the heating, for instance when doors are opened or nobody is at home.


<img width="523" height="754" alt="image" src="https://github.com/user-attachments/assets/d734b624-4827-4179-90d2-9233c4466080" /><img width="484" height="358" alt="image" src="https://github.com/user-attachments/assets/2b44944b-b49f-4df2-91f2-22f35734f6e3" />

## Helpers
* `input_boolean.verwarming_schema_gestuurd`: This is a switch to enable/disable the controlling of the heating via the schema.
* `schedule.verwarming`: This is a schedule helper in which a week-schedule can be configured to define when the "high"-heating-preset is desired.
* `input_number.woonkamer_verwarming_hoog_temperatuur`: This is the temperature which is desired to be used for the "high"-preset.
* `input_number.woonkamer_verwarming_laag_temperatuur`: This is the temperature which is desired to be used for the "low"-preset

## Dashboard card
```
type: entities
title: Verwarmingsschema
state_color: false
entities:
  - entity: input_boolean.verwarming_schema_gestuurd
    secondary_info: last-changed
  - type: simple-entity
    name: Schema tijdelijk onderbroken
    entity: input_boolean.verwarming_schema_gestuurd_tijdelijk_gedeactiveerd
    secondary_info: last-changed
    state_color: true
    tap_action:
      action: more-info
    hold_action:
      action: none
    double_tap_action: none
  - type: conditional
    conditions:
      - condition: state
        entity: schedule.verwarming
        state: "on"
    row:
      type: attribute
      name: Huidige aansturing
      icon: mdi:radiator
      format: time
      entity: schedule.verwarming
      attribute: next_event
      prefix: hoog tot
  - type: conditional
    conditions:
      - condition: state
        entity: schedule.verwarming
        state: "off"
    row:
      type: attribute
      name: Verwarming schema
      icon: mdi:radiator-disabled
      format: time
      entity: schedule.verwarming
      attribute: next_event
      prefix: laag tot
  - type: weblink
    name: Schema helper
    url: /config/helpers
    icon: mdi:home-assistant
    new_tab: false
  - type: divider
  - type: custom:multiple-entity-row
    name: Woonkamer temperatuur
    entity: climate.woonkamer_verwarming
    entities:
      - entity: sensor.huidige_temperatuur_woonkamer
        name: Huidig
      - entity: input_number.woonkamer_verwarming_hoog_temperatuur
        name: Hoog
      - entity: input_number.woonkamer_verwarming_laag_temperatuur
        name: Laag
show_header_toggle: false
```
## Automations

### "Woonkamer verwarming hoog op basis van schema"
This automation ensures the "high"-preset to be enabled.
```
alias: Woonkamer verwarming hoog op basis van schema
triggers:
  - trigger: state
    entity_id:
      - schedule.verwarming
    from: "off"
    to: "on"    
  - trigger: state
    entity_id:
      - input_boolean.verwarming_schema_gestuurd
    from: "off"
    to: "on"
  - trigger: state
    entity_id:
      - input_boolean.verwarming_schema_gestuurd_tijdelijk_gedeactiveerd
    from: "on"
    to: "off"
conditions:
  - condition: state
    entity_id: schedule.verwarming
    state: "on"
  - condition: state
    entity_id: input_boolean.verwarming_schema_gestuurd
    state: "on"
  - condition: state
    entity_id: input_boolean.verwarming_schema_gestuurd_tijdelijk_gedeactiveerd
    state: "off"
actions:
  - target:
      entity_id:
        - climate.woonkamer_verwarming
    data:
      temperature: "{{ states(\"input_number.woonkamer_verwarming_hoog_temperatuur\") }}"
    alias: Stel temperatuur in op hoog
    action: climate.set_temperature
mode: single
```

### "Woonkamer verwarming laag op basis van schema"
This automation ensures the "low"-preset to be activated.
```
alias: Woonkamer verwarming laag op basis van schema
triggers:
  - entity_id:
      - schedule.verwarming
    trigger: state
    from: "on"
    to: "off"
  - trigger: state
    entity_id:
      - input_boolean.verwarming_schema_gestuurd_tijdelijk_gedeactiveerd
    from: "on"
    to: "off"
conditions:
  - condition: state
    entity_id: schedule.verwarming
    state: "off"
  - condition: state
    entity_id: input_boolean.verwarming_schema_gestuurd
    state: "on"
  - condition: state
    entity_id: input_boolean.verwarming_schema_gestuurd_tijdelijk_gedeactiveerd
    state: "off"
actions:
  - alias: Stel temperatuur in op laag
    target:
      entity_id:
        - climate.woonkamer_verwarming
    data:
      temperature: "{{ states(\"input_number.woonkamer_verwarming_laag_temperatuur\") }}"
      hvac_mode: heat
    action: climate.set_temperature
mode: single
```

## Polling and updating
This automation keeps the script running. The script should run continuously. I use this automation and a script to prevent Home Assistant from continuously registering the start of automations and scripts.

### Automation: "automation.poll_warm_water_script_starten"
```
alias: Scripts "eBUS polling" actief houden
description: ""
triggers:
  - trigger: homeassistant
    event: start
  - trigger: state
    entity_id:
      - script.poll_warm_water_waarden
    to: "off"
  - trigger: state
    entity_id:
      - script.poll_low_prio_ebus_registers
    to: "off"
conditions: []
actions:
  - action: script.turn_on
    metadata: {}
    data: {}
    target:
      entity_id:
        - script.poll_warm_water_waarden
        - script.poll_low_prio_ebus_registers
mode: single
```

### Script: "script.poll_warm_water_waarden"
This script polls values which are used to temporarily increase the polling frequency of certain other values.
```
sequence:
  - repeat:
      while:
        - condition: template
          value_template: "{{true}}"
      sequence:
        - action: mqtt.publish
          data:
            topic: ebusd/bai/HwcDemand/get
            payload: "?1"
          alias: Poll "warm-water-vraag"
        - delay:
            seconds: 5
        - alias: Poll "flame"
          action: mqtt.publish
          data:
            topic: ebusd/bai/Flame/get
            payload: "?1"
        - delay:
            seconds: 5
        - alias: Poll "Statenumber"
          action: mqtt.publish
          data:
            topic: ebusd/bai/Statenumber/get
            payload: "?1"
        - delay:
            seconds: 5
alias: Poll high-prio ebus waarden
description: ""
icon: mdi:target
```

### Script: "script.poll_low_prio_ebus_registers"
This script runs in a loop and will poll certain values on a lower frequency.
```
sequence:
  - repeat:
      while:
        - condition: template
          value_template: "{{true}}"
      sequence:
        - action: mqtt.publish
          data:
            topic: ebusd/bai/PrEnergySumHwc1/get
            payload: "?1"
          alias: Poll "PrEnergySumHwc1"
        - action: mqtt.publish
          data:
            topic: ebusd/bai/PrEnergyCountHwc1/get
            payload: "?1"
          alias: Poll "PrEnergyCountHwc1"
        - action: mqtt.publish
          data:
            topic: ebusd/bai/HwcStarts/get
            payload: "?1"
          alias: Poll "HwcStarts"
        - action: mqtt.publish
          data:
            topic: ebusd/bai/HwcTemp/get
            payload: "?1"
          alias: Poll "HwcTemp"
        - action: mqtt.publish
          data:
            topic: ebusd/bai/HwcHours/get
            payload: "?1"
          alias: Poll "HwcHours"
        - delay:
            seconds: 60
        - action: mqtt.publish
          data:
            topic: ebusd/bai/CirPump/get
            payload: "?1"
          alias: Poll "CirPump"
        - action: mqtt.publish
          data:
            topic: ebusd/bai/PrEnergySumHc1/get
            payload: "?1"
          alias: Poll "PrEnergySumHc1"
        - action: mqtt.publish
          data:
            topic: ebusd/bai/PrEnergyCountHc1/get
            payload: "?1"
          alias: Poll "PrEnergyCountHc1"
        - action: mqtt.publish
          data:
            topic: ebusd/bai/ReturnTemp/get
            payload: "?1"
          alias: Poll "ReturnTemp"
        - alias: Poll "DisplayedRoomTemp"
          action: mqtt.publish
          data:
            topic: ebusd/350/DisplayedRoomTemp/get
            payload: "?1"
        - delay:
            seconds: 60
alias: Poll low-prio ebus registers
description: ""
icon: mdi:target-variant
```

### Script: "automation.tijdens_cv_brander_frequenter_waarden_ophalen"
```
alias: Tijdens "CV Brander" frequenter waarden ophalen
description: ""
triggers:
  - trigger: state
    entity_id:
      - sensor.ebusd_bai_flame
    to: "on"
    from: "off"
conditions: []
actions:
  - repeat:
      while:
        - condition: state
          entity_id: sensor.ebusd_bai_flame
          state: "on"
      sequence:
        - action: mqtt.publish
          metadata: {}
          data:
            topic: ebusd/bai/PrEnergySumHc1/get
            payload: "?1"
          alias: Poll "PrEnergySumHc1"
        - action: mqtt.publish
          metadata: {}
          data:
            topic: ebusd/bai/PrEnergyCountHc1/get
            payload: "?1"
          alias: Poll "PrEnergyCountHc1"
        - action: mqtt.publish
          metadata: {}
          data:
            topic: ebusd/bai/Flame/get
            payload: "?1"
          alias: Poll "flame"
        - delay:
            hours: 0
            minutes: 0
            seconds: 5
            milliseconds: 0
mode: single
```

### Script: "script.opvragen_ebusd_waarden"
Old script. Might not be needed any more.
```
alias: Alle ebusd entiteiten voorzien van de ontvangen waarden of null
description: ""
sequence:
  - data:
      topic: ebusd/list
      payload: " "
    alias: Publiceer "ebusd/list" commando voor ophalen alle bekende waarden
    action: mqtt.publish
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
        - bai/maintenancedata_HwcTempMax
        - bai/HwcHours
        - bai/HwcStarts
        - bai/HwcTemp
    alias: >-
      Definieer lijst van topics waarvan de waarde direct opgevraagd dient te
      worden
  - alias: Publiceer "get" commando's voor direct ophalen van waarden van topics
    repeat:
      count: "{{ topics | count }}"
      sequence:
        - variables:
            topic: ebusd/{{ topics[repeat.index - 1] }}/get
        - data:
            topic: "{{topic}}"
          action: mqtt.publish
  - variables:
      topics:
        - bai/Flame
        - bai/HwcWaterflow
        - bai/Statenumber
        - bai/HwcDemand
        - bai/PumpPower
    alias: >-
      Definieer lijst van topics waarvan de waarde zeer regelmatig bijgewerkt
      dient te worden
  - alias: >-
      Publiceer "get?1" commando's voor het zeer regelmatig bijwerken van de
      waarden van topics
    repeat:
      count: "{{ topics | count }}"
      sequence:
        - variables:
            topic: ebusd/{{ topics[repeat.index - 1] }}/get
        - data:
            topic: "{{topic}}"
            payload: "?1"
          action: mqtt.publish
  - variables:
      topics:
        - bai/CirPump
        - bai/PrEnergySumHc1
        - bai/PrEnergySumHwc1
        - bai/PrEnergyCountHwc1
        - bai/PrEnergyCountHc1
        - bai/RemainingBoilerblocktime
        - bai/ReturnTemp
        - bai/PumpPowerDesired
    alias: >-
      Definieer lijst van topics waarvan de waarde minder regelmatig bijgewerkt
      dient te worden
  - alias: >-
      Publiceer "get?2" commando's voor het minder regelmatig bijwerken van de
      waarden van topics
    repeat:
      count: "{{ topics | count }}"
      sequence:
        - variables:
            topic: ebusd/{{ topics[repeat.index - 1] }}/get
        - data:
            topic: "{{topic}}"
            payload: "?2"
          action: mqtt.publish
```

## Sensor gas usage for heating-circuit and for hot-water-circuit

### Script: "automation.reset_prenergy_waarden_om_middernacht"
```
alias: Reset PrEnergy* waarden om middernacht
description: ""
triggers:
  - trigger: time_pattern
    hours: "0"
    minutes: "0"
    seconds: "0"
conditions: []
actions:
  - action: script.turn_on
    metadata: {}
    data: {}
    target:
      entity_id: script.reset_prenergy_waarden
mode: single
```

### Script: "script.reset_prenergy_waarden"
```
sequence:
  - alias: Reset ebusd/bai/PrEnergyCountHwc1
    data:
      topic: ebusd/bai/PrEnergyCountHwc1/set
      payload: "0"
    action: mqtt.publish
    enabled: true
  - alias: Reset ebusd/bai/PrEnergyCountHc1
    data:
      topic: ebusd/bai/PrEnergyCountHc1/set
      payload: " 0"
    action: mqtt.publish
    enabled: true
  - alias: Reset ebusd/bai/PrEnergySumHc1
    data:
      topic: ebusd/bai/PrEnergySumHc1/set
      payload: " 0"
    action: mqtt.publish
    enabled: true
  - alias: Reset ebusd/bai/PrEnergySumHwc1
    data:
      topic: ebusd/bai/PrEnergySumHwc1/set
      payload: " 0"
    action: mqtt.publish
    enabled: true
alias: Reset PrEnergy* waarden
description: ""
```


### Helper: "sensor.gasverbruik_voor_warm_water"
```
{{ (states("sensor.ebusd_bai_prenergysumhwc1") |float) * 0.000002010000000 }}
```

### Helper: "sensor.gasverbruik_vandaag_voor_verwarming"
```
{{ (states("sensor.ebusd_bai_prenergysumhc1") |float) * 0.00000203370665409 }}
```
