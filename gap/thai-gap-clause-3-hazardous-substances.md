# ข้อ 3 วัตถุอันตรายทางการเกษตรและการควบคุมการใช้

เอกสารนี้แปลงข้อกำหนด GAP พืชเรื่องวัตถุอันตรายทางการเกษตรให้เป็น contract เชิงปฏิบัติสำหรับ SmartFarm Phase 1

## ขอบเขตของ Phase 1

Phase 1 ต้องรองรับ 4 เรื่องที่ตามสอบได้จริง:

- ทะเบียนวัตถุอันตรายหรือสารเคมีที่ฟาร์มใช้
- บันทึกการใช้ในแปลงแบบ traceable ต่อรอบปลูก
- สถานะสต็อกพื้นฐานที่อธิบายที่มาของยอดคงเหลือได้
- หลักฐานฉลาก เอกสาร และการตรวจสภาพการเก็บรักษาที่ผู้เชี่ยวชาญใช้ทวนสอบได้

## สมมติฐาน

- ใช้เป็นข้อกำหนดกลางสำหรับ GAP พืชของไทยในประเด็นข้อ 3 โดยยังไม่ผูกกับพืชเฉพาะชนิด
- ระบบยังไม่ตัดสินความถูกต้องตามกฎหมายจากฐานข้อมูลภายนอกอัตโนมัติ; การอนุมัติรายการสารต้องมาจาก scheme data หรือการทวนสอบโดยผู้เชี่ยวชาญ
- Phase 1 รองรับสต็อกระดับพื้นฐาน ไม่ทำบัญชีต้นทุน ไม่ทำ lot trace ย้อนกลับระดับคลัง และไม่ทำ workflow ขออนุมัติการใช้ล่วงหน้า

## Record Types ที่ต้องมี

### 1. `hazardous_substance_product`

ทะเบียนสารหรือผลิตภัณฑ์ที่ใช้เป็น reference กลางขององค์กร

ฟิลด์บังคับ:

- `product_name_label`: ชื่อการค้าตามฉลากแบบ verbatim
- `registration_no`: เลขทะเบียนวัตถุอันตรายหรือเลขอ้างอิงตามเอกสาร
- `active_ingredients`: รายการสารออกฤทธิ์
- `formulation`: รูปแบบสาร เช่น EC, WG, SL ถ้ามีบนฉลาก
- `manufacturer_or_registrant`
- `label_version_date`: วันที่เอกสารหรือฉลากฉบับที่ใช้อ้างอิง ถ้าระบุไม่ได้ให้ใช้วันที่อัปโหลดเอกสาร
- `evidence_label_id`: หลักฐานฉลากหรือเอกสารอ้างอิงอย่างน้อย 1 รายการ

ฟิลด์แนะนำ:

- `hazard_class`
- `expiry_date`
- `restricted_crop_targets`
- `pre_harvest_interval_days`
- `reentry_interval_hours`

กติกา:

- เมื่อเลขทะเบียนหรือฉลากเปลี่ยน ต้องสร้าง product record ใหม่แทนการแก้ไขทับ
- use record ทุกฉบับต้องอ้างถึง product record เวอร์ชันที่ใช้จริงในวันปฏิบัติงาน

### 2. `hazardous_substance_use`

บันทึกการใช้สารในแปลง เป็น record หลักที่ auditor ต้องตามสอบได้

ฟิลด์บังคับ:

- `occurred_at`
- `plot_id`
- `crop_cycle_id`
- `operator_name` หรือ `operator_user_id`
- `product_record_id`
- `reason_for_use`: ศัตรูพืช โรค หรือวัชพืชเป้าหมาย
- `application_method`
- `dose_per_area`
- `dose_unit`
- `treated_area_value`
- `treated_area_unit`
- `total_product_amount`
- `total_product_unit`

ฟิลด์แนะนำ:

- `water_volume`
- `sprayer_or_equipment_id`
- `weather_note`
- `mixing_note`
- `withholding_period_override_reason`

หลักฐานขั้นต่ำ:

- อ้างอิง product record ที่มีฉลากหรือเอกสารกำกับครบ
- แนบภาพกิจกรรมหรือเอกสารภาคสนามอย่างน้อย 1 รายการเมื่อองค์กรหรือ scheme กำหนดว่าการใช้สารชนิดนั้นต้องมี evidence ระดับเหตุการณ์

### 3. `hazardous_substance_stock_event`

ใช้สร้างยอดคงเหลือแบบอธิบายได้ ไม่ใช่ระบบคลังเต็มรูปแบบ

ชนิดเหตุการณ์:

- `opening_balance`
- `purchase`
- `usage_deduction`
- `manual_adjustment`
- `disposal`

ฟิลด์บังคับ:

- `occurred_at`
- `product_record_id`
- `event_type`
- `quantity`
- `quantity_unit`

ฟิลด์ตามบริบท:

- `reference_record_id`: ต้องมีเมื่อ `event_type = usage_deduction` และต้องอ้างถึง `hazardous_substance_use`
- `document_no`: ใช้กับ `purchase` หรือ `disposal`
- `reason`: ใช้กับ `manual_adjustment` หรือ `disposal`
- `storage_location_id`

กติกา:

- ระบบต้องคำนวณ `current_on_hand` จาก stock events ไม่เก็บเป็นค่าที่แก้ตรงได้
- `usage_deduction` ควรสร้างตาม use record โดยอัตโนมัติเมื่อหน่วยวัดสอดคล้องกัน

### 4. `hazardous_substance_storage_check`

บันทึกการตรวจสภาพพื้นที่เก็บสารเพื่อรองรับการทวนสอบเบื้องต้น

ฟิลด์บังคับ:

- `occurred_at`
- `storage_location_name`
- `inspector_name` หรือ `inspector_user_id`
- `is_locked`
- `is_separated_from_food_seed_feed`
- `has_label_visibility`
- `has_spill_or_leak`
- `has_expired_product_present`

ฟิลด์แนะนำ:

- `corrective_action`
- `next_follow_up_at`

หลักฐานขั้นต่ำ:

- ภาพสถานที่เก็บอย่างน้อย 1 รายการต่อการตรวจแต่ละครั้ง

## Evidence Matrix

| Record type | หลักฐานบังคับ | จุดประสงค์ |
| --- | --- | --- |
| `hazardous_substance_product` | ฉลากสินค้า หรือเอกสารขึ้นทะเบียน | ยืนยันตัวตนของผลิตภัณฑ์และเงื่อนไขการใช้ |
| `hazardous_substance_use` | อ้างอิง product record ที่มีฉลากครบ; ภาพ/เอกสารภาคสนามเมื่อ scheme บังคับ | ยืนยันว่ามีการใช้จริงในแปลงและผูกกับสารที่ตรวจสอบได้ |
| `hazardous_substance_stock_event.purchase` | ใบเสร็จ ใบกำกับ หรือเอกสารรับเข้า ถ้ามี | อธิบายที่มาของสต็อก |
| `hazardous_substance_stock_event.disposal` | เอกสารหรือภาพการกำจัด ถ้ามี | อธิบายการตัดสต็อกที่ไม่ใช่การใช้ |
| `hazardous_substance_storage_check` | ภาพพื้นที่เก็บ | ทวนสอบการเก็บรักษาและการแยกพื้นที่ |

## Validation ที่ควรบังคับในระบบ

- `occurred_at` ของ use record ต้องไม่เร็วกว่าวันเริ่ม crop cycle และไม่อยู่ในอนาคตเกินเวลาคลาดเคลื่อนที่องค์กรกำหนด
- จำนวนใช้และพื้นที่ที่ใช้ต้องมากกว่า 0
- `product_record_id`, `plot_id`, `crop_cycle_id` ต้องอยู่ในองค์กรเดียวกัน
- ถ้า product record มี `expiry_date` และหมดอายุก่อน `occurred_at` ของการใช้ ให้ block การส่ง record เป็นสถานะพร้อมตรวจ
- ถ้า product record มี `pre_harvest_interval_days` และมี harvest event ภายในช่วงห้ามเก็บเกี่ยว ให้สร้าง finding ระดับ blocking
- ถ้า use record ไม่มี product record ที่มีหลักฐานฉลากครบ ให้ readiness เป็น `not_ready`
- stock event แบบ `usage_deduction` ต้องหักยอดติดลบไม่ได้; ถ้ายอดไม่พอให้เตือนผู้ใช้ทันทีและส่งให้ expert review เป็น risk
- manual adjustment ต้องใส่เหตุผลทุกครั้ง

## Workflow ขั้นต่ำ

1. Compliance lead หรือ expert สร้าง product record จากฉลากหรือเอกสารจริง
2. Farmer เลือก product record ที่มีอยู่ตอนบันทึกการใช้สารในแปลง
3. ระบบสร้าง stock deduction จาก use record หรือแจ้งให้ผู้ใช้ยืนยันถ้าหน่วยไม่ตรง
4. Farmer หรือ expert บันทึกการรับเข้า ปรับปรุง หรือกำจัดสต็อกเมื่อมีเหตุการณ์
5. Expert ตรวจ use records, ดู evidence, ทวนสอบ PHI และดู storage checks ล่าสุดก่อนสรุป readiness

## จุดเสี่ยงเชิงตีความที่ต้องล็อกให้ชัด

- รายการสารที่ "อนุญาต" ต้องมาจากแหล่งข้อมูลใดและใครเป็นผู้ดูแล revision ของ scheme
- จะถือ stock ติดลบเป็น hard block ทันทีหรือเป็น finding ที่รอ expert ตัดสิน
- การกำจัดภาชนะบรรจุใช้แล้วจะอยู่ในข้อ 3 นี้หรือแตกเป็นข้อกำหนดด้านสิ่งแวดล้อม/ของเสียต่างหาก
- ถ้าฉลากไม่มี PHI หรือ REI ระบบจะถือว่าไม่บังคับ หรือให้ expert ใส่ค่า override ใน scheme

## ข้อมูลที่ควรเก็บให้พร้อมต่อ AI ภายหลัง

- เก็บ `product_name_label` แบบ verbatim และมี normalized alias แยกอีกฟิลด์หนึ่ง
- แยกหน่วยวัดเป็น code มาตรฐาน ไม่เก็บรวมกับตัวเลขในข้อความเดียว
- เก็บ target pest/disease เป็นทั้งข้อความอิสระและ code ถ้า scheme มี dictionary
- ผูก evidence แต่ละชิ้นกับบทบาทเชิงความหมาย เช่น `label`, `field_application`, `purchase_doc`, `storage_photo`
- เก็บผล review และเหตุผลการ block ต่อ record แบบ structured เพื่อใช้เป็น training label ภายหลัง

## ขอบเขตที่ยังไม่ทำใน Phase 1

- ตรวจสอบเลขทะเบียนกับฐานข้อมูลภาครัฐแบบ real-time
- optimization สูตรผสมหรือคำแนะนำการใช้สาร
- barcode/QR scan ภาคสนามเป็นเงื่อนไขบังคับ
- inventory costing, FEFO, multi-warehouse workflow เต็มรูปแบบ
