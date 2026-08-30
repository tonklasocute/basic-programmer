# บทที่ 47: Clean Code, Clean Architecture

## แนวคิดหลัก (Concept)

ตลอด Part 1-11 เราเรียนรู้**เครื่องมือ**มากมาย (โครงสร้างข้อมูล, อัลกอริทึม, OOP, Concurrency, Database, Networking, OS) แต่ยังไม่ได้พูดถึงคำถามที่สำคัญไม่แพ้กัน: **เขียนโค้ดอย่างไรให้คนอื่น (และตัวเองในอนาคต) อ่านแล้วเข้าใจง่าย และจัดระบบโครงสร้างโปรเจกต์อย่างไรให้ขยายได้โดยไม่พังทลาย** บทนี้เปิด **Part 12: Software Engineering** ด้วยสองแนวคิดพื้นฐาน — **Clean Code** (การเขียนโค้ดระดับบรรทัด/ฟังก์ชัน) และ **Clean Architecture** (การจัดโครงสร้างระดับโปรเจกต์)

- **Clean Code** — แนวทางเขียนโค้ดที่เน้น**ความอ่านง่ายและความตั้งใจที่ชัดเจน** มากกว่าความฉลาดหรือกระชับที่สุด โดยเชื่อว่า**โค้ดถูกอ่านบ่อยกว่าถูกเขียนมาก** (คนอื่นอ่านโค้ดของเรานับสิบครั้งในอนาคต แต่เขียนแค่ครั้งเดียว)
- **Meaningful Naming** — การตั้งชื่อตัวแปร/ฟังก์ชัน/Class ให้สื่อความหมายชัดเจนในตัวเอง โดยไม่ต้องอ่าน comment เพิ่ม
- **Small Functions** — หลักการที่ฟังก์ชันควรทำ**สิ่งเดียว**และทำให้ดี (คล้าย Single Responsibility Principle จากบทที่ 32 แต่ระดับฟังก์ชันแทนที่จะเป็นระดับ Class)
- **Clean Architecture** — รูปแบบการจัดโครงสร้างโปรเจกต์ที่แบ่งเป็นชั้น (layer) โดยมีกฎว่า**การพึ่งพา (dependency) ต้องชี้เข้าหาศูนย์กลางเสมอ** — ชั้นในสุด (Business Logic) ไม่รู้จักและไม่พึ่งพาชั้นนอก (Database, UI, Framework) เลย

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Clean Code มีไว้ลดต้นทุนการดูแลรักษาโค้ดในระยะยาว**: สถิติในวงการซอฟต์แวร์ชี้ว่าเวลาที่ใช้**อ่าน**โค้ดมากกว่าเวลาที่ใช้**เขียน**โค้ดหลายเท่า (อ่านโค้ดเก่าเพื่อแก้บั๊ก, เพิ่มฟีเจอร์, หรือทำความเข้าใจก่อนรีวิว) — โค้ดที่อ่านยากทำให้ทุกงานเหล่านี้ช้าลงและเสี่ยงเกิดบั๊กใหม่จากความเข้าใจผิด
- **Meaningful Naming มีไว้ทำให้โค้ดอธิบายตัวเองได้โดยไม่ต้องพึ่ง comment**: comment เก่าที่ไม่ได้อัปเดตตามโค้ดที่เปลี่ยนไปกลายเป็นข้อมูลที่ผิดพลาดและทำให้เข้าใจผิดยิ่งกว่าไม่มี comment เลย — ชื่อตัวแปรที่ดี (`days_since_last_login` แทน `d`) สื่อความหมายได้เสมอโดยไม่มีวันล้าสมัย
- **Small Functions มีไว้ลดภาระทางความคิด (cognitive load) ตอนอ่านโค้ด**: สมองมนุษย์จำสิ่งที่ซับซ้อนพร้อมกันได้จำกัด ฟังก์ชันที่ทำหลายอย่างปนกัน (คำนวณ + validate + save + log ในฟังก์ชันเดียว เหมือนตัวอย่างผิด SRP ในบทที่ 32) บังคับให้ผู้อ่านต้องจำทุกอย่างพร้อมกันเพื่อเข้าใจ ในขณะที่ฟังก์ชันเล็กๆ ที่ทำสิ่งเดียวอ่านและทดสอบทีละส่วนได้ง่ายกว่ามาก
- **Clean Architecture มีไว้ป้องกันไม่ให้ Business Logic ผูกติดกับรายละเอียดทางเทคนิคที่เปลี่ยนบ่อย**: ถ้า logic การคำนวณดอกเบี้ยธนาคาร (ซึ่งควรคงที่ตามกฎธุรกิจ) ผูกติดกับโค้ด SQL เฉพาะของฐานข้อมูลหนึ่งโดยตรง การเปลี่ยนฐานข้อมูล (จาก MySQL เป็น PostgreSQL) จะบังคับให้ต้องแก้ logic ทางธุรกิจไปด้วยทั้งที่ไม่ควรเกี่ยวข้องกันเลย — หลักการนี้คือ Dependency Inversion Principle (บทที่ 32) ที่ขยายไปสู่ระดับสถาปัตยกรรมทั้งโปรเจกต์

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Clean Code** เหมือน **การเขียนจดหมายลาป่วยที่ชัดเจนตรงไปตรงมา** เทียบกับ **จดหมายที่ใช้ภาษาซับซ้อนเกินจำเป็นจนต้องอ่านซ้ำหลายรอบ** — ทั้งคู่สื่อสารเนื้อหาเดียวกันได้ แต่แบบแรกทำให้ผู้อ่านเข้าใจทันทีโดยไม่ต้องเดา
- **Meaningful Naming** เหมือน **ป้ายชื่อลิ้นชักในครัวที่เขียนว่า "ช้อนส้อม" แทนที่จะเขียนว่า "ลิ้นชัก 3"** — คนที่มาใช้ครัวครั้งแรกหาของถูกทันทีโดยไม่ต้องเปิดดูทีละลิ้นชัก
- **Small Functions** เหมือน **สูตรอาหารที่แบ่งเป็นขั้นตอนย่อยชัดเจน (หั่นผัก, ผัดเนื้อ, ปรุงรส)** เทียบกับ **สูตรที่เขียนทุกอย่างในย่อหน้าเดียวยาวๆ** — ขั้นตอนที่แยกชัดเจนทำให้ทำตามได้ง่ายและแก้ไขเฉพาะจุดได้โดยไม่กระทบส่วนอื่น
- **Clean Architecture** เหมือน **อาคารที่แยกระบบไฟฟ้า ประปา และโครงสร้างออกจากกันชัดเจน** — เปลี่ยนบริษัทไฟฟ้าที่ใช้ (รายละเอียดทางเทคนิค) ไม่จำเป็นต้องรื้อโครงสร้างอาคาร (Business Logic) เลย เพราะทั้งสองระบบถูกออกแบบให้แยกจากกันตั้งแต่ต้น

## แผนภาพอธิบาย (Visual explanation)

### Meaningful Naming — ชื่อที่แย่ vs ชื่อที่สื่อความหมาย

```
ชื่อที่แย่:                              ชื่อที่สื่อความหมาย:

def calc(x, y, z):                      def calculate_total_price(base_price, tax_rate, discount):
    return x * (1 + y) - z                  return base_price * (1 + tax_rate) - discount

d = 5                                    days_since_last_login = 5

if u.s == 1:                            if user.status == UserStatus.ACTIVE:
    ...                                      ...

ต้องเดาความหมายจากบริบท                    เข้าใจได้ทันทีแม้ไม่เคยเห็นโค้ดนี้มาก่อน
```

### Small Functions — แยกหน้าที่ที่ปนกันออกจากกัน (ทบทวนจาก SRP บทที่ 32)

```
ฟังก์ชันใหญ่ที่ทำหลายอย่างปนกัน:              แยกเป็นฟังก์ชันเล็กที่ทำสิ่งเดียว:

def process_order(order):                def process_order(order):
    # validate (10 บรรทัด)                    validate_order(order)
    # calculate total (15 บรรทัด)              total = calculate_total(order)
    # save to database (8 บรรทัด)              save_order(order, total)
    # send email (12 บรรทัด)                   send_confirmation_email(order)
    # log (5 บรรทัด)                           log_order_processed(order)

50 บรรทัดที่ต้องอ่านทั้งหมดเพื่อเข้าใจ           อ่านแค่ชื่อฟังก์ชัน 5 บรรทัดก็เข้าใจภาพรวมได้ทันที
ทดสอบยาก (ต้อง mock ทุกอย่างพร้อมกัน)           ทดสอบแต่ละฟังก์ชันแยกกันได้ง่าย
```

### Clean Architecture — Dependency ชี้เข้าหาศูนย์กลางเสมอ

```
                    ┌─────────────────────────────────┐
                    │   Frameworks & Drivers           │   <- ชั้นนอกสุด (DB, Web Framework, UI)
                    │  ┌─────────────────────────────┐  │
                    │  │   Interface Adapters         │  │   <- แปลงข้อมูลระหว่างชั้นใน-นอก
                    │  │  ┌─────────────────────────┐  │  │
                    │  │  │   Use Cases               │  │  │   <- Business Logic เฉพาะแอปนี้
                    │  │  │  ┌─────────────────────┐  │  │  │
                    │  │  │  │   Entities            │  │  │  │   <- แกนกลาง: กฎธุรกิจหลัก
                    │  │  │  └─────────────────────┘  │  │  │
                    │  │  └─────────────────────────┘  │  │
                    │  └─────────────────────────────┘  │
                    └─────────────────────────────────┘

ลูกศร Dependency ชี้เข้าด้านในเสมอ: ───────────────────>
Entities ไม่รู้จัก Use Cases, Use Cases ไม่รู้จัก Database/Framework เลย
เปลี่ยน Database หรือ Web Framework (ชั้นนอก) โดยไม่ต้องแก้ Entities/Use Cases (ชั้นใน) เลย
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== ก่อน: โค้ดที่อ่านยาก (ชื่อไม่สื่อความหมาย, ฟังก์ชันทำหลายอย่างปนกัน) =====
def calc(o):
    t = 0
    for i in o['items']:
        t += i['p'] * i['q']
    if o['c'] == 'VIP':
        t = t * 0.9
    if t > 1000:
        t = t - 50
    db.save({'total': t, 'order_id': o['id']})
    send_email(o['customer_email'], f"Your total is {t}")
    return t

# ===== หลัง: Clean Code — ชื่อสื่อความหมาย, แยกฟังก์ชันตามหน้าที่ =====
def calculate_subtotal(order_items):
    return sum(item['price'] * item['quantity'] for item in order_items)

def apply_vip_discount(subtotal, customer_type):
    return subtotal * 0.9 if customer_type == 'VIP' else subtotal

def apply_bulk_discount(amount, threshold=1000, discount=50):
    return amount - discount if amount > threshold else amount

def process_order(order):
    subtotal = calculate_subtotal(order['items'])
    discounted = apply_vip_discount(subtotal, order['customer_type'])
    total = apply_bulk_discount(discounted)

    save_order_total(order['id'], total)
    send_order_confirmation(order['customer_email'], total)
    return total

def save_order_total(order_id, total):
    db.save({'total': total, 'order_id': order_id})

def send_order_confirmation(email, total):
    send_email(email, f"Your total is {total}")

# ===== Clean Architecture: แยก Business Logic ออกจาก Database/Framework =====
# ชั้นในสุด (Entity + Use Case) - ไม่รู้จัก Database หรือ Web Framework เลย
class Order:
    def __init__(self, items, customer_type):
        self.items = items
        self.customer_type = customer_type

    def calculate_total(self):                          # กฎธุรกิจล้วนๆ ไม่ผูกกับเทคโนโลยีใดๆ
        subtotal = sum(item.price * item.quantity for item in self.items)
        if self.customer_type == 'VIP':
            subtotal *= 0.9
        return subtotal - 50 if subtotal > 1000 else subtotal

# ชั้นนอก (Interface Adapter) - เชื่อม Business Logic เข้ากับ Database จริง
class OrderRepository:                                   # Abstraction (ทบทวนบทที่ 32: DIP)
    def save(self, order, total):
        raise NotImplementedError

class SqlOrderRepository(OrderRepository):                # implementation จริงผูกกับเทคโนโลยีเฉพาะ
    def save(self, order, total):
        db.execute("INSERT INTO orders (total) VALUES (?)", (total,))

# Use Case ประสาน Entity กับ Repository ผ่าน Abstraction เท่านั้น
def process_order_use_case(order: Order, repository: OrderRepository):
    total = order.calculate_total()                       # เรียก Business Logic ที่ไม่ผูกกับ DB เลย
    repository.save(order, total)                            # บันทึกผ่าน abstraction ไม่ใช่ SQL ตรงๆ
    return total
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. เวอร์ชัน "ก่อน" — ตัวแปรชื่อ `o`, `t`, `i`, `p`, `q`, `c` ไม่สื่อความหมายอะไรเลย ผู้อ่านต้องไล่ตามโค้ดทีละบรรทัดเพื่อเดาว่าแต่ละตัวคืออะไร และฟังก์ชัน `calc` ทำ**5 อย่างปนกัน**ในฟังก์ชันเดียว (คำนวณ subtotal, ลดราคา VIP, ลดราคาตามยอด, บันทึกฐานข้อมูล, ส่งอีเมล) ทำให้ทดสอบยาก (ต้อง mock database และ email service พร้อมกันแม้จะแค่ต้องการทดสอบการคำนวณ) และแก้ไขเสี่ยง (แก้ logic คำนวณอาจกระทบส่วนบันทึกข้อมูลโดยไม่ตั้งใจ)
2. เวอร์ชัน "หลัง" — แต่ละฟังก์ชัน (`calculate_subtotal`, `apply_vip_discount`, `apply_bulk_discount`) ทำ**สิ่งเดียว**และชื่อฟังก์ชันบอกเจตนาชัดเจนโดยไม่ต้องอ่าน body เลย `process_order` กลายเป็น**เรื่องเล่าที่อ่านจากบนลงล่างแล้วเข้าใจภาพรวมได้ทันที** (คำนวณ subtotal -> ใส่ส่วนลด VIP -> ใส่ส่วนลดยอดสั่งซื้อ -> บันทึก -> ส่งอีเมล) — นี่คือหลักการ **Function Composition** จากบทที่ 35 ที่นำมาใช้จริงในการจัดโครงสร้างโค้ด
3. `Order.calculate_total()` — เป็นเมทอดที่**ไม่รู้จักฐานข้อมูลหรือ framework ใดๆ เลย** มีแค่ logic ทางธุรกิจล้วนๆ (คำนวณราคา, ลดราคา) ทำให้ทดสอบได้ง่ายมาก (สร้าง `Order` object แล้วเรียก `calculate_total()` ตรงๆ โดยไม่ต้องต่อฐานข้อมูลจริงเลย) และไม่มีวันพังเพราะการเปลี่ยนแปลงทางเทคนิค (เปลี่ยนฐานข้อมูล, เปลี่ยน web framework)
4. `process_order_use_case(order, repository)` — รับ `repository` เป็น `OrderRepository` (abstraction) ไม่ใช่ `SqlOrderRepository` ตรงๆ (**Dependency Injection** จากบทที่ 32) ทำให้ฟังก์ชันนี้**ไม่รู้จักรายละเอียดของฐานข้อมูลจริงเลย** — ถ้าต้องเปลี่ยนจาก SQL เป็น MongoDB แค่สร้าง `MongoOrderRepository(OrderRepository)` ใหม่แล้วส่งเข้ามาแทน โดยไม่ต้องแก้ `process_order_use_case` หรือ `Order.calculate_total()` แม้แต่บรรทัดเดียว — นี่คือ Clean Architecture ที่ทำงานจริง

## Time Complexity

Clean Code และ Clean Architecture เป็นหลักการด้าน**ความสามารถในการดูแลรักษา (maintainability)** ไม่ใช่อัลกอริทึม จึงไม่มี Time Complexity ในความหมายของ Big O — ผลกระทบวัดกันในแง่**เวลาที่ใช้ในการทำความเข้าใจ แก้ไข และทดสอบโค้ดในระยะยาว** ซึ่งลดลงอย่างมีนัยสำคัญเมื่อโค้ดสะอาดและมีโครงสร้างที่ดี

## Space Complexity

การแยกฟังก์ชันและ Class ตาม Clean Code/Architecture อาจเพิ่มจำนวนไฟล์/ฟังก์ชันในโปรเจกต์ แต่**ไม่มีผลกระทบด้านหน่วยความจำขณะรันจริง** (runtime memory) อย่างมีนัยสำคัญ — เป็นเรื่องขององค์กรของโค้ด (source code organization) ไม่ใช่การใช้ทรัพยากรของโปรแกรม

## แนวทางปฏิบัติที่ดี (Best Practices)

- ตั้งชื่อตัวแปร/ฟังก์ชันให้ **บอกเจตนา** ไม่ใช่แค่บอกประเภทข้อมูล (`active_users` ดีกว่า `list1`, `calculate_shipping_cost` ดีกว่า `calc`) — ยิ่งชื่อยาวขึ้นเล็กน้อยเพื่อความชัดเจนยิ่งคุ้มค่ากว่าชื่อสั้นที่คลุมเครือเสมอ
- เขียนฟังก์ชันให้สั้น (มักแนะนำไม่เกิน 20 บรรทัดเป็นแนวทาง ไม่ใช่กฎตายตัว) และทำสิ่งเดียว ถ้าฟังก์ชันมีคำว่า "และ" ในการอธิบายว่ามันทำอะไร (เช่น "คำนวณราคาและบันทึกฐานข้อมูล") ให้พิจารณาแยกเป็นสองฟังก์ชัน
- ในการออกแบบ Clean Architecture ให้ **Business Logic (Entities, Use Cases) เป็น Pure Function ให้ได้มากที่สุด** (ทบทวนบทที่ 34) เพื่อทดสอบได้ง่ายโดยไม่ต้องพึ่งฐานข้อมูลหรือ network จริง

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ใช้ชื่อตัวแปรสั้นเกินไปเพื่อ "ประหยัดการพิมพ์" (`x`, `tmp`, `data2`) ทำให้โค้ดอ่านยากขึ้นมากในระยะยาว แม้จะเขียนเร็วขึ้นเล็กน้อยในตอนแรก
- เขียนฟังก์ชันที่ทำหลายอย่างปนกันเพราะ "สะดวกกว่าเขียนแยก" โดยไม่ตระหนักว่าจะทำให้ทดสอบและแก้ไขยากขึ้นมากในภายหลัง (เหมือนตัวอย่าง "ก่อน" ในบทเรียน)
- ผูก Business Logic เข้ากับรายละเอียดทางเทคนิคโดยตรง (เช่น เขียน SQL query ปนอยู่ในฟังก์ชันคำนวณราคา) ทำให้เปลี่ยนเทคโนโลยีทีหลังต้องแก้ logic ทางธุรกิจที่ไม่ควรเกี่ยวข้องกันเลย
- ยึดติดกับ Clean Architecture แบบสุดโต่งในโปรเจกต์เล็กๆ ที่ไม่มีแนวโน้มจะเปลี่ยนเทคโนโลยีเลย ทำให้เพิ่มความซับซ้อนโดยไม่จำเป็น (over-engineering เหมือนที่เตือนไว้ในบทที่ 32-33)

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายว่าทำไม "โค้ดถูกอ่านบ่อยกว่าถูกเขียน" ถึงเป็นหลักการสำคัญของ Clean Code
2. ยกตัวอย่างการตั้งชื่อที่แย่และปรับปรุงให้ดีขึ้น พร้อมอธิบายเหตุผล
3. Small Functions เกี่ยวข้องกับ Single Responsibility Principle (บทที่ 32) อย่างไร
4. อธิบาย Clean Architecture คืออะไร ทำไม dependency ต้องชี้เข้าหาศูนย์กลางเสมอ
5. ทำไม Business Logic ที่เขียนเป็น Pure Function (บทที่ 34) ถึงเหมาะกับ Clean Architecture มากเป็นพิเศษ

## แบบฝึกหัด (Practice Exercises)

1. หาโค้ดที่เคยเขียนในบทเรียนก่อนหน้า (หรือ Mini Project ใดก็ได้) ที่มีชื่อตัวแปรไม่ชัดเจนหรือฟังก์ชันทำหลายอย่างปนกัน แล้วปรับปรุงตามหลัก Clean Code
2. เขียนฟังก์ชัน `process_payment` ที่ทำ 4 อย่างปนกัน (validate, calculate fee, charge card, send receipt) แล้วแยกเป็นฟังก์ชันย่อยตามหลัก Small Functions
3. ออกแบบ Class `Order` ที่มี Business Logic ล้วนๆ (ไม่ผูกกับฐานข้อมูล) แล้วสร้าง `OrderRepository` (abstraction) พร้อม 2 implementation (`InMemoryOrderRepository` สำหรับทดสอบ, `SqlOrderRepository` สำหรับใช้จริง) ตามหลัก Clean Architecture

## Mini Project

**"ระบบจัดการห้องสมุดที่ออกแบบตาม Clean Architecture (Clean Library Management System)"**: เขียนโปรแกรมที่:
1. ออกแบบ Entity `Book`, `Member`, `Loan` ที่มี Business Logic ล้วนๆ (เช่น กฎการคำนวณค่าปรับเมื่อคืนหนังสือช้า) โดยไม่ผูกกับฐานข้อมูลหรือ UI ใดๆ เลย
2. สร้าง Use Case เช่น `BorrowBookUseCase`, `ReturnBookUseCase` ที่ประสาน Entity เข้ากับ Repository ผ่าน Abstraction (Dependency Injection ตามบทที่ 32)
3. Implement Repository สองแบบ: `InMemoryRepository` (สำหรับทดสอบ ไม่ต้องมีฐานข้อมูลจริง) และ `SqliteRepository` (ใช้งานจริง) แล้วพิสูจน์ว่าสลับใช้งานได้โดยไม่แก้ Use Case หรือ Entity เลย
4. เขียนชื่อตัวแปร/ฟังก์ชันทุกจุดในโปรเจกต์ให้สื่อความหมายชัดเจน และแบ่งทุกฟังก์ชันให้ทำสิ่งเดียว จากนั้นให้เพื่อนหรือ AI อ่านโค้ดโดยไม่มีคำอธิบายเพิ่มเติม แล้วดูว่าเข้าใจการทำงานของระบบได้หรือไม่

## ข้อคิดสำคัญ (Key Takeaways)

- Clean Code เน้นความอ่านง่ายมากกว่าความกระชับหรือฉลาด เพราะโค้ดถูกอ่านบ่อยกว่าถูกเขียนมาก
- Meaningful Naming ทำให้โค้ดอธิบายตัวเองได้โดยไม่ต้องพึ่ง comment ที่อาจล้าสมัยได้ตลอดเวลา
- Small Functions ที่ทำสิ่งเดียว (ต่อยอดจาก SRP บทที่ 32) ลดภาระทางความคิดของผู้อ่านและทำให้ทดสอบง่ายขึ้น
- Clean Architecture แบ่งโปรเจกต์เป็นชั้นโดยให้ dependency ชี้เข้าหาศูนย์กลางเสมอ ทำให้ Business Logic ไม่ผูกติดกับรายละเอียดทางเทคนิคที่เปลี่ยนบ่อย (ต่อยอดจาก Dependency Inversion Principle บทที่ 32)
- ทั้งสองหลักการนี้เป็นการนำแนวคิดจาก Part 6 (OOP/SOLID) และ Part 7 (Pure Functions) มาประยุกต์ใช้ในระดับที่กว้างขึ้น: ระดับบรรทัดโค้ดและระดับโครงสร้างโปรเจกต์ทั้งหมด
- บทถัดไปจะเรียน MVC, Layered, Hexagonal Architecture ซึ่งเป็นรูปแบบสถาปัตยกรรมที่เป็นรูปธรรมมากขึ้นที่นำหลักการ Clean Architecture ไปปฏิบัติจริงในรูปแบบต่างๆ

---
*บทต่อไป (บทที่ 48): MVC, Layered, Hexagonal — พิมพ์ "Next" เพื่อดำเนินการต่อ*
