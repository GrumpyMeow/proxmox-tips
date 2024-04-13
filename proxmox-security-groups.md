If you have multiple virtual appliances in Proxmox with Zabbix-clients, then it's possible to create a "Security Group" to configure a set of firewall rules.

Zabbix-server communicates with it's Zabbix client instances using TCP 10050. 
It's also possible to have an "active" Zabbix client instance which will communicate using TCP 10051 to the Zabbix-server.

# Create alias for Zabbix-server IP#
1. Navigate in Proxmox WebUI to the Proxmox Cluster node.
2. Expand the menu "Firewall"
3. Navigate to "Alias"
4. Click "Create" to create an alias
5. Give:
   name: `Zabbix-server`
   IP/CIDR: `192.168.178.10`
   Comment: `Zabbix server`

# Create Security Group "Zabbix-client"
1. Navigate in Proxmox WebUI to the Proxmox Cluster node.
2. Expand the menu "Firewall"
3. Navigate to "Security Group"
4. Click "Create" to create a group
5. Give name: `zabbix-client`
6. Confirm with the "OK" button
7. Select the group "zabbix-client"
8. On the right-side choose for "Add"
9. Configure: 
   Direction: `In`
   Action: `Accept`
   Protocol: `TCP`
   Dest.Port: `10050`
   Source: alias `zabbix-server`
   Comment: `Allow inbound Zabbix agent traffic`
   Log level: `nolog`
10. Choose again on the right side for "Add"
11. Configure: 
   Direction: `Out`
   Action: `Accept`
   Protocol: `TCP`
   Dest.Port: `10051`
   Destination: alias `zabbix-server`
   Comment: `Allow outbound Zabbix agent (active) traffic`
   Log level: `nolog`

# Insert Security Group "Zabbix-client" to "OPNsense"
1. Navigate in Proxmox to your "OPNsense" container
2. Navigate to the Firewall-menu
3. Click the "Insert: security group" button
4. Select:
   Security Group: `zabbix-client`
   Enabled: `checked`
5. Confirm with the "OK" button
   
# Create Security Group for "Zabbix-ping"
I also configured Zabbix-server to ping my network devices. I want to allow this ping-checks.
So i created a "Security Group" named `zabbix-ping`.
With a rule:
Direction: `In`
Action: `Accept`
Protocol: `icmp`
ICMP-type: `echo-request`
Source: `zabbix-server`
Comment: `Allow inbound Zabbix ping`
Log level: `nolog`
I added this security group to all my LXCs/VMs.

# Create Security Group for "Mail-delivery"
I also create a Security Group to allow mail-delivery (smtp, tcp/25) to my mail-server.

I create an alias for my mail-server:
* Name: `Mail-server`
* IP/CIDR: `192.168.178.97`
* Comment: `Mail server`

So i created a "Security Group" named `mail-delivery`.
With a rule:
Direction: `Out`
Action: `Accept`
Protocol: `tcp`
Dest Port: `80`
Destination: alias `mail-server`
Comment: `Allow mail delivery`
Log level: `nolog`
I added this security group to all my LXCs/VMs which make use of mail-delivery (Zabbix, PVE-node).
 
