# Documentation Overview

This directory contains the installation and build documentation for PC Engines APU3B2, including DTS, Home Assistant OS, Freedombox, and UEFI BIOS firmware setup.

---

## Firmware and System Images

- **APU3 UEFI firmware ROM**  
  - Repository: [`firmware/pcengines_apu3_v0.9.rom`](https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/blob/main/firmware/pcengines_apu3_v0.9.rom)  
  - GitHub Release: https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/releases/tag/v0.9.1

- **Large system images** (DTS image, ISO, WIC)  
  - Available on the same release page:  
    https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/releases/tag/v0.9.1

- **SHA256 checksums**  
  - Also visible as assets on the release page.

---

## Verification

After downloading ROM or large images, verify integrity using the SHA256 checksums:

```bash
shasum -a 256 -c sha256sums.txt

