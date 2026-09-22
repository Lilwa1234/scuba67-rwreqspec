# Implementation & Test Plan: UC06 - Toggle Sold-Out Status

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `006-toggle-sold-out.md` (UC06)[cite: 5]
* **Priority:** Phase 1 (Must Have - แก้ Pain Point ต้องเปิดแอปอื่นไปปิดเมนู)[cite: 1, 2, 5]
* **Estimated Effort:** 2 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Frontend:**
  * เพิ่ม Event Listener (Long Press / Click Hold 1 วินาที) ที่ปุ่มการ์ดเมนู[cite: 5]
  * ออกแบบ Quick Action Toggle (สวิตช์เปิด-ปิดสถานะสินค้าพร้อมขาย / สินค้าหมด)[cite: 4, 5]
  * กำหนด CSS/Style สำหรับสถานะ Sold Out (สีเทาโปร่งแสง พร้อม Badge "หมด")[cite: 5]
* **Backend:**
  * พัฒนา API `PATCH /api/v1/menus/{id}/toggle-availability`[cite: 5]
  * ส่ง WebSocket Broadcast ไปอัปเดตหน้าจอ POS อื่นๆ และแอปเจ้าของร้าน[cite: 4, 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Quick Toggle):** แคชเชียร์กดค้างและปิดสถานะเมนูได้เสร็จสิ้นภายใน 3 วินาทีบนหน้าจอขาย[cite: 4, 5]
* [ ] **TC-02 (Order Prevention):** เมนูที่ติดป้ายสินค้าหมด ต้องไม่สามารถคลิกเพิ่มลงในตะกร้าได้[cite: 5]
* [ ] **TC-03 (Toggle Revert):** เมื่อกดสลับกลับเป็น "พร้อมขาย" ปุ่มต้องกลับมาเป็นสีปกติและกดสั่งได้ทันที[cite: 5]