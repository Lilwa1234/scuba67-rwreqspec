# แผนทางเทคนิค: UC08 - บันทึกของเสียและอาหารพนักงาน

## 1. สรุปแนวทาง
1. ฟีเจอร์นี้รองรับการบันทึกตัดสต๊อกภายในและของเสียจากแคชเชียร์หรือพนักงานทั่วไปผ่านหน้าจอเดียวกัน โดยผู้ใช้จะเลือกเหตุผลจากดรอปดาวน์และให้แยกประเภทตามสาเหตุ เช่น ชงผิด, หมดอายุ, สวัสดิการพนักงาน (UC08 Main Flow 1-3, ASM-01, ASM-03)
2. ระบบจะให้ผู้ใช้กรอกรายการเมนูหรือวัตถุดิบที่เสียหายหรือใช้ภายใน และบังคับให้มีการยืนยันโดยรหัส PIN ของผู้จัดการก่อนกดยืนยันตัดสต๊อก (UC08 Main Flow 3-4, Alternative Flow 1a, ASM-02, ASM-03, ASM-05)
3. หลังกดยืนยัน ระบบจะหักจำนวนสต๊อกและบันทึกต้นทุนลง Loss & Waste Report โดยจัดเก็บเป็นค่าใช้จ่ายภายใน ไม่กระทบยอดขายจริง (UC08 Main Flow 5, Post-condition, ASM-04)
4. ความปลอดภัยจะใช้ Role-based Control ที่มีการตรวจสอบ PIN ของผู้จัดการก่อนอนุญาตให้กดยืนยันและลดสต๊อก เพื่อป้องกันการตัดยอดโดยไม่ได้รับอนุมัติ (Alternative Flow 1a, ASM-02, ASM-05)
5. กระบวนการพัฒนาเน้นหน้าจอ POS แบบบันทึกตัดสต๊อกภายใน / ของเสีย, API สำหรับบันทึกเหตุผลและยืนยัน PIN, และรายงานต้นทุนสูญเสียที่คำนวณจากสต๊อกที่ถูกหักจริง (UC08 Main Flow 1-5)

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| React (Vite) สำหรับหน้าจอ POS และฟอร์ม Internal Waste Log | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นตาม prompt; spec ไม่ระบุ framework สำหรับหน้าจอ POS หรือฟอร์มตัดสต๊อกภายใน |
| Python FastAPI สำหรับหลังบ้านและ API บันทึกตัดสต๊อก | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นตาม prompt; spec ไม่ระบุแบบ backend หรือการเชื่อมต่อฐานข้อมูล |
| PostgreSQL หรือฐานข้อมูล SQL ทั่วไป | ทีมเลือกเอง ไม่ได้มาจาก spec | จำเป็นต้องเก็บข้อมูลรายการเมนู/วัตถุดิบ เหตุผลการตัดยอด สถานะอนุมัติ และบันทึกต้นทุนสูญเสีย |
| Role-based PIN validation | ASM-02, ASM-05 | ใช้ตรวจสอบว่าผู้กดยืนยันเป็นผู้จัดการก่อนอนุญาตให้ตัดยอดสต๊อก |
| Loss & Waste Report | UC08 Main Flow 5 | เป็น report หลักสำหรับต้นทุนสูญเสียให้ทางบัญชี/ผู้จัดการตรวจสอบ |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับข้อกำหนด |
|---|---|---|
| InternalWasteLog | log_id, item_id, item_name, category, reason_code, reason_label, quantity, unit, created_by, approved_by, manager_pin_verified, created_at, approved_at, status | UC08 Main Flow 1-5; ASM-01, ASM-02, ASM-03, ASM-05 |
| InventoryItem | item_id, name, current_stock, unit, updated_at | UC08 Main Flow 2, 5; Post-condition |
| WasteReason | reason_id, code, label, type, description | UC08 Main Flow 1-3; ASM-01 |
| ExpenseLossEntry | expense_id, log_id, item_id, quantity, unit_cost, total_cost, cost_type, created_at | UC08 Main Flow 5; ASM-04 |
| ApprovalAudit | approval_id, log_id, actor_id, manager_id, pin_verified, result, approved_at | UC08 Alternative Flow 1a; ASM-02, ASM-05 |

ไม่มีฟิลด์ที่เก็บข้อมูลบัตรประชาชนหรือข้อมูลที่ไม่ได้ระบุใน spec หากมีข้อจำกัดเกี่ยวกับการบันทึกความเป็นส่วนตัวของพนักงาน จะไม่ถูกเพิ่มลงใน entity นี้ เพราะ spec ไม่ได้ระบุไว้

## 4. API / หน้าจอ

- `GET /api/pos/internal-waste/reasons` ส่งกลับรายการเหตุผลตัดสต๊อก เช่น ชงผิด, หมดอายุ, สวัสดิการพนักงาน เพื่อให้ผู้ใช้เลือกในดรอปดาวน์ (UC08 Main Flow 1, ASM-01)
- `GET /api/pos/internal-waste/items` ส่งกลับรายการเมนูหรือวัตถุดิบพร้อมปริมาณคงเหลือเพื่อให้เลือกตัดยอด (UC08 Main Flow 2, Post-condition)
- `POST /api/pos/internal-waste/logs` รับ `itemId`, `reasonCode`, `quantity`, `unit`, `createdBy`, `managerPin` และคืน `logId` พร้อมสถานะ “pending” หรือ “approved” (UC08 Main Flow 3-4, Alternative Flow 1a, ASM-02, ASM-03, ASM-05)
- `POST /api/pos/internal-waste/logs/{logId}/approve` ตรวจสอบ PIN ของผู้จัดการและอนุญาตให้ยืนยันการตัดสต๊อกได้ก็ต่อเมื่อผ่าน Role-based Control (UC08 Alternative Flow 1a, ASM-02, ASM-05)
- `GET /api/reports/loss-waste` ส่งกลับข้อมูลต้นทุนสูญเสียและสต๊อกที่ถูกหักตามเหตุผลที่เลือกไว้ เพื่อสนับสนุน Loss & Waste Report (UC08 Main Flow 5, ASM-04)
- หน้าจอ “บันทึกตัดสต๊อกภายใน / ของเสีย (Internal Waste Log)” จะแสดงรายการที่เลือก, เหตุผล, จำนวน, และช่องกรอกรหัส PIN ของผู้จัดการก่อนกดยืนยัน (UC08 Main Flow 1-4, Alternative Flow 1a)

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| ไม่มี CON ID ใน spec | ไม่มีข้อจำกัดเชิงเทคนิคหรือเงื่อนไขเฉพาะที่ต้องใช้ใน plan | ยังไม่ได้ใช้ เพราะ spec ไม่มี CON |
| ไม่มี DOM ID ใน spec | ไม่มีโครงสร้างข้อมูลหรือความสัมพันธ์ข้อมูลเฉพาะที่ต้องลงใน model | ยังไม่ได้ใช้ เพราะ spec ไม่มี DOM |
| ไม่มี IF ID ใน spec | ไม่มี interface หรือการเชื่อมต่อระบบภายนอกที่ชัดเจนต้องใช้ | ยังไม่ได้ใช้ เพราะ spec ไม่มี IF |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| ไม่มี AC ID ใน spec | `test_AC_UC08_missing_01` | spec ของ UC08 ยังไม่มี Acceptance Criteria ที่ระบุ ID ที่ชัดเจน จึงยังไม่สามารถ map ไปยัง test case ที่มี AC ID จริงได้; การทดสอบจะใช้ UC08 Main Flow, Alternative Flow 1a, และ ASM-01 ถึง ASM-05 เป็นพื้นฐานก่อนเพิ่ม AC อย่างเป็นทางการ |

## 7. ลำดับงาน
1. สร้างโครงสร้างข้อมูลสำหรับ InternalWasteLog, WasteReason, InventoryItem และ ExpenseLossEntry ให้รองรับการบันทึกเหตุผลและต้นทุนสูญเสีย (UC08 Main Flow 1-5; ASM-01, ASM-03, ASM-04)
2. สร้าง API สำหรับดึงรายการเมนู/วัตถุดิบและเหตุผลการตัดยอด เพื่อให้แคชเชียร์และผู้จัดการเห็นข้อมูลที่ต้องเลือกและกดยืนยัน (UC08 Main Flow 1-2; ASM-01)
3. สร้างหน้าจอ Internal Waste Log ใน POS พร้อมฟิลด์เลือกเหตุผล, เลือกรายการ, ระบุจำนวน, และช่องกรอกรหัส PIN ของผู้จัดการ (UC08 Main Flow 1-4; ASM-02, ASM-03)
4. สร้างการตรวจสอบ PIN ของผู้จัดการและ Role-based Control โดยปฏิเสธการกดยืนยันเมื่อ PIN ไม่ถูกต้องหรือไม่ได้รับอนุมัติ (UC08 Alternative Flow 1a; ASM-02, ASM-05)
5. สร้างกระบวนการบันทึกตัดสต๊อก หลังรหัส PIN ผ่าน และบันทึกต้นทุนสูญเสียจากสต๊อกที่หักจริง (UC08 Main Flow 4-5; ASM-04, Post-condition)
6. สร้างรายงาน Loss & Waste Report เพื่อให้เจ้าของร้าน/ผู้จัดการเห็นค่าใช้จ่ายภายในและสต๊อกที่ถูกตัดออกแล้ว (UC08 Main Flow 5; ASM-04)
7. ดำเนินการทดสอบด้านสถานะ approval, กรณี PIN ผิด, เอกสารรายงาน, และการลดสต๊อกให้ไม่กระทบยอดขายจริงเพื่อยืนยันว่า logic ตรงตาม spec (UC08 Main Flow 1-5, Alternative Flow 1a; ASM-01-05)

## 8. สิ่งที่ยังไม่ทำ
ไม่มี Open Questions ที่ค้างอยู่หลังขั้น Clarify สำหรับ UC08 ดังนั้นจะไม่ตอบคำถามใด ๆ เพิ่มเติมโดยอัตโนมัติ

ส่วนที่ยังไม่สร้างจนกว่าจะมีคำตอบเพิ่มเติม:
- รูปแบบ report detail ที่สุดท้ายของ Loss & Waste Report หากทีมต้องการรายงานแบบมีสรุปย่อยหรือแยกตามพนักงาน/วัน/เหตุผล
- โครงสร้างการเก็บประวัติ PIN และ Audit Trail ว่าจะเก็บแค่ผลลัพธ์อนุมัติหรือรวม log ของการพยายามยืนยันด้วย PIN ทุกครั้ง
- ระดับความละเอียดของ UX ในหน้าจอ Internal Waste Log หากต้องการมีป๊อปอัปหรือ modal ขั้นตอนแน่นกว่านี้

ส่วนนี้จะยังไม่สร้างจนกว่าทีมจะให้คำตอบเพิ่มเติม เพราะ spec ของ UC08 ระบุได้เพียงโจทย์หลักและเงื่อนไขยืนยัน PIN แต่ยังไม่ได้ระบุ UI detail หรือ report format แบบละเอียด
