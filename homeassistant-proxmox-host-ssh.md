# Introduction
I wanted to use Homeassistant to execute scripts on my Proxmox host. In the past i did this through setting up SSH-connection by logging in as the root.
I searched for an alternative solution which would work without SSH. I though something like a rest-api for triggering scripts would exist. And indeed those solutions did exist, but no common-used-solution that everybody is using. 
This common-used-solution seem to be "SSH". So eventually i chose to use SSH.

# Creating the "Homeassistant" user on the Proxmox host
1. Open a Shell on Proxmox via the Web-gui
2. Create the user “homeassistant” with: `useradd --shell /usr/bash --create-home homeassistant`

# Enable the "Homeassistant" user in Proxmox
1. Navigate in the Proxmox GUI to the "Server View" > “Datacenter”
2. Navigate to “Permissions” > “Users”
3. Click “Add”
4. Give "Username": `homeassistant`
5. Ensure "Realm": `Linux PAM standard authentication`
6. Ensure "Enabled": `Checked`
7. Click the button to create the user
8. Select the user `homeassistant`
9. Choose the button "Password"
10. Enter a password 

# Generate keypair and provide to remote host
1. Open "Homeassistant"
2. Install the "SSH-addon"
3. Configure the "SSH-addon" to accept a password by providing a password
4. Configure the "SSH-addon" to work on port: 22
5. Start the SSH-addon
6. Choose the the button "OPEN WEB-UI"
7. In the console crreate directory: `mkdir -p /config/key/`
8. Generate keypair: `ssh-keygen -t rsa -b 4096 -f /config/key/id_rsa`
9. Copy the public key to the Proxmox-host: `ssh-copy-id -i /config/key/id_rsa.pub homeassistant@192.168.178.2`   (192.168.178.2 is the IP# of my Proxmox-host)
10. Accept the fingerprint of the remote host (SSH addon)
11. Enter the password of the user “homeassistant” you created on the Proxmox-host

# (Optional) Allow Homeassistant to run ryzenadj
In the shell of the Proxmox-host as the root user:
1. Create a usegroup which can run "ryzenadj": `groupadd ryzenadj`
2. Make the user "homeassistant" member of the usergroup: `usermod -aG ryzenadj homeassistant`
3. Create a sudoers-file to allow for running "ryzenadj" with "sudo" without requiring a password: `nano /etc/sudoers.d/ryzenadj`
4. Put in this file:
```
## Members of the ryzenadj group may gain some privileges
%ryzenadj ALL=(ALL) NOPASSWD: /usr/local/bin/ryzenadj
```
I installed the [remote command line integration](https://github.com/koying/ha-remote-command-line) via HACS.
In homeassistant add this yaml:
```
remote_command_line:
  tdp_low:
    ssh_user: homeassistant
    ssh_host: 192.168.178.2          < IP# of my Proxmox host
    ssh_key: /config/key/id_rsa
    command_timeout: 10
    command: "sudo ./ryzenadj --stapm-limit=1000 --fast-limit=1000 --slow-limit=1000 --tctl-temp=90 --power-saving > /dev/null"

  tdp_high:
    ssh_user: homeassistant
    ssh_host: 192.168.178.2
    ssh_key: /config/key/id_rsa      < IP# of my Proxmox host
    command_timeout: 10
    command: "sudo ./ryzenadj --stapm-limit=45000 --fast-limit=45000 --slow-limit=45000 --tctl-temp=90 --max-performance > /dev/null"
```
  
