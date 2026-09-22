# Implementation & Test Plan: UC10 - Generate Sales & Settlement Report

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `010-sales-report.md` (UC10)[cite: 5]
* **Priority:** Phase 2 (Should Have - บัญชีและการกระทบยอดเงิน)[cite: 1, 2, 4, 5]
* **Estimated Effort:** 4 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Frontend:**
  * พัฒนาหน้าจอ Dashboard กราฟและสรุปยอดขาย (Gross Sales, Net Sales, Payment Breakdown)[cite: 5]
  * ตัวกรองช่วงเวลา (Date Range Picker) และปุ่ม Export PDF / XLSX[cite: 5]
* **Backend:**
  * พัฒนา Reporting Aggregation API สรุปข้อมูลยอดขายและค่าธรรมเนียม[cite: 4, 5]
  * ติดตั้ง Library สำหรับสร้างเอกสาร PDF และ Excel File Generator[cite: 5]
* **Data Verification:**
  * สร้างฟังก์ชัน Settlement Report จับคู่ยอดเงินโอนสแกนกับรายการฝั่ง Payment Gateway[cite: 4, 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Calculation Accuracy):** ยอดขายเงินสด + ยอด QR Code - ค่าธรรมเนียม ต้องตรงกับยอดสุทธิ 100% ปราศจากข้อผิดพลาดทศนิยม[cite: 4, 5]
* [ ] **TC-02 (Export Functionality):** ไฟล์ Excel และ PDF ที่ส่งออกต้องมีรูปแบบตารางที่อ่านง่ายและตัวเลขถูกต้องตรงกับหน้าจอ[cite: 5]
* [ ] **TC-03 (Performance):** การประมวลผลดึงข้อมูลรายงานย้อนหลัง 30 วัน ต้องใช้เวลาแสดงผลไม่เกิน 3 วินาที[cite: 4]