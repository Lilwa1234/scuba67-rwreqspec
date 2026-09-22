# Implementation & Test Plan: UC07 - Configure Recipe & Auto Deduct

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `007-recipe-auto-deduct.md` (UC07)[cite: 5]
* **Priority:** Phase 3 (Could Have - ระบบต้องยืดหยุ่นไม่บังคับตึงเกินไป)[cite: 1, 2, 4, 5]
* **Estimated Effort:** 5 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Frontend:**
  * หน้าจอกำหนดสูตร (Bill of Materials: BOM) เลือกผูกวัตถุดิบย่อยและหน่วยนับ (g, ml, ชิ้น)[cite: 4, 5]
  * สวิตช์เปิด-ปิดระบบตัดสต๊อกอัตโนมัติรายเมนู (Configurable Toggle)[cite: 4, 5]
* **Backend:**
  * พัฒนา Inventory Deduction Engine ทำงานแบบ Asynchronous เมื่อบิลปิดสำเร็จ[cite: 5]
  * สร้างฟังก์ชันบันทึกสต๊อกแบบยอมรับค่าติดลบ (Negative Stock Allowance) พร้อม Trigger แจ้งเตือน[cite: 5]
* **Database:**
  * ตาราง `ingredients`, `recipes`, `stock_transactions`[cite: 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Auto Deduction):** สั่งชานม 1 แก้ว สต๊อกผงชาต้องลดลง 20g, นมข้นลดลง 30ml, และแก้วลดลง 1 ใบ[cite: 5]
* [ ] **TC-02 (Negative Stock Tolerance):** หากสต๊อกเหลือ 0 แต่มีการขาย ระบบต้องยอมให้ขายได้และบันทึกเป็นสต๊อกติดลบ[cite: 5]
* [ ] **TC-03 (Feature Toggle):** หากปิดฟังก์ชันตัดสต๊อกอัตโนมัติ การขายต้องไม่ไปตัดยอดวัตถุดิบในคลัง[cite: 4, 5]