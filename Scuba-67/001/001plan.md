# แผนทางเทคนิค: UC01 - จัดการคำสั่งซื้อหน้าร้าน

## 1. สรุปแนวทาง
1. ฟีเจอร์รองรับแคชเชียร์เลือกเมนู กำหนด modifiers และสร้างคำสั่งซื้อหน้าร้าน (UC01 Main Flow 1-2)
2. หนึ่งบิลรองรับหลายรายการ หลายจำนวน และเมนูซ้ำที่มี modifiers ต่างกัน (ASM-07)
3. ตะกร้าแก้ไขจำนวน modifiers และลบรายการได้ก่อนยืนยัน พร้อมคำนวณยอดใหม่ทันที (UC01 Alternative 1a, ASM-02, ASM-05)
4. ระบบแสดงยอดบนจอ POS และหน้าจอลูกค้า พร้อมตรวจตัวเลือกบังคับก่อนยืนยัน (UC01 Main Flow 3-4, ASM-01, ASM-06, ASM-08)
5. เมื่อยืนยัน ให้สร้างคำสั่งซื้อสถานะรอชำระเงินและเตรียมข้อมูลสำหรับ KOT กับหน้าจอรับเงิน (UC01 Post-condition, ASM-04)

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| React (Vite) สำหรับหน้าบ้าน | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นตาม prompt; spec ไม่กำหนด framework |
| Python FastAPI สำหรับหลังบ้าน | ทีมเลือกเอง ไม่ได้มาจาก spec | ค่าเริ่มต้นตาม prompt; spec ไม่กำหนด framework |
| ฐานข้อมูล | ทีมเลือกเอง ไม่ได้มาจาก spec | ยังไม่เลือกระบบจัดเก็บหรือฐานข้อมูล เพราะ spec ไม่ได้กำหนด |

## 3. โมเดลข้อมูล

โมเดลต่อไปนี้เป็นข้อเสนอเชิงโครงสร้าง ไม่ได้กำหนดชนิดฐานข้อมูลหรือขอบเขตการเป็นเจ้าของข้อมูล

| Entity | ฟิลด์หลัก | รองรับข้อกำหนด |
|---|---|---|
| Order | status, total, items | UC01 Post-condition, Main Flow 3-4; ASM-04, ASM-08 |
| OrderItem | menu reference, quantity, selected modifiers, unit price, line total | UC01 Main Flow 1-3, Alternative 1a; ASM-02, ASM-05, ASM-07 |
| MenuCategory | category reference, display name | UC01 Main Flow 1 |
| MenuItem | category reference, display name, base price | UC01 Main Flow 1-3; ASM-07 |
| ModifierOption | modifier group, display name, price effect, required/default configuration, availability state | UC01 Main Flow 2, Alternative 2a; ASM-03, ASM-06 |

ไม่มี FR ID ใน spec จึงอ้างอิง UC01 flow และ ASM ที่มีอยู่แทน โดยไม่สร้าง FR ใหม่

## 4. API / หน้าจอ

- `GET /api/v1/menu/categories` รับคำขอรายการหมวดหมู่ ส่งกลับหมวดหมู่ที่ใช้เลือกเมนู (UC01 Main Flow 1)
- `GET /api/v1/menu/categories/{categoryId}/items` รับ category ID ส่งกลับเมนูและตัวเลือกเสริมพร้อมสถานะพร้อมเลือก/หมด (UC01 Main Flow 1-2, Alternative 2a; ASM-03, ASM-06)
- `POST /api/v1/orders` รับรายการเมนู จำนวน และ modifiers ส่งกลับคำสั่งซื้อสถานะ Pending Payment กับยอดรวมที่คำนวณแล้ว (UC01 Main Flow 1-4, Post-condition; ASM-01, ASM-04, ASM-07)
- หน้าจอ POS และตะกร้า รับการเลือก/แก้ไขรายการ แสดง modifiers ยอดใหม่ และให้ยืนยันเมื่อมีสินค้าและเลือกตัวเลือกบังคับครบ (UC01 Main Flow 1-4, Alternative 1a; ASM-01, ASM-02, ASM-05, ASM-06)
- หน้าจอฝั่งลูกค้า รับยอดรวมจากตะกร้าหรือคำสั่งซื้อที่ยืนยันแล้ว แสดงยอดในขั้นตอนคิดเงิน (UC01 Main Flow 3, Post-condition; ASM-08)
- ข้อมูล KOT และวิธีส่งไปยังเครื่องพิมพ์/หน้าจอรับเงินยังไม่กำหนด เพราะ spec ไม่ระบุ interface หรือกลไกส่งต่อ (UC01 Post-condition; ASM-04)

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| ไม่มี CON ID ใน spec | ไม่มี constraint ประเภท CON ให้ระบุในส่วนเทคโนโลยีหรือโครงสร้าง | ยังไม่ได้ใช้ เพราะ spec ไม่มี CON |
| ไม่มี DOM ID ใน spec | ไม่มีข้อจำกัด DOM ให้ผูกกับโมเดลหรือ API | ยังไม่ได้ใช้ เพราะ spec ไม่มี DOM |
| ไม่มี IF ID ใน spec | ไม่มี interface constraint ที่ระบุกลไก KOT/หน้าจอรับเงิน | ยังไม่ได้ใช้ เพราะ spec ไม่มี IF; รายละเอียดการเชื่อมต่อยังไม่กำหนด |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| ไม่มี AC ID ใน spec | ยังไม่มีชื่อ test ที่อ้าง AC ได้ | spec ไม่มี Acceptance Criteria จึงไม่สร้าง AC หรือ test ID ใหม่; เสนอให้เพิ่ม AC ใน spec ก่อนกำหนดชุดทดสอบตรวจรับ |

## 7. ลำดับงาน

1. ยืนยันขอบเขตข้อมูลเมนูและแหล่งข้อมูลสถานะวัตถุดิบที่หมดก่อนเชื่อมข้อมูล (UC01 Main Flow 1-2, Alternative 2a; ASM-03)
2. จัดทำแบบจำลองหมวดหมู่ เมนู modifiers และรายการในตะกร้า โดยยังไม่ผูกกับฐานข้อมูลที่ทีมไม่ได้เลือก (UC01 Main Flow 1-2; ASM-06, ASM-07)
3. ทำหน้าจอ POS สำหรับเลือกหลายเมนู หลายจำนวน และแยกรายการเมนูซ้ำตาม modifiers (UC01 Main Flow 1-2; ASM-07)
4. ทำการเลือก modifiers บังคับ/ค่าเริ่มต้น/ไม่บังคับ และแสดงสถานะตัวเลือกหมด (UC01 Main Flow 2, Alternative 2a; ASM-03, ASM-06)
5. ทำการแก้ไขตะกร้าและคำนวณราคาต่อหน่วยกับยอดรวมใหม่หลังทุกการเปลี่ยนแปลง (UC01 Main Flow 3, Alternative 1a; ASM-02, ASM-05, ASM-08)
6. ทำหน้าจอทวนรายการ ตรวจเงื่อนไขก่อนยืนยัน และแสดงยอดฝั่งลูกค้า (UC01 Main Flow 3-4; ASM-01, ASM-08)
7. ทำการสร้างคำสั่งซื้อสถานะ Pending Payment และจัดเตรียมข้อมูล KOT/ยอดรับเงิน โดยกำหนดกลไกส่งต่อหลังทีมยืนยัน interface (UC01 Post-condition; ASM-04)
8. เพิ่ม AC ใน spec แล้วจึงผูก test ตรวจรับกับ AC ID เดิม ก่อนใช้เป็นเกณฑ์ปิดงาน (ไม่มี AC ID ใน spec)

## 8. สิ่งที่ยังไม่ทำ

spec ไม่มีหัวข้อ Open Questions จึงไม่มีคำถามให้คัดลอก และไม่มีส่วนใดที่อ้าง Q ID ได้โดยไม่สร้าง ID ใหม่

รายละเอียดต่อไปนี้ยังไม่กำหนดใน spec และจะไม่ตัดสินใจแทนทีม:
- แหล่งข้อมูลเมนู ราคา modifiers และสถานะสต๊อกของวัตถุดิบเสริม
- วิธีเชื่อมต่อและพฤติกรรมเมื่อส่ง KOT หรือยอดไปหน้าจอรับเงินไม่สำเร็จ
- การเลือกฐานข้อมูลและขอบเขตการจัดเก็บข้อมูล

ส่วนที่เกี่ยวข้องกับรายละเอียดเหล่านี้จะยังไม่สร้างจนกว่าทีมจะกำหนดคำตอบ ทั้งนี้รายการข้างต้นเป็นช่องว่างที่พบในเอกสาร ไม่ใช่ Open Question ID จาก spec