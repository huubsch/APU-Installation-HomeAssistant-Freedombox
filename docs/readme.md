# Documentation Overview

This directory contains the installation and build documentation for PC Engines APU3B2, including DTS, Home Assistant OS, Freedombox, and UEFI BIOS firmware setup.

---

## Firmware and System Images

| File | Location | Notes |
|------|----------|-------|
| **APU3 UEFI firmware ROM** | [Repo](../firmware/pcengines_apu3_v0.9.rom) / [Release v0.9.1](https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/releases/tag/v0.9.1) |  |
| **Large system images** (DTS, ISO, WIC) | [Release v0.9.1](https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/releases/tag/v0.9.1) | Large files only available via Release |
| **SHA256 checksums** | [Release asset](https://github.com/huubsch/APU-Installation-HomeAssistant-Freedombox/releases/download/v0.9.1/sha256sums.txt) | Verify downloaded files |

---

## Verification

After downloading ROM or large images, verify integrity using the SHA256 checksums:

```bash
shasum -a 256 -c sha256sums.txt
