# Implementation & Test Plan: UC04 - Monitor Real-time Payment & Log

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `004-monitor-payment.md` (UC04)[cite: 5]
* **Priority:** Phase 1 (Must Have - แก้ Pain Point เงินค้างในอากาศ)[cite: 1, 2, 5]
* **Estimated Effort:** 3 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Frontend:**
  * ออกแบบหน้ารวมรายการ Transaction Feed แบบ Real-time คล้ายสลิปดิจิทัล (แบบแอปแม่มณีที่เจ้าของร้านต้องการ)[cite: 1, 2, 5]
  * เพิ่มปุ่ม "Copy Ref ID" สำหรับใช้แจ้งติดตามปัญหากับ Call Center[cite: 5]
* **Backend:**
  * พัฒนา WebSocket หรือ Server-Sent Events (SSE) ส่งแจ้งเตือนเงินเข้าจอ POS และมือถือทันที
  * จัดทำ Log Table บันทึก Payload ของ Webhook ธนาคารอย่างละเอียด[cite: 4, 5]
* **Database:**
  * ตาราง `payment_logs` (order_id, amount, bank_ref, status, gateway_response, created_at)[cite: 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Real-time Feed):** เมื่อมีเงินเข้า รายการใหม่ต้องปรากฏบน Transaction Feed ภายใน 1-2 วินาที[cite: 5]
* [ ] **TC-02 (Auditability):** ข้อมูลทุกรายการต้องมี Ref ID, เลขออร์เดอร์, และเวลาเงินเข้าอย่างครบถ้วน[cite: 4, 5]
* [ ] **TC-03 (Exception Alert):** หากยอดเงินไม่เข้าภายใน 3 นาที รายการต้องเปลี่ยนเป็นสถานะ "Pending" พร้อมเน้นสีส้ม[cite: 5]