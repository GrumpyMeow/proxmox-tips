# Check for Debian updates on machine

## Configure on Debian machine

Open a terminal on the Debian machines you want to check for updates on.

Install the required packages:
```
apt-get install unattended-upgrades apt-listchanges
```

Now create there a configuration file:
```
nano /etc/zabbix/zabbix_agent2.d/90-debian-updates.conf
```
Give that file the following content:
```
UserParameter=debian_updates.package,apt-get upgrade -s | grep -c ^Inst
UserParameter=debian_updates.security,apt-get upgrade -s | grep ^Inst | grep -c security
```
Restart the service:
```
systemctl restart zabbix-agent2
```

## Import template in Zabbix
Create a file on your desktop with the follow contents:
```
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version>6.0</version>
    <date>2023-02-20T08:27:01Z</date>
    <groups>
        <group>
            <uuid>846977d1dfed4968bc5f8bdb363285bc</uuid>
            <name>Templates/Operating systems</name>
        </group>
    </groups>
    <templates>
        <template>
            <uuid>98df8075f43e4f8da8c96c04df1e400f</uuid>
            <template>debian_updates</template>
            <name>Debian Updates</name>
            <description>Configured to check if unattended-upgrade keeps working correctly.

**Requirements**
1. Activate unattended upgrades (preferable with security only) and reboot at e.g. 02:30AM
2. Extend your Zabbix Agent 2 configuration (e.g. /etc/zabbix/zabbix_agent2.d/90-updatenotifier.conf):
UserParameter=debian_updates.package,apt-get upgrade -s | grep -c ^Inst
UserParameter=debian_updates.security,apt-get upgrade -s | grep ^Inst | grep -c security
3. packages is used for all updates and will alert after 160days
4. security updates are checked and will alert after 14days</description>
            <groups>
                <group>
                    <name>Templates/Operating systems</name>
                </group>
            </groups>
            <items>
                <item>
                    <uuid>f406fdf857d24e8aaf14e4899efba664</uuid>
                    <name>Debian Updates - Packages</name>
                    <key>debian_updates.package</key>
                    <delay>4h</delay>
                    <history>180d</history>
                    <description>Retrieves the number of currently needed updates (all).</description>
                    <tags>
                        <tag>
                            <tag>Application</tag>
                            <value>Debian Updates</value>
                        </tag>
                    </tags>
                    <triggers>
                        <trigger>
                            <uuid>205b3ab218734affb4a0217cd1d555db</uuid>
                            <expression>last(/debian_updates/debian_updates.package,#1:now-160d)&gt;=0</expression>
                            <name>Debian - Package Update available</name>
                            <priority>INFO</priority>
                            <description>Your Debian apt system reports pending updates (all).</description>
                            <manual_close>YES</manual_close>
                        </trigger>
                    </triggers>
                </item>
                <item>
                    <uuid>80665a2458eb478688f05660846c89f5</uuid>
                    <name>Debian Updates - Security</name>
                    <key>debian_updates.security</key>
                    <delay>4h</delay>
                    <description>Retrieves the number of currently needed apt security updates.</description>
                    <tags>
                        <tag>
                            <tag>Application</tag>
                            <value>Debian Updates</value>
                        </tag>
                    </tags>
                    <triggers>
                        <trigger>
                            <uuid>45e1cee36d9244ab94004697c068d9d6</uuid>
                            <expression>last(/debian_updates/debian_updates.security,#1:now-14d)&gt;=0</expression>
                            <name>Debian - Security update available</name>
                            <priority>AVERAGE</priority>
                            <description>Your Debian apt system reports pending security updates.</description>
                            <manual_close>YES</manual_close>
                        </trigger>
                    </triggers>
                </item>
            </items>
        </template>
    </templates>
</zabbix_export>
```

Open web WebUI of your Zabbix instance and navigate to "Data collection" > "Templates".
Click the "Import" button at the top-right. Select the file you created on your desktop.
Now assign the template to the hosts which you want to check for available updates.

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
