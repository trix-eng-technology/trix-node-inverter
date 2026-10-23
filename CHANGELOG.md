# Changelog

All notable changes to the TRIX Node Inverter (TN-INV) firmware are listed here.
Releases up to 3.0.3 were published under the name TRIX Deye Gateway.
Versions follow [Semantic Versioning](https://semver.org).

## [3.0.4] - 2026-10-24

### Changed
- Product renamed to TRIX Node Inverter (model TN-INV), part of the TRIX Node family. The web page and the default device name use the new name. The internal device type stays `deye-gateway`, so installed units keep receiving updates.

## [3.0.3] - 2026-10-23

First production release for the TRIX ESP32-S3 board.

### Added
- MQTT over TLS (port 8883) with broker certificate verification.
- Remote `reprovision` command: moves a node to the broker TRIX currently assigns, and returns to the previous broker by itself if the new one is unreachable for 5 minutes.
- Signed over-the-air updates through TRIX (Ed25519).
- New local web page with a power flow view and daily energy summary.

### Changed
- RS485 pins fixed to the TRIX board: RX 17, TX 18, DE/RE 16.

### Fixed
- Flash partition layout that caused new boards to restart in a loop.
- Audit log storage that was not being mounted.

[3.0.4]: https://github.com/trix-eng-technology/trix-node-inverter/releases/tag/v3.0.4
[3.0.3]: https://github.com/trix-eng-technology/trix-node-inverter/releases/tag/v3.0.3
