# Introduction
Via HACS i installed the ProxmoxVE-integration to be able to monitor my Proxmox-installation. I would like to see a persistent-notification in Home Assistant when an update is available for ProxmoxVE.


# Automations  

This is the automation to create a persistant notification when updates are available for ProxmoxVE:
```
alias: ProxmoxVE updates notificatie
description: ""
trigger:
  - platform: numeric_state
    entity_id:
      - sensor.node_pve_total_updates
    above: 0
condition: []
action:
  - service: persistent_notification.create
    data:
      message: ProxmoxVE updates beschikbaar
      notification_id: proxmoxve_updates
mode: single  
```

And an automation to close the notification when no updates are available:
```
alias: ProxmoxVE updates notificatie sluiten
description: ""
trigger:
  - platform: numeric_state
    entity_id:
      - sensor.node_pve_total_updates
    above: 0
    below: 1
condition: []
action:
  - service: persistent_notification.dismiss
    data:
      notification_id: proxmoxve_updates
mode: single
```
