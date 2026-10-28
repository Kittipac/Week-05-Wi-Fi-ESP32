# เฟสย่อยที่ 2: Auth Phase (Authentication Phase)

หลังจากที่ ESP32 สแกนพบ Access Point (AP) เป้าหมายเรียบร้อยแล้ว ESP32 จะเข้าสู่ **Auth Phase** ซึ่งเป็นกระบวนการยืนยันตัวตนระดับ Link Layer (802.11 Authentication) ก่อนที่จะเริ่มขอสิทธิ์การผูกสัมพันธ์ในระดับเครือข่าย

> [!NOTE]
> ในเครือข่าย Wi-Fi ที่ใช้ความปลอดภัยแบบ WPA2/WPA3 ขั้นตอนนี้จะเป็นแบบ **Open System Authentication** (ยินยอมให้ยืนยันตัวตนผ่านในระดับ 802.11 Link Layer ไปก่อน) แล้วค่อยไปตรวจสอบรหัสผ่านจริงใน **Four-way Handshake Phase (เฟส 4)**

---

## 1. ลำดับการแลกเปลี่ยนแพ็กเกจ Authentication (Sequence Diagram)

<p align="center"><img src="Images/Fig_03-2-Auth-Phase_2026-10-25T09-40-41_academic.svg" alt="03-2-Auth-Phase" ></p>
<p align="center"><b>รูปที่ 3 </b> ลำดับการแลกเปลี่ยนแพ็กเกจ Authentication (Sequence Diagram)</p>

---

## 2. รายละเอียดขั้นตอนทางเทคนิค (Step Breakdown)

* **s2.1**: ESP32 เริ่มต้นขั้นตอนการร้องขอการยืนยันตัวตน (Authentication Request) ไปยัง AP พร้อมกันนั้น **Auth Timer** ของระบบปฏิบัติการจะเริ่มนับเวลา เพื่อควบคุมเวลาที่ใช้ในการรอคอยแพ็กเกจตอบรับ
* **s2.2 (กรณีหมดเวลา)**: หากเวลาผ่านไปจนเกินค่า Timeout ที่กำหนดไว้แล้ว แต่ยังไม่ได้รับการตอบรับจาก AP:
  * ไดรเวอร์จะปล่อย Event ชื่อ **`WIFI_EVENT_STA_DISCONNECTED`**
  * ไดรเวอร์จะแจ้ง Result Code / Disconnect Reason ออกมาเป็น: **`WIFI_REASON_AUTH_EXPIRE`** (Value 2)
* **s2.3 (กรณีสำเร็จ)**: หากได้รับแพ็กเกจตอบรับจาก AP เป็นสถานะ Success ภายในเวลาที่กำหนด **Auth Timer จะหยุดลงทันที** และสิ้นสุดกระบวนการ Authentication ในระดับ Link Layer เพื่อเข้าสู่เฟสถัดไป
* **s2.4 (กรณีถูกปฏิเสธ)**: หาก AP ส่งแพ็กเกจ Authentication Response ตอบกลับมา แต่ระบุสถานะ**ปฏิเสธการเข้าถึง (Rejection)**:
  * ไดรเวอร์จะปล่อย Event ชื่อ **`WIFI_EVENT_STA_DISCONNECTED`**
  * ไดรเวอร์จะแจ้ง Result Code / Disconnect Reason ออกมาเป็น: **`WIFI_REASON_AUTH_FAIL`** (Value 1)

---

## 3. ตารางสรุป Result Code และสาเหตุใน Auth Phase

| Reason Code Constant | รหัสตัวเลข | สื่อความหมายถึง | สาเหตุที่อาจเกิดขึ้นในทางปฏิบัติ |
| :--- | :--- | :--- | :--- |
| `WIFI_REASON_AUTH_FAIL` | 1 | AP ปฏิเสธคำขอ Authentication | MAC Address Filtering บน AP บล็อก ESP32 ไว้ หรือ AP อยู่ในสถานะ Overload |
| `WIFI_REASON_AUTH_EXPIRE` | 2 | หมดเวลาการรอตอบรับ Auth | สัญญาณ Wi-Fi อ่อนมาก แพ็กเกจหล่นหายในอากาศ (Packet Loss) |

---

## 4. คำถามทบทวนความเข้าใจ (Checkpoints)

1. ทำไมในเครือข่าย WPA2-PSK ขั้นตอน Auth Phase นี้ถึงผ่านได้แม้เราจะพิมพ์รหัสผ่านผิด?
2. หาก Router มีการเปิดใช้งาน **MAC Address Filtering** (อนุญาตเฉพาะอุปกรณ์ที่ลงทะเบียน MAC ไว้) ESP32 จะล้มเหลวที่ขั้นตอนใด และได้ Reason Code อะไร?

**ตอบ คำถามทบทวนความเข้าใจ**
1. ทำไมในเครือข่าย WPA2-PSK ขั้นตอน Auth Phase นี้ถึงผ่านได้แม้เราจะพิมพ์รหัสผ่านผิด?

เพราะขั้นตอน Authentication เบื้องต้นเป็นเพียงการตรวจสอบว่าอุปกรณ์สามารถเข้าร่วมเครือข่ายได้หรือไม่ แต่ยังไม่ได้ตรวจสอบรหัสผ่าน WPA2-PSK อย่างสมบูรณ์ การตรวจสอบรหัสผ่านจริงเกิดขึ้นในขั้นตอน 4-Way Handshake หากรหัสผ่านผิด การจับคู่กุญแจเข้ารหัสจะล้มเหลวและไม่สามารถเชื่อมต่อเครือข่ายเพื่อรับส่งข้อมูลได้ตามปกติ

2. หาก Router เปิดใช้งาน MAC Address Filtering และ ESP32 ไม่ได้ลงทะเบียน MAC ไว้ จะล้มเหลวที่ขั้นตอนใด และได้ Reason Code อะไร?

ESP32 จะถูกปฏิเสธการเข้าร่วมเครือข่ายในขั้นตอน Association เนื่องจาก MAC Address ไม่ได้รับอนุญาต โดยอาจพบ Reason Code 17 (WLAN_REASON_ASSOC_TOOMANY) ไม่ใช่รหัสที่ถูกต้องสำหรับกรณีนี้เสมอไป ทั้งนี้ Reason Code ที่พบจริงขึ้นอยู่กับวิธีที่ Router ปฏิเสธอุปกรณ์ และอาจเป็นรหัสที่เกี่ยวข้องกับการปฏิเสธ Association เช่น 1 (UNSPECIFIED) หรือ 17 (AP unable to handle additional stations)
