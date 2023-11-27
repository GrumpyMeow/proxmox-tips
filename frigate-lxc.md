# Introduction
I've tried many different apps to monitor my security camera, but i keep on comming back to Frigate and it's free.
* I found the UI of Shinobi counter-intuitive. Also the live-stream was very much delayed. I chose to switch to Frigate again when the Testflight-period expired of the Shinobi-iOS app.

# Create container
1. I used the TTeck Docker script to create the container with: 
   `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/docker.sh)"`
2. During the script i chose "Debian 12", hostname "frigate", disable ipv6
3. I also chose to make the container "Privileged". This as i intend to give the container access to my GPU.
4. I also had to choose the option "enabled fuse OverlayFS". This as my Proxmox host is using ZFS.
5. I choose NOT to install "Portainer" or the "Portainer agent"
6. I chose to install "Docker Compose"

# Bind mounts
1. Stop the created container
2. Make a directory for storage on the proxmox host via the Proxmox shell: `mkdir /root/frigate`
2. Also in the Proxmox shell configure the mount for storage on the container: `pct set 211 -mp0 /root/frigate,mp=/cctv_clips`
3. Add an mount to passthrough the GPU: `nano /etc/pve/lxc/211.conf`
4. Add the following line: `lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file 0, 0`
5. I chose to remove mounting of USB-devices from the LXC configuration file. This as i will not use that.
6. Choose to start the container with: `pct start 211`
   

# Install Frigate
1. Via the Proxmox Web-UI open the console of the created container
2. In the console, run: `cd /opt`
3. Create a docker compose file: `nano docker-compose.yaml` and put this in this file
```
version: '3.9'

services:

  frigate:
    container_name: frigate
    privileged: true
    restart: unless-stopped
    image: ghcr.io/blakeblackshear/frigate:stable
    shm_size: "128mb"
    devices:
      - /dev/dri/renderD128
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /opt/frigate/config:/config:ro
      - /cctv_clips:/media/frigate
      - type: tmpfs
        target: /tmp/cache
        tmpfs:
          size: 1000000000
    ports:
      - "5000:5000"
      - "1935:1935" # RTMP feeds
    environment:
      FRIGATE_RTSP_PASSWORD: "myPassword"
      LIBVA_DRIVER_NAME: "radeonsi"
```

# Continue
1. Run: `mkdir -p /opt/frigate/config`
1. Run: `cd /opt`
2. Run: `docker compose pull`
3. Run: `nano /opt/frigate/config/config.yml`
4. Put this in the file:

```
mqtt:
  enabled: false
ffmpeg:
  hwaccel_args: preset-vaapi
detectors:
  coral:
    type: edgetpu
    device: usb
#Global Object Settings
objects:
  track:
    - person
  filters:
    person:
      min_area: 5000
      max_area: 100000
cameras:
  FrontCam:
    ffmpeg:
      inputs:
        # High Resolution Stream
        - path: rtsp://192.168.22.1:7447/P6C9GcVoZ8mRmYor
          roles:
            - record
        # Low Resolution Stream
        - path: rtsp://192.168.22.1:7447/s0Wju46GtlXua3J4
          roles:
            - detect
    detect:
      width: 480
      height: 360
      fps: 24
```

1. Run: `docker compose pull -d`



Based on:
* https://www.homeautomationguy.io/blog/running-frigate-on-proxmox
* https://github.com/blakeblackshear/frigate/discussions/1111


Todo:
I suspect this also needs to be added to the 211.conf file:
```
lxc.cgroup2.devices.allow: c 226:128 rwm
```
