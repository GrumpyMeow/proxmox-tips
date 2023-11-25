# Introduction
I recently bought a [MinisForum UM450XT](https://store.minisforum.de/products/minisforum-venus-series-um560) to be used as my homeserver. The version i bought has got a Ryzen 7 4800H as the cpu.
This CPU is very powerfull but will use up-to 45watt of power. As this machine is 24/7/365 running, making it run as efficiently as possible makes sense. When nobody is at home or during the night the system can be throttled without anybody noticing.

# CPU Scaling Governor
I started out with using the [Proxmox VE CPU Scaling Governor script from TTeck](https://tteck.github.io/Proxmox/) to select the Powersave-profile. This did work, but kept the system running idle at 15watt.

# BIOS setting
In the BIOS i noticed an option with which the TDP can be selected. The default was a TDP of 45watt. I selected a TDP of 15watt which resulted in a 8watts being used. Selecting this TDP of course seriously capped the power/speed of the CPU.

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

Running this command would make the system run at a very low power usage TDP of 1watt and resulted in an acutal usage of:
```
./ryzenadj --stapm-limit=1000 --fast-limit=1000 --slow-limit=1000 --tctl-temp=90 --info  --power-saving 
```

This would:
./ryzenadj --stapm-limit=45000 --fast-limit=45000 --slow-limit=1000 --tctl-temp=90 --info  --power-saving 


# Benchmarking
For comparing the various scenario's i used the built-in benchmark functionality of 7zip. Which i installed with:
```
apt install -y 7zip
```

To run a benchmark i used the following command on the Proxmox host via the Shell:
```
7zz b
```

The first result i got was:
```
root@pve:~# 7zz b

7-Zip (z) 22.01 (x64) : Copyright (c) 1999-2022 Igor Pavlov : 2022-07-15
 64-bit locale=en_US.UTF-8 Threads:16

Compiler: 12.2.0 GCC 12.2.0
Linux : 6.5.11-4-pve : #1 SMP PREEMPT_DYNAMIC PMX 6.5.11-4 (2023-11-20T10:19Z) : x86_64
PageSize:4KB THP:madvise hwcap:2 hwcap2:2
AMD Ryzen 7 4800H with Radeon Graphics (860F01) 

1T CPU Freq (MHz):   386   388   391   392   391   389   391
8T CPU Freq (MHz): 788% 383   787% 383  

RAM size:   15415 MB,  # CPU hardware threads:  16
RAM usage:   3559 MB,  # Benchmark threads:     16

                       Compressing  |                  Decompressing
Dict     Speed Usage    R/U Rating  |      Speed Usage    R/U Rating
         KiB/s     %   MIPS   MIPS  |      KiB/s     %   MIPS   MIPS

22:       6310  1103    557   6139  |      54633  1272    366   4659
23:       5922  1104    546   6035  |      51009  1203    367   4413
24:       5686  1085    563   6114  |      51079  1220    367   4482
25:       5496  1077    582   6275  |      49455  1202    366   4400
----------------------------------  | ------------------------------
Avr:      5854  1092    562   6141  |      51544  1224    367   4488
Tot:            1158    464   5315
````

```
root@pve:~# ryzenadj/RyzenAdj/build/ryzenadj --info
CPU Family: Renoir
SMU BIOS Interface Version: 18
Version: v0.14.0 
PM Table Version: 370005
|        Name         |   Value   |     Parameter      |
|---------------------|-----------|--------------------|
| STAPM LIMIT         |     1.000 | stapm-limit        |
| STAPM VALUE         |     4.230 |                    |
| PPT LIMIT FAST      |     1.000 | fast-limit         |
| PPT VALUE FAST      |     4.239 |                    |
| PPT LIMIT SLOW      |     1.000 | slow-limit         |
| PPT VALUE SLOW      |     3.923 |                    |
| StapmTimeConst      |   275.000 | stapm-time         |
| SlowPPTTimeConst    |     5.000 | slow-time          |
| PPT LIMIT APU       |    54.000 | apu-slow-limit     |
| PPT VALUE APU       |     3.923 |                    |
| TDC LIMIT VDD       |    58.000 | vrm-current        |
| TDC VALUE VDD       |     0.780 |                    |
| TDC LIMIT SOC       |    15.000 | vrmsoc-current     |
| TDC VALUE SOC       |     1.292 |                    |
| EDC LIMIT VDD       |    96.000 | vrmmax-current     |
| EDC VALUE VDD       |     7.501 |                    |
| EDC LIMIT SOC       |    20.000 | vrmsocmax-current  |
| EDC VALUE SOC       |     0.000 |                    |
| THM LIMIT CORE      |    90.000 | tctl-temp          |
| THM VALUE CORE      |    29.657 |                    |
| STT LIMIT APU       |     0.000 | apu-skin-temp      |
| STT VALUE APU       |     0.000 |                    |
| STT LIMIT dGPU      |     0.000 | dgpu-skin-temp     |
| STT VALUE dGPU      |     0.000 |                    |
| CCLK Boost SETPOINT |    95.000 | power-saving /     |
| CCLK BUSY VALUE     |    99.615 | max-performance    |
```

Retrieving the Scaling Governor with:
```
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

# Benchmark results
| Scaling governor | Idle watt | Benchmarking watt | stapm-limit  |  fast-limit | slow-limit | threads | 8T MHZ | AVG Compressing MIPS | AVG Decompressing MIPS | 
|------------------|------------|-------------------|--------------|-------------|------------|---------|-------|-------------------|--------------------|
| ondemand         |       8.7w |             11.0w |        1.000 |       1.000 |      1.000 |      16 |   380 |             6.001 |              4.380 |
| ondemand         | 8.7w - 12w |             71.1w |       45.000 |      45.000 |     45.000 |      16 | 3.017 |            47.122 |             53.957 |
| powersave        | 8.7w - 12w |             18.1w |       45.000 |      45.000 |     45.000 |      16 | 1.381 |            23.744 |              18.329 |
