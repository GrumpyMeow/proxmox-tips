# NixOS KDE/Plasma desktop in a LXC-container on Proxmox
My preference is to re-use as much as possible of my tech-stuff. More tech-stuff means more effort to keep things working.
This preference resulted in me running a small mini pc for my smart-home, but also using that pc for desktop usage.
Currently i'm running a [Minisforum](https://www.minisforum.com/) um4800xt mini-pc. 
This mini-pc is running Proxmox as it's host-os. In VMs/LXCs i'm running: Frigate, Home assistant, OPNsense, Immich and more...

Instead of having a seperate machine to run as a desktop-computer, i'm running full Linux Desktop environments also on this mini-pc. The desktops having full GPU access and have HDMI output for video and audio. 
It's kind-of mind-boggling what kind of magic is behind enabling this ability.

For a long while i've creating my desktop environment based on Debian, but recently felt comfortable to try it using NixOS. I'm happy to share my knowledge/experience how to achieve this.

## How?
It all boils down to running a LXC-container with these properties:
* Devices should be available for the Proxmox-host (thus device-drivers should be installed)
* An Privileged LXC-container with extra low-level privileges
* Passing through the device-files of the host to the LXC-container.

## Know issues
* Sometimes the screen blacks-out for a while when i move my mouse. Currently i don't why this happens. I've noticed that this happens when i move my mouse on the login-screen (SDDM-greeter), but also when i move my mouse when making a snapshot with Spectacle.

## Device-files

### Display passthrough
It all starts with passing through the device-files of the display to the LXC-container. These device-files are in [/dev/dri/](https://en.wikipedia.org/wiki/Direct_Rendering_Infrastructure) of your Proxmox-host.
To "see" these device-files, you can connect with the Console/Shell of your Proxmox-host and run: `ls /dev/dri -lai`, which should display something like this:
```
949 drwxr-xr-x   3 root root       100 Jun  1 00:14 .
  1 drwxr-xr-x  19 root root      4740 Jun 11 08:57 ..
961 drw-rw----   2 root root        80 Jun 11 08:57 by-path
951 crw-rw----+  1 root video 226,   1 Jun 11 08:57 card1
950 crw-rw-rw-+  1 root   303 226, 128 Jun 11 08:57 renderD128
```
The device-files "card1" and "renderD128" are related and for the single display-device my mini-pc has. If you have more display-devices, you should see more "card*" and "renderD*" entries. 
The name "card1" is somewhat weird, previously i had the name "card0" here. I don't know why that changed on my machine. 
It's important to take notice of the major device-number" `226`. As i only have one device, i've chosen to passthrough the whole `/dev/dri/`-folder.
The passthrough is done using this LXC-configuration snippet:
```
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.cgroup2.devices.allow: c 226:* rwm
```
When you got your LXC-container up and running, doing a `ls /dev/dri -lai` in your LXC container should display something like this:
```
949 drwxr-xr-x   3 root root        100  1 jun 00:14 .
  1 drwxr-xr-x  13 root root        800 11 jun 08:57 ..
961 drw-rw----   2 root root         80 11 jun 08:57 by-path
951 crw-rw----+  1 root     44 226,   1 11 jun 08:57 card1
950 crw-rw-rw-+  1 root render 226, 128 11 jun 08:57 renderD128
```
As you can see the output is almost the same as the output we got on the Proxmox-host itself. 
The only difference is that in the LXC-container the groupid "44" is displayed instead of the groupname "video", but this is not important or an issue.

## Sound passthrough
The passthrough of the sound-devices is the same procedure as the "Display passthrough". 
On the Console/Shell of your Proxmox-host run: `ls /dev/dri -lai`, which should display something like this:
```
580 drwxr-xr-x   3 root root      300 Jun  1 00:14 .
  1 drwxr-xr-x  19 root root     4740 Jun 11 08:57 ..
921 drwxr-xr-x   2 root root       80 Jun 11 08:57 by-path
919 crw-rw----+  1 root audio 116,  7 Jun 11 08:57 controlC0
930 crw-rw----+  1 root audio 116, 11 Jun 11 08:57 controlC1
909 crw-rw----+  1 root audio 116,  6 Jun 11 08:57 hwC0D0
927 crw-rw----+  1 root audio 116, 10 Jun 11 08:57 hwC1D0
905 crw-rw----+  1 root audio 116,  2 Jun 11 08:57 pcmC0D3p
906 crw-rw----+  1 root audio 116,  3 Jun 11 08:57 pcmC0D7p
907 crw-rw----+  1 root audio 116,  4 Jun 11 08:57 pcmC0D8p
908 crw-rw----+  1 root audio 116,  5 Jun 11 08:57 pcmC0D9p
926 crw-rw----+  1 root audio 116,  9 Jun 11 08:57 pcmC1D0c
925 crw-rw----+  1 root audio 116,  8 Jun 11 08:57 pcmC1D0p
582 crw-rw----+  1 root audio 116,  1 Jun 11 08:57 seq
581 crw-rw----+  1 root audio 116, 33 Jun 11 08:57 timer
```
The passthrough is done using this LXC-configuration snippet:
```
lxc.mount.entry: /dev/snd dev/snd none bind,optional,create=dir
lxc.cgroup.devices.allow: c 116:* rwm
```


```
lxc.apparmor.profile: unconfined
lxc.cap.drop:
lxc.cap.drop: sys_time sys_module sys_rawio
lxc.cgroup2.devices.allow: a
lxc.mount.auto: cgroup:rw
lxc.mount.auto: sys:rw
```

Passthrough input devices (keyboard, mouse):
```
lxc.mount.entry: /dev/input dev/input none bind,optional,create=dir
lxc.cgroup2.devices.allow: c 13:* rwm
```

Passthrough TTY:
```
lxc.tty.max: 6

lxc.cgroup2.devices.allow: c 4:* rwm

lxc.mount.entry: /dev/tty0 dev/tty0 none bind,create=file 0 0

lxc.mount.entry: /dev/tty7 dev/tty7 none bind,optional,create=file

lxc.mount.entry: /dev/tty8 dev/tty8 none bind,create=file 0 0
```

Passthough framebuffer:
```
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.cgroup2.devices.allow: c 29:0 rwm
```

Passthrough fuse:
```
lxc.mount.entry: /dev/fuse dev/fuse none bind,create=file,optional
lxc.cgroup2.devices.allow: c 10:229 rwm
```
