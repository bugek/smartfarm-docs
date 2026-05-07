# Fertilizer Application Requirement (GAP Phase 1.2)

เอกสารนี้เป็น product/API handoff สำหรับระบบบันทึกการใส่ปุ๋ยใน SmartFarm GAP Phase 1.2 โดยแยก flow นี้ออกจาก Chemical use / Hazardous substance เพื่อให้ชาวสวนกรอกง่าย ผู้เชี่ยวชาญตรวจสอบได้ และ readiness report ย้อนกลับถึงแปลงกับรอบการผลิตได้ครบ

## Scope

Phase 1.2 ต้องรองรับ record type ใหม่:

- `fertilizer_application`

Record นี้ใช้เมื่อมีการใส่ปุ๋ยหรือปรับปรุงธาตุอาหารพืชในแปลง ไม่ว่าจะเป็นปุ๋ยเคมี ปุ๋ยอินทรีย์ ปุ๋ยชีวภาพ ปูนปรับสภาพดิน หรือวัสดุปรับปรุงดินที่องค์กรนับเป็นปัจจัยธาตุอาหาร

ไม่ใช้ record นี้กับ:

- การพ่นหรือใช้สารป้องกันกำจัดศัตรูพืช ให้ใช้ `hazardous_substance_use`
- ทะเบียนสารเคมีหรือวัตถุอันตราย ให้ใช้ hazardous substance product/stock records
- การซื้อสต็อกหรือบัญชีต้นทุนปุ๋ยเต็มรูปแบบ ยังไม่อยู่ใน Phase 1.2
- คำแนะนำอัตราปุ๋ยอัตโนมัติ ยังไม่อยู่ใน Phase 1.2

## Assumptions

- องค์กรมี farm site, plot, crop cycle และ role พื้นฐานตาม Phase 1 แล้ว
- การใส่ปุ๋ยเป็น GAP record ที่ผูกกับ `plot_id` และ `crop_cycle_id` เสมอ
- Phase 1.2 เก็บข้อมูลเพื่อ traceability และ review readiness ไม่ใช่ระบบคำนวณแผนธาตุอาหารหรือ inventory
- Record และ evidence เป็น append-only; การแก้ไขใช้ supersession ตาม contract หลัก

## Input Category Separation

UX และ API ต้องแยกประเภทปัจจัยการผลิตตั้งแต่ต้นทางเพื่อไม่ให้ farmer เลือกผิด flow

| Category | ความหมาย | ตัวอย่าง | Record/API หลัก |
| --- | --- | --- | --- |
| Fertilizer / nutrient input | ใช้เพิ่มธาตุอาหารหรือปรับปรุงดินเพื่อการเจริญเติบโตของพืช | ปุ๋ย 15-15-15, ยูเรีย 46-0-0, ปุ๋ยคอก, ปุ๋ยหมัก, น้ำหมัก, ปูนโดโลไมต์ | `fertilizer_application` |
| Chemical / pesticide use | ใช้ควบคุมศัตรูพืช โรค วัชพืช หรือเป็นสารออกฤทธิ์ตามฉลาก | ยาฆ่าแมลง, สารป้องกันกำจัดเชื้อรา, สารกำจัดวัชพืช | `hazardous_substance_use` |
| Hazardous substance product | ทะเบียนกลางของวัตถุอันตรายหรือสารเคมีที่ต้องอ้างอิงฉลาก/เลขทะเบียน | product record ของสารกำจัดศัตรูพืช | `hazardous_substance_product` |
| Other farm input | ปัจจัยอื่นที่ไม่ใช่ธาตุอาหารและไม่ใช่วัตถุอันตราย | เมล็ดพันธุ์, วัสดุคลุมดิน, อุปกรณ์ | ยังไม่สร้าง record เฉพาะใน Phase 1.2 เว้นแต่ scheme กำหนด |

Rule สำคัญ: `fertilizer_application` อาจเก็บชื่อผลิตภัณฑ์และสูตรปุ๋ยได้ แต่ไม่ต้องอ้าง `hazardous_substance_product_id` เป็นค่าเริ่มต้น ยกเว้นองค์กรหรือ scheme ระบุว่าปุ๋ย/วัสดุบางชนิดต้องถูกควบคุมเป็นวัตถุอันตรายด้วย

## Minimum Data Contract

ทุก `fertilizer_application` ต้องมีข้อมูลขั้นต่ำต่อไปนี้

| Field | Required | Notes |
| --- | --- | --- |
| `org_id` | yes | จาก session/org context |
| `site_id` | yes | derive จาก plot ได้ แต่ API response ควร expose เพื่อรายงาน |
| `plot_id` | yes | แปลงที่ใส่ปุ๋ย |
| `crop_cycle_id` | yes | รอบการผลิตที่ record นี้นับเข้า readiness |
| `occurred_at` | yes | วันที่/เวลาที่ใส่ปุ๋ยจริง |
| `fertilizer_name` | yes | ชื่อผลิตภัณฑ์หรือชื่อวัสดุที่ชาวสวนรู้จัก |
| `fertilizer_type` | yes | enum: `chemical`, `organic`, `biofertilizer`, `soil_amendment`, `other` |
| `npk` | conditional | object `{ n, p, k }`; ต้องมีเมื่อทราบสูตร เช่น 15-15-15; nullable เมื่อไม่มีสูตรบนวัสดุ |
| `quantity_value` | yes | จำนวนที่ใช้ มากกว่า 0 |
| `quantity_unit` | yes | เช่น `kg`, `g`, `l`, `ml`, `bag`, `sack`, `ton` |
| `application_method` | yes | enum หรือ scheme value เช่น หว่าน, โรยโคน, ใส่ร่อง, ละลายน้ำรด, fertigation, พ่นทางใบ |
| `treated_area_value` | yes | พื้นที่ที่ใส่จริง มากกว่า 0 |
| `treated_area_unit` | yes | เช่น `rai`, `sqm`, `ha` |
| `operator_name` or `operator_user_id` | yes | ผู้ปฏิบัติงาน |
| `reason_or_growth_stage` | yes | เหตุผลหรือระยะพืช เช่น รองพื้น, หลังย้ายปลูก, เร่งใบ, ติดผล |
| `notes` | no | free text |
| `evidence_ids` | no at submit, required for readiness when scheme requires | image/video/document attached at record level |
| `supersedes_record_id` | no | ใช้เมื่อแก้ไข record เดิม |

Recommended fields for later reporting:

- `brand_name`
- `formula_label_text` เช่น `15-15-15`, `46-0-0`, `ปุ๋ยคอก`
- `source_or_supplier`
- `lot_no`
- `water_volume_value` และ `water_volume_unit` เมื่อผสมน้ำ
- `equipment_name`
- `weather_note`
- `rate_value`, `rate_unit`, `rate_basis` เมื่อผู้ใช้ทราบอัตราต่อพื้นที่

## Evidence Requirement

Phase 1.2 ต้องรองรับ evidence ชนิด `image`, `video`, และ `document` ตาม evidence strategy หลัก

Evidence ที่อนุญาต:

- รูปถุง/ฉลาก/ภาชนะปุ๋ย
- รูปการใส่ปุ๋ยในแปลงหรือภาพหลังใส่
- วิดีโอสั้นแสดงกิจกรรมการใส่ในแปลง
- เอกสาร เช่น ใบซื้อ ใบรับรองปุ๋ยอินทรีย์ หรือบันทึกภาคสนามที่ scan/upload

Readiness rule:

- ระบบต้องยอมให้ farmer submit record ได้แม้ยังไม่มี evidence
- ถ้า scheme กำหนด evidence สำหรับ `fertilizer_application` แต่ record ยังไม่มี evidence ชนิดที่ต้องการ readiness item นั้นต้องไม่เป็น `ready`
- เฉพาะ record-level evidence เท่านั้นที่นับเข้า readiness
- Cycle-level evidence แสดงในรายงานเป็น context แต่ไม่แทน proof ของการใส่ปุ๋ยรายการนั้น

## Workflow

### Farmer

1. เปิด crop cycle ที่กำลังทำงาน
2. เลือกเมนู "บันทึกการใส่ปุ๋ย" แยกจากเมนู "บันทึกการใช้สารเคมี"
3. เลือกแปลงและระบบผูก crop cycle ให้อัตโนมัติเมื่ออยู่ใน cycle context
4. กรอกวันที่ ชื่อ/ชนิดปุ๋ย สูตร N-P-K ถ้ามี ปริมาณ หน่วย วิธีใส่ พื้นที่ ผู้ใส่ และเหตุผล/ระยะพืช
5. แนบรูป/วิดีโอ/เอกสารได้ทันที หรือส่ง record ก่อนแล้วตามแนบ evidence ภายหลัง
6. Submit แล้ว record ถูกเก็บเป็น current record หรือ superseding record ตามที่ผู้ใช้เลือก

### Expert

1. เปิด crop cycle และดู history การใส่ปุ๋ยตามแปลง/รอบการผลิต
2. ตรวจ structured fields, evidence, และความสอดคล้องของปริมาณ/พื้นที่/ระยะพืช
3. ตั้ง review state เป็น `reviewed`, `needs_more_evidence`, หรือ `blocking`
4. ใส่ comment ให้ farmer เห็นเมื่อหลักฐานไม่พอ ข้อมูลขัดแย้ง หรือพบความเสี่ยงด้าน GAP

### Readiness Report

รายงานต้องแสดง:

- รายการใส่ปุ๋ยตามลำดับเวลาใน crop cycle
- แปลง วันที่ ชื่อปุ๋ย สูตร ปริมาณ วิธีใส่ พื้นที่ ผู้ปฏิบัติงาน และเหตุผล/ระยะพืช
- evidence ที่ผูกกับแต่ละ record
- review ล่าสุดของ expert ต่อ current record
- superseded history เมื่อมีการแก้ไข

## Validation

API และ UI ต้องบังคับอย่างน้อย:

- `plot_id` และ `crop_cycle_id` ต้องอยู่ใน org เดียวกัน
- `occurred_at` ต้องอยู่ในช่วง crop cycle หรือถูก flag ให้ expert review ถ้าอยู่นอกช่วง
- `quantity_value` และ `treated_area_value` ต้องมากกว่า 0
- `npk.n`, `npk.p`, `npk.k` ต้องเป็นตัวเลขไม่ติดลบเมื่อกรอก
- ถ้า `fertilizer_type = chemical` และชื่อ/ฉลากเข้าข่ายสารกำจัดศัตรูพืช UI ต้องเตือนให้ใช้ chemical flow แทน
- Record ที่ supersede ต้องอ้าง record เดิมใน crop cycle เดียวกัน
- Evidence ที่นับ readiness ต้องมี `record_id` ของ fertilizer application record นั้น

## Acceptance Criteria (Thai)

### สำหรับชาวสวน

- ชาวสวนเห็นเมนูบันทึกการใส่ปุ๋ยเป็น flow แยกจากการใช้สารเคมีอย่างชัดเจน
- ชาวสวนสามารถบันทึกแปลง รอบการผลิต วันที่ ชนิด/สูตรปุ๋ย N-P-K ถ้ามี ปริมาณ หน่วย วิธีใส่ พื้นที่ ผู้ปฏิบัติงาน เหตุผล/ระยะพืช หมายเหตุ และหลักฐานได้
- ชาวสวนสามารถส่ง record ได้แม้ยังไม่ได้แนบหลักฐานครบ และเห็นสถานะว่าหลักฐานยังไม่พอสำหรับ readiness
- ชาวสวนสามารถเปิด history การใส่ปุ๋ยของแปลงในรอบการผลิตนั้นได้
- ถ้ากรอกข้อมูลผิด ชาวสวนต้องสร้าง record แก้ไขที่อ้างถึง record เดิม ไม่แก้ทับประวัติเดิม

### สำหรับผู้เชี่ยวชาญ

- ผู้เชี่ยวชาญเห็นรายการใส่ปุ๋ยแยกจากรายการใช้สารเคมีใน crop cycle เดียวกัน
- ผู้เชี่ยวชาญตรวจข้อมูล structured fields และหลักฐานรูป/วิดีโอ/เอกสารของแต่ละ record ได้
- ผู้เชี่ยวชาญตั้งสถานะ review เป็น `reviewed`, `needs_more_evidence`, หรือ `blocking` ได้ โดย review ผูกกับ record version นั้น
- ถ้าหลักฐานไม่ครบ readiness ของรายการใส่ปุ๋ยต้องไม่เป็น `ready` จนกว่าจะมีหลักฐานที่ scheme ต้องการและผ่าน review
- Readiness report ต้องแสดง current record, evidence, review ล่าสุด และ history ที่ถูก supersede เพื่อให้ตรวจสอบย้อนหลังได้

## Implementation Handoff

Tech Lead:

- เพิ่ม record type `fertilizer_application` ใน scheme/record schema layer
- ใช้ `GapRecord` append-only model เดิม ไม่สร้างตารางแก้ทับประวัติ
- เพิ่ม validation ตาม section นี้ และ expose list/filter by `plot_id`, `crop_cycle_id`, and record type
- ทำให้ readiness engine นับ required evidence และ expert review ของ record type นี้เหมือน record type อื่น

Platform Engineer:

- เพิ่ม farmer UI flow "บันทึกการใส่ปุ๋ย" ที่ไม่ปนกับ chemical use
- รองรับ mobile browser evidence capture สำหรับ image/video/document
- เพิ่ม crop-cycle history view filtered to fertilizer applications
- เพิ่ม expert review surface และ readiness report rendering สำหรับ fields ของ record type นี้

Contract of record:

- Product/API requirement: `gap/fertilizer-application-phase-1-2.md`
- Shared evidence rules: `gap/evidence-strategy.md`
- Shared Phase 1 data model: `architecture/implementation-handoff.md`
