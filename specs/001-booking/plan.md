# แผนงานสำหรับฟีเจอร์ จองคิวตรวจสุขภาพ (Booking)

## 1. สรุปแนวทาง

ฟีเจอร์นี้ให้ผู้รับบริการที่ยืนยันตัวตนแล้วเลือกแพ็กเกจ วัน และช่วงเวลาตรวจสุขภาพ เพื่อให้ได้รับหมายเลขคิวอย่างปลอดภัยและรวดเร็ว โดยระบบจะตรวจสถานะการจองซ้ำก่อนยืนยันรายการ เพื่อป้องกันการจองซ้อนกันในวันเดียวกัน ระบบจะคงข้อมูลการจองและจำนวนที่นั่งคงเหลือไว้สำหรับการค้นหาช่วงว่างและยืนยันการจองตามเงื่อนไขใน FR-BKG-01 ถึง FR-BKG-06

ผู้ใช้หลักคือผู้รับบริการที่ผ่านการยืนยันตัวตนแล้ว และผู้ดูแลหรือเจ้าหน้าที่ที่ต้องตรวจสอบบัญชีการจองและ log การเข้าถึงข้อมูลผู้รับบริการในกรณีที่จำเป็น

แนวทางสร้างระบบประกอบด้วย 3 ส่วนหลักคือหน้าจอจองคิว, API การคำนวณช่วงเวลาและการยืนยันการจอง, และบริการรองรับการแจ้งเตือน/การส่งซ้ำ/ audit log เพื่อให้ตรงกับ Spec และความต้องการด้านความปลอดภัย

สถาปัตยกรรมจะพึ่ง MySQL เป็นแหล่งข้อมูลหลัก, FastAPI สำหรับบริการหลังบ้าน, และ React (Vite) สำหรับหน้าบ้าน เพื่อให้สามารถแสดงช่วงเวลาและจำนวนที่นั่งคงเหลือแบบ real-time ให้เพียงพอสำหรับการทำงานของผู้ใช้ 200 คนในภาวะพร้อมกัน

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| MySQL | CON-TECH-01 | ใช้เป็นฐานข้อมูลหลักสำหรับการจอง, slot, queue, notification, audit log |
| React (Vite) | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้สำหรับหน้าแสดงวันและช่วงเวลา, สถานะคิว, และหน้าจอยืนยันการจอง |
| Python FastAPI | ทีมเลือกเอง ไม่ได้มาจาก spec | ใช้สำหรับ API จัดการการค้นหาช่วงว่าง การป้องกันการจองซ้ำ และการยืนยันการจอง |
| ระบบแจ้งเตือน SMS/LINE แบบ asynchronous | IF-NOT-01 | ควรใช้ adapter หรือ outbox queue เพื่อให้การจองไม่รอผลการส่งข้อความ |
| HIS integration via HN | IF-HIS-01 | ดึงข้อมูลผู้รับบริการจาก HIS โดยใช้เลขบัตรประชาชนภายนอกแล้วเก็บเฉพาะ HN ในระบบ |
| Identity verification gateway | IF-IDP-01 | ทุกการเข้าถึงข้อมูลผู้รับบริการต้องได้รับการตรวจสอบ identity ก่อนเสมอ |
| Audit log store | DOM-PDPA-01 | บันทึกผู้เข้าถึง เวลา และรหัสผู้รับบริการในรูปแบบที่เก็บได้ไม่น้อยกว่า 1 ปี |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับ FR / Constraint |
|---|---|---|
| PatientProfile | patient_id, HN, verified_at, status | IF-IDP-01, IF-HIS-01, FR-BKG-02, FR-BKG-04 |
| Booking | booking_id, patient_id, HN, package_id, booking_date, slot_id, queue_no, status, created_at, confirmed_at, notification_status | FR-BKG-02, FR-BKG-04, FR-BKG-05 |
| BookingSlot | slot_id, booking_date, slot_start, slot_end, capacity, remaining_capacity, package_id, version | FR-BKG-01, FR-BKG-03, FR-BKG-04, FR-BKG-06 |
| Package | package_id, name, required_time, active_flag | FR-BKG-01, FR-BKG-06 |
| BookingQueue | queue_id, booking_id, queue_no, issue_date, status | FR-BKG-02, FR-BKG-04 |
| NotificationOutbox | notification_id, booking_id, channel, payload, message_status, retry_count, next_retry_at, created_at | FR-BKG-05, NFR-REL-02, IF-NOT-01 |
| AuditLog | audit_id, actor_id, accessed_at, patient_hn, action, resource_type, resource_id | DOM-PDPA-01 |

หมายเหตุ: ตาม IF-HIS-01 ระบบไม่เก็บเลขบัตรประชาชนในตารางการจองและไม่ควรมีฟิลด์ national_id หรือ id_card_number ใน entity Booking, BookingSlot, NotificationOutbox หรือ AuditLog ที่แสดงให้เห็นเป็นข้อมูลประจำตัวแบบสาธารณสมบัติ

## 4. API / หน้าจอ

### หน้าจอ
- UI-01: `/booking` — แสดงวันที่ภายใน 30 วันข้างหน้า, ช่วงเวลาว่าง, จำนวนที่นั่งคงเหลือ, และแพ็กเกจที่เลือก รองรับ FR-BKG-01, FR-BKG-06
- UI-02: `/booking/confirm` — หน้าแสดงสรุปการจองและยืนยันผลลัพธ์ รองรับ FR-BKG-03, FR-BKG-04, FR-BKG-05
- UI-03: `/booking/result` — แสดงหมายเลขคิวและสถานะการส่งข้อความยืนยัน รองรับ FR-BKG-04, FR-BKG-05

### API
- GET /api/booking/slots?fromDate=&toDate=&packageId= — คืนค่าช่วงเวลาว่างพร้อม remaining capacity สำหรับแต่ละช่วงเวลา รองรับ FR-BKG-01, FR-BKG-06
- POST /api/booking/availability/check — ตรวจสอบว่าผู้รับบริการมีคิวที่ยังไม่ได้ใช้ในวันที่เลือกหรือไม่ และคืนผลว่าปฏิเสธหรือยอมให้จอง รองรับ FR-BKG-02
- POST /api/booking/confirm — รับข้อมูลแพ็กเกจ วัน และช่วงเวลา ยืนยันการจอง, บันทึก booking, ลด remaining capacity, สร้าง queue_no, และส่งคำขอแจ้งเตือน รองรับ FR-BKG-03, FR-BKG-04
- POST /api/booking/notifications/retry — ส่งซ้ำข้อความแจ้งเตือนตาม queue ของการส่งที่ล้มเหลว รองรับ FR-BKG-05, NFR-REL-02
- GET /api/booking/{bookingId}/status — คืนสถานะการจองและสถานะการส่งข้อความยืนยัน รองรับ FR-BKG-05
- POST /api/audit/logs — บันทึก audit log เมื่อมีการเข้าถึงข้อมูลผู้รับบริการ รองรับ DOM-PDPA-01

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| CON-TECH-01 | MySQL ใช้เป็นฐานข้อมูลหลักของ Booking, Slot, Queue, NotificationOutbox, AuditLog | ใช้แล้ว |
| DOM-PDPA-01 | AuditLog entity และ API /api/audit/logs ถูกออกแบบให้บันทึกผู้เข้าถึง เวลา และ HN พร้อมเก็บไม่น้อยกว่า 1 ปี | ใช้แล้ว |
| IF-IDP-01 | UI และ backend validation ที่ต้องยืนยัน identity ก่อนเข้าถึงข้อมูลผู้รับบริการ รวมถึง guard layer ก่อนเปิดข้อมูล booking | ใช้แล้ว |
| IF-HIS-01 | PatientProfile entity และโครงสร้างข้อมูลจองใช้ HN เป็นข้อต่อกับ HIS และไม่เก็บเลขบัตรประชาชนในตารางการจอง | ใช้แล้ว |
| IF-NOT-01 | NotificationOutbox และ API /api/booking/confirm จะส่งคำขอแจ้งเตือนแบบ asynchronous และไม่รอผลตอบกลับ | ใช้แล้ว |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| AC-BKG-01 | test_AC_BKG_01_success_booking_reduces_capacity | สร้างข้อมูล slot ที่มีที่นั่งว่าง 1 ที่ ให้ผู้ใช้ยืนยันการจอง แล้วตรวจว่ารายการจองถูกบันทึก, queue_no ถูกสร้าง, และ remaining capacity กลายเป็น 0 |
| AC-BKG-02 | test_AC_BKG_02_reject_duplicate_active_booking | สร้าง booking เดิมที่ยังไม่ได้ใช้ในวันเดียวกัน จากนั้นลองจองอีกครั้งในวันเดียวกัน ตรวจว่าระบบปฏิเสธและแสดงหมายเลขคิวเดิม |
| AC-BKG-03 | test_AC_BKG_03_show_alternatives_when_slot_full | ทำให้ slot เหลือ 1 ที่แล้วมีผู้ใช้อีกคนยืนยันก่อน ระหว่างยืนยันของผู้ใช้ ให้ตรวจว่าควรเห็น “ช่วงเวลาเต็ม” และได้รับ 3 ตัวเลือกที่ใกล้เคียงโดยไม่มีการสร้างจองซ้อน |
| AC-BKG-04 | test_AC_BKG_04_booking_saved_when_notification_fails | จำลองการส่ง SMS/LINE ไม่ตอบสนอง แล้วตรวจว่าการจองยังถูกบันทึก, queue_no ยังแสดงบนหน้าจอ, และมีรายการที่ต้องส่งซ้ำภายใน 5 นาที |
| AC-BKG-05 | test_AC_BKG_05_booking_slot_query_p95_under_2s | ใช้ load test 200 concurrent users เพื่อเรียก API ค้นหาช่วงว่าง แล้วตรวจว่า p95 <= 2 วินาที |
| AC-BKG-06 | test_AC_BKG_06_audit_log_written_after_access | จำลองการเปิดดูข้อมูลการจองของผู้รับบริการ แล้วตรวจว่ามี audit log ที่มี actor, time, และ HN / รหัสผู้รับบริการ |

## 7. ลำดับงาน

1. สร้าง schema ของ BookingSlot, Booking, Payment/Package reference และ queue_no logic ตาม FR-BKG-01 และ AC-BKG-01
2. สร้าง API ค้นหาช่วงเวลาและจำนวนที่นั่งคงเหลือตามแพ็กเกจและวันที่ภายใน 30 วัน พร้อมตรวจ performance สำหรับ NFR-PERF-01
3. สร้างกฎป้องกันการจองซ้ำและแสดงหมายเลขคิวเดิมตาม FR-BKG-02 และ AC-BKG-02
4. สร้าง flow จองพร้อมตรวจสถานะ slot full และแสดง 3 ตัวเลือกใกล้เคียงตาม FR-BKG-03 และ AC-BKG-03
5. สร้าง flow ยืนยันการจอง บันทึก booking ลด remaining capacity และออกหมายเลขคิว ตาม FR-BKG-04 และ AC-BKG-01
6. สร้างระบบแจ้งเตือนแบบ asynchronous และ outbox retry เพื่อรองรับ FR-BKG-05, NFR-REL-02, AC-BKG-04
7. สร้าง identity gate, HIS lookup, และ audit log เพื่อรองรับ IF-IDP-01, IF-HIS-01, DOM-PDPA-01, AC-BKG-06
8. ทดสอบ end-to-end สำหรับทุก AC และตรวจความสอดคล้องกับ Open Questions ก่อนปิดงาน

## 8. สิ่งที่ยังไม่ทำ

- Q-01 “ช่วงเวลาใกล้เคียง” นับเฉพาะวันเดียวกัน หรือรวมวันถัดไปด้วย? -> ถามพยาบาลคัดกรอง
  ส่วนที่เกี่ยวข้องกับข้อนี้จะยังไม่สร้างจนกว่าจะได้คำตอบ

- Q-02 หมายเลขคิวรีเซ็ตรายวัน หรือนับต่อเนื่อง? -> ถามเจ้าหน้าที่เวชระเบียน
  ส่วนที่เกี่ยวข้องกับข้อนี้จะยังไม่สร้างจนกว่าจะได้คำตอบ

> หมายเหตุ: สถานะ spec ปัจจุบันเป็น Draft v1 ยังไม่ได้ผ่านขั้น clarify อย่างเป็นทางการ ดังนั้นแผนนี้จะต้องอิงกับข้อกำหนดปัจจุบันและคอยปรับเมื่อทีมชัดเจนเรื่อง open questions
