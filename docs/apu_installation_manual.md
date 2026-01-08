# Installation of FreedomBox and/or Home Assistant on PC Engines APU3B2 (Headless SBC)

## Prerequisites

**Hardware:**

* PC Engines APU3B2 board with mSATA module
* Boot/utility media: microSD card or USB flash drive (minimum 8 GB; microSD recommended)
* Serial cable (RS232 to USB)

**Host system:**

* Linux laptop with:

  * minicom (port settings: 115200 8N1)
  * nmap

## Problem Description

There are two common ways to install Home Assistant or FreedomBox:

1. Using a Docker container on an existing Linux system
2. Using the Home Assistant OS image or FreedomBox image

The first procedure is simpler and works well on Alpine Linux.

The PC Engines APU uses a legacy BIOS (MBR-based, no UEFI). Neither the Home Assistant OS image nor the FreedomBox image will boot without updating the BIOS to UEFI.

**Warning:** Updating the BIOS carries a risk of bricking the board. A SPI module is available for recovery at: [https://www.pcengines.ch/spi1a.htm](https://www.pcengines.ch/spi1a.htm)

---

## A. On a Linux System Using a Docker Container

### 1. Installing Alpine Linux on the APU (Legacy BIOS, Recommended)

#### 1.1 Preparing the Boot Media

1. Download the Alpine Linux installer from: [https://alpinelinux.org/downloads/](https://alpinelinux.org/downloads/), selecting `alpine-extended-3.23.0-x86_64.iso`
2. Write the ISO image to a microSD card (recommended) or USB flash drive using BalenaEtcher, dd, or an equivalent tool.

*Why microSD is recommended:*

* Reliable boot from APU BIOS
* Can remain inserted after installation
* Easy file transfer and backups
* No USB port permanently occupied

#### 1.2 Hardware Preparation

* Install the mSATA module in the left-most slot next to the serial connector
* Connect the serial cable to the laptop
* Open a text console on the laptop (Ctrl+Alt+F1) — do not use the graphical terminal
* Start minicom as root:

```
sudo minicom -D /dev/ttyUSB0
```

#### 1.3 Booting the Alpine Installer

1. Insert prepared microSD or USB
2. Power on the APU
3. Press F10 at BIOS prompt to select boot device
4. At the `boot:` prompt, quickly type `/` to interrupt default boot

```
/boot/vmlinuz-lts modules=loop,squashfs,sd-mod,usb-storage nomodeset console=ttyS0,115200 initrd=/boot/initramfs-lts
```

#### 1.4 Installing Alpine Linux to mSATA

* Log in as root (no password initially)
* Start installer:

```
setup-alpine
```

* Follow prompts and install Alpine to mSATA
* Identify disks:

```
lsblk
```

#### 1.5 Enabling Persistent Serial Console

1. Edit `/boot/extlinux.conf`:

   * Add `SERIAL 0 115200` as the first line
   * In `APPEND`, replace `quiet` with:

```
console=ttyS0,115200
```

2. Make configuration persistent by editing `/etc/update-extlinux.conf`:

```
serial_port=0
serial_baud=115200
default_kernel_opts="console=ttyS0,115200"
```

* Apply changes:

```
update-extlinux
cat /boot/extlinux.conf
```

* Reboot; system now boots with serial console output

### 2. Base System Configuration

#### 2.1 Update Alpine Linux

```
apk update
apk upgrade
reboot
cat /etc/alpine-release
```

#### 2.2 Enable Community Repository

* Edit `/etc/apk/repositories` and uncomment the appropriate community repo, for example:

```
https://dl-cdn.alpinelinux.org/alpine/v3.23/community
```

* Update packages:

```
apk update
apk upgrade
```

### 3. Installing Docker

```
apk add docker docker-cli docker-compose
rc-update add docker boot
service docker start
docker version
```

### 4. Installing Home Assistant Container

```
docker run -d \
  --name homeassistant \
  --restart=unless-stopped \
  -e TZ=Europe/Paris \
  -v homeassistant_config:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
```

* Access via browser at: `http://<ip-address>:8123`

#### 4.1 Finding the IP Address

```
nmap -p 8123 --open 192.168.1.0/24
nmap -p 8123 --open -Pn 192.168.1.0/24  # if ICMP is blocked
nmap -p 8123 --open -Pn -oG - 192.168.1.0/24 | awk '/8123\/open/ {print $2}'  # optional: extract IP only
```

* Complete initial onboarding in browser

### 5. Using the microSD Card for Maintenance

#### 5.1 Manual Mount Example

```
lsblk
mkdir /mnt/sdcard
mount /dev/mmcblk0p1 /mnt/sdcard
```

#### 5.2 Persistent Mount via /etc/fstab

```
/dev/mmcblk0p1  /mnt/sdcard  vfat  noauto,nofail  0  0
# or using UUID (recommended)
blkid /dev/mmcblk0p1
UUID=XXXX-YYYY  /mnt/sdcard  vfat  noauto,nofail  0  0
mount /mnt/sdcard
```

### 6. Recovery and Maintenance Workflow

#### 6.1 Typical Use Cases

* Backup Home Assistant configurations
* Transfer scripts or firmware
* Collect logs from non-networked system
* Emergency access for misconfigured network

#### 6.2 Offline Recovery

1. Power down APU
2. Insert microSD into another Linux system
3. Copy required files
4. Reinsert card into APU
5. Boot and mount card to apply changes

#### 6.3 Emergency Boot / Rescue

* Keep Alpine installer on microSD
* Boot from microSD to repair mSATA filesystem or configuration

---

## B. Building and Installing UEFI BIOS on APU Using DTS

### 1. Overview

Use Dasharo Tools Suite (documentation at [https://docs.dasharo.com/dasharo-tools-suite/overview/](https://docs.dasharo.com/dasharo-tools-suite/overview/)) for building UEFI BIOS. Full functionality requires a paid subscription. Without it, manually build DTS and firmware.

Releases for APU are listed at [https://docs.dasharo.com/variants/pc_engines/releases_uefi/](https://docs.dasharo.com/variants/pc_engines/releases_uefi/) (latest: v0.9.1, Nov 2025). No prebuilt ROMs available.

The built APU3 UEFI firmware ROM is available in the repository ([`firmware/pcengines_apu3_v0.9.rom`](https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/blob/main/firmware/pcengines_apu3_v0.9.rom)) and as a GitHub Release asset ([v0.9.1 Release](https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/releases/tag/v0.9.1)). Large system images, including the built DTS image, ISO, and WIC files, are provided only via the GitHub Release. Always verify downloads using the included SHA256 checksums.


### 2. Building DTS

1. Install Docker:

   * Engine: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)
   * Linux post-install: [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)
2. Build DTS via Dasharo Tools Suite overview. Uses Yocto for UEFI BIOS; compatible with APU SeaBIOS (CSM).

### 3. Building UEFI BIOS

```
mkdir Dasharo-build
cd Dasharo-build
```

* Clone coreboot:

```
git clone https://github.com/Dasharo/coreboot.git
cd coreboot
git checkout 217612e6
git submodule update --init --checkout
```

* Clone edk2:

```
cd ..
git clone https://github.com/Dasharo/edk2.git
cd edk2
git checkout 42934b12
git submodule update --init --checkout
```

* Build BIOS ROM (from coreboot):

```
cd ../coreboot
./build.sh apu3
```

* Resulting ROM: `coreboot/pcengines_apu3_v0.9.rom`

### 4. Flashing BIOS Firmware

1. Write DTS image to microSD, ROM to USB
2. Connect serial cable, start minicom
3. Boot from microSD, press Delete
4. Select serial console: `console=ttyS0,115200`
5. Drop to bash shell, mount USB:

```
mkdir /run/mount/usb
mount /dev/sdb1 /run/mount/usb
ls /run/mount/usb  # verify apu3.rom
```

6. Flash ROM:

```
flashrom -w /run/mount/usb/apu3.rom -p internal:boardmismatch=force -c W25Q64BV/W25Q64CV/W25Q64FV
```

7. Power off, remove USB, reboot

### 5. Installing Home Assistant OS on mSATA

1. Download HAOS image & checksum to USB:

```
wget https://github.com/home-assistant/operating-system/releases/download/16.3/haos_generic-x86-64-16.3.img.xz
wget https://github.com/home-assistant/operating-system/releases/download/16.3/haos_generic-x86-64-16.3.img.xz.sha256
```

2. Mount USB, verify image
3. Identify devices and write image to mSATA (usually /dev/sda):

```
xzcat /run/mount/usb/haos_generic-x86-64-*.img.xz | dd of=/dev/sda status=progress
```

4. Power off, remove microSD/USB, attach network cable, power on
5. Find IP: `nmap -p 8123 --open 192.168.1.0/24`
6. Complete setup via browser at `<IP>:8123`

### 6. Installing FreedomBox on mSATA

1. Download image & signature:

```
wget https://ftp.freedombox.org/pub/freedombox/hardware/amd64/stable/freedombox-trixie_all-amd64.img.xz
wget https://ftp.freedombox.org/pub/freedombox/hardware/amd64/stable/freedombox-trixie_all-amd64.img.xz.sig
```

2. Install like Home Assistant OS (mount USB, decompress, write to mSATA, power cycle)
