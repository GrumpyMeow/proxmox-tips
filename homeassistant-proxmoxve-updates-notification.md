# Introduction
Via HACS i installed the ProxmoxVE-integration to be able to monitor my Proxmox-installation. I would like to see a persistent-notification in Home Assistant when an update is available for ProxmoxVE.


# Automations  

This is the automation to create a persistant notification when updates are available for ProxmoxVE:
```
alias: ProxmoxVE updates notificatie
trigger:
  - platform: numeric_state
    entity_id:
      - sensor.node_pve_total_updates
    above: 0
action:
  - service: persistent_notification.create
    data:
      message: '{{ states("sensor.node_pve_total_updates") }} updates beschikbaar voor ProxmoxVE'
      notification_id: proxmoxve_updates
```

And an automation to close the notification when no updates are available:
```
alias: ProxmoxVE updates notificatie sluiten
trigger:
  - platform: numeric_state
    entity_id:
      - sensor.node_pve_total_updates
    below: 1
action:
  - service: persistent_notification.dismiss
    data:
      notification_id: proxmoxve_updates
```
