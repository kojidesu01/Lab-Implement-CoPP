# CoPP-Lab

โปรเจกต์ Capstone สำหรับจัดเก็บแล็บ **Control Plane Policing (CoPP)** เพื่อส่งขึ้น GitHub และใช้ตรวจสอบการทดสอบ Telnet ก่อนและหลังปรับนโยบาย CoPP

> **สถานะหลักฐาน:** ZIP นี้มีโครงสร้างโปรเจกต์และ README แต่ยังไม่พบไฟล์ export จาก CML, running-config หรือ packet capture ใน workspace จึงยังไม่สามารถยืนยันผลการทดลองจริงจากหลักฐานต้นฉบับได้ โปรดเพิ่มไฟล์จริงตามรายการด้านล่างก่อนใช้เป็นหลักฐานส่งงาน

## โครงสร้างไฟล์

```text
CoPP-Lab/
├── README.md
├── topology/
│   └── .gitkeep
├── configs/
│   └── .gitkeep
└── captures/
    └── .gitkeep
```

ไฟล์ที่ต้องเพิ่มจากแล็บจริง:

| ตำแหน่ง | เนื้อหาที่ต้องเป็นไฟล์จริง |
| --- | --- |
| `topology/CoPP-Capstone-Lab.yaml` | ไฟล์ topology ที่ export จาก CML |
| `configs/R1-config.txt` | running-config ของ R1 |
| `configs/R2-config.txt` | running-config ของ R2 |
| `configs/A1-config.txt` | running-config ของ A1 |
| `captures/telnet-blocked.pcap` | packet capture ระหว่างกรณี Telnet ถูกบล็อก |
| `captures/telnet-allowed.pcap` | packet capture ระหว่างกรณี Telnet ผ่านได้ |

ไม่มีโฟลเดอร์ `screenshots` ตามขอบเขตงานนี้ ไฟล์ `.gitkeep` มีไว้ให้ Git ติดตามโฟลเดอร์ที่ยังว่าง และลบได้เมื่อใส่ไฟล์จริงแล้ว

## ข้อมูลสำหรับทดสอบ

| รายการ | ค่า |
| --- | --- |
| Username | `admin` |
| Password | `cisco123` |

ใช้บัญชีนี้เฉพาะในสภาพแวดล้อมแล็บที่ได้รับอนุญาต เนื่องจาก README ใน GitHub จะเปิดเผยรหัสผ่านนี้ต่อผู้ที่เข้าถึง repository ได้

## การทดสอบและหลักฐาน

1. นำเข้าไฟล์ CML YAML และเริ่มอุปกรณ์ตาม topology จริง
2. ตรวจการเชื่อมต่อและสถานะ CoPP บน R1, R2 และ A1 ด้วยคำสั่งที่อุปกรณ์รองรับ เช่น `show running-config`, `show policy-map control-plane` และ `show policy-map interface control-plane`
3. ทดสอบ Telnet ด้วยบัญชี `admin` / `cisco123` ในสถานการณ์ที่นโยบายบล็อก และบันทึกไฟล์ `captures/telnet-blocked.pcap`
4. ทดสอบอีกครั้งในสถานการณ์ที่นโยบายอนุญาต และบันทึกไฟล์ `captures/telnet-allowed.pcap`
5. บันทึกผล CLI จริง (เวลา, ต้นทาง, ปลายทาง, คำสั่ง, ผลลัพธ์ และตัวนับนโยบาย) ลงในตารางด้านล่างหลังตรวจหลักฐาน

## ผลการทดลองจริง

| กรณี | ผล CLI จริง | หลักฐาน packet | สถานะยืนยัน |
| --- | --- | --- | --- |
| Telnet ถูกบล็อก | ยังไม่มีบันทึกผล CLI ใน workspace | `captures/telnet-blocked.pcap` ยังขาด | รอตรวจจากแล็บจริง |
| Telnet ผ่านได้ | ยังไม่มีบันทึกผล CLI ใน workspace | `captures/telnet-allowed.pcap` ยังขาด | รอตรวจจากแล็บจริง |

ยังไม่ควรสรุปว่าทดสอบผ่านหรือไม่ผ่านจนกว่าจะตรวจไฟล์ capture และผล CLI จากการทดลองจริง ผลที่ต้องการพิสูจน์คือ Telnet ถูกบล็อกเมื่อใช้นโยบายจำกัด และผ่านได้เมื่อใช้นโยบายอนุญาต

## ก่อนนำขึ้น GitHub

ตรวจว่าไฟล์ YAML, config และ PCAP เป็นไฟล์ export จากแล็บจริง ชื่อไฟล์ตรงกับรายการด้านบน และไม่มีข้อมูลลับอื่นนอกเหนือจากบัญชีทดสอบที่ตั้งใจเผยแพร่
# Lab-Implement-CoPP
