# แผนทางเทคนิค: UC09 - ใช้งานโหมดออฟไลน์และซิงก์ข้อมูล

## 1. สรุปแนวทาง
1. ฟีเจอร์นี้ช่วยให้ระบบตรวจพบการหลุดของอินเทอร์เน็ตและสลับเครื่อง POS เป็น Offline Mode ทันที โดยแสดงแถบสถานะสีส้มและยอมให้แคชเชียร์ดำเนินการขายต่อได้ในรูปแบบเงินสดเท่านั้น (UC09 Main Flow 1-2, ASM-01, ASM-07)
2. รายการธุรกรรมจะถูกบันทึกลงใน Local Database ของเครื่อง POS โดยใช้ SQLite หรือ IndexedDB เพื่อให้มีข้อมูลครบถ้วนแม้ไม่มีสัญญาณอินเทอร์เน็ต (UC09 Main Flow 3, ASM-02)
3. เมื่อกลับมาออนไลน์ ระบบจะเริ่ม Background Sync โดยอัตโนมัติ ส่งเฉพาะข้อมูลที่ค้างอยู่ พร้อมใช้ UUID + Timestamp เพื่อป้องกันการชนกันหรือเขียนทับระหว่าง Reconcile ข้อมูลขึ้นคลาวด์ (UC09 Main Flow 4, Alternative Flow 4a, ASM-03, ASM-04, ASM-08)
4. หลัง Sync สำเร็จ ระบบจะอัปโหลดข้อมูลบิลภายใน 30 วินาที และยอดขายรวมบนสมาร์ทโฟนของเจ้าของร้านจะอัปเดตเพิ่มขึ้นตรงกัน 100% พร้อมเปลี่ยนสถานะกลับเป็น Online (UC09 Main Flow 5-6, ASM-04, ASM-05, ASM-07)
5. แนวทางพัฒนาจะตัดแยกชัดเจนระหว่าง Network Status Monitor, Local Persistence Layer, Background Sync Worker, และ Pending Sync Indicator เพื่อให้การซิงก์และการแจ้งเตือนสอดคล้องกับข้อกำหนดระหว่างโหมดออฟไลน์และออนไลน์ (UC09 Main Flow 1-6, Alternative Flow 4a, ASM-06, ASM-08)

## 2. เทคโนโลยีที่ใช้

| สิ่งที่เลือก | มาจาก | หมายเหตุ |
|---|---|---|
| React (Vite) สำหรับหน้าจอ POS และแถบสถานะ Offline/Online | ทีมเลือกเอง ไม่ได้มาจาก spec | spec ไม่ระบุ framework UI แต่ต้องรองรับการสลับโหมดและแสดงสัญลักษณ์เตือนบน POS |
| Python FastAPI สำหรับ API และ Background Sync Worker | ทีมเลือกเอง ไม่ได้มาจาก spec | spec ไม่ระบุ framework backend แต่ต้องรองรับ Sync Queue และ Reconcile กับคลาวด์ |
| SQLite หรือ IndexedDB บนเครื่อง POS | ASM-02 | ใช้เป็น Local Database หลักสำหรับบันทึกธุรกรรมต่อเนื่องขณะไม่มีอินเทอร์เน็ต |
| Network Status Monitor | ASM-07 | ตรวจจับการเชื่อมต่อและสลับสภาวะออนไลน์/ออฟไลน์อัตโนมัติ |
| Background Sync Worker | ASM-04, ASM-08 | ทำงานเมื่อกลับมาออนไลน์ โดยอัปโหลดข้อมูลค้างอยู่อัตโนมัติภายใน 30 วินาที |
| Queue / Pending Sync State | ASM-06, ASM-08 | ใช้ติดตามรายการที่ยังไม่ได้อัปโหลด และแสดงสัญลักษณ์เตือนบนหน้าจอ |
| UUID + Timestamp Conflict Resolution | ASM-03 | ใช้ป้องกันการเขียนทับหรือเลขออร์เดอร์ชนกันระหว่างการ reconcile กับคลาวด์ |
| WebSocket / SSE สำหรับการอัปเดตยอดขาย | ASM-05 | ใช้ส่งข้อมูลยอดขายรวมไปยังสมาร์ทโฟนของเจ้าของร้านหลัง sync สำเร็จ |

## 3. โมเดลข้อมูล

| Entity | ฟิลด์หลัก | รองรับข้อกำหนด |
|---|---|---|
| TransactionLog | transaction_id, order_id, uuid, cashier_id, payment_type, amount, created_at, synced_at, status | UC09 Main Flow 2-5; ASM-01, ASM-02, ASM-03, ASM-08 |
| LocalTransactionStore | local_id, transaction_id, payload_json, store_id, created_at, updated_at, retry_count, status | UC09 Main Flow 3, Alternative Flow 4a; ASM-02, ASM-06, ASM-08 |
| SyncQueueItem | queue_id, transaction_id, retry_at, last_attempt_at, attempt_count, sync_status, error_code | UC09 Main Flow 4, Alternative Flow 4a; ASM-04, ASM-06, ASM-08 |
| NetworkStatus | device_id, is_online, changed_at, mode_label, status_color | UC09 Main Flow 1, 6; ASM-07 |
| OwnerSalesSummary | summary_id, store_id, total_sales, updated_at, sync_source | UC09 Main Flow 5-6; ASM-05 |
| PendingAlert | alert_id, queue_id, message, visible_on_pos, created_at | UC09 Alternative Flow 4a; ASM-06, ASM-08 |

ข้อมูลที่เก็บจะเน้นเฉพาะธุรกรรม, สถานะออนไลน์/ออฟไลน์, รายการค้างส่ง และผลรวมยอดขายเท่านั้น ไม่มีการจัดเก็บข้อมูลที่ไม่เกี่ยวข้องกับสภาวะการขายและข้อมูล Sync ตามข้อกำหนดของ UC09

## 4. API / หน้าจอ

- `GET /api/network/status` ส่งกลับสถานะตามสัญญาณอินเทอร์เน็ตปัจจุบันและโหมดออนไลน์/ออฟไลน์ที่เครื่อง POS อยู่ (UC09 Main Flow 1, 6; ASM-07)
- `POST /api/pos/transactions/local` รับ payload บิล/ธุรกรรมจากหน้าจอ POS และบันทึกลง Local Database พร้อม UUID + Timestamp (UC09 Main Flow 2-3; ASM-01, ASM-02, ASM-03)
- `GET /api/pos/transactions/pending` ส่งกลับรายการที่ยังไม่ได้ sync เพื่อใช้แสดงสถานะค้างส่งบนหน้าจอ POS (UC09 Alternative Flow 4a; ASM-06, ASM-08)
- `POST /api/sync/run` เริ่ม Background Sync อย่างอัตโนมัติเมื่ออินเทอร์เน็ตกลับมาสำเร็จ โดยส่งเฉพาะข้อมูลค้างอยู่ และบันทึกผลลัพธ์แต่ละรายการ (UC09 Main Flow 4; ASM-04, ASM-08)
- `GET /api/sales/summary` ส่งกลับยอดขายรวมที่พร้อมแสดงบนสมาร์ทโฟนของเจ้าของร้านหลัง Reconcile สำเร็จ (UC09 Main Flow 5-6; ASM-05)
- `WS /ws/network-status` หรือ `SSE /stream/network` ส่ง event เมื่อเข้าสู่ Offline Mode หรือกลับเป็น Online และอัปเดตสถานะบน UI (UC09 Main Flow 1, 6; ASM-07)
- หน้าจอ POS: แสดงแถบสถานะสีส้ม/เขียว, สัญลักษณ์เตือน pending sync, และรายการสถานะอัปโหลดสุดท้ายของธุรกรรม (UC09 Main Flow 1, 4-6; ASM-06, ASM-07)
- หน้าจอเจ้าของร้าน: แสดงยอดขายรวมจากข้อมูลที่ sync สำเร็จ และรายงานสถานะอัปเดตตามสภาวะออนไลน์/ออฟไลน์ (UC09 Main Flow 5-6; ASM-05)

## 5. ตารางตรวจ Constraints

| Constraint ID | ถูกนำไปใช้ที่ไหนใน plan | สถานะ |
|---|---|---|
| ไม่มี CON ID ใน spec | ไม่มีข้อจำกัดด้านโครงสร้างระบบหรือสถาปัตยกรรมที่ต้องใช้ใน plan นี้ | ยังไม่ได้ใช้ เพราะ spec ไม่มี CON |
| ไม่มี DOM ID ใน spec | ไม่มีข้อจำกัดด้านข้อมูลหรือการเก็บข้อมูลที่ต้องใช้ใน plan นี้ | ยังไม่ได้ใช้ เพราะ spec ไม่มี DOM |
| ไม่มี IF ID ใน spec | ไม่มีข้อกำหนดด้านระบบภายนอกหรือ interface ที่ต้องใช้ใน plan นี้ | ยังไม่ได้ใช้ เพราะ spec ไม่มี IF |
| ASM-01 | โหมดออฟไลน์รองรับชำระเงินสดเท่านั้นและยังคงขายได้ตามปกติ | ใช้แล้ว |
| ASM-02 | Local storage layer ใช้ SQLite หรือ IndexedDB สำหรับบันทึกธุรกรรมต่อเนื่อง | ใช้แล้ว |
| ASM-03 | UUID + Timestamp สำหรับป้องกันการชนกันขณะ reconcile | ใช้แล้ว |
| ASM-04 | Background Sync หลังกลับออนไลน์ภายใน 30 วินาที | ใช้แล้ว |
| ASM-05 | ยอดขายรวมอัปเดตตรงกัน 100% หลัง sync สำเร็จ | ใช้แล้ว |
| ASM-06 | Pending alert + Auto Retry ทุก 3 นาที | ใช้แล้ว |
| ASM-07 | Network Status Monitor สำหรับสลับสถานะ Online/Offline | ใช้แล้ว |
| ASM-08 | Sync เฉพาะข้อมูลค้างอยู่ และตรวจสอบสถานะรายการแต่ละรายการจนกว่าจะสำเร็จ | ใช้แล้ว |

## 6. แผนทดสอบจาก Acceptance Criteria

| AC ID | ชื่อ test | ทดสอบอย่างไร |
|---|---|---|
| ไม่มี AC ID ใน spec | `test_UC09_offline_mode_cash_only_sale` | ทดสอบว่าเมื่ออินเทอร์เน็ตขาดหาย ระบบแสดง Offline Mode สีส้ม และแคชเชียร์ยังสามารถรับออร์เดอร์ คิดเงิน และพิมพ์ใบเสร็จได้โดยรองรับชำระเงินสดเท่านั้น |
| ไม่มี AC ID ใน spec | `test_UC09_local_persistence_transactions` | ทดสอบว่า TransactionLog ถูกบันทึกลง SQLite หรือ IndexedDB และยังคงมีข้อมูลครบถ้วนเมื่อไม่มีอินเทอร์เน็ต |
| ไม่มี AC ID ใน spec | `test_UC09_reconcile_uuid_timestamp` | ทดสอบว่าเมื่อมีข้อมูลซ้ำหรือเลขออร์เดอร์ชนกัน ระบบใช้ UUID + Timestamp ให้ข้อมูลไม่เขียนทับและไม่เกิดความขัดแย้งในการ merge ข้อมูล |
| ไม่มี AC ID ใน spec | `test_UC09_background_sync_30s` | ทดสอบว่าหลังสัญญาณกลับมาสำเร็จ ระบบเริ่ม Background Sync โดยอัตโนมัติและอัปโหลดข้อมูลค้างอยู่ภายใน 30 วินาที |
| ไม่มี AC ID ใน spec | `test_UC09_pending_alert_retry_3m` | ทดสอบว่าเมื่อรายการยังไม่ได้ sync ระบบแสดงสัญลักษณ์เตือนและเริ่ม Auto Retry ทุก 3 นาทีจนกว่าจะสำเร็จ |
| ไม่มี AC ID ใน spec | `test_UC09_network_status_switch` | ทดสอบว่า Network Status Monitor จะเข้าสู่ Offline Mode ทันทีเมื่ออินเทอร์เน็ตหลุด และกลับเป็น Online เมื่อเชื่อมต่อได้อีกครั้ง |
| ไม่มี AC ID ใน spec | `test_UC09_sync_only_pending_data` | ทดสอบว่า Background Sync ส่งเฉพาะธุรกรรมและบิลที่ค้างอยู่ ไม่ได้ส่งข้อมูลซ้ำทั้งหมดกลับไปอีกครั้ง |
| ไม่มี AC ID ใน spec | `test_UC09_owner_sales_summary_sync` | ทดสอบว่าหลัง Sync สำเร็จ ยอดขายรวมบนสมาร์ทโฟนของเจ้าของร้านอัปเดตเพิ่มขึ้นตรงกัน 100% |

> หมายเหตุ: spec ปัจจุบันยังไม่มี AC ID หรือ AC list อย่างเป็นทางการ จึงใช้ชื่อ test ที่สรุปจากปรัชญาของ Main Flow, Alternative Flow, และ Assumptions โดยไม่สร้าง ID ใหม่

## 7. ลำดับงาน
1. กำหนดโครงสร้าง Offline/Online state และ Network Status Monitor เพื่อสลับแต่ละเครื่อง POS ตามสัญญาณอินเทอร์เน็ต (UC09 Main Flow 1, 6; ASM-07)
2. สร้าง Local Persistence Layer บนเครื่อง POS ด้วย SQLite หรือ IndexedDB และเก็บ TransactionLog/SyncQueueItem เพื่อรองรับธุรกรรมในโหมดออฟไลน์ (UC09 Main Flow 2-3; ASM-01, ASM-02)
3. จัดทำ Sync Queue และ Pending Alert ให้ตรวจสอบสถานะการอัปโหลดของแต่ละรายการพร้อมสัญลักษณ์เตือนและ retry policy ทุก 3 นาที (UC09 Alternative Flow 4a; ASM-06, ASM-08)
4. พัฒนา Background Sync Worker ที่ทำงานเมื่อกลับมาออนไลน์ เพื่อส่งเฉพาะข้อมูลค้างอยู่และบันทึกผล resync อย่างมีประสิทธิภาพ (UC09 Main Flow 4; ASM-04, ASM-08)
5. เพิ่ม Conflict Resolution ด้วย UUID + Timestamp เพื่อป้องกันการเขียนทับหรือเลขออร์เดอร์ชนกันระหว่างการ reconcile ข้อมูลกับคลาวด์ (UC09 Main Flow 4; ASM-03)
6. สร้าง API และ event stream สำหรับยอดขายรวมที่ส่งไปยังสมาร์ทโฟนของเจ้าของร้าน เพื่อให้อัปเดตตรงกัน 100% หลัง Sync สำเร็จ (UC09 Main Flow 5-6; ASM-05)
7. ทดสอบการเปลี่ยนโหมดออฟไลน์/ออนไลน์แบบ real network condition และตรวจว่ารายการค้างส่งสามารถส่งสำเร็จครบถ้วนโดยไม่ซ้ำเกินความจำเป็น (UC09 Main Flow 1-6, Alternative Flow 4a; ASM-04, ASM-06, ASM-08)
8. ตรวจสอบการอัปเดต UI และการแจ้งเตือนอีกครั้ง เพื่อยืนยันว่าแถบสถานะสีส้ม/เขียวและ icon pending sync แสดงผลถูกต้องแบบ end-to-end (UC09 Main Flow 1, 4-6; ASM-06, ASM-07)

## 8. สิ่งที่ยังไม่ทำ
ไม่มี Open Questions ใน spec หลังจาก Clarify แล้ว ขณะนี้ไม่มีข้อข้องใจที่ต้องรอคำตอบเพื่อเริ่มพัฒนา

ส่วนที่ยังไม่สร้างจนกว่าจะมีคำตอบเพิ่มเติม:
- รูปแบบ payload ของ Background Sync ที่จะส่งไปยังคลาวด์แบบจริง หากทีมต้องการให้มี contract ระหว่าง POS และ Cloud service ที่ชัดขึ้น
- ระดับความละเอียดของ alert dashboard สำหรับรายการค้างส่ง ถ้าทีมต้องการแยกแยะตามวัน/ร้าน/เครื่อง POS เพิ่มเติม
- กลไก replay / dead-letter handling สำหรับกรณี Cloud ไม่ตอบหรือมีข้อผิดพลาดทางเครือข่ายที่ยาวกว่า retry 3 นาที หากต้องการมีกฎการหยุด/ยกเลิก/บันทึกเฉพาะข้อความที่สำคัญ

ส่วนดังกล่าวยังไม่สร้างจนกว่าทีมจะให้ข้อมูลเพิ่มเติม เพราะ spec ปัจจุบันไม่ได้ระบุ technical contract ของ cloud sync หรือ fallback behavior แบบละเอียดมากพอสำหรับการพัฒนาใน production
