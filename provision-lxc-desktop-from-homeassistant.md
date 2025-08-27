In conformance to `when you have a hammer, everything is a nail` i have a command in Home Assistant which provisions a fresh desktop environment.
Triggering the [`create_desktop` command](https://github.com/koying/ha-remote-command-line) will create:
* An unprivileged/unconfined LXC container
* Debian v12 LXC is upgraded to v13
* Mounted GPU-devices, USB, sound and low-level devices
* Plasma-desktop with Chromium and other apps i regularly use

I have this running on my Minisforum UM480xt machine with directly connected monitor. When i start the container, the Proxmox console will automatically switch to the screen of the container. It runs (near?) native using my Proxmox connected monitor:
<img width="663" height="511" alt="image" src="https://github.com/user-attachments/assets/4df2d4b7-ec84-41cf-ab35-d1937ddf02e0" />

The only thing i notice that i'm running inside an LXC container is that things like Snap and Flatpak have issues.

```
remote_command_line:
  create_desktop:
    ssh_user: root
    ssh_host: hub.lan
    ssh_key: /config/key/id_ed25519
    command_timeout: 900
    command: |
      ctid="800"
      ctname="desktop"
      user="sander"
      password="Passw0rd!"
      template="/var/lib/pve/local-btrfs/template/cache/debian-12-standard_12.7-1_amd64.tar.zst"

      logger "Create container $ctid"
      pct create $ctid $template \
      --storage "local-btrfs" \
      --rootfs volume=local-btrfs:16 \
      --start false \
      --timezone host \
      --unprivileged 0 \
      --cores 4 \
      --features nesting=1,fuse=1,force_rw_sys=1,mount=sysfs\;cgroup \
      --tty=6 \
      --hostname $ctname \
      --memory 4096 \
      --password $password \
      --onboot 0 \
      --arch amd64 \
      --ostype debian \
      --force 1 \
      --description "Big Beautiful Desktop" \
      --hookscript local:snippets/chvt-desktop.pl \
      --unique=1 \
      --net0 name=eth0,bridge=vmbr0,hwaddr=66:66:66:66:66:66,ip=dhcp,type=veth

      logger "Configuring container $ctid"
      cat <<EOF >> /etc/pve/lxc/$ctid.conf
      lxc.apparmor.profile: unconfined
      lxc.cap.drop:
      lxc.cap.drop: sys_time sys_module sys_rawio
      lxc.cgroup2.devices.allow: a

      lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
      lxc.cgroup2.devices.allow: c 226:* rwm

      lxc.mount.entry: /dev/tty7 dev/tty7 none bind,optional,create=file
      lxc.cgroup2.devices.allow: c 4:7 rwm

      lxc.mount.entry: /dev/tty8 dev/tty8 none bind,create=file 0 0
      lxc.cgroup2.devices.allow: c 4:8 rwm

      lxc.mount.entry: /dev/tty0 dev/tty0 none bind,create=file 0 0
      lxc.cgroup2.devices.allow: c 4:0 rwm

      lxc.mount.entry: /dev/input dev/input none bind,optional,create=dir
      lxc.cgroup2.devices.allow: c 13:* rwm

      lxc.mount.entry: /dev/snd dev/snd none bind,optional,create=dir
      lxc.cgroup.devices.allow = c 116:* rwm

      lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
      lxc.cgroup2.devices.allow: c 29:0 rwm

      lxc.mount.entry: /dev/fuse dev/fuse none bind,create=file,optional
      lxc.cgroup2.devices.allow: c 10:229 rwm

      lxc.mount.auto: cgroup:rw
      lxc.mount.auto: sys:rw
      lxc.tty.max: 6
      EOF

      pct snapshot $ctid "Post-configuration"

      logger "Starting container"
      pct start $ctid

      logger "Waiting for network connectivity"
      sleep 10

      logger "Provisioning SSH public keys"
      pct exec $ctid -- sh -c "echo \"....\" >> /root/.ssh/authorized_keys"
      pct exec $ctid -- sh -c "echo \"...\" >> /root/.ssh/authorized_keys"

      pct exec $ctid -- sh -c "apt update"

      logger "Install essential tools"
      pct exec $ctid -- sh -c "apt install -y sudo curl apt-transport-https nano"

      logger "Configure container timezone"      
      pct exec $ctid -- sh -c "
        apt install -y locales;
        locale-gen en_US.UTF-8;
        locale-gen nl_NL.UTF-8;
        localedef -i nl_NL -f UTF-8 nl_NL.UTF-8
      "

      logger "Configuring keyboard"
      pct exec $ctid -- sh -c "tee /etc/default/keyboard <<EOF
      XKBMODEL=\"pc105\"
      XKBLAYOUT=\"nl\"
      XKBVARIANT=\"\"
      XKBOPTIONS=\"\"
      EOF"
      pct exec $ctid -- sh -c "dpkg-reconfigure -f noninteractive keyboard-configuration"
      pct exec $ctid -- sh -c "echo \"keyboard-configuration keyboard-configuration/xkb-keymap select nl\" | debconf-set-selections"


      logger "Upgrading Debian 12 to 13"
      pct exec $ctid -- sh -c "
        DEBIAN_FRONTEND=noninteractive;
        sed -i 's/bookworm/trixie/g' /etc/apt/sources.list;
        apt update;
        DEBIAN_FRONTEND=noninteractive apt-get -o Dpkg::Options::=\"--force-confdef\" -o Dpkg::Options::=\"--force-confold\" dist-upgrade -y
      "

      logger "Update container"
      pct exec $ctid -- sh -c "apt full-upgrade -y"

      pct snapshot $ctid "Post-update"

      logger "Create user $user"
      pct exec $ctid -- sh -c "useradd --create-home -s /bin/bash $user -G users,sudo,video,render,input,audio,lp,systemd-journal,systemd-network"

      logger "Configuring user $user for passwordless sudo"
      pct exec $ctid -- sh -c "
        echo \"$user ALL=(ALL) NOPASSWD:ALL\" | tee /etc/sudoers.d/$user;
        chmod 440 /etc/sudoers.d/$user;
        echo \"$user:$password\" | chpasswd
      "

      pct snapshot $ctid "User-$user-created"

      logger "Install base software"
      pct exec $ctid -- su - $user -c "sudo DEBIAN_FRONTEND=noninteractive apt install -y build-essential libglvnd-dev pkg-config"
      logger "Install Desktop environment"
      pct exec $ctid -- su - $user -c "sudo DEBIAN_FRONTEND=noninteractive apt install -y plasma-desktop sddm"
      logger "Install additional Desktop software"
      pct exec $ctid -- su - $user -c "sudo DEBIAN_FRONTEND=noninteractive apt install -y konsole xterm kcalc dolphin kate ark remmina kde-spectacle"

      logger "Configuring user policy"
      pct exec $ctid -- sh -c "echo 'polkit.addRule(function(action, subject) {if (action.id == \"org.freedesktop.NetworkManager.network-control\" && subject.user == \"$user\") {return polkit.Result.YES;}});' > /etc/polkit-1/rules.d/49-nm-network-control.rules"

      logger "Installing applications"
      pct exec $ctid -- su - $user -c "sudo apt -y install chromium"
      pct exec $ctid -- su - $user -c "sudo apt -y install radeontop"
      pct exec $ctid -- su - $user -c "sudo apt -y install gimp"      
      pct exec $ctid -- su - $user -c "sudo apt -y install kcharselect"
      pct exec $ctid -- su - $user -c "sudo apt -y install filelight"
      pct exec $ctid -- su - $user -c "sudo apt -y install okular"
      pct exec $ctid -- su - $user -c "sudo apt -y install skanpage"
      pct exec $ctid -- su - $user -c "sudo apt -y install thunderbird thunderbird-l10n-nl"
      pct exec $ctid -- su - $user -c "sudo apt -y install vlc"
      pct exec $ctid -- su - $user -c "sudo apt -y install kdenlive"
      pct exec $ctid -- su - $user -c "sudo apt -y install libreoffice-calc"
      pct exec $ctid -- su - $user -c "sudo apt -y install ksystemlog"
      pct exec $ctid -- su - $user -c "sudo apt -y install kwave"
      pct exec $ctid -- su - $user -c "sudo apt -y install webext-plasma-browser-integration"
      pct exec $ctid -- su - $user -c "sudo apt -y install golang"
      pct exec $ctid -- su - $user -c "sudo apt -y install flatpak"

      logger "Installing dutch spelling"
      pct exec $ctid -- su - $user -c "sudo apt -y install aspell-nl hunspell-nl"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --file KDE/Sonnet.conf --group General --key defaultLanguage nl_NL"

      logger "Set user language"
      pct exec $ctid -- su - $user -c "
        kwriteconfig5 --file plasma-localerc --group Formats --key LANG nl_NL.UTF-8;
        kwriteconfig5 --file plasma-localerc --group Translations --key LANGUAGE nl
      "

      logger "Set keyboard layout"
      pct exec $ctid -- su - $user -c "
        kwriteconfig5 --file kxkbrc --group Layout --key LayoutList nl;
        kwriteconfig5 --file kxkbrc --group Layout --key Model pc105;
        kwriteconfig5 --file kxkbrc --group Layout --key Use true;
        kwriteconfig5 --file kxkbrc --group Layout --key VariantList us
      "

      logger "Configuring daily automatic updating"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --file PlasmaDiscoverUpdates --group Global --key UseUnattendedUpdates true"
        
      logger "Configuring Screen timeout"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --file ~/.config/kscreenlockerrc --group Daemon --key Timeout 300"

      logger "Installing Tailscale"
      pct exec $ctid -- su - $user -c "
        curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null;
        curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list;
        sudo apt-get update;
        sudo apt-get install -y tailscale
      "
      logger "Install TriliumNext desktop v0.98.0"
      pct exec $ctid -- su - $user -c "
        wget https://github.com/TriliumNext/Trilium/releases/download/v0.98.0/TriliumNotes-v0.98.0-linux-x64.deb;
        sudo dpkg -i TriliumNotes-v0.98.0-linux-x64.deb;
        rm TriliumNotes-v0.98.0-linux-x64.deb
      "

      logger "Install Bitwarden desktop"
      pct exec $ctid -- su - $user -c "
        wget \"https://bitwarden.com/download/?app=desktop&platform=linux&variant=deb\" -O bitwarden.deb;
        sudo dpkg -i bitwarden.deb;
        rm bitwarden.deb
      "

      logger "Installing VSCode"
      pct exec $ctid -- su - $user -c "
        echo \"code code/add-microsoft-repo boolean true\" | sudo debconf-set-selections;
        sudo apt-get install wget gpg;
        wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg;
        sudo install -D -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/microsoft.gpg;
        rm -f microsoft.gpg
      "
      pct exec $ctid -- sh -c "tee /etc/apt/sources.list.d/vscode.sources <<EOF
      Types: deb
      URIs: https://packages.microsoft.com/repos/code
      Suites: stable
      Components: main
      Architectures: amd64,arm64,armhf
      Signed-By: /usr/share/keyrings/microsoft.gpg
      EOF"
      pct exec $ctid -- su - $user -c "
        sudo apt update;
        sudo apt install -y code
      "

      logger "Installing new version xRDP Server"
      pct exec $ctid -- su - $user -c "
        wget https://www.c-nergy.be/downloads/xRDP/xrdp-installer-1.5.4.zip;
        unzip xrdp-installer-1.5.4.zip;
        rm xrdp-installer-1.5.4.zip;
        chmod +x  xrdp-installer-1.5.4.sh;
        ./xrdp-installer-1.5.4.sh -d -u -p
      "

      logger "Configuring kwinrc"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --file kwinrc --group \$Version --key update_info update_info=kwin.upd:replace-scalein-with-scale,kwin.upd:port-minimizeanimation-effect-to-js,kwin.upd:port-scale-effect-to-js,kwin.upd:port-dimscreen-effect-to-js,kwin.upd:auto-bordersize,kwin.upd:animation-speed,kwin.upd:desktop-grid-click-behavior,kwin.upd:no-swap-encourage,kwin.upd:make-translucency-effect-disabled-by-default,kwin.upd:remove-flip-switch-effect,kwin.upd:remove-cover-switch-effect,kwin.upd:remove-cubeslide-effect,kwin.upd:remove-xrender-backend,kwin.upd:enable-scale-effect-by-default,kwin.upd:overview-group-plugin-id,kwin.upd:animation-speed-cleanup,kwin.upd:replace-cascaded-zerocornered"

      logger "Configuring Breeze Dark theme"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --group General --key ColorScheme BreezeDark"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --file ~/.config/plasmarc --group Theme --key name breeze-dark"

      logger "Configuring Screen Animations"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --file ~/.config/kwinrc --group Compositing --key Enabled true"
      pct exec $ctid -- su - $user -c "kwriteconfig5 --file ~/.config/kwinrc --group Plugins --key wobblywindowsEnabled true"

      logger "Cleaning container"
      pct exec $ctid -- su - $user -c "
        sudo apt purge -y bluez pulseaudio-module-bluetooth;
        sudo apt purge -y powerdevil upower;
        sudo systemctl daemon-reload;
        sudo systemctl enable sddm;
        sudo apt -y autoremove
      "

      logger "Uninstall fwupd"
      pct exec $ctid -- su - $user -c "
        sudo systemctl disable --now fwupd.service;
        sudo apt -y remove fwupd plasma-discover-backend-fwupd
      "

      logger "Disable baloo indexer by default (performance tweak)"
      pct exec $ctid -- su - $user -c "
        balooctl6 disable
      "

      logger "Uninstalling khelpcenter"
      pct exec $ctid -- su - $user -c "
        sudo apt purge -y khelpcenter
      "

      logger "Installing KDE-zeroconf"
      pct exec $ctid -- su - $user -c "
        sudo apt install -y kde-zeroconf
      "
      pct snapshot $ctid "Post-install"

      logger "Rebooting"
      pct exec $ctid -- su - $user -c "sudo reboot now"
```
