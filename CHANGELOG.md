# Changelog

## 3.0.3 (2026-10-23)

รุ่น production แรกบนบอร์ด TRIX ESP32-S3

- ขา RS485 ล็อกตามบอร์ด: RX 17, TX 18, DE/RE 16
- MQTT over TLS ตรวจใบรับรอง broker (TrixNode 1.2.0)
- คำสั่ง `reprovision` ย้ายเครื่องไป broker ที่ TRIX แจกตอนนี้จากระยะไกล ต่อไม่ได้ใน 5 นาทีกลับค่าเดิมเอง
- OTA ผ่าน TRIX พร้อมลายเซ็น Ed25519
- หน้าเว็บใหม่: power flow รูปบ้าน, การ์ดพลังงานรายวัน
- แก้ตาราง partition (บูตวนบนบอร์ดใหม่) และ audit log ที่ไม่เคยทำงาน
