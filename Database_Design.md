# โครงสร้างฐานข้อมูล ระบบจัดการอู่ซ่อมรถ

22 ตาราง แบ่งเป็น 6 หมวด ออกแบบสำหรับอู่สาขาเดียว จึงไม่มีการแยกข้อมูลตามสาขา

อ้างอิงกระบวนการทำงานที่ `Scenario.md` และผังความสัมพันธ์ที่ `diagrams/er-database.html`

---

## 1. ข้อมูลหลัก

### `Vehicle_Brands` — ยี่ห้อรถ

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `brand_id` | `INT` | **PK** |  |
| `brand_name` | `VARCHAR(50)` | UNIQUE | Toyota, Honda, Isuzu, Yamaha |
| `is_active` | `BOOLEAN` |  | ปิดการใช้งานได้โดยไม่ต้องลบ |

### `Vehicle_Models` — รุ่นรถ

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `model_id` | `INT` | **PK** |  |
| `brand_id` | `INT` | FK → `Vehicle_Brands` |  |
| `model_name` | `VARCHAR(80)` |  | Revo, Civic, Wave110i, PCX |
| `vehicle_type` | `ENUM('Car','Motorcycle')` |  | ประเภทรถเก็บที่ระดับรุ่น ไม่เก็บซ้ำที่ตัวรถ |
| `is_active` | `BOOLEAN` |  |  |

ประเภทรถเก็บไว้ที่ระดับรุ่น ไม่ได้เก็บซ้ำที่ตัวรถของลูกค้า เพราะรุ่นหนึ่งเป็นได้แค่ประเภทเดียว ถ้าเก็บสองที่แล้วแก้ไม่พร้อมกันจะขัดกันเอง

### `Products` — อะไหล่และค่าแรง

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `product_id` | `INT` | **PK** |  |
| `code` | `VARCHAR(30)` | UNIQUE | รหัสสินค้า / SKU |
| `name` | `VARCHAR(150)` |  | ชื่อรายการ |
| `type` | `ENUM('Part','Labor')` |  | แยกอะไหล่กับค่าแรงด้วยฟิลด์นี้ |
| `selling_price` | `DECIMAL(10,2)` |  | ราคาขายมาตรฐาน เป็นราคาก่อน VAT |
| `reorder_point` | `INT` |  | จุดสั่งซื้อ ใช้แจ้งเตือนของใกล้หมด |
| `lifespan_months` | `INT` | NULL ได้ | รอบเปลี่ยนถัดไป นับเป็นเดือน เช่น 6 คือหกเดือน 24 คือสองปี ว่างได้ถ้าอะไหล่ชิ้นนั้นไม่มีรอบเปลี่ยน |
| `is_active` | `BOOLEAN` |  |  |

ค่าแรงเก็บในตารางเดียวกับอะไหล่ แยกด้วย `type` เพราะทั้งสองอย่างขึ้นบิลเหมือนกัน ต่างกันแค่หมวดภาษีและไม่มีสต็อก

### `Employees` — พนักงานและช่าง

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `employee_id` | `INT` | **PK** |  |
| `name` | `VARCHAR(100)` |  |  |
| `role` | `ENUM('Admin','Employee','Mechanic')` |  | Admin เจ้าของอู่ / Employee พนักงานหน้าร้าน / Mechanic ช่าง |
| `username` | `VARCHAR(50)` | UNIQUE | ใช้ล็อกอิน มีครบทุก role |
| `password_hash` | `VARCHAR(255)` |  | เก็บเป็น hash เท่านั้น |
| `is_active` | `BOOLEAN` |  |  |

### `Suppliers` — ร้านขายอะไหล่

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `supplier_id` | `INT` | **PK** |  |
| `name` | `VARCHAR(150)` |  |  |
| `contact_phone` | `VARCHAR(20)` |  |  |
| `is_active` | `BOOLEAN` |  |  |

### `Shop_Settings` — ตั้งค่าอู่

ตารางนี้มีแถวเดียวในระบบ ใช้เก็บข้อมูลที่ต้องปรับได้โดยไม่ต้องแก้โค้ด

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `setting_id` | `INT` | **PK** |  |
| `shop_name` | `VARCHAR(150)` |  | ใช้เป็นหัวกระดาษของเอกสารทุกใบ |
| `shop_address` | `VARCHAR(255)` |  |  |
| `shop_phone` | `VARCHAR(20)` |  |  |
| `tax_id` | `CHAR(13)` |  | เลขประจำตัวผู้เสียภาษีของอู่ |
| `is_vat_registered` | `BOOLEAN` |  | อู่จดทะเบียน VAT แล้วหรือยัง |
| `vat_rate` | `DECIMAL(5,2)` |  | ค่าเริ่มต้น 7.00 |
| `wht_rate` | `DECIMAL(5,2)` |  | ค่าเริ่มต้น 3.00 |

เมื่อ `is_vat_registered` เป็นเท็จ ระบบจะไม่คิด VAT บังคับ `vat_amount` เป็นศูนย์ และเปลี่ยนหัวเอกสารจากใบกำกับภาษีเป็นใบเสร็จรับเงินธรรมดา

---

## 2. ลูกค้าและรถ

### `Customers`

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `customer_id` | `INT` | **PK** |  |
| `phone` | `VARCHAR(20)` | UNIQUE | ตัวค้นหลัก แก้ไขได้เมื่อลูกค้าเปลี่ยนเบอร์ |
| `name` | `VARCHAR(100)` |  |  |
| `is_corporate` | `BOOLEAN` |  | นิติบุคคลหรือไม่ ใช้ตัดสินว่าต้องหักภาษี ณ ที่จ่ายไหม |
| `tax_id` | `CHAR(13)` | NULL ได้ | เลขผู้เสียภาษี กรณีเป็นนิติบุคคล |
| `created_at` | `DATETIME` |  |  |

### `Vehicles`

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `vehicle_id` | `INT` | **PK** |  |
| `customer_id` | `INT` | FK → `Customers` | เปลี่ยนได้เมื่อรถเปลี่ยนเจ้าของ |
| `model_id` | `INT` | FK → `Vehicle_Models` |  |
| `license_plate` | `VARCHAR(20)` |  | ทะเบียนรถ |
| `year` | `SMALLINT` |  | ปีรถ |

---

## 3. งานบริการ

### `Jobs` — ใบงานซ่อม

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `job_id` | `INT` | **PK** |  |
| `job_number` | `VARCHAR(20)` | UNIQUE | รูปแบบ `JOB-YYMM-0001` รันเลขใหม่ทุกต้นเดือน |
| `vehicle_id` | `INT` | FK → `Vehicles` |  |
| `mileage_in` | `INT` |  | เลขไมล์ตอนรับรถ |
| `symptom_note` | `TEXT` |  | อาการเสียเบื้องต้น |
| `status` | `ENUM('Queue','Fixing','Waiting_Part','Done','Closed','Cancelled')` |  | `Done` คือซ่อมเสร็จรอส่งมอบ `Closed` คือเก็บเงินและส่งมอบแล้ว |
| `created_at` | `DATETIME` |  |  |
| `closed_at` | `DATETIME` | NULL ได้ |  |

`Done` คือซ่อมเสร็จรอส่งมอบ `Closed` คือเก็บเงินและส่งมอบแล้ว ต้องแยกกันเพราะแดชบอร์ดนับสองอย่างนี้คนละช่อง

รถหนึ่งคันมีใบงานที่ยังไม่ปิดได้ครั้งละหนึ่งใบ

```sql
CREATE UNIQUE INDEX uq_vehicle_open_job ON Jobs (vehicle_id)
  WHERE status NOT IN ('Closed', 'Cancelled');
```

MySQL ไม่รองรับ partial index ให้ย้ายไปเช็คในชั้น application แทน

### `Job_Mechanics` — ช่างที่รับผิดชอบใบงาน

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `job_mechanic_id` | `INT` | **PK** |  |
| `job_id` | `INT` | FK → `Jobs` |  |
| `employee_id` | `INT` | FK → `Employees` |  |
| `is_lead` | `BOOLEAN` |  | ช่างหลักของใบงาน มีได้คนเดียว |
| `labor_share` | `DECIMAL(5,2)` |  | สัดส่วนค่าแรงที่รับผิดชอบ หน่วยเป็นเปอร์เซ็นต์ |

ตอนบันทึกต้องตรวจว่าผลรวม `labor_share` ของใบงานเท่ากับ 100 และมี `is_lead` เป็นจริงแค่แถวเดียว

### `Quotations` — ใบเสนอราคา

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `quotation_id` | `INT` | **PK** |  |
| `job_id` | `INT` | FK → `Jobs` |  |
| `version` | `INT` |  | v1, v2, ... |
| `status` | `ENUM('Draft','Approved','Rejected')` |  |  |
| `approved_by` | `INT` | FK → `Employees` NULL ได้ | ว่างได้ถ้ายังไม่อนุมัติ |
| `approved_at` | `DATETIME` | NULL ได้ |  |
| `created_at` | `DATETIME` |  |  |

ยอดรวมและ VAT ไม่ได้เก็บไว้ในตารางนี้ แต่คำนวณสดจาก `Quotation_Items` ตอนสร้าง PDF เพราะราคาต่อหน่วยถูกล็อกไว้ในรายการอยู่แล้ว เก็บยอดรวมซ้ำอีกที่มีแต่จะเสี่ยงตัวเลขไม่ตรงกัน

### `Quotation_Items` — รายการในใบเสนอราคา

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `quote_item_id` | `INT` | **PK** |  |
| `quotation_id` | `INT` | FK → `Quotations` |  |
| `product_id` | `INT` | FK → `Products` |  |
| `qty` | `DECIMAL(10,2)` |  |  |
| `unit_price` | `DECIMAL(10,2)` |  | ราคาต่อหน่วย ณ วันเสนอ ก่อน VAT |

`unit_price` เป็นสำเนาของราคา ณ ตอนนั้น ไม่ใช่การอ้างอิงไปที่ `Products.selling_price` ราคากลางจะเปลี่ยนทีหลังก็ไม่กระทบใบที่เสนอไปแล้ว

### `Job_Items` — รายการที่ใช้จริง

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `job_item_id` | `INT` | **PK** |  |
| `job_id` | `INT` | FK → `Jobs` |  |
| `product_id` | `INT` | FK → `Products` |  |
| `lot_id` | `INT` | FK → `Inventory_Lots` NULL ได้ | ว่างได้ เพราะค่าแรงไม่มี Lot |
| `qty` | `DECIMAL(10,2)` |  |  |
| `selling_price` | `DECIMAL(10,2)` |  | ราคาที่ตกลงกับลูกค้า ก่อน VAT |
| `actual_cost` | `DECIMAL(10,2)` |  | ต้นทุนจริงต่อหน่วย ดึงมาจาก Lot ที่ตัด |

**การตัดข้าม Lot** ถ้าอะไหล่ตัวเดียวต้องตัดจากหลาย Lot เช่นต้องการ 10 ชิ้นแต่ Lot เก่าเหลือ 6 ต้องเอาจาก Lot ถัดไปอีก 4 ให้แตกเป็นสองแถว แถวละ Lot เพราะแต่ละ Lot มีต้นทุนไม่เท่ากัน ถ้ายัดใน 1 แถว `actual_cost` จะเก็บได้ค่าเดียวและกำไรจะผิด

**กรณีสต็อกไม่พอ** ตัดเท่าที่มีก่อนแล้วเปลี่ยน `Jobs.status` เป็น `Waiting_Part` พอของที่ค้างมาถึงค่อยเบิกส่วนที่เหลือเป็นแถวใหม่

---

## 4. จัดซื้อและคลังสินค้า

### `Purchase_Orders` — ใบสั่งซื้อ

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `po_id` | `INT` | **PK** |  |
| `po_number` | `VARCHAR(20)` | UNIQUE |  |
| `supplier_id` | `INT` | FK → `Suppliers` |  |
| `status` | `ENUM('Pending','Partial','Received','Cancelled')` |  |  |
| `order_date` | `DATE` |  |  |

### `Purchase_Order_Items` — รายการในใบสั่งซื้อ

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `po_item_id` | `INT` | **PK** |  |
| `po_id` | `INT` | FK → `Purchase_Orders` |  |
| `product_id` | `INT` | FK → `Products` |  |
| `qty` | `DECIMAL(10,2)` |  | จำนวนที่สั่ง |
| `received_qty` | `DECIMAL(10,2)` |  | จำนวนที่รับมาแล้วจริง |
| `cost_price` | `DECIMAL(10,2)` |  | ต้นทุนต่อชิ้นที่ตกลงกับร้าน |

รับของไม่ครบให้กดรับเท่าที่มา ระบบสร้าง Lot ของจำนวนนั้นและตั้งใบสั่งซื้อเป็น `Partial` ของที่ตามมาทีหลังจะสร้าง Lot ใหม่แยก เพราะอาจได้ต้นทุนคนละราคา

### `Inventory_Lots` — ล็อตสินค้าและต้นทุน

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `lot_id` | `INT` | **PK** |  |
| `product_id` | `INT` | FK → `Products` |  |
| `po_id` | `INT` | FK → `Purchase_Orders` NULL ได้ | ว่างได้กรณีซื้อด่วน |
| `supplier_id` | `INT` | FK → `Suppliers` NULL ได้ | ว่างได้ แต่จำเป็นเพราะซื้อด่วนไม่มีใบสั่งซื้อ |
| `receive_date` | `DATETIME` |  | วันที่ของเข้า ใช้เรียงคิว FIFO |
| `unit_cost` | `DECIMAL(10,2)` |  | ต้นทุนต่อชิ้นของรอบนี้ |
| `initial_qty` | `DECIMAL(10,2)` |  | จำนวนที่รับเข้า |
| `remaining_qty` | `DECIMAL(10,2)` |  | จำนวนที่เหลือให้ตัด |

`supplier_id` จำเป็นเพราะการซื้อด่วนไม่มีใบสั่งซื้อ ถ้าไม่มีฟิลด์นี้จะไม่รู้ว่าของล็อตนั้นซื้อมาจากร้านไหน

### `Inventory_Transactions` — ความเคลื่อนไหวสต็อก

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `transaction_id` | `BIGINT` | **PK** |  |
| `product_id` | `INT` | FK → `Products` |  |
| `lot_id` | `INT` | FK → `Inventory_Lots` |  |
| `job_id` | `INT` | FK → `Jobs` NULL ได้ | ว่างได้กรณีเป็นการรับของเข้า |
| `qty` | `DECIMAL(10,2)` |  | บวกคือรับเข้าหรือคืนสต็อก ลบคือเบิกใช้ |
| `type` | `ENUM('Receive','Consume','Rollback','Adjust')` |  |  |
| `note` | `VARCHAR(255)` | NULL ได้ | บังคับกรอกเมื่อ `type` เป็น `Adjust` |
| `created_at` | `DATETIME` |  |  |
| `created_by` | `INT` | FK → `Employees` |  |

`Rollback` ใช้ตอนคืนของที่จองไว้กลับเข้าสต็อกเมื่อลูกค้าไม่อนุมัติ ส่วน `Adjust` ใช้ตอนนับสต็อกแล้วของจริงไม่ตรงกับระบบ

ทุก role ปรับสต็อกได้ แต่ทุกครั้งต้องมีเหตุผลใน `note` และระบบบันทึก `created_by` ไว้เสมอ

---

## 5. การเงินและโปรโมชั่น

### `Promotions`

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `promotion_id` | `INT` | **PK** |  |
| `code` | `VARCHAR(30)` | UNIQUE | เช่น RAINY26 |
| `name` | `VARCHAR(150)` |  |  |
| `discount_type` | `ENUM('Percent','Fixed_Amount')` |  |  |
| `discount_value` | `DECIMAL(10,2)` |  |  |
| `scope` | `ENUM('All','Part','Labor')` |  | จำเป็นสำหรับแคมเปญแบบฟรีค่าแรง |
| `start_date` | `DATE` |  |  |
| `end_date` | `DATE` |  |  |
| `is_active` | `BOOLEAN` |  |  |

`scope` จำเป็นสำหรับแคมเปญแบบฟรีค่าแรง ซึ่งลดเฉพาะหมวดค่าแรงเท่านั้น

### `Invoices` — ใบเสร็จรับเงิน

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `invoice_id` | `INT` | **PK** |  |
| `job_id` | `INT` | FK → `Jobs` UNIQUE | หนึ่งใบงานออกได้หนึ่งบิล |
| `promotion_id` | `INT` | FK → `Promotions` NULL ได้ | หนึ่งบิลใช้ได้หนึ่งโปร |
| `part_total` | `DECIMAL(12,2)` |  | ยอดอะไหล่ก่อนหักส่วนลด ก่อน VAT |
| `labor_total` | `DECIMAL(12,2)` |  | ยอดค่าแรงก่อนหักส่วนลด |
| `discount_parts` | `DECIMAL(12,2)` |  | ส่วนลดที่ตกกับหมวดอะไหล่ |
| `discount_labor` | `DECIMAL(12,2)` |  | ส่วนลดที่ตกกับหมวดค่าแรง |
| `vat_rate` | `DECIMAL(5,2)` |  | อัตรา VAT ณ วันออกบิล |
| `vat_amount` | `DECIMAL(12,2)` |  |  |
| `wht_rate` | `DECIMAL(5,2)` |  | อัตราหัก ณ ที่จ่าย ณ วันออกบิล |
| `wht_amount` | `DECIMAL(12,2)` |  |  |
| `net_total` | `DECIMAL(12,2)` |  | ยอดที่ลูกค้าจ่ายจริง |
| `gross_profit` | `DECIMAL(12,2)` |  | กำไรขั้นต้นของบิลนี้ |
| `status` | `ENUM('Unpaid','Partial','Paid')` |  |  |
| `issue_date` | `DATETIME` |  |  |

**การปันส่วนลด** กรณี `scope` เป็น `All`

```
ส่วนลดรวม   = discount_value  หรือ  (part_total + labor_total) × discount_value%
discount_parts = ส่วนลดรวม × part_total / (part_total + labor_total)
discount_labor = ส่วนลดรวม − discount_parts
```

ถ้า `scope` เป็น `Part` ให้ลงที่ `discount_parts` ทั้งก้อน ถ้าเป็น `Labor` ก็ลงที่ `discount_labor` ทั้งก้อน

ที่ต้องแยกสองช่องเพราะฐานภาษีของสองหมวดคนละตัว ถ้าเก็บส่วนลดรวมเป็นก้อนเดียวจะคำนวณ VAT กับภาษีหัก ณ ที่จ่ายไม่ได้

**การคิดภาษี**

```
vat_amount = (part_total − discount_parts) × vat_rate%
             เป็น 0 ถ้า Shop_Settings.is_vat_registered เป็นเท็จ

wht_amount = (labor_total − discount_labor) × wht_rate%
             เป็น 0 ถ้า Customers.is_corporate เป็นเท็จ
```

`vat_rate` และ `wht_rate` คัดลอกมาจาก `Shop_Settings` ตอนออกบิลแล้วเก็บติดไว้กับบิล ไม่ได้อ่านสดทุกครั้ง บิลเก่าจึงไม่เปลี่ยนตามถ้าวันหน้ามีการแก้อัตราภาษี

**กำไรขั้นต้น**

```
gross_profit = (part_total − discount_parts) + (labor_total − discount_labor)
             − SUM(Job_Items.actual_cost × qty)
```

### `Payments` — รับเงินจากลูกค้า

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `payment_id` | `INT` | **PK** |  |
| `invoice_id` | `INT` | FK → `Invoices` |  |
| `method` | `ENUM('Cash','Transfer','Card')` |  |  |
| `amount` | `DECIMAL(12,2)` |  |  |
| `paid_at` | `DATETIME` |  |  |
| `received_by` | `INT` | FK → `Employees` |  |

หนึ่งบิลมีได้หลายรายการจ่าย รองรับการจ่ายแบ่งงวด

### `Supplier_Payments` — จ่ายเงินให้ร้านอะไหล่

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `supplier_payment_id` | `INT` | **PK** |  |
| `supplier_id` | `INT` | FK → `Suppliers` |  |
| `po_id` | `INT` | FK → `Purchase_Orders` NULL ได้ | ว่างได้ |
| `lot_id` | `INT` | FK → `Inventory_Lots` NULL ได้ | ใช้แทน `po_id` กรณีซื้อด่วน |
| `method` | `ENUM('Cash','Transfer','Petty_Cash')` |  | `Petty_Cash` คือการตัดเงินสดย่อยของอู่ |
| `amount` | `DECIMAL(12,2)` |  |  |
| `paid_at` | `DATETIME` |  |  |

`Petty_Cash` คือการตัดเงินสดย่อยของอู่ ใช้ตอนช่างวิ่งไปซื้อของด่วนแล้วควักเงินไปก่อน

---

## 6. แจ้งเตือน

### `Maintenance_Alerts`

| ฟิลด์ | ชนิดข้อมูล | คีย์ | รายละเอียด |
|---|---|---|---|
| `alert_id` | `INT` | **PK** |  |
| `vehicle_id` | `INT` | FK → `Vehicles` |  |
| `job_id` | `INT` | FK → `Jobs` | งานที่เป็นต้นทางของการแจ้งเตือน |
| `product_id` | `INT` | FK → `Products` |  |
| `due_date` | `DATE` |  | วันที่ปิดงาน บวก `Products.lifespan_months` เดือน เป็นตัวเดียวที่ใช้ยิงแจ้งเตือน |
| `is_contacted` | `BOOLEAN` |  |  |
| `contacted_by` | `INT` | FK → `Employees` NULL ได้ |  |
| `contacted_at` | `DATETIME` | NULL ได้ |  |
| `result` | `ENUM('Pending','Booked','Declined','No_Answer')` |  | ใช้วัด KPI ของพนักงานหน้าร้าน |

`result` ใช้วัด KPI ของพนักงานหน้าร้านว่าโทรตามลูกค้าแล้วได้ผลแค่ไหน

---

## สิทธิ์ตาม `Employees.role`

| สิทธิ์ | Admin | Employee | Mechanic |
|---|:---:|:---:|:---:|
| เปิด / ปิดใบงาน | ✓ | ✓ | ✓ |
| ออกและอนุมัติใบเสนอราคา | ✓ | ✓ | ✓ |
| ปรับสต็อก | ✓ | ✓ | ✓ |
| เห็นกำไรและต้นทุน | ✓ | — | — |
| จัดการโปรโมชั่น | ✓ | ✓ | — |
| เพิ่ม / ลบพนักงาน | ✓ | — | — |

Admin คือเจ้าของอู่ Employee คือพนักงานหน้าร้าน Mechanic คือช่าง งานประจำวันอย่างการเปิดใบงาน อนุมัติใบเสนอราคา และปรับสต็อก ทำได้ทุก role เพราะทุกการกระทำมี `created_by` หรือ `approved_by` บันทึกไว้อยู่แล้ว ส่วนที่ล็อกไว้จริง ๆ คือตัวเลขกำไรและต้นทุน ซึ่งเห็นได้เฉพาะเจ้าของ

---

## หมายเหตุการอ่านผัง ER

ผัง `diagrams/er-database.html` ลากเส้นเฉพาะความสัมพันธ์เชิงโครงสร้าง 19 เส้น ส่วน foreign key ที่เป็นการอ้างอิงข้อมูลหลักทั่วไป เช่น `product_id` ที่โผล่ในหลายตาราง แสดงเป็นฟิลด์ในกล่องแทนการลากเส้น ไม่งั้นเส้นจะพันกันจนอ่านไม่ออก

`Shop_Settings` ไม่มีเส้นเชื่อมกับใครเพราะเป็นตารางตั้งค่าที่มีแถวเดียว ระบบอ่านค่าจากตารางนี้ตอนออกบิลแล้วคัดลอกอัตราภาษีไปเก็บไว้ที่ `Invoices`
