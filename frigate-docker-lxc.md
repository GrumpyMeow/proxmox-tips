
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

Create template file (i'm using btrfs):
```
tar --exclude=dev --exclude=sys --exclude=proc -czvf /var/lib/pve/local-btrfs/template/cache/frigate_template.tar.gz ./
```
After creating the template, you can remove the raw LXC frigate container:
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
    --unprivileged 0
```    
* Activeer feature: Nesting
* Change option: console to "console"

Start Proxmox LXC container:
```
pct start 998
pct enter 998
/sbin/ip link set dev eth0 up
dhclient
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
#mkdir -p /dev/shm
mkdir -p /config/shm
ln -s /config/shm/ /dev/

mkdir -p /dev/shm
chmod 777 /dev/shm
#mount -t tmpfs -o size=1024m devshm /dev/shm
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
  # Required: name of the detector
  detector_name:
    # Required: type of the detector
    # Frigate provided types include 'cpu', 'edgetpu', 'openvino' and 'tensorrt' (default: shown below)
    # Additional detector types can also be plugged in.
    # Detectors may require additional configuration.
    # Refer to the Detectors configuration page for more information.
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
  dummy_camera: # <--- this will be changed to your actual camera later
    enabled: true
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
        - path: rtsp://127.0.0.1:8554/dummy_camera # <--- the name here must match the name of the camera in restream
          input_args: preset-rtsp-restream
          roles:
            - record
            - audio # <- only necessary if audio detection is enabled
  dummy_camera_sub: # <--- this will be changed to your actual camera later
    enabled: true
    ffmpeg:
      output_args:
        record: preset-record-generic-audio-copy
      inputs:
        - path: rtsp://127.0.0.1:8554/dummy_camera_sub # <--- the name here must match the name of the camera in restream
          input_args: preset-rtsp-restream
          roles:
            - detect

motion:
  # Optional: The threshold passed to cv2.threshold to determine if a pixel is different enough to be counted as motion. (default: 30)
  # Increasing this value will make motion detection less sensitive and decreasing it will make motion detection more sensitive.
  # The value should be between 1 and 255.
  threshold: 30
  # Optional: The percentage of the image used to detect lightning or other substantial changes where motion detection
  #           needs to recalibrate. (default: shown below)
  # Increasing this value will make motion detection more likely to consider lightning or ir mode changes as valid motion.
  # Decreasing this value will make motion detection more likely to ignore large amounts of motion such as a person approaching
  # a doorbell camera.
  lightning_threshold: 0.8
  # Optional: Minimum size in pixels in the resized motion image that counts as motion (default: 10)
  # Increasing this value will prevent smaller areas of motion from being detected. Decreasing will
  # make motion detection more sensitive to smaller moving objects.
  # As a rule of thumb:
  #  - 10 - high sensitivity
  #  - 30 - medium sensitivity
  #  - 50 - low sensitivity
  contour_area: 10
  # Optional: Alpha value passed to cv2.accumulateWeighted when averaging frames to determine the background (default: 0.01)
  # Higher values mean the current frame impacts the average a lot, and a new object will be averaged into the background faster.
  # Low values will cause things like moving shadows to be detected as motion for longer.
  # https://www.geeksforgeeks.org/background-subtraction-in-an-image-using-concept-of-running-average/
  frame_alpha: 0.01
  # Optional: Height of the resized motion frame  (default: 100)
  # Higher values will result in more granular motion detection at the expense of higher CPU usage.
  # Lower values result in less CPU, but small changes may not register as motion.
  frame_height: 100
  # Optional: motion mask
  # NOTE: see docs for more detailed info on creating masks
  #mask: 0,900,1080,900,1080,1920,0,1920
  # Optional: improve contrast (default: shown below)
  # Enables dynamic contrast improvement. This should help improve night detections at the cost of making motion detection more sensitive
  # for daytime.
  improve_contrast: False
  # Optional: Delay when updating camera motion through MQTT from ON -> OFF (default: shown below).
  mqtt_off_delay: 30

live:
  # Optional: Set the name of the stream that should be used for live view
  # in frigate WebUI. (default: name of camera)
  stream_name: dummy_camera_sub
  # Optional: Set the height of the jsmpeg stream. (default: 720)
  # This must be less than or equal to the height of the detect stream. Lower resolutions
  # reduce bandwidth required for viewing the jsmpeg stream. Width is computed to match known aspect ratio.
  height: 720
  # Optional: Set the encode quality of the jsmpeg stream (default: shown below)
  # 1 is the highest quality, and 31 is the lowest. Lower quality feeds utilize less CPU resources.
  quality: 8

# Optional: Record configuration
# NOTE: Can be overridden at the camera level
record:
  # Optional: Enable recording (default: shown below)
  # WARNING: If recording is disabled in the config, turning it on via
  #          the UI or MQTT later will have no effect.
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

On the proxmox host a log file is created at: `tail -f /var/log/frigate.log`
Log files are available: `cd /dev/shm/logs`
See which ports are listening: `netstat -l`
S6-overlay files are at: `/etc/s6-overlay/s6-rc.d`

When an error occurs S6-overlay will terminate the container. It's possible to temporarily remove the `lxc.init.cmd` from the config-file to be able to have more time to fix stuff in the container.

Resources:
* https://www.buzzwrd.me/index.php/2021/03/10/creating-lxc-containers-from-docker-and-oci-images/
* https://github.com/pimox/pimox7/issues/160
* https://forum.proxmox.com/threads/permission-denied-failed-to-exec-sbin-init.118710/
* https://forum.proxmox.com/threads/cant-start-lxc.108036/
* https://forum.proxmox.com/threads/how-to-migrate-a-regular-lxc-container-to-a-proxmox-lxc-container.24138/
* https://serverfault.com/questions/731400/how-to-migrate-a-regular-lxc-container-to-a-proxmox-lxc-container
* https://gist.github.com/midoriiro/58b6d16d1578e030e7078917a5872290
* https://github.com/just-containers/s6-overlay
