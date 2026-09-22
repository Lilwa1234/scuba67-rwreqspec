# Use Case Specification: UC03 - Process Dynamic QR Payment

* **Use Case ID:** UC03[cite: 5]
* **Use Case Name:** ชำระเงินด้วย Dynamic QR Code (Process Dynamic QR Payment)[cite: 5]
* **Actor หลัก:** แคชเชียร์ (Cashier)[cite: 5]
* **Actor รอง:** ลูกค้า (Customer), ระบบชำระเงินภายนอก (Payment Gateway / Bank API)[cite: 5]
* **Pre-condition:** แคชเชียร์สรุปรายการออร์เดอร์และกดเลือกช่องทางชำระเงินผ่าน QR Code[cite: 5]
* **Post-condition:** ระบบบันทึกสถานะการชำระเงินสำเร็จ ลิ้นชักเก็บเงินไม่เปิด และพิมพ์ใบเสร็จรับเงิน[cite: 5]

## Main Flow (ลำดับขั้นตอนปกติ)
1. แคชเชียร์เลือกวิธีชำระเงินเป็น "QR Code สแกนจ่าย"[cite: 5]
2. ระบบส่งยอดรวมสุทธิไปยัง Payment Gateway เพื่อสร้าง Dynamic QR Code ที่ระบุยอดเงินตรงกับบิลนั้นโดยอัตโนมัติ[cite: 5]
3. หน้าจอแสดงผลสำหรับลูกค้า (Customer Facing Display) แสดงภาพ QR Code พร้อมยอดเงิน[cite: 5]
4. ลูกค้าสแกนและยืนยันการโอนเงินผ่าน Mobile Banking[cite: 5]
5. ระบบ Payment Gateway ส่งสัญญาณยืนยัน (Webhook) มายังเครื่อง POS แบบเรียลไทม์[cite: 5]
6. หน้าจอ POS แสดงสถานะ "ชำระเงินสำเร็จ" พิมพ์ใบเสร็จ และปิดออร์เดอร์โดยอัตโนมัติ[cite: 5]

## Alternative / Exception Flows (ขั้นตอนทางเลือก / กรณีข้อผิดพลาด)
* **4a. ลูกค้าเปลี่ยนใจชำระเป็นเงินสด:** แคชเชียร์สามารถกดยกเลิก QR Code เพื่อเปลี่ยนเป็นช่องทางเงินสด และให้ระบบคำนวณเงินทอนตามปกติ[cite: 5]
* **5a. สัญญาณตอบรับจากเครือข่ายล่าช้า:** แคชเชียร์สามารถกดปุ่ม "ตรวจสอบสถานะยอดเงิน (Re-check)" เพื่อสั่งให้ระบบสอบถามสถานะไปยัง Gateway ซ้ำโดยตรง[cite: 5]