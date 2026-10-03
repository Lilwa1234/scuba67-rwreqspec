# CHANGELOG ของ spec

บันทึกการเปลี่ยนแปลงของ spec ทุกเวอร์ชัน ทุกการแก้ต้องผ่าน CR ใน `specs/changes/` (ห้ามแก้เงียบ)

> ⚠️ TEMPLATE: ช่อง `<...>` ทีมต้องกรอกหลังตัดสิน CR-001 แล้ว

## SPEC-POS-001 v3 (<ปปปป-ดด-วว>) — CR-001

ไฟล์: `<path ของ spec ที่แก้>` | tag: `<feature>-v3` | CR: `specs/changes/CR-001.md`

### เพิ่ม (Added)
- <FR-POS-xx ...>
- <AC-POS-xx (FR-POS-xx) ...>
- <Q-xx ...>

### แก้ (Changed)
- <ID เดิม>: <ก่อน> -> <หลัง>
- หัว spec: Status <เดิม> -> v3, Updated <วันที่>

### ลบ / เลื่อน (Removed / Deferred)
- <ID หรือรายการที่ลบ / ย้ายไป Out of scope เพื่อแลก>

## SPEC-POS-001 v2 (<ปปปป-ดด-วว>)
- <สรุปจาก clarify รอบก่อน ถ้าทีมนับเป็น v2>

## SPEC-POS-001 v1 (2569-09-23)
- ฉบับแรก: FR-POS-01 ถึง FR-POS-06, NFR-PERF-01, NFR-PERF-02, NFR-SEC-01, NFR-REL-01, AC-POS-01 ถึง AC-POS-06

---

## ส่วนที่ต้องแก้ใน spec.md เมื่อ CR-001 ได้รับการยอมรับ (checklist)

- [ ] หัว spec: `Status: v3 | Updated: <วันที่>` และเพิ่มบรรทัดเวอร์ชัน เช่น `Version: v3 (CR-001)`
- [ ] เพิ่ม FR / AC ใหม่ใช้เลขต่อจากเดิม (ห้ามเปลี่ยนเลขเดิม)
- [ ] อัปเดตตาราง Traceability
- [ ] เพิ่ม Q ใหม่ในหัวข้อ Assumptions & Open Questions
- [ ] commit ไฟล์ spec + `specs/changes/CR-001.md` + `specs/CHANGELOG.md` ในคำสั่งเดียว ข้อความขึ้นต้น `CR-001: ...`
- [ ] `git tag <feature>-v3`
