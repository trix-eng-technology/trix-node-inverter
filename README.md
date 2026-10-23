# TRIX Deye Gateway firmware

Firmware สำหรับอุปกรณ์ TRIX Deye Gateway (ESP32-S3) อ่านค่าจากอินเวอร์เตอร์ไฮบริด Deye ผ่าน RS485 Modbus RTU แล้วส่งขึ้นระบบมอนิเตอร์ TRIX (https://trix.technology) ผ่าน MQTT แบบเข้ารหัส TLS

repo นี้มีเฉพาะไฟล์ firmware สำเร็จรูปและเอกสาร ไม่มี source code ดาวน์โหลดไฟล์ได้ที่หน้า [Releases](../../releases)

## ความสามารถ

- อ่านค่า PV, แบตเตอรี่, grid, load, GEN port และพลังงานสะสม ทุกประมาณ 2 วินาที
- ส่ง telemetry ขึ้น TRIX ผ่าน MQTT over TLS (พอร์ต 8883) ใช้งานได้จากเครือข่ายใดก็ได้ที่มีอินเทอร์เน็ต
- ตั้งค่าครั้งแรกแบบ zero-touch: ใส่ WiFi และ provisioning URL จาก TRIX เครื่องดึงค่า broker และรหัสเอง
- อัปเดต firmware อัตโนมัติผ่าน TRIX (OTA) ตรวจลายเซ็น Ed25519 และ sha256 ก่อนติดตั้ง
- หน้าเว็บในเครื่อง: dashboard, power flow, ประวัติพลังงาน, ตั้งค่าอินเวอร์เตอร์ (ต้องปลดล็อก admin), audit log
- MQTT สำรองสำหรับ Home Assistant (auto-discovery) เปิดได้จากหน้าเว็บ

## ฮาร์ดแวร์ที่รองรับ

| รายการ | ค่า |
|---|---|
| บอร์ด | TRIX gateway ESP32-S3-WROOM-1U, flash 8 MB ขึ้นไป |
| RS485 RX (RO) | GPIO 17 |
| RS485 TX (DI) | GPIO 18 |
| RS485 DE/RE | GPIO 16 |
| โมดูล RS485 | 3.3 V (MAX3485 / SP3485) |
| ปุ่มล้างค่า | BOOT (GPIO 0) กดค้าง 5 วินาทีขณะเครื่องทำงาน |

ขา RS485 ล็อกไว้ใน firmware ตามบอร์ด เปลี่ยนจากหน้าเว็บไม่ได้

## อินเวอร์เตอร์ที่รองรับ

| รุ่น | ทดสอบกับเครื่องจริง |
|---|---|
| Deye SG04LP1 (เช่น SUN-5K-SG04LP1-EU-SM2) | ใช่ |
| Deye SG05LP1-EU | ใช่ |
| Deye SG04LP3, SG05LP3 (3 เฟส) | ยังไม่ทดสอบ (ใช้ register map จากเอกสาร) |

ค่าเริ่มต้น Modbus: 9600 8N1, slave id 1 เปลี่ยนได้ในหน้า Modbus

## ไฟล์ใน Release

| ไฟล์ | ใช้ทำอะไร |
|---|---|
| `deye-gateway-<version>-factory.bin` | แฟลชบอร์ดใหม่ครั้งแรกผ่านสาย USB เขียนที่ address `0x0` ไฟล์เดียวจบ |
| `deye-gateway-<version>-ota.bin` | อัปขึ้น TRIX (`firmwares/new`, device type `deye-gateway`) ให้เครื่องที่ติดตั้งแล้วอัปเดตเอง |
| `deye-gateway-manifest.json` | สำหรับแฟลชผ่านเบราว์เซอร์ด้วย ESP Web Tools |
| `SHA256SUMS` | ตรวจว่าไฟล์ที่ดาวน์โหลดมาครบและไม่ถูกแก้ |

ตรวจไฟล์ก่อนใช้:

```
shasum -a 256 -c SHA256SUMS
```

## แฟลชบอร์ดใหม่ (ครั้งแรกเท่านั้น)

ต้องมี Python และ esptool (`pip install esptool`) ต่อบอร์ดเข้าคอมพิวเตอร์ด้วยสาย USB แล้วรัน (แทน `<PORT>` ด้วยพอร์ตของบอร์ด เช่น `/dev/cu.usbserial-10` หรือ `COM5`):

```
esptool.py --chip esp32s3 --port <PORT> erase_flash
esptool.py --chip esp32s3 --port <PORT> --baud 460800 write_flash 0x0 deye-gateway-<version>-factory.bin
```

หลังจากนี้อัปเดตผ่าน TRIX ได้ตลอด ไม่ต้องใช้สายอีก

## ตั้งค่าครั้งแรก

1. จ่ายไฟให้บอร์ด เครื่องจะเปิด WiFi ชื่อ `trix-setup-xxxxxx`
2. ใช้มือถือต่อ WiFi นั้น แล้วเปิด http://192.168.4.1 (ส่วนใหญ่หน้าจะเด้งขึ้นเอง)
3. กรอก
   - WiFi SSID และรหัสผ่าน ของสถานที่ติดตั้ง
   - Device id (มีค่าให้แล้ว เปลี่ยนได้ ใช้ a-z 0-9 และ `-`)
   - TRIX provisioning URL: คัดลอกจาก QR หรือลิงก์ในหน้า portal ของ site บน TRIX
     รูปแบบ `https://trix.technology/api/portal/<token>/config.json`
   - ส่วน manual broker เว้นว่าง
4. กด Save and reboot เครื่องจะต่อ WiFi ดึงค่าจาก TRIX แล้วเริ่มส่งข้อมูล

หน้าตั้งค่าเปิดค้าง 10 นาที ถ้าต่อ WiFi ไม่ได้นาน 5 นาทีหลังบูต เครื่องจะเปิดหน้าตั้งค่าเองอีกครั้ง

## หน้าเว็บในเครื่อง

เปิด `http://<IP ของเครื่อง>/` จากเครือข่ายเดียวกัน ครั้งแรกจะให้ตั้งรหัสผ่านเว็บ การเขียนค่าเข้าอินเวอร์เตอร์ การกู้ค่า และการอัป firmware ต้องตั้งและปลดล็อกรหัส admin แยกอีกชั้น

## ความปลอดภัย

- MQTT ใช้ TLS และตรวจใบรับรองของ broker ก่อนส่งรหัส
- OTA รับเฉพาะ firmware ที่ TRIX เซ็นด้วย Ed25519 type ตรงกับเครื่อง และเวอร์ชันใหม่กว่าเท่านั้น
- ในตัว firmware ไม่มีรหัส WiFi หรือรหัส MQTT ค่าเหล่านี้เก็บในหน่วยความจำของแต่ละเครื่องตอนตั้งค่า

## License

Proprietary ดู [LICENSE](LICENSE) ติดต่อ TRIX ที่ https://trix.technology
