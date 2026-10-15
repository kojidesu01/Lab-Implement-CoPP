# Lab Implement CoPP

Capstone Project Part III: ทดลอง **Control Plane Policing (CoPP)** บน Cisco Modeling Labs (CML) ผ่าน Cisco DevNet Sandbox เพื่อศึกษาการควบคุมทราฟฟิกที่เข้าสู่ Control Plane ของเราเตอร์

## ผู้จัดทำ

| ชื่อ | รหัสนักศึกษา |
| --- | --- |
| นาย ฐิติพงศ์ กาญจนดำรงนันท์ | 633020290-2 |
| นาย ธีรเมธ สายคำ | 673380273-9 |
| นางสาว ปภาวรินทร์ นาเมืองรักษ์ | 673380275-5 |
| นางสาว ปรนันท์ บุสดีวงค์ | 673380276-3 |

[ดูสไลด์นำเสนอบน Canva](https://canva.link/0pdby32q597h2uc)

## วัตถุประสงค์

- ศึกษาความต่างระหว่าง Control Plane และ Data Plane และการทำงานของ CoPP กับ Modular QoS CLI (MQC)
- สร้าง topology R1–R2–A1 และกำหนดค่า EIGRP AS 123
- ใช้ ACL, Class Map และ Policy Map เพื่อควบคุมทราฟฟิกที่เข้าสู่ Control Plane ของ R1
- ตรวจพฤติกรรม Telnet จากไฟล์ packet capture และเปรียบเทียบทราฟฟิกที่มุ่งเข้า R1 กับทราฟฟิกที่ R1 ส่งต่อไป A1

## สภาพแวดล้อมและการเชื่อมต่อ

| อุปกรณ์ | Image ตามสไลด์ | บทบาท | IP ที่ใช้ในไฟล์แล็บ |
| --- | --- | --- | --- |
| R1 | CSR1000v / IOS XE 17.03.08a | เราเตอร์ที่ติดตั้ง CoPP | `10.10.1.1/24`, `172.16.12.1/30` |
| R2 | Cisco IOL | ต้นทางทราฟฟิกทดสอบ | `172.16.12.2/30` |
| A1 | Cisco IOL-L2 | สวิตช์ปลายทางทดสอบ | `10.10.1.4/24` |

```text
A1 (10.10.1.4) ── R1 (10.10.1.1 / 172.16.12.1) ── R2 (172.16.12.2)
```

## ไฟล์ใน Repository

```text
.
├── README.md
├── topology/
│   └── Lab_26.1.6-Implement_CoPP.yaml
├── configs/
│   ├── R1-config.txt
│   ├── R2-config.txt
│   └── A1-config.txt
└── captures/
    ├── Telnet_Blocked.pcap
    └── Telnet_pass.pcap
```

- [`topology/Lab_26.1.6-Implement_CoPP.yaml`](topology/Lab_26.1.6-Implement_CoPP.yaml): ไฟล์ topology สำหรับนำเข้า CML ซึ่งมี configuration ฝังอยู่
- [`configs/`](configs/): ไฟล์ configuration แยกตามอุปกรณ์สำหรับอ่านและตรวจคำสั่งสำคัญ
- [`captures/`](captures/): ไฟล์จับแพ็กเก็ตจริงของการทดสอบ Telnet สองกรณี

## การกำหนดค่า CoPP ที่สำคัญ

ใน [`R1-config.txt`](configs/R1-config.txt) มี ACL `TELNET` สำหรับ TCP พอร์ต 23, Class Map `CM-TELNET` และ Policy Map `PM-COPP` ที่กำหนด `conform-action drop` และ `exceed-action drop` ให้คลาส Telnet จากนั้นผูกนโยบายกับ `control-plane` ขาเข้า ส่วน EIGRP, SSH และ ICMP ถูกแยกเป็นคลาสของตนเอง

R1 และ R2 กำหนด EIGRP AS 123 ในเครือข่าย `172.16.12.0/30` ตามไฟล์ config สไลด์นำเสนอรายงานว่า EIGRP Neighbor R1–R2 ทำงานได้ แต่ Repository ยังไม่มีผลคำสั่ง `show ip eigrp neighbors` สำหรับตรวจยืนยันสถานะ ณ เวลาทดสอบ

## บัญชีทดสอบในแล็บ

| Username | Password |
| --- | --- |
| `admin` | `cisco123` |

บัญชีนี้มีไว้ใช้ในสภาพแวดล้อมแล็บเท่านั้น ไม่ควรนำรหัสผ่านเดียวกันไปใช้กับระบบอื่น

## ผลการทดลองจากไฟล์ PCAP

| กรณี | ต้นทาง → ปลายทาง | สิ่งที่พบใน capture | ข้อสรุปที่ยืนยันได้ |
| --- | --- | --- | --- |
| [`Telnet_Blocked.pcap`](captures/Telnet_Blocked.pcap) | R2 `172.16.12.2` → R1 `172.16.12.1:23` | พบ TCP SYN 6 แพ็กเก็ตจาก 2 การพยายามเชื่อมต่อ ไม่มี SYN-ACK ในไฟล์ | ไม่เห็นการตั้ง TCP session ไปยัง R1 ในช่วงที่จับแพ็กเก็ต; สอดคล้องกับนโยบาย `CM-TELNET` ที่ drop ทราฟฟิกเข้าสู่ Control Plane |
| [`Telnet_pass.pcap`](captures/Telnet_pass.pcap) | R2 `172.16.12.2` → A1 `10.10.1.4:23` | พบ SYN, SYN-ACK, ACK และข้อมูล Telnet แลกเปลี่ยนกัน | TCP session ไปยัง A1 ตั้งขึ้นและมีการสื่อสาร Telnet; เป็นทราฟฟิกที่ R1 ส่งต่อ ไม่ใช่การ Telnet เข้าสู่ Control Plane ของ R1 |

PCAP กรณีที่ผ่านแสดงการเชื่อมต่อและการแลกเปลี่ยนข้อมูล แต่ไม่ได้ใช้ยืนยันผลการล็อกอินหรือพิสูจน์การเปลี่ยนนโยบาย CoPP บนปลายทางเดียวกัน การระบุสาเหตุการ drop อย่างเด็ดขาดควรแนบตัวนับจาก `show policy-map control-plane` หรือคำสั่งที่ image รองรับด้วย

## วิธีตรวจซ้ำ

1. นำเข้าไฟล์ YAML ใน CML และเริ่ม R1, R2 และ A1
2. ตรวจ interface และ EIGRP ด้วย `show ip interface brief` และ `show ip eigrp neighbors`
3. ตรวจนโยบายบน R1 ด้วย `show running-config` และ `show policy-map control-plane` หรือคำสั่งเทียบเท่าที่ image รองรับ
4. จาก R2 ทดลอง Telnet ไป `172.16.12.1` และ `10.10.1.4` แล้วเปรียบเทียบกับ PCAP สองไฟล์
5. บันทึกผล CLI และตัวนับ CoPP หลังการทดสอบ เพื่อแยกผลของนโยบายออกจากปัญหาการเชื่อมต่ออื่น
