# บทที่ 32: SOLID Principles

## แนวคิดหลัก (Concept)

สองบทที่ผ่านมาสอนเครื่องมือของ OOP (Class, Object, Encapsulation, Abstraction, Inheritance, Polymorphism) แต่เครื่องมือเหล่านี้**ใช้ผิดวิธีก็ได้** เช่น สร้าง Inheritance hierarchy ที่ลึกเกินไป หรือ Class ที่ทำหน้าที่มากเกินไปจนแก้ไขทีไรพังทุกที บทนี้แนะนำ **SOLID** — ชุดหลักการออกแบบ 5 ข้อที่ทำให้โค้ด OOP ยืดหยุ่น ดูแลรักษาง่าย และขยายได้โดยไม่ต้องแก้โค้ดเดิม

- **S — Single Responsibility Principle (SRP)** — แต่ละ Class ควรมี**เหตุผลเดียว**ที่ทำให้ต้องแก้ไขมัน (มีหน้าที่รับผิดชอบเดียว)
- **O — Open/Closed Principle (OCP)** — Class ควร**เปิดให้ขยาย**ได้ (extension) แต่**ปิดไม่ให้แก้ไข**โค้ดเดิม (modification) — เพิ่มความสามารถใหม่ด้วยการเพิ่มโค้ดใหม่ ไม่ใช่แก้โค้ดที่มีอยู่แล้ว
- **L — Liskov Substitution Principle (LSP)** — Object ของ Child Class ต้องสามารถแทนที่ Object ของ Parent Class ได้ โดยไม่ทำให้โปรแกรมทำงานผิดพลาด
- **I — Interface Segregation Principle (ISP)** — ไม่ควรบังคับให้ Class หนึ่งต้อง implement method ที่มันไม่ได้ใช้จริง (แยก interface ใหญ่ให้เป็น interface เล็กๆ ที่เจาะจงกว่า)
- **D — Dependency Inversion Principle (DIP)** — Class ระดับสูงไม่ควรพึ่งพา Class ระดับล่างโดยตรง ทั้งคู่ควรพึ่งพา**นามธรรม (abstraction)** ร่วมกันแทน

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **มีไว้ป้องกัน "โค้ดที่แก้ตรงไหนก็พังตรงนั้น"**: เมื่อไม่ทำตาม SRP, Class หนึ่งอาจทำหลายหน้าที่ปนกัน (เช่น คำนวณดอกเบี้ยด้วย, บันทึกลงไฟล์ด้วย, ส่งอีเมลแจ้งเตือนด้วย) การแก้ไขส่วนหนึ่งเสี่ยงกระทบส่วนอื่นที่ไม่เกี่ยวข้องกันเลย
- **OCP มีไว้ป้องกันการแก้โค้ดเดิมที่ทดสอบผ่านแล้วและใช้งานจริงอยู่**: ทุกครั้งที่แก้โค้ดที่มีอยู่แล้ว มีความเสี่ยงที่จะทำให้ฟีเจอร์เดิมพัง (regression) — ถ้าออกแบบให้ขยายได้ด้วยการเพิ่ม Class ใหม่แทนการแก้ของเดิม (เหมือนที่ทำใน Mini Project บทที่ 31 ตอนเพิ่ม `CheckingAccount`) ความเสี่ยงนี้จะหายไป
- **LSP มีไว้รับประกันว่า Polymorphism (บทที่แล้ว) ทำงานถูกต้องจริง**: ถ้า Child Class ผิดสัญญาที่ Parent Class ให้ไว้ (เช่น Child โยน exception ที่ Parent ไม่เคยโยน หรือคืนค่าประเภทต่างไป) โค้ดที่เขียนโดยอาศัย Polymorphism จะพังโดยไม่คาดคิด แม้จะดูเหมือนใช้ Inheritance ถูกต้องทางไวยากรณ์ก็ตาม
- **ISP และ DIP มีไว้ลด "การผูกติดกันแน่นเกินไป" (tight coupling)**: ถ้า Class หนึ่งพึ่งพา Class อื่นโดยตรงมากเกินไป การเปลี่ยนแปลง Class หนึ่งจะบังคับให้ต้องแก้ Class ที่พึ่งพามันด้วยเสมอ ทำให้ระบบแก้ไขยากขึ้นเรื่อยๆ เมื่อโตขึ้น

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **SRP** เหมือน **พนักงานร้านอาหารที่มีหน้าที่ชัดเจนแยกกัน** — พ่อครัวทำอาหาร, พนักงานเสิร์ฟรับออเดอร์, แคชเชียร์คิดเงิน — ถ้าคนคนเดียวต้องทำทุกอย่าง เมื่อมีปัญหาเรื่องเงิน (ต้องฝึกแคชเชียร์ใหม่) จะกระทบการทำอาหารและเสิร์ฟไปด้วยโดยไม่จำเป็น
- **OCP** เหมือน **ปลั๊กไฟที่เสียบอุปกรณ์ใหม่ได้โดยไม่ต้องรื้อสายไฟในผนัง** — เพิ่มอุปกรณ์ไฟฟ้าใหม่ (ขยาย) ทำได้แค่เสียบปลั๊ก โดยไม่ต้องแก้ไขระบบไฟฟ้าเดิมในผนัง (ปิดการแก้ไข)
- **LSP** เหมือน **แบตเตอรี่ AA จากยี่ห้อไหนก็ใช้แทนกันได้ในอุปกรณ์เดียวกัน** — ถ้าแบตเตอรี่ยี่ห้อหนึ่งมีขนาดหรือแรงดันไฟต่างไปจากมาตรฐาน AA มันจะทำให้อุปกรณ์เสียหายหรือทำงานผิดพลาด แม้จะติดป้ายว่าเป็น "AA" เหมือนกัน
- **ISP** เหมือน **รีโมทคอนโทรลที่ออกแบบปุ่มเฉพาะสำหรับแต่ละอุปกรณ์** — แทนที่จะให้รีโมทตัวเดียวมีปุ่มทุกฟังก์ชันของทุกอุปกรณ์ (ทำให้ผู้ใช้งง) แยกรีโมทเป็นชุดเล็กๆ ที่ตรงกับความต้องการของแต่ละอุปกรณ์จริงๆ
- **DIP** เหมือน **ปลั๊กไฟมาตรฐานที่อุปกรณ์ไฟฟ้าทุกชนิดใช้ร่วมกันได้** — เครื่องใช้ไฟฟ้า (ระดับสูง) ไม่จำเป็นต้องรู้ว่าไฟฟ้ามาจากโรงไฟฟ้าถ่านหินหรือพลังงานแสงอาทิตย์ (ระดับล่าง) ทั้งคู่พึ่งพา "มาตรฐานปลั๊กไฟ" (นามธรรม) ร่วมกันแทน

## แผนภาพอธิบาย (Visual explanation)

### SRP — แยกหน้าที่ที่ปนกันออกจากกัน

```
ผิดหลัก SRP (Class เดียวทำหลายหน้าที่):        ถูกหลัก SRP (แยกหน้าที่ชัดเจน):

class Report:                                 class Report:
    def calculate_data(self): ...                 def calculate_data(self): ...   <- แค่คำนวณข้อมูล
    def format_as_pdf(self): ...
    def save_to_database(self): ...           class ReportFormatter:
    def send_email(self): ...                     def format_as_pdf(self, report): ...  <- แค่จัดรูปแบบ

3 เหตุผลที่ทำให้ Class นี้ต้องแก้ไข:            class ReportRepository:
  - เปลี่ยนสูตรคำนวณ                                def save(self, report): ...        <- แค่บันทึก
  - เปลี่ยนรูปแบบ PDF
  - เปลี่ยนวิธีเชื่อมต่อฐานข้อมูล                  class ReportMailer:
  - เปลี่ยนผู้ให้บริการอีเมล                          def send(self, report): ...        <- แค่ส่งอีเมล
  (ทั้งหมดกระทบ Class เดียวกัน!)
                                               แต่ละ Class มีเหตุผลเดียวที่ต้องแก้ไข
```

### OCP — ขยายด้วยการเพิ่ม Class ใหม่ ไม่แก้ของเดิม

```
ผิดหลัก OCP (ต้องแก้โค้ดเดิมทุกครั้งที่เพิ่มชนิดใหม่):

def calculate_discount(customer_type, price):
    if customer_type == "regular":
        return price * 0.95
    elif customer_type == "vip":              <- ทุกครั้งที่เพิ่มชนิดลูกค้าใหม่
        return price * 0.85                      ต้องมาแก้ฟังก์ชันนี้เพิ่ม if/elif
    elif customer_type == "new_vvip_type":       เสี่ยงทำของเดิมพังโดยไม่ตั้งใจ
        return price * 0.75

ถูกหลัก OCP (เพิ่ม Class ใหม่ ไม่แตะโค้ดเดิม):

class DiscountStrategy:
    def calculate(self, price): raise NotImplementedError

class RegularDiscount(DiscountStrategy):
    def calculate(self, price): return price * 0.95

class VipDiscount(DiscountStrategy):
    def calculate(self, price): return price * 0.85

# เพิ่มลูกค้าประเภทใหม่ = เพิ่ม Class ใหม่เท่านั้น ไม่แก้ RegularDiscount/VipDiscount เลย
class NewVvipDiscount(DiscountStrategy):
    def calculate(self, price): return price * 0.75
```

### LSP — Child Class ต้องแทนที่ Parent ได้โดยไม่ทำให้พัง

```
ผิดหลัก LSP (Square ทำลายสัญญาของ Rectangle):

class Rectangle:                              square = Square(4)
    def set_width(self, w): self.width = w    square.set_width(5)   <- ตาม LSP ควรได้ width=5, height=4
    def set_height(self, h): self.height = h  print(square.height)  <- แต่ Square บังคับ height=width เสมอ
                                                                        ผลจริง: height กลายเป็น 5 ด้วย!
class Square(Rectangle):                                              (พฤติกรรมขัดกับที่ Rectangle สัญญาไว้)
    def set_width(self, w):
        self.width = self.height = w          โค้ดที่คาดหวังพฤติกรรมแบบ Rectangle จะพังทันที
    def set_height(self, h):                  เมื่อได้รับ Square มาแทน -> ละเมิด LSP
        self.width = self.height = h
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== S: Single Responsibility — แยกหน้าที่ชัดเจน =====
class Invoice:
    def __init__(self, amount):
        self.amount = amount

    def calculate_total(self, tax_rate):           # หน้าที่เดียว: คำนวณยอดรวม
        return self.amount * (1 + tax_rate)

class InvoicePrinter:
    def print_invoice(self, invoice, total):        # หน้าที่เดียว: แสดงผล
        print(f"Invoice total: {total:.2f}")

class InvoiceRepository:
    def save(self, invoice):                          # หน้าที่เดียว: บันทึกข้อมูล
        print(f"Saving invoice of {invoice.amount} to database...")

# ===== O: Open/Closed — ขยายผ่าน Polymorphism แทนการแก้ if/elif เดิม =====
class DiscountStrategy:
    def calculate(self, price):
        raise NotImplementedError

class RegularDiscount(DiscountStrategy):
    def calculate(self, price):
        return price * 0.95

class VipDiscount(DiscountStrategy):
    def calculate(self, price):
        return price * 0.85

def checkout(price, discount_strategy: DiscountStrategy):    # ไม่ต้องแก้ฟังก์ชันนี้เลยเมื่อเพิ่มส่วนลดชนิดใหม่
    return discount_strategy.calculate(price)

# ===== L: Liskov Substitution — Child ต้องแทนที่ Parent ได้จริง =====
class Bird:
    def move(self):
        return "moving"

class FlyingBird(Bird):                                  # แยกออกจาก Bird ทั่วไป แทนที่จะสมมติว่านกทุกตัวบินได้
    def move(self):
        return "flying"

class Penguin(Bird):                                      # Penguin ไม่บิน แต่ยังคง "move" ได้ตามสัญญาของ Bird
    def move(self):
        return "swimming"                                    # ไม่ผิดสัญญา เพราะ Bird แค่สัญญาว่า "move ได้" ไม่ได้สัญญาว่า "บิน"

# ===== D: Dependency Inversion — พึ่งพา Abstraction ไม่ใช่ implementation ตรงๆ =====
class NotificationSender:                                  # นามธรรม (abstraction) กลาง
    def send(self, message):
        raise NotImplementedError

class EmailSender(NotificationSender):
    def send(self, message):
        print(f"Email sent: {message}")

class SmsSender(NotificationSender):
    def send(self, message):
        print(f"SMS sent: {message}")

class OrderService:                                        # Class ระดับสูง
    def __init__(self, sender: NotificationSender):          # พึ่งพา abstraction ไม่ใช่ EmailSender/SmsSender ตรงๆ
        self.sender = sender

    def place_order(self):
        self.sender.send("Your order has been placed!")

order_service = OrderService(EmailSender())    # สลับเป็น SmsSender() ได้ทันทีโดยไม่แก้ OrderService เลย
order_service.place_order()
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `Invoice`, `InvoicePrinter`, `InvoiceRepository` — แต่ละ Class มีเหตุผลเดียวที่ทำให้ต้องแก้ไข: ถ้าสูตรภาษีเปลี่ยน แก้แค่ `Invoice`, ถ้ารูปแบบการแสดงผลเปลี่ยน แก้แค่ `InvoicePrinter`, ถ้าเปลี่ยนฐานข้อมูล แก้แค่ `InvoiceRepository` — การแก้ไขแต่ละเรื่องไม่กระทบ Class อื่นเลย (เทียบกับตัวอย่าง "ผิดหลัก SRP" ในแผนภาพที่รวมทุกอย่างไว้ Class เดียว)
2. `checkout(price, discount_strategy)` — ฟังก์ชันนี้ **ไม่รู้จักชนิดของส่วนลดที่แน่นอนเลย** รู้แค่ว่ามันเป็น `DiscountStrategy` ที่มี `calculate()` (อาศัย Polymorphism จากบทที่แล้ว) เมื่อต้องเพิ่มส่วนลดชนิดใหม่ (เช่น `NewVvipDiscount`) แค่สร้าง Class ใหม่ที่สืบทอดจาก `DiscountStrategy` โดย**ไม่ต้องแก้ `checkout()` หรือ Class ส่วนลดเดิมเลยแม้แต่บรรทัดเดียว**
3. `Penguin(Bird)` — ออกแบบให้ `Bird.move()` สัญญาแค่ว่า "เคลื่อนที่ได้" ไม่ได้สัญญาว่า "ต้องบินได้" ทำให้ `Penguin` (ที่ว่ายน้ำแทนที่จะบิน) ยังคง**แทนที่ `Bird` ได้อย่างถูกต้องตาม LSP** — ถ้าออกแบบผิดโดยให้ `Bird.move()` สัญญาว่า "บินได้" ตั้งแต่ต้น (เหมือนตัวอย่าง `Rectangle`/`Square` ที่ผิดหลัก LSP) `Penguin` จะต้องเขียนโค้ดที่ขัดแย้งกับสัญญาของ Parent ทันที
4. `OrderService.__init__(self, sender: NotificationSender)` — รับ `sender` เป็นพารามิเตอร์ผ่าน type hint `NotificationSender` (นามธรรม) แทนที่จะสร้าง `EmailSender()` ขึ้นมาเองข้างในตรงๆ (`self.sender = EmailSender()`) เทคนิคนี้เรียกว่า **Dependency Injection** — ทำให้เปลี่ยนช่องทางแจ้งเตือนจาก Email เป็น SMS ได้เพียงแค่ส่ง object คนละชนิดเข้ามาตอนสร้าง `OrderService` โดยไม่ต้องแก้โค้ดภายใน `OrderService` เลยแม้แต่บรรทัดเดียว

## Time Complexity

SOLID Principles เป็นหลักการ**ออกแบบโครงสร้างโค้ด** ไม่ใช่อัลกอริทึม จึงไม่มี Time Complexity ในความหมายของ Big O (บทที่ 20) โดยตรง — ผลกระทบของ SOLID วัดกันในแง่ **ความง่ายในการดูแลรักษาและขยายโค้ดในระยะยาว (maintainability)** แทน ไม่ใช่ความเร็วในการรันโปรแกรม

## Space Complexity

การแยก Class ตาม SRP อาจทำให้จำนวน Class ในระบบเพิ่มขึ้น (พื้นที่หน่วยความจำสำหรับ object แต่ละตัวเพิ่มขึ้นเล็กน้อยตามจำนวน Class) แต่**ไม่มีนัยสำคัญ**เมื่อเทียบกับประโยชน์ด้าน maintainability ที่ได้กลับมา — SOLID ไม่ใช่เทคนิคที่ออกแบบมาเพื่อประหยัดหน่วยความจำ

## แนวทางปฏิบัติที่ดี (Best Practices)

- เมื่อพบว่า Class หนึ่งมี method ที่ทำหน้าที่ต่างกันโดยสิ้นเชิง (เช่น ทั้งคำนวณและบันทึกไฟล์) ให้พิจารณาแยกเป็นหลาย Class ตาม SRP ทันที แม้จะทำให้จำนวนไฟล์/Class เพิ่มขึ้นก็ตาม
- ก่อนเขียน `if/elif` ยาวๆ ตรวจสอบชนิดของข้อมูล (เชื่อมโยงบทที่ 10) ให้ถามตัวเองว่า "ถ้าเพิ่มเงื่อนไขใหม่ในอนาคต จะต้องมาแก้ฟังก์ชันนี้ไหม" ถ้าใช่ ให้พิจารณาออกแบบด้วย Polymorphism ตาม OCP แทน
- เมื่อออกแบบ Inheritance hierarchy เสมอถามว่า "Child Class ทุกตัวสามารถใช้แทน Parent ได้ในทุกที่ที่ Parent ถูกใช้งานหรือไม่ โดยไม่ทำให้พฤติกรรมของโปรแกรมเปลี่ยนไปในทางที่ไม่คาดคิด" ตาม LSP
- รับพารามิเตอร์เป็น abstraction (Parent Class หรือ interface) แทนการสร้าง object ที่พึ่งพาโดยตรงข้างในเสมอ (Dependency Injection) เพื่อให้ทดสอบและสลับ implementation ได้ง่ายในภายหลัง

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ใช้ SOLID Principles อย่างสุดโต่งเกินไปในโปรเจกต์ขนาดเล็กที่ไม่มีแนวโน้มจะขยายจริง ทำให้โค้ดซับซ้อนเกินความจำเป็น (over-engineering) — SOLID มีไว้แก้ปัญหาความซับซ้อนที่เพิ่มขึ้นตามขนาดจริง ไม่ใช่กฎที่ต้องท่องจำใช้ทุกกรณีโดยไม่คิด
- ออกแบบ Child Class ที่ผิดสัญญาของ Parent (ละเมิด LSP) โดยไม่รู้ตัว เช่น Child ที่โยน exception ในสถานการณ์ที่ Parent ไม่เคยโยน ทำให้โค้ดที่พึ่งพา Polymorphism พังโดยไม่คาดคิด
- สร้าง interface/Parent Class ที่ใหญ่เกินไปจนบังคับให้ Child Class ต้อง implement method ที่ไม่เกี่ยวข้องกับตัวเองเลย (ละเมิด ISP) เช่น บังคับให้สัตว์ทุกชนิดต้องมี `fly()` ทั้งที่มีแค่บางชนิดบินได้
- สร้าง object ที่ต้องพึ่งพา (dependency) ขึ้นมาเองข้างใน Class โดยตรง (เช่น `self.sender = EmailSender()`) แทนที่จะรับผ่านพารามิเตอร์ ทำให้ทดสอบยากและสลับ implementation ไม่ได้ (ละเมิด DIP)

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบาย SOLID Principles ทั้ง 5 ข้อโดยย่อ พร้อมยกตัวอย่างการละเมิดแต่ละข้อ
2. ทำไม Open/Closed Principle ถึงแนะนำให้ "เพิ่ม Class ใหม่" แทนการแก้ `if/elif` เดิม? มีข้อเสียอะไรบ้างถ้าใช้มากเกินไป
3. อธิบาย Liskov Substitution Principle ด้วยตัวอย่าง Rectangle/Square ว่าทำไมถึงเป็นตัวอย่างคลาสสิกของการละเมิด LSP
4. Dependency Injection คืออะไร เกี่ยวข้องกับ Dependency Inversion Principle อย่างไร
5. SOLID Principles ทั้งหมดพึ่งพาแนวคิด Inheritance และ Polymorphism (บทที่แล้ว) อย่างไรบ้าง

## แบบฝึกหัด (Practice Exercises)

1. หา Class หนึ่งในโค้ดที่เคยเขียนมา (หรือใน Mini Project ก่อนหน้า) ที่ละเมิด SRP แล้วลองแยกมันออกเป็นหลาย Class ตามหน้าที่ที่แท้จริง
2. เขียนระบบคำนวณค่าจัดส่งสินค้าที่ใช้ OCP: มี `ShippingStrategy` (abstraction) และ Child Class อย่างน้อย 3 แบบ (`StandardShipping`, `ExpressShipping`, `InternationalShipping`) ที่คำนวณค่าจัดส่งต่างกัน โดยฟังก์ชันหลักที่เรียกใช้ต้องไม่มี `if/elif` เช็คชนิดเลย
3. ระบุว่าโค้ดต่อไปนี้ละเมิด LSP หรือไม่ พร้อมอธิบายเหตุผล: Parent Class `FileStorage` มี method `save(data)` ที่คืนค่า `True` เสมอเมื่อสำเร็จ แต่ Child Class `ReadOnlyStorage` override `save()` ให้โยน exception ทุกครั้งที่ถูกเรียก

## Mini Project

**"ระบบประมวลผลคำสั่งซื้อออนไลน์ (Order Processing System)"**: เขียนโปรแกรมที่ยึดหลัก SOLID ทั้ง 5 ข้อ:
1. **SRP**: แยก Class `Order` (เก็บข้อมูลคำสั่งซื้อ), `OrderValidator` (ตรวจสอบความถูกต้อง), `PriceCalculator` (คำนวณราคารวม), `OrderRepository` (บันทึกข้อมูล) ออกจากกันชัดเจน
2. **OCP**: ออกแบบระบบคำนวณส่วนลดด้วย `DiscountStrategy` (abstraction) ที่รองรับส่วนลดหลายแบบ (ตามฤดูกาล, ตามสมาชิก, ตามโค้ดคูปอง) โดยเพิ่มแบบใหม่ได้โดยไม่แก้โค้ดเดิม
3. **LSP**: ออกแบบ `PaymentMethod` (abstraction) พร้อม Child Class `CreditCardPayment`, `BankTransferPayment`, `CashOnDeliveryPayment` ที่ทุกตัวแทนที่กันได้จริงโดยไม่ทำให้ระบบพัง
4. **DIP**: ให้ `OrderService` (ระดับสูง) รับ `PaymentMethod` และ `NotificationSender` (abstraction ทั้งคู่) ผ่าน constructor แทนการสร้างขึ้นเองข้างใน แล้วทดสอบสลับ implementation หลายแบบโดยไม่แก้ `OrderService` เลย

## ข้อคิดสำคัญ (Key Takeaways)

- SOLID คือชุดหลักการออกแบบ 5 ข้อ (SRP, OCP, LSP, ISP, DIP) ที่สร้างบนพื้นฐาน Encapsulation, Abstraction, Inheritance, และ Polymorphism ที่เรียนมาในสองบทก่อนหน้า
- SRP ทำให้แต่ละ Class มีเหตุผลเดียวที่ต้องแก้ไข ลดผลกระทบข้ามส่วนที่ไม่เกี่ยวข้องกัน
- OCP ทำให้ขยายความสามารถได้ด้วยการเพิ่ม Class ใหม่ผ่าน Polymorphism แทนการแก้โค้ดเดิมที่ทดสอบผ่านแล้ว
- LSP รับประกันว่า Child Class แทนที่ Parent ได้จริงโดยไม่ทำลายพฤติกรรมที่คาดหวัง — เป็นเงื่อนไขที่ทำให้ Polymorphism ใช้งานได้อย่างปลอดภัย
- ISP และ DIP ลดการผูกติดกันแน่นเกินไประหว่าง Class ทำให้ระบบทดสอบง่ายและสลับ implementation ได้โดยไม่กระทบโค้ดส่วนอื่น
- SOLID ไม่ใช่กฎตายตัวที่ต้องใช้ทุกกรณี — ควรใช้เมื่อความซับซ้อนของระบบเพิ่มขึ้นจริงๆ ไม่ใช่ใช้แบบสุดโต่งจนเกิด over-engineering ในโปรเจกต์เล็กๆ
- บทถัดไปจะเรียน Design Patterns ซึ่งเป็นรูปแบบการแก้ปัญหาการออกแบบที่พบซ้ำๆ บ่อยครั้ง โดยหลาย pattern สร้างขึ้นบนพื้นฐานของ SOLID Principles ที่เรียนในบทนี้โดยตรง (เช่น Strategy Pattern ที่ตัวอย่าง `DiscountStrategy` ในบทนี้ก็คือรูปแบบหนึ่งของมัน)

---
*บทต่อไป (บทที่ 33): Design Patterns — พิมพ์ "Next" เพื่อดำเนินการต่อ*
