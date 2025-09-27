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
