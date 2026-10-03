# แผนทางเทคนิค: UC05 - จัดการเมนูและราคาตัวเลือก

## 1. สรุปแนวทาง
1. ฟีเจอร์นี้รองรับการจัดการเมนูและราคาตัวเลือกเสริมจากเจ้าของร้านหรือผู้จัดการด้วยสิทธิ์ Admin Role ทั้งในระบบหลังบ้านและแอป POS (UC05 Main Flow 1, ASM-05)
2. ระบบจะใช้หน้าจอจัดการเมนูแบบ Single Page Form เพื่อรับข้อมูลชื่อเมนู หมวดหมู่ ราคาเริ่มต้น และราคาบวกเพิ่มของ Modifier Group เมื่อมีการเลือกใช้ (UC05 Main Flow 2-3, ASM-04)
3. หลังกดยืนยัน ระบบจะบันทึกข้อมูลลงฐานข้อมูลผ่าน CRUD API และทำ Instant Sync ให้หน้าจอขายหน้าร้านอัปเดตทันทีโดยไม่ต้อง Restart เครื่อง (UC05 Main Flow 4-5, ASM-03, ASM-06)
4. หากเมนูเคยมีประวัติการขายจะไม่ถูกลบถาวร แต่จะใช้ Soft Delete โดยเปลี่ยนเป็น `is_active = false` และซ่อนจากหน้าจอขาย แต่ยังคงชื่อเมนูในรายงานย้อนหลัง (UC05 Alternative 2a, ASM-01)
5. ถ้าการซิงก์หรือการแสดงผลบนหน้าจอขายไม่สำเร็จ ระบบจะคงข้อมูลหลักในฐานข้อมูลให้พร้อมใช้งาน และเรียกดึงข้อมูลกลับเมื่อเชื่อมต่อกลับมา (UC05 Alternative 5a, ASM-06)

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| React (Vite) สำหรับหน้าจอจัดการเมนูและ POS | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นตาม prompt; spec ไม่ระบุ framework สำหรับหน้าจอจัดการเมนูหรือหน้าจอขาย |
| Python FastAPI สำหรับหลังบ้าน | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นตาม prompt; spec ไม่ระบุ framework สำหรับ CRUD API หรือการ sync |
| PostgreSQL หรือฐานข้อมูล SQL ทั่วไป | ทีมเลือกเอง ไม่ได้มาจาก spec | spec ไม่ระบุแบบฐานข้อมูล แต่ต้องเก็บเมนู หมวดหมู่ Modifier Group และสถานะเมนูอย่างชัดเจน |
| WebSocket / SSE สำหรับ Instant Sync | ASM-03 | ใช้เพื่อให้อัปเดตข้อมูลเมนูและราคาไปยังหน้าจอขายทันทีเมื่อมีการเปลี่ยนแปลง |
| Soft Delete (`is_active = false`) | ASM-01 | สอดคล้องกับข้อกำหนดว่าห้ามลบเมนูถาวรเมื่อมีประวัติการขาย |
| Single Page Form | ASM-04 | ใช้สำหรับจัดเก็บข้อมูลเมนูและ Modifier Group ในหน้าเดียวก่อนยืนยันบันทึก |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับข้อกำหนด |
|---|---|---|
| Menu | menu_id, name, category_id, base_price, is_active, created_at, updated_at | UC05 Main Flow 2, 4-5; ASM-01, ASM-04, ASM-06 |
| Category | category_id, name, is_active | UC05 Main Flow 2; ASM-04 |
| ModifierGroup | modifier_group_id, menu_id, name, is_required, is_active | UC05 Main Flow 3; ASM-02, ASM-04 |
| ModifierOption | modifier_option_id, modifier_group_id, label, price_delta, is_active | UC05 Main Flow 3; ASM-02, ASM-04 |
| MenuSyncEvent | sync_id, menu_id, event_type, payload, triggered_at, status | UC05 Main Flow 5; ASM-03, ASM-06 |
| AuditLog | log_id, actor_id, action, target_type, target_id, changed_at | UC05 Main Flow 1-5; ASM-05, ASM-06 |

ไม่มีฟิลด์ที่เก็บข้อมูลส่วนบุคคลหรือข้อมูลที่ไม่ได้ระบุใน UC05 หากมีข้อมูลที่เกี่ยวข้องกับเมนูหรือ modifier จะถูกเก็บเฉพาะในฟิลด์ที่ใช้ในการจัดการเมนูและราคาตัวเลือกเสริมเท่านั้น เช่น ชื่อเมนู หมวดหมู่ ราคาเริ่มต้น และราคาเพิ่มจาก modifier

## 4. API / หน้าจอ

- `GET /api/admin/menus` ส่งกลับรายการเมนูพร้อมหมวดหมู่และสถานะ `is_active` เพื่อให้เจ้าของร้านหรือผู้จัดการเห็นข้อมูลเมนูทั้งหมด (UC05 Main Flow 1; ASM-05)
- `POST /api/admin/menus` รับ `name`, `categoryId`, `basePrice`, `isActive` และส่งกลับเมนูที่สร้างใหม่ พร้อมสถานะสำเร็จ (UC05 Main Flow 2, 4; ASM-04, ASM-06)
- `PATCH /api/admin/menus/{menuId}` รับข้อมูลที่ต้องปรับปรุง เช่น ราคา, หมวดหมู่, สถานะเมนู และส่งกลับเมนูที่อัปเดต (UC05 Main Flow 2, 4; ASM-04, ASM-06)
- `DELETE /api/admin/menus/{menuId}` ไม่ลบจริง แต่ใช้ soft delete โดยตั้ง `is_active = false` และบันทึกเหตุการณ์เพื่อรายงานย้อนหลัง (UC05 Alternative 2a; ASM-01)
- `POST /api/admin/menus/{menuId}/modifier-groups` รับ `groupName`, `isRequired`, `options` และส่งกลับกลุ่ม modifier ที่ผูกกับเมนู (UC05 Main Flow 3; ASM-02, ASM-04)
- `PATCH /api/admin/menus/{menuId}/modifier-groups/{groupId}` อัปเดตราคาบวกเพิ่มและข้อมูลกลุ่ม modifier ของเมนู (UC05 Main Flow 3; ASM-02, ASM-04)
- `GET /api/pos/menu-catalog` ส่งกลับเมนูที่พร้อมขายและ modifier ที่ผูกกับเมนูให้หน้าจอขายแสดงผล (UC05 Main Flow 5; ASM-03, ASM-06)
- `POST /api/admin/sync/menu-events` รับ event payload จาก CRUD API เพื่อส่งข้อมูลไปยัง POS และอัปเดต real-time (UC05 Main Flow 5; ASM-03, ASM-06)
- หน้าจอจัดการเมนู: แสดงฟอร์มกรอกข้อมูลเมนูและ modifier ในหน้าเดียว พร้อม validation ก่อนอัปเดต (UC05 Main Flow 2-4; ASM-04)
- หน้าจอ POS: แสดงเมนูที่ active เท่านั้น และซ่อนเมนูที่ถูก soft delete ออกจากการขาย แต่ยังคงให้รายงานย้อนหลังเห็นข้อมูลเดิม (UC05 Alternative 2a; ASM-01)

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| ไม่มี CON ID ใน spec | ไม่มีข้อกำหนดเชิงเทคนิคที่ต้องใช้ใน plan | ยังไม่ได้ใช้ เพราะ spec ไม่มี CON |
| ไม่มี DOM ID ใน spec | ไม่มีข้อจำกัดด้านข้อมูลหรือโครงสร้างข้อมูลเฉพาะ | ยังไม่ได้ใช้ เพราะ spec ไม่มี DOM |
| ไม่มี IF ID ใน spec | ไม่มีเงื่อนไข interface หรือการเชื่อมต่อระบบภายนอกที่ชัดเจน | ยังไม่ได้ใช้ เพราะ spec ไม่มี IF |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| ไม่มี AC ID ใน spec | `test_AC_UC05_missing_01` | spec ของ UC05 ยังไม่มี Acceptance Criteria ที่ระบุ ID อย่างชัดเจน จึงไม่สามารถผูก test ที่มี AC ID จริงได้ โดยทีมควรเพิ่ม AC ให้ชัดก่อนเริ่มเขียน Automated Test อย่างเป็นทางการ |

## 7. ลำดับงาน

1. กำหนดโครงสร้างเมนู หมวดหมู่ Modifier Group และ Modifier Option ให้สอดคล้องกับเอกสาร spec และสถานะ `is_active` สำหรับ soft delete (UC05 Main Flow 2-3; ASM-01, ASM-02, ASM-04)
2. สร้าง API สำหรับจัดการเมนูในระบบหลังบ้าน พร้อม validation การกรอกข้อมูลชื่อเมนู หมวดหมู่ ราคาเริ่มต้น และราคามูลค่า modifier ก่อนบันทึก (UC05 Main Flow 2-4; ASM-04, ASM-06)
3. สร้างฟอร์มจัดการเมนูแบบ Single Page Form และฟังก์ชันผูก Modifier Group พร้อมค่าราคาบวกเพิ่มในหน้าเดียว (UC05 Main Flow 3; ASM-02, ASM-04)
4. สร้างการบันทึกเมนูและ modifier ลงฐานข้อมูลผ่าน CRUD API รวมการบันทึก audit log สำหรับผู้ใช้งาน Admin Role (UC05 Main Flow 1-4; ASM-05, ASM-06)
5. สร้างการ sync real-time ไปยังหน้าจอขายผ่าน WebSocket/SSE พร้อมตรวจสอบว่าเมนูและราคาอัปเดตทันทีโดยไม่ต้อง restart เครื่อง (UC05 Main Flow 5; ASM-03, ASM-06)
6. สร้างโค้ดซ่อนเมนูแบบ soft delete สำหรับเมนูที่เคยมีประวัติการขาย และคงข้อมูลไว้ในรายงานย้อนหลัง (UC05 Alternative 2a; ASM-01)
7. สร้างฟังก์ชัน fallback เมื่อการ sync ขัดข้อง ให้ฐานข้อมูลยังคงพร้อมใช้งานและดึงข้อมูลกลับเมื่อเชื่อมต่อคืน (UC05 Alternative 5a; ASM-06)
8. ดำเนินแผนทดสอบเชิงฟังก์ชันสำหรับการเปิดหน้า, การสร้างเมนู, การผูก modifier, การ soft delete, และการ sync เพื่อยืนยันว่าทุกเมนูมีข้อมูลที่จำเป็นและเค้าโครงซิงก์ทำงานได้ (UC05 Main Flow 1-5, Alternative 2a, 5a; ASM-01-06)

## 8. สิ่งที่ยังไม่ทำ

ไม่มี Open Questions ใน spec ขณะนี้ จึงไม่ต้องคัดลอกคำถามใด ๆ มาไว้ที่นี่

ส่วนที่ยังไม่สร้างจนกว่าจะมีคำตอบเพิ่มเติม:
- การกำหนดรูปแบบ payload และ protocol ของ Real-time Sync ที่ใช้จริงระหว่าง backend กับ POS จนกว่าทีมจะตัดสินใจชัดเจน
- การกำหนดคำแนะนำการจัดเก็บข้อมูลหรือการ index ของตารางเมนู/หมวดหมู่/Modifier Group หากต้องรองรับปริมาณข้อมูลจริงมากกว่า sandbox
- การกำหนดตัวอย่าง UI/UX อย่างละเอียดสำหรับหน้าจอจัดการเมนูและหน้าจอขายหากทีมต้องการโครงสร้างการแสดงผลที่เฉพาะมากขึ้น

ส่วนเหล่านี้ยังไม่สร้างจนกว่าทีมจะให้คำตอบเพิ่มเติม เพราะ spec ไม่ได้ระบุรายละเอียดเชิง interface หรือข้อกำหนดเชิงประสบการณ์ของผู้ใช้แบบละเอียดมากพอสำหรับการพัฒนาโดยตรง
