# Implementation & Test Plan: UC08 - Record Waste & Staff Meals

## 1. ข้อมูลการวางแผน (Planning Overview)
* **Target Spec:** `008-record-waste.md` (UC08)[cite: 5]
* **Priority:** Phase 3 (Could Have - แก้ปัญหาตัดสต๊อกคลาดเคลื่อนจากการกินเอง/ทำผิด)[cite: 1, 2, 4, 5]
* **Estimated Effort:** 3 วันทำการ

## 2. รายละเอียดงานพัฒนา (Technical Tasks)
* **Frontend:**
  * พัฒนาหน้าจอแยกสำหรับ "บันทึกของเสียและสวัสดิการ (Internal Waste Log)"[cite: 4, 5]
  * ดรอปดาวน์เลือกเหตุผล: ชงผิด (Mistake), หมดอายุ (Expired), สวัสดิการพนักงาน (Staff Meal)[cite: 5]
  * ช่องใส่รหัส PIN อนุมัติการตัดของเสียสำหรับระดับ Manager
* **Backend:**
  * พัฒนา API `POST /api/v1/inventory/waste`[cite: 5]
  * แยกประเภทต้นทุนเป็น Expense / Loss โดยไม่นำไปคำนวณใน Gross Sales[cite: 4, 5]

## 3. เกณฑ์การตรวจรับและการทดสอบ (Acceptance Criteria & Test Cases)
* [ ] **TC-01 (Stock Deduction):** บันทึกของเสียชานม 1 แก้ว สต๊อกวัตถุดิบต้องลดลงตามสูตร[cite: 5]
* [ ] **TC-02 (Financial Integrity):** ยอดขายหน้าร้าน (Sales Revenue) ต้องไม่เพิ่มขึ้นจากรายการของเสีย[cite: 4, 5]
* [ ] **TC-03 (Role-based Control):** พนักงานทั่วไปไม่สามารถกดยืนยันได้หากไม่กรอกรหัส PIN ของผู้จัดการ[cite: 4]