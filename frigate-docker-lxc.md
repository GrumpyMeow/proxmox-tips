
Create raw LXC container from Docker container image:
```
apt install skopeo umoci jq
lxc-create frigate -t oci -- --url docker://ghcr.io/blakeblackshear/frigate:stable 
```
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/833ef741-5e41-45e6-a538-64f58381ee9c)


Install extra packages in the container rootfs:
```
cd /var/lib/lxc/frigate/rootfs/

/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt update
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install init -y
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install ifupdown2 -y
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install isc-dhcp-client -y
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install dnsutils -y
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install inetutils-ping -y
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install nano -y
/usr/sbin/chroot /var/lib/lxc/frigate/rootfs/ apt install net-tools -y
```

Create template file (i'm using btrfs, you maybe need to use `/var/lib/vz/`):
```
tar --exclude=dev --exclude=sys --exclude=proc -czvf /var/lib/pve/local-btrfs/template/cache/frigate_template.tar.gz ./
```
After creating the Proxmox-LXC-template file, you can remove the raw LXC frigate container:
```
cd /
lxc-destroy frigate
```

Create LXC container:
```
pct create 998 /var/lib/pve/local-btrfs/template/cache/frigate_template.tar.gz \
    -hostname frigate-test -memory 2048 \
    -net0 name=eth0,hwaddr=52:4A:5E:26:58:D8,bridge=vmbr0 \
    -storage local-btrfs -password Passw0rd! \
    --unprivileged 0 --features nesting=1 --cmode console
```    
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/569ed24c-e621-4df4-901c-c4aef24235f7)

Start Proxmox LXC container:
```
pct start 998
pct enter 998
/sbin/ip link set dev eth0 up
dhclient
ip a
```

Edit the init file with: `nano /init`
Add:
```
#!/bin/sh -e

/sbin/ip link set dev eth0 up
dhclient

sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1

export S6_LOGGING_SCRIPT="T 1 n0 s10000000 T"
export S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0
#export CCACHE_DIR=/root/.ccache
#export CCACHE_MAXSIZE=2G

mkdir -p /tmp/cache
chmod 777 /tmp/cache
mount -t tmpfs -o size=4096m tmpcache /tmp/cache

rm -rf /dev/shm
mkdir -p /config/shm
ln -s /config/shm/ /dev/

mkdir -p /dev/shm
chmod 777 /dev/shm
mkdir -p /dev/shm/logs/frigate/
chmod 777 /dev/shm/logs/frigate
```

Also add in "init" file before the `export path` statement:
```
addpath /bin
addpath /usr/bin
addpath /command

addpath /usr/lib/btbn-ffmpeg/bin
addpath /usr/local/go2rtc/bin
addpath /usr/local/nginx/sbin

export PATH
```

Create configuration file:
```
mkdir -p /config
cat >/config/config.yml <<'EOL'
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

Create media folder and download a sample video file:
```
mkdir -p /media/frigate
cd /media/frigate
wget -q https://github.com/intel-iot-devkit/sample-videos/raw/master/person-bicycle-car-detection.mp4
chmod -R 777 /media
```
Increase go2rtc timeout:
```
nano /etc/s6-overlay/s6-rc.d/go2rtc-healthcheck/run

--connect-timeout 100
sleep 100s

```

Run:
```
exit      < leave the container
pct stop 998   
```

Edit the lxc config file with `nano /etc/pve/lxc/998.conf`:
Add in this file:
```
lxc.init.cmd: /init
lxc.log.level: 3
lxc.console.logfile: /var/log/frigate.log
```
Now start the LXC container:
```
pct start 998 --debug
pct enter 998
```
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/1421a307-6879-47fb-95fb-6283f9cdd2c0)

Here the error `[Errno 99] Cannot assign requested address` is displayed, which terminates the container.
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/cafe7ca8-6614-4b07-95fe-7ab205c8f79e)

![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/0f9ba820-e6cb-46c6-a1d9-8e638cb956e3)


* On the proxmox host a log file is created at: `tail -f /var/log/frigate.log`
* Log files are available: `cd /dev/shm/logs`
* See which ports are listening: `netstat -l`
* S6-overlay files are at: `/etc/s6-overlay/s6-rc.d`

When an error occurs S6-overlay will terminate the container. It's possible to temporarily remove the `lxc.init.cmd` from the config-file to be able to have more time to fix stuff in the container.

This is the error which i'm now struggling with to resolve:
```
2024-03-28 20:03:23.344001672  [INFO] Preparing new go2rtc config...
2024-03-28 20:03:23.349380398  [INFO] Starting Frigate...
2024-03-28 20:03:23.951748584  [INFO] Starting go2rtc...
2024-03-28 20:03:24.132432205  20:03:24.132 INF go2rtc version 1.8.4 linux/amd64
2024-03-28 20:03:24.133339287  20:03:24.133 INF [api] listen addr=:1984
2024-03-28 20:03:24.133420029  20:03:24.133 INF [rtsp] listen addr=:8554
2024-03-28 20:03:24.133949806  20:03:24.133 INF [webrtc] listen addr=:8555
2024-03-28 20:03:26.080959291  [2024-03-28 20:03:26] frigate.app                    INFO    : Starting Frigate (0.13.2-6476f8a)
2024-03-28 20:03:26.201767502  [2024-03-28 20:03:26] peewee_migrate.logs            INFO    : Starting migrations
2024-03-28 20:03:26.211708182  [2024-03-28 20:03:26] peewee_migrate.logs            INFO    : There is nothing to migrate
2024-03-28 20:03:26.224045329  [2024-03-28 20:03:26] frigate.app                    INFO    : Recording process started: 796
2024-03-28 20:03:26.230855328  [2024-03-28 20:03:26] frigate.app                    INFO    : go2rtc process pid: 138
2024-03-28 20:03:26.238905991  [Errno 99] Cannot assign requested address
2024-03-28 20:03:26.243526196  [2024-03-28 20:03:26] frigate.record.maintainer      INFO    : Exiting recording maintenance...
2024-03-28 20:03:27.617150972  [INFO] Service Frigate exited with code 1 (by signal 0)
s6-rc: info: service legacy-services: stopping
s6-rc: info: service legacy-services successfully stopped
s6-rc: info: service nginx: stopping
s6-rc: info: service go2rtc-healthcheck: stopping
2024-03-28 20:03:27.647206157  [INFO] The go2rtc-healthcheck service exited with code 256 (by signal 15)
s6-rc: info: service go2rtc-healthcheck successfully stopped
2024-03-28 20:03:27.755264504  [INFO] Service NGINX exited with code 0 (by signal 0)
```

Resources:
* https://www.buzzwrd.me/index.php/2021/03/10/creating-lxc-containers-from-docker-and-oci-images/
* https://github.com/pimox/pimox7/issues/160
* https://forum.proxmox.com/threads/permission-denied-failed-to-exec-sbin-init.118710/
* https://forum.proxmox.com/threads/cant-start-lxc.108036/
* https://forum.proxmox.com/threads/how-to-migrate-a-regular-lxc-container-to-a-proxmox-lxc-container.24138/
* https://serverfault.com/questions/731400/how-to-migrate-a-regular-lxc-container-to-a-proxmox-lxc-container
* https://gist.github.com/midoriiro/58b6d16d1578e030e7078917a5872290
* https://github.com/just-containers/s6-overlay
