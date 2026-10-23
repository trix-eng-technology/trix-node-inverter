# TRIX Deye Gateway firmware

Firmware for the TRIX Deye Gateway (ESP32-S3). It reads a Deye hybrid inverter over RS485 Modbus RTU and sends the data to the TRIX monitoring platform (https://trix.technology) over MQTT with TLS.

This repository holds the ready-to-flash firmware files and documentation only. It contains no source code. Download the files from [Releases](../../releases).

## Features

- Reads PV, battery, grid, load, GEN port and energy counters about every 2 seconds
- Sends telemetry to TRIX over MQTT over TLS (port 8883), from any network with internet access
- Zero-touch setup: enter WiFi and the TRIX provisioning URL, and the device fetches its broker and credentials by itself
- Automatic firmware updates through TRIX (OTA), checked against an Ed25519 signature and sha256 before install
- Built-in web page: dashboard, power flow, energy history, inverter settings (behind an admin unlock), audit log
- Optional second MQTT connection for Home Assistant (auto-discovery), enabled from the web page

## Supported hardware

| Item | Value |
|---|---|
| Board | TRIX gateway ESP32-S3-WROOM-1U, 8 MB flash or more |
| RS485 RX (RO) | GPIO 17 |
| RS485 TX (DI) | GPIO 18 |
| RS485 DE/RE | GPIO 16 |
| RS485 module | 3.3 V (MAX3485 / SP3485) |
| Reset button | BOOT (GPIO 0), hold 5 seconds while running |

The RS485 pins are fixed in the firmware to match the board and cannot be changed from the web page.

## Supported inverters

| Model | Tested on a real unit |
|---|---|
| Deye SG04LP1 (e.g. SUN-5K-SG04LP1-EU-SM2) | Yes |
| Deye SG05LP1-EU | Yes |
| Deye SG04LP3, SG05LP3 (three-phase) | Not yet (register map from documentation) |

Modbus defaults: 9600 8N1, slave id 1. Both can be changed on the Modbus page.

## Release files

| File | Use |
|---|---|
| `deye-gateway-<version>-factory.bin` | First flash of a new board over USB, written at address `0x0`, single file |
| `deye-gateway-<version>-ota.bin` | Upload to TRIX (`firmwares/new`, device type `deye-gateway`) so installed devices update themselves |
| `deye-gateway-manifest.json` | For flashing from a browser with ESP Web Tools |
| `SHA256SUMS` | Checks that the downloaded files are complete and unmodified |

Check the files before use:

```
shasum -a 256 -c SHA256SUMS
```

## Flashing a new board (first time only)

You need Python and esptool (`pip install esptool`). Connect the board with a USB cable and run (replace `<PORT>` with the board's port, e.g. `/dev/cu.usbserial-10` or `COM5`):

```
esptool.py --chip esp32s3 --port <PORT> erase_flash
esptool.py --chip esp32s3 --port <PORT> --baud 460800 write_flash 0x0 deye-gateway-<version>-factory.bin
```

After this, all updates come through TRIX. No cable is needed again.

## First-time setup

1. Power the board. It opens a WiFi network named `trix-setup-xxxxxx`.
2. Join that network with a phone and open http://192.168.4.1 (the page usually opens by itself).
3. Fill in:
   - WiFi SSID and password of the installation site
   - Device id (pre-filled, can be changed; use a-z, 0-9 and `-`)
   - TRIX provisioning URL: copy it from the QR code or link on the site's portal page in TRIX,
     in the form `https://trix.technology/api/portal/<token>/config.json`
   - Leave the manual broker section empty
4. Press Save and reboot. The device joins WiFi, fetches its settings from TRIX and starts sending data.

The setup page stays open for 10 minutes. If the device cannot join WiFi for 5 minutes after booting, it opens the setup page again by itself.

## Built-in web page

Open `http://<device IP>/` from the same network. The first visit asks you to set a web password. Writing settings to the inverter, restoring a backup and uploading firmware need a separate admin password, set and unlocked on top of that.

## Security

- MQTT uses TLS and checks the broker's certificate before sending credentials.
- OTA accepts only firmware signed by TRIX with Ed25519, of the matching device type and a newer version.
- The firmware contains no WiFi or MQTT passwords. These are stored in each device's own memory during setup.

## License

Proprietary, see [LICENSE](LICENSE). Contact TRIX at https://trix.technology
