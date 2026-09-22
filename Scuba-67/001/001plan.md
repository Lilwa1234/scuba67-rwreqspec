# Implementation & Test Plan: UC01 - Manage Sales Order

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `001-manage-sales-order.md` (UC01)[cite: 5]
* **Priority:** Phase 1 (Must Have - Core MVP)
* **Estimated Effort:** 4 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Frontend:**
  * พัฒนาหน้าจอ POS Grid Layout แสดงหมวดหมู่และปุ่มเมนูที่ตอบสนองต่อการสัมผัสรวดเร็ว[cite: 4]
  * สร้างหน้าต่าง Pop-up แสดง Modifier Groups (เช่น อุณหภูมิ ร้อน/เย็น/ปั่น, ระดับความหวาน)[cite: 4, 5]
  * สร้างแถบสรุปตะกร้าสินค้า (Cart Summary) พร้อมปุ่มเพิ่ม/ลดจำนวน และปุ่มลบรายการ[cite: 5]
* **Backend:**
  * พัฒนา API `POST /api/v1/orders` สำหรับสร้างคำสั่งซื้อใหม่[cite: 5]
  * พัฒนา Order Calculation Engine คำนวณราคาสินค้า, ราคา Modifiers, ส่วนลด และภาษี[cite: 4, 5]
* **Database:**
  * ตาราง `orders`, `order_items`, `menu_items`, `modifiers`[cite: 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Happy Path):** เลือกเมนูและเลือก Modifier ร้อน/เย็น/ปั่น ยอดเงินรวมต้องคำนวณเพิ่มขึ้นทันทีแบบ Real-time[cite: 4, 5]
* [ ] **TC-02 (Performance):** ระบบต้องตอบสนองต่อการสัมผัสหน้าจอภายในเวลาไม่เกิน 1 วินาที[cite: 4]
* [ ] **TC-03 (Exception Flow):** สามารถลบรายการออกจากตะกร้าก่อนกดยืนยัน และยอดเงินรวมต้องลดลงอย่างถูกต้อง 100%[cite: 4, 5]