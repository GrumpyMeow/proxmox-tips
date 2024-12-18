# Create NixOS environments as Proxmox LXC containers
Created on: 18-12-2024, NixOS release 24.11, Proxmox v8.3.1

## Get a NixOS container template from hydra.nixos.org

1. Open in browser: [https://hydra.nixos.org/project/nixos](https://hydra.nixos.org/project/nixos)
2. Select the release: [at this time: `nixos:release-24.11`](https://hydra.nixos.org/jobset/nixos/release-24.11)
3. Navigate to the tab: `Jobs` [nixos:release-24.11 Jobs](https://hydra.nixos.org/jobset/nixos/release-24.11#tabs-jobs)
4. On the Jobs-tab use the search-bar to search for: `nixos.proxmoxLXC.x86_64-linux`
5. Click a green checkmark which sits behind the correct CPU-arch
6. Copy the download link of `nixos-system-x86_64-linux.tar.xz`
7. In the Proxmox Web UI, navigate to the "Storage" and choose `CT Template`
8. Click the button `Download from URL` and paste the download link ([at this time](https://hydra.nixos.org/build/282070945/download/1/nixos-system-x86_64-linux.tar.xz))

## Create a NixOS environment as a Proxmox LXC container
* The exact parameters for `cmode`, `unprivileged`, `ostype`, and `features` is needed for getting this container running.
* The container will get an IP via DHCP
* The root user in the container is passwordless
* SSH is enabled, also for the root user

Run on your Proxmox host:
```
pct create 300 local-btrfs:vztmpl/nixos-system-x86_64-linux.tar.xz --tags nixos --cmode console \
  -ostype nixos --arch amd64 -net0 name=eth0,bridge=vmbr0,hwaddr=BC:24:11:8B:6C:00,ip=dhcp,type=veth \
  -storage local-btrfs --hostname qbittorrent --swap 0 --memory 2048 --unprivileged 0 --features nesting=1

pct resize 300 rootfs +2G
pct start 300
pct enter 300
```
In the shell of CT 300:
1. Run: `source /etc/set-environment`
2. Clear the root password: `passwd --delete root`
3. Start editting: `nano /etc/nixos/configuration.nix`
4. Add the following minimal configuration to allow for remote deployment via SSH
```
{ config, modulesPath, pkgs, lib, ... }:
{
  imports = [ (modulesPath + "/virtualisation/proxmox-lxc.nix") ];
  nix.settings = { sandbox = false; };  
  security.pam.services.sshd.allowNullPassword = true;
  services.openssh = {
    enable = true;
    openFirewall = true;
    settings = {
        PermitRootLogin = "yes";
        PasswordAuthentication = true;
        PermitEmptyPasswords = "yes";
    };
  };
  system.stateVersion = "24.11";
}
```
4. Run the following commands
```
nix-channel --update
nixos-rebuild switch --upgrade
```

I myself use this container as the target to deploy a flake target onto, by:
```
nixos-rebuild switch --flake .#qbittorrent --target-host root@192.168.1.119
```


