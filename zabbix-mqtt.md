# Monitoring MQTT with Zabbix

## MQTT-plugin
The Zabbix-agent2 version has support for MQTT using the [MQTT-plugin](https://www.zabbix.com/documentation/current/en/manual/appendix/config/zabbix_agent2_plugins/mqtt_plugin). 
I've decided that i wanted to use the agent within my Zabbix-LXC-container would be the best instance to have it monitor MQTT.

### Install Zabbix-agent2 in Zabbix-LXC-container
My Zabbix-LXC-container was setup with the older Zabbix-agent (v1) which does not support the MQTT-plugin. This meant that i had to switch from the currently installed agent (v1) and to install the new agent (v2).

I disabled the service of the current intstalled agent (v1):
```
systemctl stop zabbix-agent
systemctl disable zabbix-agent
```

I installed and enabled the new agent (v2):
```
apt install -y zabbix-agent2
systemctl enable zabbix-agent2
```

I renamed the example provisioned `mqtt.conf` file:
```
mv /etc/zabbix/zabbix_agent2.d/plugins/mqtt.conf /etc/zabbix/zabbix_agent2.d/plugins/mqtt.conf.org
```

I created a new `mqtt.conf` file:
```
nano /etc/zabbix/zabbix_agent2.d/plugins/mqtt.conf
```
And put this in the file:
```
Plugins.MQTT.Sessions.hassaddon.Url=tcp://192.168.178.9:1883
Plugins.MQTT.Sessions.hassaddon.Topic=$SYS/broker/clients/active
Plugins.MQTT.Sessions.hassaddon.User=mosquitto
Plugins.MQTT.Sessions.hassaddon.Password=Passw0rd!
```
I'm not actually sure about the value of the topic.

I restarted the agent (v2) service:
```
systemctl restart zabbix-agent2
systemctl status zabbix-agent2
```

If everything went well, uninstall the old agent (v1):
```
apt purge zabbix-agent
apt autoremove
```

## Create the item "Broker clients active"
These steps describe how to create an item which pulls data from your MQTT-broker into Zabbix via the agent:
1. In Zabbix i navigated to "Data collection" > "Hosts"
2. For the entry of my Zabbix server i chose for the link `Items`
3. In the top-right-corner i chose for the "Create item" button.
4. I enter the following values for the fields:
   * Name: `Broker clients active`
   * Type: `Zabbix agent (active)`
   * Key: `mqtt.get["hassaddon","$SYS/broker/clients/active"]`
   * Type of information: `Numeric (unsigned)`
   * Enabled: `checked`
5. I checked the checkbox in front of the newly item and clicked the buttons "Enable" and "Execute now" at the bottom of the screen.

## Create the trigger "Mosquitto: Active clients has changed"
These steps describe how to create a trigger which will raise a problem if the item-value changes:
1. In Zabbix i navigated to "Data collection" > "Hosts"
2. For the entry of my Zabbix server i chose for the link `Triggers`
3. In the top-right-corner i chose for the "Create trigger" button.
4. I enter the following values for the fields:
   * Name: `Mosquitto: Active clients has changed`
   * Event name: `Mosquitto: Active clients has changed`
   * Operational data: `From {ITEM.LASTVALUE1} to {ITEM.LASTVALUE2}`
   * Severity: `Warning`
   * Expression: `last(/Zabbix server/mqtt.get["hassaddon","$SYS/broker/clients/active"],#1)<>last(/Zabbix server/mqtt.get["hassaddon","$SYS/broker/clients/active"],#2)`
   * OK event generation: `None`
   * Allow manual close: `checked`   
   * Enabled: `checked`



## Other items
I also create items for the following MQTT topics..

### Item: EbusD ebus running
* Name: `EbusD ebus running`
* Type: `Zabbix agent (active)`
* Key: `mqtt.get["hassaddon","ebusd/global/running"]`
* Type of information: `Text`
* Enabled: `checked`

### Item: EBusD Ebus signal
* Name: `EbusD ebus running`
* Type: `Zabbix agent (active)`
* Key: `mqtt.get["hassaddon","ebusd/global/signal"]`
* Type of information: `Text`
* Enabled: `checked`

### Item: EbusD update check
* Name: `EbusD update check`
* Type: `Zabbix agent (active)`
* Key: `mqtt.get["hassaddon","ebusd/global/updatecheck"]`
* Type of information: `Text`
* Enabled: `checked`

### Item: Frigate available
* Name: `Frigate available`
* Type: `Zabbix agent (active)`
* Key: `mqtt.get["hassaddon","frigate/available"]`
* Type of information: `Text`
* Enabled: `checked`

## Other triggers

### Trigger: Frigate available
* Name: `Frigate available`
* Event name: `Frigate available: not online`
* Severity: `High`
* Expression: `last(/Zabbix server/mqtt.get["hassaddon","frigate/available"])<>"online"`
* OK event generation: `Expression`
* Allow manual close: `UN-checked`
* Description: `Frigate is not available`
* Enabled: `checked`

