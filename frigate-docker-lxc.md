With these notes it is possible to take the official Frigate (Docker) container and convert it into a fully working Proxmox LXC container. Thus running it without Docker as virtualization layer. The LXC container should not suffer from the issues which are known for having a ZFS filesystem and nested Docker.

It's actually working :-S
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/4dd103e2-8e29-4f74-a359-3fb2d99be532)

# Initialization
We will using the init-system which is part of the Frigate Docker container. This init-system (`s6-overlay`) manages the several parallel processes for Frigate. Normally in LXC-containers `systemd` is used, but for this container this is not neccesary.

# Process

## Creating the template

Create raw LXC container from Docker container image. Run this on your Proxmox host:
```
apt install skopeo umoci jq
lxc-create frigate -t oci -- --url docker://ghcr.io/blakeblackshear/frigate:stable 
```
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/833ef741-5e41-45e6-a538-64f58381ee9c)
You will now have a LXC container running which is not managed by Proxmox. The commando `lxc-ls` should now show a container named "frigate" next to the Proxmox managed containers.
The filesystem of the running Frigate container is here: `/var/lib/lxc/frigate/rootfs`

Change to the rootfs directory of the Frigate LXC container:
```
cd /var/lib/lxc/frigate/rootfs/
```
Create the Frigate media and config folders:
```
mkdir -p media/frigate
mkdir -p config
```

Edit the init file with: `nano init`. We will be adding some statements to provide DHCP on eth0 in the container. Also we will create a ramdisk for the cache.Add the parts between `# Begin` and `# End` to the file:
```
#!/bin/sh -e

# Begin
/sbin/ip link set dev eth0 up
/sbin/ip link set dev lo up
dhclient

export S6_LOGGING_SCRIPT="T 1 n0 s10000000 T"
export S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0
#export CCACHE_DIR=/root/.ccache
#export CCACHE_MAXSIZE=2G

mkdir -p /tmp/cache
chmod 777 /tmp/cache
mount -t tmpfs -o size=1024m tmpcache /tmp/cache

rm -rf /dev/shm
mkdir -p /config/shm
ln -s /config/shm/ /dev/

mkdir -p /dev/shm
chmod 777 /dev/shm
mkdir -p /dev/shm/logs/frigate/
chmod 777 /dev/shm/logs/frigate
# End

# This is the first program launched at container start.
# We don't know where our binaries are and we cannot guarantee
# that the default PATH can access them.
# So this script needs to be entirely self-contained until it has
# at least /command, /usr/bin and /bin in its PATH.
...
```

Also add this part between `# Begin` and `# End` to the init file:
```
addpath /bin
addpath /usr/bin
addpath /command

# Begin
addpath /usr/lib/btbn-ffmpeg/bin
addpath /usr/local/go2rtc/bin
addpath /usr/local/nginx/sbin
# End

export PATH
...
```
Save and close Nano with CTRL-O, <Enter>, CTRL-X

For running this LXC container within Proxmox some packages need to be installed in the container rootsfs.
```
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt update
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install init isc-dhcp-client -y
```
The package `init` might not be needed anymore.

Lookup in which folder the Proxmox templates are situated by running `cat /etc/pve/storage.cfg`. On my system the output is:
```
dir: local
        disable
        path /var/lib/vz
        content vztmpl,backup,iso

btrfs: local-btrfs
        path /var/lib/pve/local-btrfs
        content rootdir,vztmpl,images,backup,iso
```
In my system the storage named `local` is disabled and the one named `local-btrfs` is used. So i'll be using the path `/var/lib/pve/local-btrfs` in the next command. You might to use path `/var/lib/vz`. If i run `ls /var/lib/pve/local-btrfs` i should see previously downloaded CT templates.

Create the template file by creating an archive from the rootfs filesystem. This command might take a few minutes:
```
ctstorage="/var/lib/pve/local-btrfs"
tar --exclude=dev --exclude=sys --exclude=proc -czf $ctstorage/template/cache/frigate_template.tar.gz ./
```
After creating the Proxmox-LXC-template, you can remove the unmanaged LXC frigate container:
```
cd /
lxc-destroy frigate
```

## Creating the Frigate container in Proxmox

We can create now create a Promox managed Frigate LXC container. This can be done by using the Proxmox Web UI, but also via the command-line:
```
ctid="998"
ctstorage="local-btrfs"
storage="local-btrfs"
pct create $ctid /var/lib/pve/$ctstorage/template/cache/frigate_template.tar.gz \
    -hostname frigate-test -memory 2048 --cores 2 \
    -net0 name=eth0,hwaddr=52:4A:5E:26:58:D8,bridge=vmbr0 \
    -storage $storage -password Passw0rd! \
    --unprivileged 0 --features nesting=1 --cmode console
```    
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/569ed24c-e621-4df4-901c-c4aef24235f7)

For updating the Frigate container in the future, it's smart to keep data stored outside of the Frigate container. The following command is what i use:
```
configpath="/root/frigate"
mediapath="/zpool-storage/media/Frigate"
ctid="998"
mkdir -p $configpath
mkdir -p $mediapath
chmod -R 777 $mediapath
chmod -R 777 $configpath
pct set $ctid -mp0 $configpath,mp=/config/
pct set $ctid -mp1 $mediapath,mp=/media/frigate
```

In the media folder we bound to the container, we will download a sample video file:
```
mediapath="/zpool-storage/media/Frigate"
cd $mediapath
wget -q https://github.com/intel-iot-devkit/sample-videos/raw/master/person-bicycle-car-detection.mp4
```

Here we will be creating an initial configuration file in the folder i've just bound to the container:
```
configpath="/root/frigate"
cat >$configpath/config.yml <<'EOL'
detectors:
  detector_name:
    type: cpu

mqtt:
  enabled: False

go2rtc:
  streams:
    dummy_camera:
      - "ffmpeg:/media/frigate/person-bicycle-car-detection.mp4"
    dummy_camera_sub:
      - "ffmpeg:/media/frigate/person-bicycle-car-detection.mp4#video=h264#width=640"

  webrtc:
    candidates:
      #- 192.168.1.1:8555
      - stun:8555

cameras:
  dummy_camera:
    enabled: true
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
        - path: rtsp://127.0.0.1:8554/dummy_camera 
          input_args: preset-rtsp-restream
          roles:
            - record
            - audio 
  dummy_camera_sub: 
    enabled: true
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
        - path: rtsp://127.0.0.1:8554/dummy_camera_sub 
          input_args: preset-rtsp-restream
          roles:
            - detect

motion:
  threshold: 30
  lightning_threshold: 0.8
  contour_area: 10
  frame_alpha: 0.01
  frame_height: 100
  improve_contrast: False
  mqtt_off_delay: 30

live:
  stream_name: dummy_camera_sub
  height: 720
  quality: 8

record:
  enabled: False

objects:
  track:
    - person
    - car
    - bicycle
EOL
```
The configuration and sample video file are probably not optimal. This as ffmpeg logs quite a few errors.

Edit the lxc config file with:
```
ctid="998"
nano /etc/pve/lxc/$ctid.conf
```

Add in this file:
```
lxc.init.cmd: /init
lxc.log.level: 3
lxc.console.logfile: /var/log/frigate.log
```


## Starting the container

Start Proxmox LXC container:
```
ctid="998"
pct start $ctid
pct status $ctid
```

In Proxmox you should the logging output of Frigate rolling by on the console of the container.

You now probably need to get the IP-address of the container. You can do this:
```
ctid="998"
pct enter $ctid
ip a
...
exit
```
The container should've received an IP-address via DHCP from your router. 

You should be able to access Frigate with the IP-address on port 5000, for example: `http://192.168.178.156:5000/` 

# Notes
* On the proxmox host a log file is created at: `tail -f /var/log/frigate.log`
* Log files in the container are created at: `cd /dev/shm/logs`
* Frigate S6-overlay files are at: `/etc/s6-overlay/s6-rc.d`

Resources:
* https://www.buzzwrd.me/index.php/2021/03/10/creating-lxc-containers-from-docker-and-oci-images/
* https://github.com/pimox/pimox7/issues/160
* https://forum.proxmox.com/threads/permission-denied-failed-to-exec-sbin-init.118710/
* https://forum.proxmox.com/threads/cant-start-lxc.108036/
* https://forum.proxmox.com/threads/how-to-migrate-a-regular-lxc-container-to-a-proxmox-lxc-container.24138/
* https://serverfault.com/questions/731400/how-to-migrate-a-regular-lxc-container-to-a-proxmox-lxc-container
* https://gist.github.com/midoriiro/58b6d16d1578e030e7078917a5872290
* https://github.com/just-containers/s6-overlay
* https://unix.stackexchange.com/questions/119100/cannot-connect-to-any-localhost-connections

# Hardware acceleration
I'm able to get Hardware-acceleration on my AMD APU with the notes below.
BUT! When i put the "preset-vaapi" in my config file, the memory usage explodes making the system unusable. This is apparently a known issue. If i configure another stream, then the hw-acceleration does work flawless. 

```
lxc.apparmor.profile: unconfined
lxc.cap.drop: 
lxc.cap.drop: sys_time sys_module sys_rawio
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
lxc.cgroup2.devices.allow: c 226:128 rwm
```
Add with `nano /init`:
```
export LIBVA_DRIVER_NAME=radeonsi
```
Add with `nano /config/config.yml`:
```
ffmpeg:
  hwaccel_args: preset-vaapi
```
# Notes
* After modifying the config via the web-ui. Saving will stop the LXC container. And should manually be started.
