# Implementation & Test Plan: UC09 - Operate Offline & Sync Data

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `009-offline-sync.md` (UC09)[cite: 5]
* **Priority:** Phase 2 (Should Have - ความต่อเนื่องของธุรกิจหน้าร้าน)[cite: 4, 5]
* **Estimated Effort:** 6 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Client Architecture:**
  * ใช้ Local Database (SQLite / IndexedDB) เป็นแหล่งเก็บข้อมูลหลักของเครื่อง POS[cite: 5]
  * สร้าง Network Status Monitor ตรวจจับสถานะการเชื่อมต่อเน็ตแบบ Real-time[cite: 5]
* **Sync Engine:**
  * พัฒนา Background Sync Worker ทำหน้าที่ Reconcile ข้อมูลออร์เดอร์ขึ้นคลาวด์เมื่อสถานะเปลี่ยนเป็น Online[cite: 5]
  * สร้างระบบ Conflict Resolution ป้องกันการเขียนทับเลขออร์เดอร์ (ใช้ UUID + Timestamp)
* **Frontend:**
  * แสดงแถบแจ้งสถานะบนแถบบาร์ด้านบน: สีเขียว (Online), สีส้ม (Offline Mode)[cite: 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Offline Sales):** ปิด Wi-Fi ของเครื่อง POS ระบบต้องยังคงเปิดบิล รับเงินสด และพิมพ์ใบเสร็จได้ตามปกติ[cite: 4, 5]
* [ ] **TC-02 (Auto Sync):** เมื่อเปิด Wi-Fi อีกครั้ง ข้อมูลบิลที่ค้างอยู่ต้องถูกอัปโหลดขึ้นคลาวด์ภายใน 30 วินาที[cite: 5]
* [ ] **TC-03 (Data Consistency):** ยอดขายบนแอปมือถือของเจ้าของร้านต้องอัปเดตยอดเพิ่มตรงกับบิลออฟไลน์ 100%[cite: 4, 5]