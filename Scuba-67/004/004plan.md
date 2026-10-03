# Plan: UC04 - Monitor Real-time Payment & Log

## 1. สรุปแนวทาง
1. ระบบจะสร้าง Transaction Feed ที่อัปเดตแบบเรียลไทม์ผ่าน WebSocket หรือ SSE เพื่อให้รายการเงินเข้าใหม่ปรากฏภายใน 1–2 วินาที โดยไม่ต้องกดรีเฟรชเอง (ASM-01).
2. ผู้ใช้จะเห็นข้อมูลสำคัญเป็นคู่คือ Order ID และ Bank Ref ID เพื่อใช้ตรวจสอบความถูกต้องก่อนยืนยันปิดบิล (ASM-04, ASM-05).
3. หากไม่มียอดเข้าภายใน 3 นาที ระบบจะเปลี่ยนสถานะเป็น Pending และแสดงแถบสีส้มพร้อมให้กด Copy Ref ID เพื่อติดตามกับ Gateway หรือ Call Center ได้ทันที (ASM-02, ASM-03).
4. ระบบจะบันทึกรายการที่ผิดพลาดหรือค้างไว้ใน Discrepancy / Pending Log เพื่อให้ทีมประสานกับผู้ให้บริการได้อย่างรวดเร็ว (ASM-05, ASM-08).
5. มุมมองการเข้าถึงจะแยกตามบทบาท: แคชเชียร์เห็นบน POS ส่วนเจ้าของร้านและผู้จัดการสามารถดูบน POS และแอปพลิเคชันมือถือ (ASM-06).

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| Frontend Web App (React + Vite) | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้สำหรับหน้า Transaction Feed, Pending Banner, ปุ่ม Copy Ref ID และตารางรายงานเบื้องต้น |
| Backend API (Python FastAPI) | ทีมเลือกเอง ไม่ได้มาจาก spec | จัดการดึงข้อมูลกองธุรกรรม, event stream, สถานะ Pending, และรายงาน discrepancy |
| Realtime transport (WebSocket หรือ SSE) | ASM-01 | จัดส่งรายการใหม่ให้ client ภายใน 1–2 วินาที โดยปราศจากการรีเฟรชเอง |
| Database (PostgreSQL) | ทีมเลือกเอง ไม่ได้มาจาก spec | เก็บ payment_logs, transaction_feed, discrepancy_log และข้อมูลจำแนกตาม order_id |
| Webhook processor | ASM-07 | รับสัญญาณยืนยันจากระบบภายนอกเพื่ออัปเดตสถานะชำระเงินและปิดบิลอัตโนมัติ |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับอะไรใน spec |
|---|---|---|
| orders | order_id, order_number, total_amount, status, created_at, closed_at, payment_channel | ASM-04, ASM-05, ASM-07 |
| payment_logs | payment_id, order_id, bank_ref_id, amount, status, transaction_time, webhook_received_at, created_at | ASM-01, ASM-02, ASM-03, ASM-04, ASM-05, ASM-07 |
| transaction_feed | feed_id, order_id, bank_ref_id, amount, event_time, display_status, is_pending, source_channel | ASM-01, ASM-02, ASM-03, ASM-04, ASM-06 |
| discrepancy_log | discrepancy_id, order_id, bank_ref_id, expected_amount, actual_amount, discrepancy_reason, logged_at, reviewed_by | ASM-05, ASM-08 |

หมายเหตุ: ในโมเดลนี้จะไม่เก็บข้อมูลที่ไม่จำเป็น เช่น เลขบัตรประชาชนหรือข้อมูลส่วนบุคคลอื่นใด เพราะ spec ของ UC04 ไม่ได้กำหนดให้มีข้อมูลประเภทดังกล่าว และไม่ควรเพิ่มฟิลด์ที่ไม่จำเป็นให้มากเกินไป

## 4. API / หน้าจอ

- GET /api/transactions — ดึงรายการเงินเข้าแบบเรียงตามเวลาล่าสุดพร้อม Order ID, amount, timestamp, Bank Ref ID; รองรับ ASM-01, ASM-04, ASM-06
- WS /ws/transactions หรือ /api/transactions/stream — ส่ง event ใหม่แบบเรียลไทม์ไปยัง UI; รองรับ ASM-01
- GET /api/transactions/:orderId — ดึงประวัติธุรกรรมของออร์เดอร์เดียว พร้อม Ref ID และสถานะ; รองรับ ASM-04, ASM-05
- GET /api/discrepancies — อ่านรายงาน Discrepancy / Pending Log; รองรับ ASM-05, ASM-08
- POST /api/transactions/:id/verify — บันทึกว่าผู้ใช้ได้ตรวจสอบแล้ว; รองรับ ASM-07
- หน้า Transaction Feed — แสดงรายการสลิปดิจิทัล, Pending banner, Order ID + Bank Ref ID, และปุ่ม Copy Ref ID; รองรับ ASM-02, ASM-03, ASM-04, ASM-06
- หน้า Discrepancy / Pending Log — แสดงรายการยอดเงินไม่ตรงหรือค้างระหว่างตรวจสอบ; รองรับ ASM-05, ASM-08

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| ไม่มี CON / DOM / IF ใน spec ของ UC04 | ไม่พบ constraint ที่ระบุในไฟล์ spec ปัจจุบัน; แผนใช้เกณฑ์จาก ASM-01 ถึง ASM-08 เป็นหลัก | ยังไม่ได้ใช้ เพราะ spec ไม่มี ID constraint ที่กำหนด |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_01_realtime_feed_1_to_2_seconds | ตรวจว่าเมื่อมีรายการใหม่เข้ามาใน backend ระบบส่ง event ผ่าน WebSocket/SSE และ UI ปรากฎภายใน 1–2 วินาที |
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_02_pending_after_3_minutes | จำลองเวลาที่ไม่มียอดเงินโอนเข้าภายใน 3 นาที แล้วตรวจว่าระบบแสดง Pending + แถบสีส้ม |
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_03_copy_ref_id_when_gateway_failure | จำลอง gateway timeout แล้วตรวจว่าปุ่ม Copy Ref ID ปรากฏและคัดลอกค่า Bank Ref ID ได้ถูกต้อง |
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_04_order_and_bank_ref_visible | ตรวจว่า UI แสดง Order ID และ Bank Ref ID คู่กันบน Transaction Feed |
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_05_discrepancy_log_for_mismatch | จำลองยอดเงินไม่ตรงหรือผิดพลาด ตรวจว่า item ถูกย้ายไป Discrepancy Log | 
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_06_role_based_access | ตรวจว่าระดับสิทธิ์แคชเชียร์เห็นบน POS แต่เจ้าของ/ผู้จัดการเห็นทั้ง POS และ mobile |
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_07_webhook_closes_bill_automatically | จำลอง Webhook success แล้วตรวจว่าสถานะเปลี่ยนเป็นยืนยันแล้วและปิดบิลอัตโนมัติ |
| ไม่มี AC ID ใน spec ของ UC04 | test_UC04_ASM_08_pending_log_generated | ตรวจว่ารายการค้างหรือผิดพลาดถูกส่งไปยัง Discrepancy / Pending Log อย่างถูกต้อง |

หมายเหตุ: ไฟล์ spec ปัจจุบันมีสภาพเป็น Use Case Specification ที่ระบุหลักการและ Assumptions อย่างชัดเจน แต่ไม่มี AC ID ที่สามารถโยงเป็นตารางได้โดยตรง จึงใช้ ASM-ID เป็นตัวชี้วัดการทดสอบตามคำตอบที่ได้รับจากทีม

## 7. ลำดับงาน

1. กำหนด schema ของ payment_logs, transaction_feed และ discrepancy_log เพื่อรองรับ Order ID, Bank Ref ID, pending state, mismatch tracking; รองรับ ASM-04, ASM-05
2. สร้าง realtime event channel และ API ดึงข้อมูล feed; รองรับ ASM-01
3. พัฒนา UI Transaction Feed สำหรับเครื่อง POS และ mobile view ตามสิทธิ์; รองรับ ASM-04, ASM-06
4. เพิ่ม logic คำนวณ pending status เมื่อไม่มียอดเข้าภายใน 3 นาที และแสดงแถบสีส้ม; รองรับ ASM-02, ASM-03
5. เพิ่มปุ่ม Copy Ref ID และจัดการข้อมูลติดตามกับ Call Center / Payment Gateway; รองรับ ASM-03
6. ผนวก Webhook status handler เพื่อยืนยันและปิดบิลอัตโนมัติ; รองรับ ASM-07
7. เพิ่มรายงาน Discrepancy / Pending Log สำหรับรายการผิดพลาดหรือค้างระหว่างตรวจสอบ; รองรับ ASM-05, ASM-08
8. ทดสอบ end-to-end สำหรับ real-time, pending, mismatch, role-based access และ webhook outcomes; รองรับ ASM-01 ถึง ASM-08

## 8. สิ่งที่ยังไม่ทำ

- ไม่มี Open Questions ที่ค้างอยู่ใน spec ของ UC04 หลังจากขั้น clarify เสร็จสิ้น
- ส่วนที่เกี่ยวข้องกับข้อเสนอแนะเพิ่มเติมหรือการเชื่อมต่อ Gateway จริงจะยังไม่สร้างจนกว่าจะได้รับคำตอบเพิ่มเติมจากทีม
