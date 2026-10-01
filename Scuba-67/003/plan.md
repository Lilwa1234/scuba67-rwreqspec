# แผนทางเทคนิค: UC03 - Process Dynamic QR Payment

## 1. สรุปแนวทาง
1. ฟีเจอร์นี้สร้างกระบวนการชำระเงินด้วย Dynamic QR Code เพื่อให้แคชเชียร์และลูกค้าเห็นสถานะการชำระเงินแบบเรียลไทม์ และใช้ยอดสุทธิของบิลเป็นค่าเงินที่ล็อกใน QR โดยตรง (UC03 Main Flow 2-7; ASM-03, ASM-05)
2. ระบบจะเรียก Payment Gateway เพื่อสร้าง QR Code และเก็บ Ref ID, สถานะ, และข้อมูลตอบกลับจาก Gateway ไว้ใน log เพื่อใช้ตรวจสอบย้อนหลัง (UC03 Main Flow 2; ASM-07)
3. หากไม่มี Webhook ยืนยันภายใน 3 นาที ระบบจะปรับสถานะเป็น "Pending / รอยืนยันจากธนาคาร" และเปิดให้แคชเชียร์กด "ตรวจสอบยอดเงิน (Re-check)" เพื่อสอบถามสถานะอีกครั้ง (UC03 Main Flow 5; ASM-01)
4. หากลูกค้าเปลี่ยนใจเป็นเงินสดหรือจอแสดงผลลูกค้าเกิดขัดข้อง ระบบจะยกเลิก QR หรือสลับไปชำระด้วยเงินสด และป้องกันไม่ให้เกิดบิลซ้ำซ้อน (UC03 Alternative 4a; ASM-02, ASM-06)
5. เมื่อได้รับ Webhook ยืนยันสำเร็จ ระบบจะแสดงสถานะ "ชำระเงินสำเร็จ" และพิมพ์ใบเสร็จรับเงินเฉพาะกรณีที่ชำระสำเร็จเท่านั้น โดยไม่ปิดออร์เดอร์ในกรณียอดเงินผิดปกติ (UC03 Main Flow 6-7; ASM-04, ASM-03)

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| React (Vite) สำหรับหน้า POS และ Customer Facing Display | ทีมเลือกเอง ไม่ได้มาจาก spec | Spec ไม่มีข้อกำหนดเกี่ยวกับ framework สำหรับส่วนติดต่อผู้ใช้ |
| Python FastAPI สำหรับหลังบ้าน | ทีมเลือกเอง ไม่ได้มาจาก spec | Spec ไม่มีข้อกำหนดเกี่ยวกับ framework สำหรับบริการชำระเงินและ webhook |
| ฐานข้อมูลเชิงสัมพันธ์ | ทีมเลือกเอง ไม่ได้มาจาก spec | Spec ระบุว่าต้องมีตาราง `payment_logs` แต่ไม่ระบุ DB engine ที่ต้องใช้ |
| Payment Gateway Integration | ASM-01, ASM-03, ASM-07 | ใช้สำหรับสร้าง Dynamic QR Code, ตรวจสอบสถานะ, และรองรับ webhook ยืนยันชำระเงิน |
| Webhook Event Handler | UC03 Main Flow 5-6, ASM-01, ASM-04 | ใช้เพื่อรับสัญญาณยืนยันจาก Gateway และเปลี่ยนสถานะเงินตามผลจริง |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับข้อกำหนด |
|---|---|---|
| Order | orderId, totalAmount, netAmount, status, cashierId, createdAt | UC03 Main Flow 1-7; ASM-03, ASM-05 |
| PaymentSession | paymentSessionId, orderId, refId, qrPayload, amount, status, timeoutAt, createdAt | UC03 Main Flow 2-7; ASM-01, ASM-03, ASM-05, ASM-07 |
| PaymentLog | logId, orderId, refId, amount, status, gatewayResponse, createdAt | UC03 Main Flow 5-7; ASM-04, ASM-07 |
| DiscrepancyLog | discrepancyId, orderId, refId, expectedAmount, actualAmount, reason, createdAt, reviewedBy | UC03 Alternative 5b; ASM-03 |
| CashFallbackTransaction | fallbackId, orderId, originalPaymentMethod, newPaymentMethod, amount, status, createdAt | UC03 Alternative 4a; ASM-02 |

หมายเหตุ: ไม่มีฟิลด์ใดที่จัดเก็บเลขบัตรประชาชนหรือข้อมูลบัตรเครดิตตามข้อกำหนดของฟีเจอร์นี้ และไม่พบข้อจำกัดด้านข้อมูลที่ต้องห้ามเก็บไว้ใน spec

## 4. API / หน้าจอ

- `POST /api/v1/payments/create-qr` รับ orderId, netAmount, cashierId และคืน qrPayload, refId, status (UC03 Main Flow 2; ASM-03, ASM-07)
- `GET /api/v1/payments/{paymentSessionId}/status` ส่งกลับสถานะปัจจุบัน เช่น รอชำระเงิน / ชำระเงินสำเร็จ / รอยืนยันจากธนาคาร (UC03 Main Flow 3-7; ASM-01, ASM-05)
- `POST /api/v1/payments/webhook` รับ payload จาก Payment Gateway และอัปเดตสถานะของออร์เดอร์เมื่อได้รับยืนยันสำเร็จ (UC03 Main Flow 5-6; ASM-01, ASM-04)
- `POST /api/v1/payments/recheck` รับ orderId หรือ refId เพื่อสอบถามสถานะอีกครั้งไปยัง Gateway (UC03 Alternative 5a; ASM-01)
- `POST /api/v1/payments/cancel-qr` รับ orderId หรือ paymentSessionId เพื่อยกเลิก QR และสลับไปใช้เงินสด (UC03 Alternative 4a; ASM-02)
- หน้า POS: แสดงสถานะ QR, ปุ่ม "ตรวจสอบยอดเงิน (Re-check)" และปุ่ม "เปลี่ยนเป็นเงินสด" พร้อมแถบสีส้มเมื่ออยู่ใน Pending (UC03 Main Flow 5; UC03 Alternative 4a, 5a; ASM-01, ASM-02, ASM-05)
- หน้า Customer Facing Display: แสดง QR Code พร้อมยอดเงินและสถานะ "รอชำระเงิน (Loading/QR Active)" (UC03 Main Flow 3; ASM-05)

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| ไม่มี CON ID ใน spec | ไม่มีข้อจำกัดด้านเทคโนโลยีที่ต้องใช้ | ยังไม่ได้ใช้ เพราะ spec ไม่มี CON ที่ระบุชัดเจน |
| ไม่มี DOM ID ใน spec | ไม่มีข้อจำกัดด้านข้อมูลหรือโครงสร้างฐานข้อมูลที่ชัดเจน | ยังไม่ได้ใช้ เพราะ spec ไม่มี DOM ที่ระบุชัดเจน |
| ไม่มี IF ID ใน spec | ไม่มี interface เภสัชที่ระบุรูปแบบการเชื่อมต่อภายนอกอย่างชัดเจน | ยังไม่ได้ใช้ เพราะ spec ไม่มี IF ที่ระบุชัดเจน |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| ไม่มี AC ID ใน spec | test_no_AC_defined | ปัจจุบัน spec ไม่มี Acceptance Criteria ที่มี ID ระบุชัดเจน จึงไม่สามารถสร้าง test ที่ยึด AC ID ใดได้; ต้องรอทีมกำหนด AC ก่อนทำตัวทดสอบแบบตรวจรับอย่างเป็นทางการ |

## 7. ลำดับงาน
1. กำหนดโครงสร้าง PaymentSession และ PaymentLog ให้มี Ref ID, ข้อมูล Gateway response, และ status tracking ที่ชัดเจน (UC03 Main Flow 2, 5-7; ASM-07)
2. สร้างบริการสร้าง QR Code จากยอดสุทธิและบันทึกความสัมพันธ์กับออร์เดอร์และจำนวนเงิน (UC03 Main Flow 2; ASM-03)
3. สร้างหน้า Customer Facing Display และ POS Screen เพื่อแสดง QR, สถานะรอชำระเงิน, สถานะ Pending, และปุ่ม Re-check / เปลี่ยนเป็นเงินสด (UC03 Main Flow 3, 5; ASM-05, ASM-02)
4. สร้าง webhook receiver และ service เปลี่ยนสถานะเป็นสำเร็จเมื่อได้รับยืนยันจาก Gateway (UC03 Main Flow 5-6; ASM-01, ASM-04)
5. สร้าง timeout check 3 นาที เพื่อเปลี่ยนสถานะเป็น Pending / รอยืนยันจากธนาคาร และแจ้งแคชเชียร์ให้กด Re-check (UC03 Main Flow 5; ASM-01)
6. สร้าง flow ยกเลิก QR และสลับไปชำระเป็นเงินสด พร้อมป้องกันบิลซ้ำซ้อน (UC03 Alternative 4a; ASM-02)
7. สร้างระบบบันทึก Discrepancy / Pending Log เมื่อยอดเงินผิดปกติหรือไม่ตรงกับบิลสุทธิ และไม่ปิดออร์เดอร์ (UC03 Alternative 5b; ASM-03)
8. ดำเนินการทดสอบหลัก: happy path, timeout pending, manual re-check, cash fallback, discrepancy handling, and printing receipt only on success (UC03 Main Flow 2-7; UC03 Alternative 4a, 5a, 5b; ASM-01, ASM-02, ASM-03, ASM-04)

## 8. สิ่งที่ยังไม่ทำ
ไม่มี Open Questions ใน spec หลังจากขั้น clarify จึงไม่มีรายการ Open Questions ที่ต้องคัดลอกมาไว้ที่นี่

ส่วนที่ยังไม่สร้างจนกว่าจะได้คำตอบเพิ่มเติม:
- รูปแบบ payload และ status code ที่ Payment Gateway จริงจะส่งกลับอย่างเป็นทางการ
- วิธีจัดการ retry หรือ re-check ที่ชัดเจนเมื่อ Gateway ตอบกลับซ้ำหรือส่งข้อมูลล่าช้าเกิน 3 นาที
- กฎการแสดงผลและกลไก fallback ที่แน่นอนสำหรับเครื่องแสดงผลลูกค้าเมื่ออุปกรณ์ปิดหรือเชื่อมต่อไม่ได้

