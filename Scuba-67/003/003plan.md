# Implementation & Test Plan: UC03 - Process Dynamic QR Payment

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `003-qr-payment.md` (UC03)[cite: 5]
* **Priority:** Phase 1 (Must Have - Core Payment)
* **Estimated Effort:** 5 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **External Integration:**
  * เชื่อมต่อ API ผู้ให้บริการ Payment Gateway (PromptPay QR Generator API)[cite: 5]
* **Backend:**
  * พัฒนา Webhook Receiver `POST /api/v1/payments/webhook` สำหรับรับ Payment Confirmation[cite: 5]
  * สร้างระบบตรวจสอบ Transaction Timeout ภายใน 3 นาที[cite: 5]
  * พัฒนา API `POST /api/v1/payments/recheck` เพื่อตรวจสอบยอดเงินแบบ Manual[cite: 5]
* **Frontend:**
  * พัฒนา Customer Facing Display UI สำหรับแสดง QR Code และยอดเงินที่ต้องชำระ[cite: 5]
  * หน้าจอ Loading พร้อมปุ่ม "เปลี่ยนเป็นเงินสด" และปุ่ม "ตรวจสอบยอดเงิน"[cite: 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Happy Path):** ยอดเงินใน QR Code ต้องตรงกับยอดสุทธิของบิล 100% โดยลูกค้าไม่ต้องพิมพ์ตัวเลขเอง[cite: 4, 5]
* [ ] **TC-02 (Auto-Confirm):** เมื่อจำลอง Webhook ยืนยันเงินเข้า หน้าจอ POS ต้องเปลี่ยนสถานะเป็นสำเร็จและปิดบิลภายใน 3 วินาที[cite: 5]
* [ ] **TC-03 (Fallback):** สามารถกดยกเลิก QR เพื่อกลับไปคิดเงินสดและคำนวณเงินทอนได้โดยบิลไม่ซ้ำซ้อน[cite: 5]