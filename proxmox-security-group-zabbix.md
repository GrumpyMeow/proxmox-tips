If you have multiple virtual appliances in Proxmox with Zabbix-clients, then it's possible to create a "Security Group" to configure a set of firewall rules.

Zabbix-server communicates with it's client using TCP 10050.

# Create alias for Zabbix-server IP#
1. Navigate in Proxmox WebUI to the Proxmox Cluster node.
2. Expand the menu "Firewall"
3. Navigate to "Alias"
4. Click "Create" to create an alias
5. Give:
   name: `Zabbix-server`
   IP/CIDR: `192.168.178.10`
   Comment: `Zabbix server`

# Create Security Group "Zabbix"
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
   Comment: `Allow inbound Zabbix traffic`
   Log level: `nolog`
   
   
