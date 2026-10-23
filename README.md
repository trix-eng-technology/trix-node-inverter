# TRIX Deye Gateway

Official firmware releases for the **TRIX Deye Gateway**, the inverter monitoring gateway made by Trix Engineering and Technology Co., Ltd.

The gateway connects to a Deye hybrid inverter over RS485 (Modbus RTU) and reports live data to the [TRIX monitoring platform](https://trix.technology) over encrypted MQTT. Firmware updates are delivered automatically through TRIX.

This repository publishes signed, ready-to-install firmware and its documentation. Source code is not published. Download files from [Releases](../../releases).

## Overview

| | |
|---|---|
| Product | TRIX Deye Gateway |
| Device type in TRIX | `deye-gateway` |
| Current release | see [Releases](../../releases/latest) |
| Hardware | TRIX gateway board, ESP32-S3-WROOM-1U, 8 MB flash or more |
| Connectivity | WiFi, MQTT over TLS (port 8883) |
| Updates | Over the air through TRIX, Ed25519 signed |

## Features

- Reads PV, battery, grid, load and GEN port values and energy counters about every 2 seconds
- Reports to TRIX from any network with internet access, encrypted with TLS
- Zero-touch commissioning: the installer enters WiFi and the site's provisioning URL, and the device fetches its broker and credentials from TRIX
- Automatic, signed firmware updates through TRIX
- Built-in local web page: dashboard, power flow, energy history, inverter settings behind an admin unlock, audit log
- Optional local MQTT output for Home Assistant (auto-discovery)

## Hardware

| Signal | Board connection |
|---|---|
| RS485 RX (RO) | GPIO 17 |
| RS485 TX (DI) | GPIO 18 |
| RS485 DE/RE | GPIO 16 |
| RS485 transceiver | 3.3 V (MAX3485 / SP3485) |
| Reset button | BOOT (GPIO 0), hold 5 seconds while running |

The RS485 pins are fixed in firmware to match the TRIX board.

## Inverter compatibility

| Model | Status |
|---|---|
| Deye SG04LP1 (e.g. SUN-5K-SG04LP1-EU-SM2) | Verified on site |
| Deye SG05LP1-EU | Verified on site |
| Deye SG04LP3, SG05LP3 (three-phase) | Supported from the published register map, not yet verified on site |

Modbus defaults: 9600 baud, 8N1, slave id 1 (adjustable on the Modbus page).

## Release files

| File | Purpose |
|---|---|
| `deye-gateway-<version>-factory.bin` | Factory image for the first flash of a new board over USB, written at `0x0` |
| `deye-gateway-<version>-ota.bin` | Update image, uploaded to TRIX so installed devices update themselves |
| `deye-gateway-manifest.json` | Manifest for flashing from a browser with ESP Web Tools |
| `SHA256SUMS` | Checksums of the images |

Verify the download before use:

```
shasum -a 256 -c SHA256SUMS
```

## Installation

### 1. Flash a new board (once)

Requires Python and esptool (`pip install esptool`). Connect the board over USB and run, replacing `<PORT>` with the board's serial port (for example `/dev/cu.usbserial-10` or `COM5`):

```
esptool.py --chip esp32s3 --port <PORT> erase_flash
esptool.py --chip esp32s3 --port <PORT> --baud 460800 write_flash 0x0 deye-gateway-<version>-factory.bin
```

All later updates are delivered over the air.

### 2. Commission on site

1. Power the gateway. It opens a WiFi network named `trix-setup-xxxxxx`.
2. Join it with a phone and open http://192.168.4.1 (the page usually opens by itself).
3. Enter:
   - the site's WiFi name and password
   - the device id (pre-filled; letters a-z, digits and `-`)
   - the TRIX provisioning URL, copied from the QR code or link on the site's portal page in TRIX:
     `https://trix.technology/api/portal/<token>/config.json`
   - leave the manual broker section empty
4. Press **Save and reboot**. The gateway joins WiFi, fetches its settings from TRIX and starts reporting.

The setup page stays open for 10 minutes. If the gateway cannot join WiFi within 5 minutes of booting, it reopens the setup page by itself.

### 3. Local web page

Open `http://<gateway IP>/` from the same network. The first visit asks for a web password. Writing inverter settings, restoring a backup and uploading firmware also require a separate admin password.

## Updates

Installed gateways check TRIX for updates after boot and every 6 hours, and on request from TRIX. An update is installed only if it is signed by TRIX, matches the device type and is newer than the running version. See [CHANGELOG](CHANGELOG.md) for release history.

## Security

- MQTT runs over TLS and the broker's certificate is verified before credentials are sent.
- Firmware updates are verified with an Ed25519 signature and a sha256 checksum before installation.
- No WiFi or MQTT passwords are built into the firmware; each device stores its own after commissioning.

To report a vulnerability, see [SECURITY](SECURITY.md).

## Support

For installation, warranty and technical support, contact Trix Engineering and Technology Co., Ltd. at https://trix.technology/contact

Writing settings to an inverter changes how an electrical installation operates. Inverter settings should be changed only by a qualified person.

## License and trademarks

The firmware is proprietary and licensed for use on TRIX gateway hardware only. See [LICENSE](LICENSE).

Deye is a trademark of its respective owner. TRIX is not affiliated with or endorsed by Deye.

Copyright (c) 2026 Trix Engineering and Technology Co., Ltd.
