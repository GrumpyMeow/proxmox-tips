# Introduction
I recently bought a [MinisForum UM450XT](https://store.minisforum.de/products/minisforum-venus-series-um560) to be used as my homeserver. The version i bought has got a Ryzen 7 4800H as the cpu.
This CPU is very powerfull but will use up-to 45watt of power. As this machine is 24/7/365 running, making it run as efficiently as possible makes sense.

# CPU Scaling Governor
I started out with using [Proxmox VE CPU Scaling Governor script from TTeck](https://tteck.github.io/Proxmox/) which did work, but kept the system running idle at 15watt.

# BIOS setting
I was able to select a TDP of 15watt from the BIOS, which also reduced the power usage. But this capped the CPU speed.

# RyzenAdj
I came across [RyzenAdj](https://github.com/FlyGoat/RyzenAdj) which allowed for doing the same as the BIOS setting, but then on-the-fly.

I had to compile the application on the Proxmox host. For which i used this script:
```
apt install build-essential cmake libpci-dev git
git clone https://github.com/FlyGoat/RyzenAdj.git
cd RyzenAdj
git fetch --prune
git pull
rm -r win32
rm -rf build
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
if [ -d ~/.local/bin ]; then ln -s ryzenadj ~/.local/bin/ryzenadj && echo "symlinked to ~/.local/bin/ryzenadj"; fi
if [ -d ~/.bin ]; then ln -s ryzenadj ~/.bin/ryzenadj && echo "symlinked to ~/.bin/ryzenadj"; fi
if [ -d /usr/local/bin ]; then ln -s ryzenadj /usr/local/bin/ryzenadj && echo "symlinked to /usr/local/bin/ryzenadj"; fi
```

Running this command would make the system run at a very fast low power usage:
```
./ryzenadj --stapm-limit=1000 --fast-limit=1000 --slow-limit=1000 --tctl-temp=90 --info  --power-saving 
```

This would:
./ryzenadj --stapm-limit=45000 --fast-limit=45000 --slow-limit=1000 --tctl-temp=90 --info  --power-saving 
