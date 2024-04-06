
# Script "Apt full-upgrade"
* Name: `Apt full-upgrade`
* Scope: `Manual host action`
* Type: `Script`
* Execute on: `Zabbix agent`
* Commands: `apt update && apt full-upgrade`
* Host group: `Selected` `Linux servers`
* User group: `Zabbix administrators`
* Required host permissions: `Write`
* Enable confirmation: `checked`
* Confirmation text: `Are you sure you want to do a full-upgrade?`

# Allow running commands on Linux server
Allow running commands on a machine:
1. Edit configuration file: `nano /etc/zabbix/zabbix_agent2.conf`
2. Add at the bottom of the file: `AllowKey=system.run[*]`

# Run zabbix-agent2 as root
Open SSH terminal on a machine you want to run commands on: 
```
ssh root@192.168.178.xxx
```
We are gonna edit the configuration of the zabbix-agent2 service:
```
systemctl edit zabbix-agent2
```
Add at the specified line:
```
[Service]
User=root
Group=root
```

Restart service:
```
systemctl daemon-reload
systemctl restart zabbix-agent2
```
