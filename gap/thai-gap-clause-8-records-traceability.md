# ข้อ 8 เอกสาร บันทึกข้อมูล และการตามสอบ

เอกสารนี้แปลงข้อกำหนด GAP พืชเรื่องเอกสาร บันทึกข้อมูล และการตามสอบให้เป็น product contract สำหรับ SmartFarm Phase 1 โดยเน้นสิ่งที่เกษตรกรบันทึกได้จริง ผู้เชี่ยวชาญตรวจได้จริง และพร้อมใช้ใน readiness report

## ขอบเขตของเอกสารนี้

เอกสารนี้ครอบคลุมงาน 5 กลุ่มที่ข้อ 8 ต้องตามสอบได้:

- การเก็บเอกสารและบันทึกข้อมูลให้ครบตามฤดูกาลผลิต
- การผูกเอกสารสนับสนุนจากข้อกำหนดอื่นเข้ากับรอบการผลิต
- การสร้างรุ่นผลิตผลหรือ lot ที่ตามกลับถึงแปลงและวันเก็บเกี่ยวได้
- การบันทึกผู้รับซื้อ ปริมาณที่ขาย และการแยกกักเมื่อพบปัญหา
- การทบทวนวิธีปฏิบัติและการจัดการข้อร้องเรียน

## สมมติฐาน

- ใช้ข้อ 8 ของ มกษ. 9001-2556 เป็นฐานหลัก และใช้คำอธิบายตัวอย่างเอกสารจากแนวปฏิบัติ มกษ. 9001(G)-2564 เป็นตัวช่วยตีความรายการเอกสารที่ควรมี
- ใน SmartFarm Phase 1 คำว่า "ฤดูกาลผลิต" ให้ผูกกับ `crop_cycle` เป็นหลัก เพื่อให้แยกเอกสารและบันทึกตามรอบการผลิตได้ชัด
- ระบบต้องตามสอบได้ระดับ `crop_cycle` -> `harvest_lot` -> `sale_dispatch` -> `buyer` แต่ยังไม่ทำ serial trace ระดับกล่องย่อยหรือ pallet
- เอกสารสำคัญจากข้อกำหนดอื่น เช่น ผลวิเคราะห์น้ำ/ดิน ใบเสร็จปัจจัยการผลิต และประวัติการฝึกอบรม อาจเก็บเป็น document record หรือเป็น evidence ที่ผูกกับรอบการผลิต โดยต้องค้นคืนได้จาก readiness report

## Mapping กับข้อกำหนดข้อ 8

| ข้อกำหนด GAP | ความหมายใน SmartFarm |
| --- | --- |
| 8.12 / 3.8.1 เอกสารและบันทึกต้องเป็นปัจจุบัน ครบถ้วน และลงชื่อผู้ปฏิบัติงาน | ทุก record ใช้ `created_by`, `created_at` และเก็บใน `crop_cycle` ที่เกี่ยวข้อง |
| 8.13 / 3.8.2 จัดเก็บเอกสารเป็นหมวดหมู่แยกตามฤดูกาลผลิต | ใช้ `season document register` เป็นทะเบียนกลางของ `crop_cycle` |
| 8.14 / 3.8.3 ระบุรุ่นผลิตผล/แหล่งผลิต/วันเก็บเกี่ยวให้ตามสอบได้ | ใช้ `harvest_lot` ผูก `plot_id`, `crop_cycle_id`, `harvest_date`, `lot_code` |
| 8.15 / 3.8.4 บันทึกผู้รับซื้อและปริมาณที่จำหน่าย | ใช้ `sale_dispatch` ผูก buyer, quantity, และ `lot_ids` |
| 8.16 / 3.8.5 เก็บเอกสารเพื่อการตามสอบและเรียกคืนได้ | เก็บแบบ append-only และกำหนด retention ขั้นต่ำตามรอบผลิต |
| 8.17-8.18 / 3.8.6-3.8.7 แยกผลิตผลเมื่อมีปัญหา แจ้งผู้ซื้อ และบันทึกสาเหตุ/แก้ไข | ใช้ `traceability_incident` และ flag lot ที่ได้รับผลกระทบ |
| 8.19 / 3.8.8 ทบทวนวิธีปฏิบัติอย่างน้อยปีละ 1 ครั้ง | ใช้ `practice_review` |
| 8.20 / 3.8.9 แก้ไขข้อร้องเรียนและเก็บบันทึก | ใช้ `complaint_case` |

## สิ่งที่ Phase 1 ต้องรองรับ

### 1. season document register

ทะเบียนเอกสารกลางของรอบการผลิต ใช้รวบรวมเอกสารและบันทึกที่ผู้ตรวจต้องเห็นว่ามีครบในฤดูกาลนั้น

ฟิลด์บังคับ:

- `crop_cycle_id`
- `document_type`
- `document_date`
- `owner_role` (`farmer`, `expert`, `compliance_lead`, `external`)
- `summary`
- `evidence_ids`
- `created_by`
- `created_at`

ฟิลด์แนะนำ:

- `related_record_type`
- `issuer_name`
- `document_no`
- `valid_from`
- `valid_until`
- `note`

ชนิด `document_type` ที่ควรมีใน Phase 1:

- `water_or_soil_test_result`
- `input_purchase_document`
- `training_record`
- `health_check_record`
- `other_gap_supporting_document`

กติกา:

- ทุกเอกสารในทะเบียนต้องผูกกับ `crop_cycle_id` เสมอ แม้ต้นทางจะเป็นเอกสารระดับองค์กร
- readiness report ต้องดึงทะเบียนนี้ไปแสดงเป็น appendix แยกตาม `document_type`
- ผู้ใช้ห้ามอัปเดตทับเอกสารเดิม; ถ้าอัปโหลดฉบับใหม่ให้ supersede ฉบับเดิม

### 2. `harvest_lot`

รุ่นผลิตผลหรือ lot ขั้นต่ำที่ใช้ตามสอบย้อนกลับไปยังแปลง วันเก็บเกี่ยว และรอบการผลิต

ฟิลด์บังคับ:

- `crop_cycle_id`
- `plot_id`
- `lot_code`
- `harvest_date`
- `produce_name`
- `quantity`
- `quantity_unit`
- `status` (`open`, `in_storage`, `dispatched`, `on_hold`, `disposed`)
- `created_by`

ฟิลด์แนะนำ:

- `harvest_shift`
- `grade`
- `storage_location`
- `pack_date`
- `note`

กติกา:

- `lot_code` ต้อง unique ภายในองค์กร
- 1 lot ต้องอ้างกลับได้อย่างน้อยถึง `plot_id`, `crop_cycle_id`, และ `harvest_date`
- ถ้า lot ถูกแยกขายหลายครั้ง ให้ยอดคงเหลือคำนวณจาก `sale_dispatch` แบบ append-only ไม่แก้ค่าทับที่ lot

### 3. `sale_dispatch`

บันทึกการจำหน่ายหรือส่งมอบผลิตผลตามข้อกำหนดเรื่องผู้รับซื้อและปริมาณที่จำหน่าย

ฟิลด์บังคับ:

- `crop_cycle_id`
- `dispatch_date`
- `buyer_name`
- `buyer_type` (`collector`, `market`, `packer`, `retailer`, `other`)
- `quantity`
- `quantity_unit`
- `lot_ids`
- `created_by`

ฟิลด์แนะนำ:

- `buyer_contact`
- `destination_name`
- `delivery_document_no`
- `vehicle_plate`
- `price_note`
- `evidence_ids`

กติกา:

- ต้องเลือก `lot_ids` อย่างน้อย 1 ค่าเสมอ
- 1 dispatch อ้าง lot ได้หลาย lot และ 1 lot ถูกอ้างได้หลาย dispatch จนกว่ายอดจะหมด
- readiness report ต้องแสดงเส้นทาง `crop_cycle -> lot -> buyer` แบบอ่านง่าย

### 4. `traceability_incident`

เคสปัญหาความปลอดภัยหรือคุณภาพที่ต้องแยกกักผลิตผล แจ้งผู้ซื้อ และบันทึกสาเหตุพร้อมแนวทางแก้ไข

ฟิลด์บังคับ:

- `crop_cycle_id`
- `detected_at`
- `issue_type` (`safety`, `quality`, `traceability_gap`, `other`)
- `issue_summary`
- `affected_lot_ids`
- `containment_action`
- `buyer_notified` (`yes`, `no`, `not_applicable`)
- `root_cause_note`
- `corrective_action`
- `created_by`

ฟิลด์แนะนำ:

- `buyer_notification_at`
- `buyer_notification_method`
- `resolution_due_at`
- `closed_at`
- `evidence_ids`

กติกา:

- ถ้า `buyer_notified = yes` ต้องเก็บเวลาแจ้งหรือหลักฐานการแจ้งอย่างน้อย 1 รายการ
- lot ที่อยู่ใน incident ต้องถูกมองเห็นเป็น `on_hold` หรือมี flag เตือนในหน้าตามสอบ
- expert หรือ compliance lead ต้อง review incident ก่อน readiness กลับเป็นปกติ

### 5. `practice_review`

บันทึกการทบทวนวิธีปฏิบัติอย่างน้อยปีละ 1 ครั้ง หรือหลังเกิดปัญหาสำคัญ

ฟิลด์บังคับ:

- `organization_id`
- `review_date`
- `review_scope`
- `reviewer_name`
- `findings_summary`
- `improvement_actions`

ฟิลด์แนะนำ:

- `related_crop_cycle_ids`
- `next_review_due_at`
- `evidence_ids`

กติกา:

- Phase 1 รองรับอย่างน้อยระดับองค์กร ไม่จำเป็นต้องทำ workflow อนุมัติหลายขั้น
- readiness report ระดับ crop cycle ควร link ไปยัง practice review ล่าสุดขององค์กรได้

### 6. `complaint_case`

บันทึกข้อร้องเรียนที่เกี่ยวข้องกับความปลอดภัย คุณภาพ หรือการตามสอบของผลิตผล

ฟิลด์บังคับ:

- `received_at`
- `complainant_name`
- `complaint_channel`
- `complaint_summary`
- `related_lot_ids`
- `investigation_note`
- `resolution_note`
- `status` (`open`, `investigating`, `resolved`, `closed`)

ฟิลด์แนะนำ:

- `related_buyer_name`
- `crop_cycle_id`
- `closed_at`
- `owner_user_id`
- `evidence_ids`

กติกา:

- ถ้าผูกกับ lot ได้ ต้องผูก `related_lot_ids` เพื่อให้ trace ย้อนกลับได้ทันที
- complaint ที่ยังไม่ `resolved` หรือ `closed` ควรขึ้นเป็น risk ในมุม compliance lead

## ความเชื่อมโยงหลักที่ระบบต้องรักษา

### Buyer, lot, cycle

| สิ่งที่ต้องเชื่อม | ฟิลด์ขั้นต่ำ | เหตุผลทาง GAP |
| --- | --- | --- |
| เอกสารต่อรอบการผลิต | `crop_cycle_id` | แยกเอกสารและบันทึกตามฤดูกาลผลิต |
| lot ต่อแปลงและวันเก็บเกี่ยว | `plot_id`, `crop_cycle_id`, `harvest_date`, `lot_code` | ตามสอบที่มาของผลิตผล |
| การขายต่อ lot | `sale_dispatch.lot_ids` | รู้ว่าผลิตผล lot ไหนถูกส่งให้ใคร |
| การขายต่อผู้รับซื้อ | `buyer_name`, `dispatch_date`, `quantity` | รองรับข้อกำหนดเรื่องผู้รับซื้อและปริมาณจำหน่าย |
| incident ต่อ lot และ buyer | `affected_lot_ids`, `buyer_notified` | แยกกัก แจ้งผู้ซื้อ และติดตามการแก้ไข |
| complaint ต่อ lot/cycle | `related_lot_ids`, `crop_cycle_id` | สืบย้อนกลับและทบทวนกระบวนการได้ |

## สิ่งที่ readiness report ต้องแสดง

- รายการเอกสารสนับสนุนของรอบการผลิต แยกตามชนิดเอกสาร
- ตาราง lot ทั้งหมดของรอบการผลิต พร้อมวันเก็บเกี่ยว ปริมาณ และสถานะ
- ตารางการขายหรือส่งมอบที่ระบุผู้รับซื้อ ปริมาณ และ lot ที่เกี่ยวข้อง
- incident และ complaint ที่เกี่ยวข้องกับ lot หรือ buyer ในรอบการผลิตนั้น
- ลิงก์ไปยัง practice review ล่าสุดที่ครอบคลุมรอบการผลิตหรือองค์กรเดียวกัน

## Now กับ Next

### Now (ต้องมีใน Phase 1)

- season document register สำหรับเก็บเอกสารสนับสนุนที่ค้นคืนได้ตามรอบการผลิต
- harvest lot ขั้นต่ำที่ย้อนกลับถึงแปลง วันเก็บเกี่ยว และปริมาณได้
- sale dispatch log ที่ผูก buyer กับ lot และ quantity ได้
- incident log สำหรับการแยกกัก แจ้งผู้ซื้อ และบันทึกสาเหตุ/การแก้ไข
- practice review และ complaint log แบบเรียบง่ายแต่ query ได้
- การเก็บเอกสารและบันทึกแบบ append-only พร้อม actor และเวลา

### Next (ยังไม่ต้องทำใน Phase 1)

- QR หรือ barcode label สำหรับ lot
- recall workflow แบบหลายขั้นพร้อมสถานะย่อย
- buyer master, contract pricing, invoice reconciliation
- warehouse movement แบบละเอียดหรือ serial trace ระดับกล่อง
- complaint SLA, escalation automation, และ analytics dashboard เต็มรูปแบบ
- retention policy automation เกินกว่าการเตือนหรือแสดงอายุเอกสาร

## ข้อกำหนดเชิงระบบที่ห้ามตก

- ทุก record และ document ต้องมี `created_by`, `created_at`, และ context ขององค์กร
- เอกสารและบันทึกต้องค้นได้ทั้งจากมุม `crop_cycle`, `lot`, และ `buyer`
- ระบบต้องเก็บย้อนหลังอย่างน้อย 2 ฤดูกาลผลิตต่อเนื่อง หรือกำหนด retention ที่ยาวกว่าได้ตามองค์กร
- การ export readiness report ต้องไม่ต้องประกอบข้อมูลเองนอกระบบ
- ถ้าพบ lot ที่อยู่ใน incident หรือ complaint เปิดอยู่ ระบบต้องแสดง flag ชัดในหน้าตามสอบ

## ขอบเขตที่ยังไม่ทำในเอกสารนี้

- รายละเอียด schema ของเอกสารเฉพาะพืชหรือ commodity-specific overlay
- แบบฟอร์มการรับรองภาครัฐนอกระบบปฏิบัติการของฟาร์ม
- การออกแบบหน้าจอ, สิทธิ์ย่อยระดับ field, และรายละเอียด API endpoint
