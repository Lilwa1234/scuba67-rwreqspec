# Implementation & Test Plan: UC02 - Send Order to Kitchen

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `002-send-kitchen.md` (UC02)[cite: 5]
* **Priority:** Phase 1 (Must Have - ลดปัญหาการเดินส่งกระดาษแมนนวล)[cite: 1, 2, 5]
* **Estimated Effort:** 3 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Hardware & Network:**
  * เชื่อมต่อ ESC/POS Network Thermal Printer ผ่าน Local LAN (IP Address)[cite: 4, 5]
* **Backend:**
  * พัฒนา Print Dispatcher Service เพื่อจัด Format ใบแจ้งงานครัว (KOT) แยกตามสถานีเตรียม[cite: 5]
  * สร้างคิวงานการพิมพ์ (Print Queue) ป้องกันปัญหางานพิมพ์ชนกัน
* **Frontend:**
  * เพิ่ม Notification Modal บนหน้าจอ POS แจ้งเตือนสถานะการส่งงานครัวสำเร็จหรือล้มเหลว[cite: 5]
  * เพิ่มปุ่ม "พิมพ์ใบครัวซ้ำ (Reprint KOT)" ในหน้ารายละเอียดออร์เดอร์[cite: 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Happy Path):** เมื่อกดยืนยันออร์เดอร์ เครื่องพิมพ์ครัวต้องพิมพ์ใบสั่งงานออกภายใน 2 วินาที[cite: 5]
* [ ] **TC-02 (Data Accuracy):** ข้อความในใบ KOT ต้องแสดงชื่อเมนู, ตัวเลือกพิเศษ (Modifiers) และเลขออร์เดอร์ครบถ้วนชัดเจน[cite: 5]
* [ ] **TC-03 (Exception Flow):** ดึงสายแลนเครื่องพิมพ์ออก ระบบหน้าร้านต้องแจ้งเตือนข้อผิดพลาดทันที และสั่ง Reprint ได้เมื่อเสียบสายกลับ[cite: 5]