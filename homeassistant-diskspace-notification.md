# Introduction
I just got an error in Homeassistant about the file-system being readonly. This was caused by Homeassistant running out-of-disk-space in the VM as the disk was only 32gb. I would've liked to have received a notification for this. To resolve the error itself i increased the sized of the disk via Proxmox.

# Data-disk
I also chose to take this opportunity to add a second disk to the Home Assistant VM and to make this the data-disk for Home Assistant.
1. Via Proxmox i added a 40gb on SCSI:1 with writetrough-caching, enabled discard, enabled backup. It's important to choose the size to match or be larger than the current disk on which Home Assistant is installed or an errormessage will be displayed in Home Assistant.
2. Rebooted the Home Assistant VM to get Home Assistant to learn about the newly added disk
3. Navigate in Home Assistant to "Settings" > "Storage"
4. I chose "Move data-disk" and selected the only available option "drive-scsi1"

# Sensors
I would expect that Homeassistant or "Home Assistant Supervisor" to provide information about diskspace, but apparently not.  
But the "systemmonitor" integration does provide this information. With this Yaml the diskspace information will become available:
```
sensor:
  - platform: systemmonitor
    resources:
      - type: processor_use
      - type: disk_use_percent
        arg: /
      - type: disk_use_percent
        arg: /config
      - type: disk_use
        arg: /
      - type: disk_free
        arg: /
```
After reload/reboot Home Assistant i got the following sensors:
* `sensor.processor_use`
* `sensor.disk_use_percent`
* `sensor.disk_use_percent_config`
* `sensor.disk_use`
* `sensor.disk_free`


# Automation
With the following automation i will receive a notification on my iPhone when more than 90% is used:
```
alias: Home Assistant controleer vrije diskruimte
description: ""
trigger:
  - platform: numeric_state
    entity_id:
      - sensor.disk_use_percent
    above: 90
  - platform: numeric_state
    entity_id:
      - sensor.disk_use_percent_config
    above: 90
action:
  - service: notify.mobile_app_iphone_van_sander
    data:
      message: Weinig vrije diskruimte voor Home Assisant
mode: single
```
