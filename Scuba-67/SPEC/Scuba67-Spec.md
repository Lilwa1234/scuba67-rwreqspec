# แผนงานสำหรับฟีเจอร์ การสั่งซื้อและชำระเงินด้วย Dynamic QR Code (Sales Ordering & Payment)

## 1. สรุปแนวทาง

ฟีเจอร์นี้ให้พนักงานแคชเชียร์สร้างคำสั่งซื้อ เลือกตัวเลือกเมนู และรับชำระเงินผ่าน Dynamic QR Code ที่ล็อกยอดเงินได้อย่างรวดเร็วและแม่นยำ พร้อมทั้งส่งคำสั่งซื้อเข้าครัวผ่านเครือข่าย Local LAN โดยตรงเพื่อลดการทำงานแบบกึ่งแมนนวล ระบบจะตรวจสอบสถานะการชำระเงินผ่าน Webhook และจัดเก็บ Transaction Log แบบเรียลไทม์เพื่อป้องกันปัญหาเงินค้างในอากาศตามเงื่อนไขใน FR-POS-01 ถึง FR-POS-06

ผู้ใช้หลักคือแคชเชียร์หน้าร้านที่ต้องทำรายการขายให้เสร็จสิ้นภายใน 1 นาที, พนักงานครัวที่รับใบแจ้งงาน (KOT), และเจ้าของร้านที่ต้องตรวจสอบยอดเงินเข้าผ่าน Transaction Feed

สถาปัตยกรรมจะใช้ SQLite/IndexedDB บนเครื่อง POS สำหรับจัดเก็บข้อมูลแบบ Local-First เพื่อรองรับการทำงานแบบ Offline, ใช้ Node.js/Go backend สำหรับ API จัดการออร์เดอร์และการพิมพ์ KOT, และ React (Vite) สำหรับหน้าจอ POS หน้าร้านและจอลูกค้า โดยมี WebSocket ช่วยกระจายสถานะเงินเข้าและสถานะสินค้าหมดแบบเรียลไทม์

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| SQLite (Local) / PostgreSQL (Cloud) | CON-TECH-01 | ใช้ SQLite หน้าร้านเพื่อให้ขายออฟไลน์ได้ และซิงก์ขึ้น PostgreSQL บน Cloud |
| ESC/POS Network Driver | CON-HW-01 | ส่งคำสั่งพิมพ์ตรงไปยังเครื่องพิมพ์ความร้อนในครัวผ่านพอร์ต LAN 9100 |
| Payment Gateway SDK (PromptPay) | IF-PAY-01 | ใช้สร้าง Dynamic QR Code แบบระบุจำนวนเงิน และรับ Webhook Callback |
| React (Vite) + TailwindCSS | ทีมเลือกเอง ไม่ได้มาจาก spec | พัฒนาหน้าจอ POS Grid และ Customer Facing Display ที่ตอบสนองต่อการสัมผัสรวดเร็ว |
| Node.js (Express / Fastify) | ทีมเลือกเอง ไม่ได้มาจาก spec | รองรับ API จัดการคำสั่งซื้อ และจัดการคิวงานพิมพ์ใบแจ้งงานครัว |
| WebSocket Service | IF-NOT-01 | แจ้งเตือนสถานะเงินเข้าจอแคชเชียร์ และกระจายสถานะ Sold-out แบบทันที |
| Audit & Transaction Store | DOM-FIN-01 | บันทึกประวัติธุรกรรมการเงินและ Webhook Payload เก็บไว้ไม่น้อยกว่า 90 วัน |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับ FR / Constraint |
|---|---|---|
| Order | order_id, order_no, order_status, total_amount, payment_method, is_offline, created_at | FR-POS-01, FR-POS-04, NFR-REL-01 |
| OrderItem | item_id, order_id, menu_id, quantity, unit_price, subtotal, notes | FR-POS-01 |
| OrderItemModifier | item_modifier_id, item_id, modifier_id, extra_price | FR-POS-01 |
| MenuItem | menu_id, category_id, name, base_price, is_available, updated_at | FR-POS-01, FR-POS-02 |
| PaymentTransaction | transaction_id, order_id, gateway_ref_id, amount, status, qr_payload, confirmed_at | FR-POS-04, FR-POS-05, FR-POS-06, DOM-FIN-01 |
| KitchenPrintQueue | print_id, order_id, station_id, payload, print_status, retry_count | FR-POS-03, CON-HW-01 |
| PaymentAuditLog | log_id, transaction_id, raw_webhook_payload, verified_status, logged_at | DOM-FIN-01, NFR-SEC-01 |

หมายเหตุ: เพื่อความถูกต้องทางการเงิน ฟิลด์ราคาและยอดเงินทั้งหมด (amount, base_price, extra_price) จะใช้ชนิดข้อมูล Decimal(10,2) หรือจัดเก็บเป็นหน่วยสตางค์ (Integer) เพื่อป้องกันความคลาดเคลื่อนของทศนิยม

## 4. API / หน้าจอ

### หน้าจอ
- UI-01: `/pos` — หน้าจอขายหลัก แสดงหมวดหมู่สินค้า, ปุ่มเมนู, ปุ่มกดค้างสลับ Sold Out, ตะกร้าสินค้า และปุ่มชำระเงิน รองรับ FR-POS-01, FR-POS-02
- UI-02: `/display` — หน้าจอฝั่งลูกค้า (Customer Facing Display) แสดงสรุปรายการบิล ยอดเงินรวม และภาพ Dynamic QR Code รองรับ FR-POS-04
- UI-03: `/pos/transactions` — หน้าต่างแสดง Transaction Feed ยอดเงินเข้าเรียลไทม์ พร้อมปุ่มกด Re-check สถานะจาก Gateway รองรับ FR-POS-05, FR-POS-06

### API
- GET /api/v1/menus?availableOnly=false — คืนค่ารายการเมนู หมวดหมู่ และสถานะความพร้อมจำหน่าย รองรับ FR-POS-01, FR-POS-02
- PATCH /api/v1/menus/{menuId}/availability — อัปเดตสถานะเมนู (พร้อมขาย / สินค้าหมด) รองรับ FR-POS-02
- POST /api/v1/orders — สร้างคำสั่งซื้อ คำนวณยอดเงิน และส่งคำขอพิมพ์ใบครัวเข้าคิวพิมพ์ รองรับ FR-POS-01, FR-POS-03
- POST /api/v1/payments/dynamic-qr — ส่งยอดเงินรวมเพื่อขอสร้าง Dynamic QR Code จาก Gateway รองรับ FR-POS-04
- POST /api/v1/payments/webhook — รับสัญญาณยืนยันการชำระเงินจากธนาคาร/Gateway ตรวจสอบ Signature และปิดบิล รองรับ FR-POS-05, NFR-SEC-01
- POST /api/v1/payments/recheck — ตรวจสอบสถานะการโอนเงินโดยตรงจาก Gateway เมื่อเกิด Timeout รองรับ FR-POS-06

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| CON-TECH-01 | SQLite ใช้งานบนเครื่อง POS หน้าร้าน เก็บข้อมูล Order, MenuItem, Transaction ชั่วคราว รองรับการทำงาน Offline | ใช้แล้ว |
| CON-HW-01 | KitchenPrintQueue และ Service สั่งพิมพ์ผ่าน ESC/POS Socket (TCP/IP Port 9100) ไปยังเครื่องพิมพ์ครัว | ใช้แล้ว |
| IF-PAY-01 | การเรียก API สร้าง QR และ Webhook Receiver Endpoint ใน `/api/v1/payments/*` | ใช้แล้ว |
| DOM-FIN-01 | PaymentAuditLog entity และการบันทึก Raw Webhook Payload ลงฐานข้อมูล พร้อมนโยบายเก็บ 90 วัน | ใช้แล้ว |
| IF-NOT-01 | WebSocket Service สำหรับบรอดแคสต์สถานะ Sold-out และสถานะเงินเข้าเรียลไทม์ | ใช้แล้ว |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| AC-POS-01 | test_AC_POS_01_dynamic_qr_success_closes_order | จำลองบิล 125 บาท สร้าง Dynamic QR แล้วส่ง Webhook จำลองเงินเข้า ตรวจว่าสถานะ Order เปลี่ยนเป็น PAID และ Transaction Log ถูกบันทึกภายใน 3 วินาที |
| AC-POS-02 | test_AC_POS_02_toggle_sold_out_blocks_ordering | จำลองการกดค้างสลับสถานะเมนูเป็น Sold Out ตรวจว่าปุ่มเปลี่ยนเป็นสีเทา และ API ปฏิเสธการเพิ่มเมนูนี้ลงในคำสั่งซื้อ |
| AC-POS-03 | test_AC_POS_03_order_confirmation_dispatches_kot | กดยืนยันคำสั่งซื้อ ตรวจว่าระบบส่ง Payload งานพิมพ์ไปยัง Socket ของเครื่องพิมพ์ครัว และพิมพ์ใบเสร็จ KOT ออกมาถูกต้องภายใน 2 วินาที |
| AC-POS-04 | test_AC_POS_04_payment_timeout_enables_recheck | จำลองสถานการณ์ไม่มี Webhook ตอบกลับเกิน 3 นาที ตรวจว่าหน้าจอเปลี่ยนเป็นสถานะ Pending และปุ่ม Re-check เปิดให้กดส่งคำขอตรวจสอบได้ |
| AC-POS-05 | test_AC_POS_05_offline_sales_and_sync | ปิดการเชื่อมต่อเครือข่าย ทำการขายเงินสด 1 บิล ตรวจว่าบิลถูกบันทึกลง Local SQLite และเมื่อต่อเน็ตข้อมูลต้องถูกส่งขึ้น Cloud อัตโนมัติ |
| AC-POS-06 | test_AC_POS_06_audit_log_captures_transaction | ตรวจสอบตาราง PaymentAuditLog หลังการชำระเงินสำเร็จ ว่ามี Order ID, Gateway Ref ID, ยอดเงิน และ Raw Payload บันทึกครบถ้วน |

## 7. ลำดับงาน

1. สร้างโครงสร้างฐานข้อมูล Local SQLite (Order, MenuItem, PaymentTransaction, AuditLog) ตาม DOM-FIN-01 และ CON-TECH-01
2. พัฒนาระบบจัดการตะกร้าสินค้า การเลือก Modifiers และการคำนวณราคาบิลหน้าร้าน ตาม FR-POS-01
3. พัฒนาระบบ Quick Sold-out Toggle ที่หน้าจอขาย พร้อมส่ง WebSocket แจ้งเตือน ตาม FR-POS-02, IF-NOT-01 และ AC-POS-02
4. พัฒนา Kitchen Dispatcher Service เชื่อมต่อเครื่องพิมพ์ ESC/POS ผ่าน Local LAN ตาม FR-POS-03, CON-HW-01 และ AC-POS-03
5. พัฒนาระบบเชื่อมต่อ Payment Gateway เพื่อสร้าง Dynamic QR Code แสดงบน Customer Display ตาม FR-POS-04 และ AC-POS-01
6. พัฒนา Webhook Receiver พร้อมระบบตรวจสอบ Signature และบันทึก Audit Log ตาม FR-POS-05, DOM-FIN-01, NFR-SEC-01 และ AC-POS-06
7. พัฒนาระบบตรวจจับ Timeout 3 นาที และปุ่ม Manual Re-check ตาม FR-POS-06 และ AC-POS-04
8. ทดสอบสถานะ Offline Mode, การทำ Local Caching และการซิงก์ข้อมูลขึ้น Cloud ตาม NFR-REL-01 และ AC-POS-05 ก่อนส่งมอบ

## 8. สิ่งที่ยังไม่ทำ

- Q-01 กรณีลูกค้าสแกนจ่ายแล้วแต่ Webhook หลุดและสลิปในมือถือลูกค้าขึ้นโอนสำเร็จ แคชเชียร์สามารถกด "Force Pass พร้อมแนบภาพสลิป" ได้ทันที หรือต้องให้ผู้จัดการใส่รหัส PIN? -> ถามพี่เบิร์ด (เจ้าของร้าน)
  *ส่วนฟังก์ชันปุ่ม Force Pass จะยังไม่พัฒนาลงในระบบจนกว่าจะได้ข้อสรุป*

- Q-02 รายการเครื่องดื่มประเภท "ปั่น" ต้องแยกพิมพ์ใบแจ้งงานไปยังเคาน์เตอร์บาร์น้ำจุดที่ 2 หรือไม่? -> ถามพนักงานหน้าร้าน
  *ส่วนการตั้งค่าพิมพ์แยกสถานี (Multi-station Routing) จะยังคงใช้การพิมพ์รวมใบเดียวก่อนจนกว่าจะได้คำตอบ*

> หมายเหตุ: เอกสาร Spec และ Plan ฉบับนี้อ้างอิงจากบทสัมภาษณ์ร้านจริงเวอร์ชัน Draft v1 โดยมุ่งเน้นการแก้ปัญหาคอขวดด้านความเร็วหน้าร้านและปัญหาเงินในอากาศเป็นสำคัญ