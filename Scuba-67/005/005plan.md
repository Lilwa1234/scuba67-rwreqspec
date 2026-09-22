# Implementation & Test Plan: UC05 - Manage Menu & Modifiers

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `005-manage-menu.md` (UC05)[cite: 5]
* **Priority:** Phase 2 (Should Have - ลดความซ้ำซ้อนของการตั้งราคาตัวเลือก)[cite: 1, 2, 5]
* **Estimated Effort:** 4 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Frontend:**
  * พัฒนาหน้าจอ Menu & Modifier Management ในแอป POS โดยตรง (ไม่ต้องสลับแอปไปจัดการ)[cite: 1, 2, 5]
  * ออกแบบ UI ให้กำหนดราคาบวกเพิ่มของแต่ละ Modifier ได้ในหน้าเดียว (Single Page Form)[cite: 5]
* **Backend:**
  * CRUD APIs สำหรับ `/api/v1/menus`, `/api/v1/categories`, `/api/v1/modifiers`[cite: 5]
  * ระบบ Soft Delete เมนู (เปลี่ยน `is_active = false`) เพื่อรักษาประวัติการขายย้อนหลัง[cite: 5]
* **Database:**
  * ตาราง `menu_items`, `categories`, `modifier_groups`, `modifier_options`[cite: 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Modifier Pricing):** กำหนดราคา "ปั่น +10 บาท" เมื่อสั่งหน้าร้านต้องคิดราคาเพิ่ม 10 บาทถูกต้อง 100%[cite: 4, 5]
* [ ] **TC-02 (Soft Delete):** เมนูที่ถูกซ่อนต้องไม่แสดงบนหน้าจอขาย แต่ยอดขายในรายงานย้อนหลังยังคงมีชื่อเมนูเดิม[cite: 5]
* [ ] **TC-03 (Instant Sync):** เมื่อบันทึกราคาใหม่ ข้อมูลบนหน้าจอขายต้องอัปเดตทันทีโดยไม่ต้อง Restart เครื่อง[cite: 5]