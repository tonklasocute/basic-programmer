# บทที่ 48: MVC, Layered, Hexagonal

## แนวคิดหลัก (Concept)

บทที่แล้วสอนหลักการ Clean Architecture ในระดับแนวคิด (dependency ชี้เข้าหาศูนย์กลาง) บทนี้แนะนำ**สถาปัตยกรรมที่เป็นรูปธรรม 3 แบบ**ที่นำหลักการนั้นไปปฏิบัติจริงในรูปแบบต่างๆ กัน — แต่ละแบบมีจุดเน้นและความเข้มงวดของการแยกชั้นต่างกัน ตั้งแต่แบบที่นิยมที่สุดและเข้าใจง่าย (MVC) ไปจนถึงแบบที่เข้มงวดที่สุด (Hexagonal)

- **MVC (Model-View-Controller)** — แบ่งโค้ดเป็น 3 ส่วน: **Model** (ข้อมูลและ business logic), **View** (การแสดงผลต่อผู้ใช้), **Controller** (รับ input จากผู้ใช้แล้วประสานงานระหว่าง Model กับ View)
- **Layered Architecture (สถาปัตยกรรมแบบชั้น)** — แบ่งโค้ดเป็นชั้นแนวนอนตามหน้าที่ (เช่น Presentation -> Business Logic -> Data Access) แต่ละชั้นเรียกใช้ได้เฉพาะชั้นที่อยู่ต่ำกว่าตัวเองเท่านั้น
- **Hexagonal Architecture (Ports and Adapters)** — สถาปัตยกรรมที่วาง Business Logic ไว้ตรงกลางเป็น "หกเหลี่ยม" ล้อมรอบด้วย **Port** (interface ที่กำหนดว่า Business Logic ต้องการอะไรจากภายนอก) และ **Adapter** (implementation จริงที่เชื่อมต่อกับเทคโนโลยีภายนอก เช่น ฐานข้อมูล, Web API)

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **MVC มีไว้แยกความกังวลระหว่าง "ข้อมูล" กับ "การแสดงผล" ซึ่งเป็นปัญหาคลาสสิกที่สุดของการพัฒนาแอปที่มี UI**: ถ้าปนโค้ดแสดงผล (HTML) กับ logic คำนวณข้อมูลไว้ในที่เดียวกัน การเปลี่ยนหน้าตา UI (เช่น ปรับ layout) เสี่ยงกระทบ logic การคำนวณโดยไม่ตั้งใจ — MVC แยกทั้งสองส่วนออกจากกันชัดเจน ทำให้แก้ไขแยกกันได้อย่างปลอดภัย (เชื่อมโยงกับ SRP บทที่ 32)
- **Layered Architecture มีไว้จัดระเบียบการไหลของข้อมูลให้คาดเดาได้**: เมื่อโปรเจกต์ใหญ่ขึ้น ถ้าทุกส่วนของโค้ดเรียกกันไปมาแบบไม่มีทิศทาง (ทุกอย่างเรียกทุกอย่างได้) จะติดตามยากว่าอะไรพึ่งพาอะไร — การบังคับให้แต่ละชั้นเรียกได้แค่ชั้นล่างกว่าเท่านั้น สร้างกฎที่ชัดเจนและป้องกันการพึ่งพาแบบวนกลับไปกลับมา (circular dependency)
- **Hexagonal Architecture มีไว้ทดสอบ Business Logic ได้โดยไม่ต้องมีฐานข้อมูลหรือ Web Framework จริง**: MVC และ Layered ยังคงมีทิศทางที่ Business Logic อาจถูกเรียกจาก Controller/Presentation Layer ที่ผูกกับ framework เฉพาะ — Hexagonal เอา Business Logic ไว้ตรงกลางแท้จริงและกำหนด Port ที่ทุกการเชื่อมต่อจากภายนอก (ไม่ว่าจะเป็น Web, CLI, Database, Test) ต้องผ่าน Adapter เดียวกันเสมอ ทำให้สลับหรือทดสอบส่วนไหนก็ได้อย่างอิสระอย่างแท้จริง

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **MVC** เหมือน **ร้านอาหารที่แยกครัว (Model) เมนู/การจัดโต๊ะ (View) และพนักงานเสิร์ฟ (Controller) ออกจากกัน** — เปลี่ยนการตกแต่งร้าน (View) ไม่กระทบสูตรอาหารในครัว (Model) เลย พนักงานเสิร์ฟ (Controller) เป็นตัวกลางที่รับออเดอร์จากลูกค้าแล้วส่งต่อให้ครัว จากนั้นนำอาหารมาเสิร์ฟ
- **Layered Architecture** เหมือน **สายการผลิตในโรงงานที่วัตถุดิบไหลไปทางเดียว**: วัตถุดิบ -> แปรรูป -> บรรจุภัณฑ์ -> จัดส่ง — แผนกบรรจุภัณฑ์ไม่มีทางส่งงานย้อนกลับไปแผนกวัตถุดิบได้ ทุกอย่างไหลไปข้างหน้าตามลำดับชั้นที่กำหนดไว้ชัดเจน
- **Hexagonal Architecture** เหมือน **ปลั๊กไฟมาตรฐานที่อุปกรณ์ไฟฟ้าใดๆ ก็เสียบใช้ได้** (ทบทวนจากตัวอย่าง DIP บทที่ 32) — ตัวเครื่องใช้ไฟฟ้า (Business Logic) ไม่สนใจว่าไฟมาจากโรงไฟฟ้าถ่านหินหรือโซลาร์เซลล์ (Adapter ต่างๆ) ขอแค่เสียบผ่านปลั๊กมาตรฐาน (Port) เดียวกันก็ใช้งานได้เหมือนกันหมด

## แผนภาพอธิบาย (Visual explanation)

### MVC — การไหลของข้อมูลเมื่อผู้ใช้กระทำบางอย่าง

```
ผู้ใช้คลิกปุ่ม "ซื้อสินค้า"
        |
        v
   Controller  <-- รับ input, ตัดสินใจว่าต้องทำอะไร
        |
        v
     Model  <-- ประมวลผล business logic (เช่น ตรวจสต็อก, คำนวณราคา, บันทึกฐานข้อมูล)
        |
        v
   Controller  <-- รับผลลัพธ์จาก Model
        |
        v
      View  <-- แสดงผลลัพธ์กลับให้ผู้ใช้เห็น (เช่น "ซื้อสำเร็จ")

Model ไม่รู้จัก View เลย, View ไม่มี business logic เลย, Controller เป็นแค่ตัวกลางประสานงาน
```

### Layered Architecture — ข้อมูลไหลทางเดียวผ่านชั้นต่างๆ

```
┌─────────────────────────────────────┐
│   Presentation Layer                  │   <- รับ request, ส่ง response (เช่น REST API endpoint)
└─────────────────┬─────────────────────┘
                   │ เรียกลงมาชั้นล่างเท่านั้น
                   v
┌─────────────────────────────────────┐
│   Business Logic Layer (Service)      │   <- กฎทางธุรกิจ, การคำนวณ, validation
└─────────────────┬─────────────────────┘
                   │ เรียกลงมาชั้นล่างเท่านั้น
                   v
┌─────────────────────────────────────┐
│   Data Access Layer (Repository)      │   <- คุยกับฐานข้อมูลโดยตรง
└─────────────────────────────────────┘

Presentation Layer ห้ามคุยกับ Data Access Layer ตรงๆ ต้องผ่าน Business Logic Layer เสมอ
(ป้องกันการข้ามชั้นที่ทำให้ dependency สับสนไม่มีทิศทาง)
```

### Hexagonal Architecture — Business Logic ตรงกลาง ล้อมด้วย Port/Adapter

```
        Web Adapter          CLI Adapter          Test Adapter
             │                    │                     │
             v                    v                     v
        ┌────────────── Primary Port ──────────────┐
        │                                            │
        │            Business Logic (แกนกลาง)          │
        │         ไม่รู้จักเทคโนโลยีภายนอกใดๆ เลย          │
        │                                            │
        └────────────── Secondary Port ─────────────┘
             ^                    ^                     ^
             │                    │                     │
       SQL Adapter          Email Adapter        Payment Adapter

Primary Port: ทางเข้าสู่ระบบ (ผู้ใช้เรียกระบบผ่านทางไหนก็ได้ Web/CLI/Test)
Secondary Port: ทางออกจากระบบ (ระบบเรียกใช้บริการภายนอกผ่านทางไหนก็ได้ SQL/Email/Payment)
Business Logic กลางไม่รู้จัก Adapter ตัวไหนเลย รู้จักแค่ Port (abstraction) เท่านั้น
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== MVC =====
class OrderModel:                                    # Model: business logic ล้วนๆ ไม่รู้จัก View เลย
    def __init__(self, items, stock_checker):
        self.items = items
        self.stock_checker = stock_checker

    def place_order(self):
        for item in self.items:
            if not self.stock_checker.has_stock(item):
                return {"success": False, "reason": f"{item} out of stock"}
        return {"success": True, "total": sum(i.price for i in self.items)}

class OrderView:                                       # View: แสดงผลล้วนๆ ไม่มี business logic เลย
    def render_success(self, total):
        return f"Order placed! Total: {total}"

    def render_failure(self, reason):
        return f"Order failed: {reason}"

class OrderController:                                  # Controller: ประสานงาน Model กับ View
    def __init__(self, model, view):
        self.model = model
        self.view = view

    def handle_order_request(self):
        result = self.model.place_order()
        if result["success"]:
            return self.view.render_success(result["total"])
        return self.view.render_failure(result["reason"])

# ===== Layered Architecture =====
class OrderRepository:                                   # Data Access Layer
    def save(self, order_data):
        print(f"Saving to database: {order_data}")

class OrderService:                                        # Business Logic Layer
    def __init__(self, repository: OrderRepository):
        self.repository = repository

    def create_order(self, items):
        total = sum(item.price for item in items)          # business logic
        self.repository.save({"items": items, "total": total})  # เรียกชั้นล่างเท่านั้น
        return total

class OrderAPI:                                              # Presentation Layer
    def __init__(self, service: OrderService):
        self.service = service

    def post_order(self, request_data):                       # รับ request, เรียก Business Logic Layer เท่านั้น
        total = self.service.create_order(request_data["items"])
        return {"status": 200, "total": total}                   # ไม่คุยกับ Repository ตรงๆ เด็ดขาด

# ===== Hexagonal Architecture =====
from abc import ABC, abstractmethod

class NotificationPort(ABC):                                 # Secondary Port (abstraction)
    @abstractmethod
    def notify(self, message): ...

class OrderBusinessLogic:                                      # แกนกลาง: ไม่รู้จัก Adapter ใดๆ เลย
    def __init__(self, notifier: NotificationPort):              # พึ่งพา Port เท่านั้น (DIP บทที่ 32)
        self.notifier = notifier

    def complete_order(self, order_id):
        # ... business logic การปิดออเดอร์ ...
        self.notifier.notify(f"Order {order_id} completed!")       # เรียกผ่าน Port ไม่รู้ว่าจะไปที่ Email/SMS/Log

class EmailAdapter(NotificationPort):                            # Adapter จริงตัวที่ 1
    def notify(self, message):
        print(f"[Email] {message}")

class SmsAdapter(NotificationPort):                              # Adapter จริงตัวที่ 2 - สลับใช้ได้ทันที
    def notify(self, message):
        print(f"[SMS] {message}")

logic = OrderBusinessLogic(notifier=EmailAdapter())              # ใช้ Email
logic.complete_order(101)
logic2 = OrderBusinessLogic(notifier=SmsAdapter())                 # เปลี่ยนเป็น SMS โดยไม่แก้ OrderBusinessLogic เลย
logic2.complete_order(102)
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `OrderController.handle_order_request()` — เป็นจุดเดียวที่ **Model และ View มาเจอกัน** Controller เรียก `self.model.place_order()` เพื่อประมวลผล business logic แล้วนำผลลัพธ์ไปส่งต่อให้ `self.view` เพื่อแสดงผล — สังเกตว่า **`OrderModel` ไม่มีการอ้างอิงถึง `OrderView` เลยแม้แต่น้อย** และ `OrderView` ก็ไม่มี logic การตรวจสต็อกหรือคำนวณราคาเลย ทั้งสองฝั่งถูกแยกออกจากกันอย่างสมบูรณ์โดยมี Controller เป็นสะพานเชื่อม
2. `OrderAPI.post_order()` (Presentation Layer) — เรียก `self.service.create_order(...)` (Business Logic Layer) เท่านั้น **ไม่มีทางเรียก `OrderRepository` ตรงๆ ได้เลยตามกฎของ Layered Architecture** แม้จะสะดวกกว่าถ้าทำได้ก็ตาม — การบังคับกฎนี้ทำให้ business logic (การคำนวณ `total`) อยู่ในที่เดียวเสมอ (`OrderService`) ไม่กระจัดกระจายไปตาม Presentation Layer หลายจุดที่อาจลืมคำนวณให้ตรงกัน
3. `OrderBusinessLogic.__init__(self, notifier: NotificationPort)` — รับ `notifier` เป็น **Port** (abstract class ที่นิยามด้วย `ABC`/`abstractmethod`) ไม่ใช่ `EmailAdapter` หรือ `SmsAdapter` โดยตรง — เมื่อเรียก `self.notifier.notify(...)` ข้างใน `complete_order()` โค้ดนี้**ไม่มีทางรู้เลยว่าข้อความจะไปลง Email หรือ SMS จริงๆ** เพราะรู้จักแค่ว่า `notifier` มี method `notify()` ตาม Port ที่กำหนดไว้เท่านั้น
4. `logic = OrderBusinessLogic(notifier=EmailAdapter())` เทียบกับ `logic2 = OrderBusinessLogic(notifier=SmsAdapter())` — แสดงให้เห็นว่า**การสลับ Adapter ทำได้ทันทีโดยไม่ต้องแก้ `OrderBusinessLogic` แม้แต่บรรทัดเดียว** เพราะทั้ง `EmailAdapter` และ `SmsAdapter` implement `NotificationPort` เดียวกัน (ตรงตาม Liskov Substitution Principle จากบทที่ 32 ด้วย) — นี่คือจุดที่ Hexagonal Architecture เข้มงวดกว่า MVC/Layered ตรงที่ **แม้แต่การแจ้งเตือนก็ยังผ่าน Port/Adapter** ไม่ใช่แค่ฐานข้อมูลเท่านั้น

## Time Complexity

MVC, Layered, และ Hexagonal เป็นรูปแบบการจัดโครงสร้างโค้ด ไม่ใช่อัลกอริทึม จึงไม่มี Big O — ผลกระทบที่วัดได้คือ**จำนวนชั้นที่ข้อมูลต้องเดินทางผ่าน**ก่อนถึงปลายทาง:
- MVC: request ผ่าน Controller -> Model -> Controller -> View (4 จุด)
- Layered: request ผ่านทุกชั้นเรียงลำดับ (Presentation -> Business -> Data Access) โดยไม่ข้ามชั้นได้
- Hexagonal: request ผ่าน Adapter -> Port -> Business Logic -> Port -> Adapter (มีชั้น abstraction เพิ่มจาก Port ทั้งสองฝั่ง)

ยิ่งมีชั้นมากขึ้น ยิ่งมี**overhead เล็กน้อยจากการเรียกฟังก์ชันเพิ่ม** แต่แลกมาด้วยความสามารถในการดูแลรักษาและทดสอบที่ดีขึ้นมาก — เป็น trade-off ระหว่างประสิทธิภาพเล็กน้อยกับ maintainability ที่คุ้มค่าในโปรเจกต์ขนาดกลางถึงใหญ่

## Space Complexity

จำนวน Class/Interface ที่เพิ่มขึ้นจากการแบ่งชั้น (Model+View+Controller หรือ Port+Adapter หลายตัว) ใช้หน่วยความจำเพิ่มขึ้นเล็กน้อยที่ **ไม่มีนัยสำคัญ** เมื่อเทียบกับข้อมูลจริงที่โปรแกรมประมวลผล

## แนวทางปฏิบัติที่ดี (Best Practices)

- เลือก **MVC** สำหรับแอปที่มี UI ชัดเจน (เว็บแอป, mobile app) เพราะเข้าใจง่ายและมี framework รองรับมากมาย (Django, Rails, Spring MVC)
- เลือก **Layered Architecture** สำหรับระบบ backend ทั่วไปที่ไม่มี UI ซับซ้อน (เช่น REST API service) เพราะกฎ "เรียกลงชั้นล่างเท่านั้น" เข้าใจและบังคับใช้ง่าย
- เลือก **Hexagonal Architecture** เมื่อ Business Logic ซับซ้อนมากและต้องการทดสอบแยกจากเทคโนโลยีภายนอกอย่างเข้มงวด (เช่น ระบบการเงิน, ระบบที่มีโอกาสเปลี่ยนฐานข้อมูล/framework บ่อย) แต่ตระหนักว่ามีความซับซ้อนเพิ่มขึ้นจากการนิยาม Port จำนวนมาก — ไม่คุ้มค่าสำหรับโปรเจกต์เล็กๆ

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ใส่ business logic ไว้ใน Controller (MVC) หรือ Presentation Layer (Layered) โดยตรง แทนที่จะอยู่ใน Model/Business Logic Layer ทำให้ผิดหลักการที่ตั้งใจแยกไว้ตั้งแต่ต้น (Controller/Presentation ควรเป็นแค่ทางผ่าน ไม่ใช่ที่อยู่ของ logic)
- ให้ Presentation Layer เรียก Data Access Layer ข้ามชั้น Business Logic Layer ไปตรงๆ เพื่อความสะดวก ทำให้ business logic กระจัดกระจายและตรวจสอบความถูกต้องไม่ครบถ้วน
- ใช้ Hexagonal Architecture ในโปรเจกต์ขนาดเล็กที่ไม่มีความซับซ้อนพอ ทำให้ต้องเขียน Port/Adapter จำนวนมากโดยไม่ได้ประโยชน์คุ้มค่า (over-engineering เหมือนที่เตือนซ้ำหลายบทก่อนหน้า)
- สร้าง Model ที่มีการอ้างอิงกลับไปยัง View หรือ Controller (circular dependency) ทำลายจุดประสงค์หลักของการแยกชั้นที่ต้องการให้ Model เป็นอิสระจากการแสดงผล

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบาย MVC ทั้งสามส่วน พร้อมยกตัวอย่างว่าอะไรควรอยู่ใน Model และอะไรไม่ควรอยู่ใน Controller
2. Layered Architecture มีกฎอะไรที่ป้องกันไม่ให้เกิด dependency ที่สับสน
3. Hexagonal Architecture ต่างจาก Layered Architecture อย่างไร ทำไมถึงเรียกว่า "Ports and Adapters"
4. ทำไม Business Logic ใน Hexagonal Architecture ถึงทดสอบได้ง่ายกว่าสถาปัตยกรรมอื่น
5. ถ้าต้องเลือกสถาปัตยกรรมสำหรับ REST API backend ธรรมดาที่ไม่ซับซ้อนมาก ควรเลือกแบบไหน เพราะอะไร

## แบบฝึกหัด (Practice Exercises)

1. ปรับโค้ด MVC ในตัวอย่างให้เพิ่ม `PaymentModel` ที่ตรวจสอบการชำระเงินก่อนสร้างออเดอร์ โดยยังคงแยก Model/View/Controller อย่างถูกต้อง
2. ออกแบบ Layered Architecture สำหรับระบบจองห้องประชุม (Presentation, Business Logic, Data Access) พร้อมพิสูจน์ว่า Presentation Layer ไม่มีทางเรียก Data Access Layer ข้ามชั้นได้
3. เขียน Hexagonal Architecture สำหรับระบบแจ้งเตือน ที่มี Port `StoragePort` (บันทึกข้อมูล) และ Adapter อย่างน้อย 2 แบบ (`FileStorageAdapter`, `DatabaseStorageAdapter`) แล้วพิสูจน์ว่าสลับใช้งานได้โดยไม่แก้ Business Logic

## Mini Project

**"ระบบจัดการงานที่ออกแบบได้ทั้ง 3 สถาปัตยกรรม (Task Manager: MVC vs Layered vs Hexagonal Comparison)"**: เขียนโปรแกรมที่:
1. ออกแบบระบบจัดการงาน (Task Manager) เดียวกันด้วยทั้ง 3 สถาปัตยกรรมแยกกัน (โฟลเดอร์แยกกัน) — ฟีเจอร์เหมือนกันทุกประการ (เพิ่มงาน, ทำเครื่องหมายเสร็จ, ลบงาน, แสดงรายการ)
2. เขียนเทสต์ชุดเดียวกันสำหรับ Business Logic ของทั้ง 3 แบบ แล้วเปรียบเทียบว่าแบบไหนทดสอบง่ายที่สุด/ยากที่สุด และทำไม
3. จำลองการเปลี่ยนแปลง 2 อย่าง: (ก) เปลี่ยนจาก console UI เป็น web UI (ข) เปลี่ยนจากเก็บข้อมูลในไฟล์เป็นฐานข้อมูล SQL แล้ววัดว่าแต่ละสถาปัตยกรรมต้องแก้โค้ดกี่ไฟล์/กี่บรรทัดสำหรับการเปลี่ยนแปลงแต่ละอย่าง
4. เขียนรายงานสรุปเปรียบเทียบทั้ง 3 สถาปัตยกรรม พร้อมข้อสรุปว่าแบบไหนเหมาะกับสถานการณ์ไหนจากประสบการณ์จริงที่ได้ลองทำ

## ข้อคิดสำคัญ (Key Takeaways)

- MVC แยก Model (ข้อมูล/logic), View (แสดงผล), และ Controller (ประสานงาน) ออกจากกัน เหมาะกับแอปที่มี UI ชัดเจน
- Layered Architecture แบ่งเป็นชั้นแนวนอนที่เรียกได้แค่ชั้นล่างกว่าเท่านั้น สร้างทิศทางการไหลของข้อมูลที่คาดเดาได้
- Hexagonal Architecture วาง Business Logic ไว้ตรงกลางล้อมด้วย Port/Adapter ทำให้ทดสอบและสลับเทคโนโลยีภายนอกได้อย่างอิสระที่สุด แต่มีความซับซ้อนสูงสุดด้วย
- ทั้งสามสถาปัตยกรรมล้วนสร้างบนหลักการเดียวกันจากบทที่แล้ว (Clean Architecture, Dependency Inversion จากบทที่ 32) เพียงมีระดับความเข้มงวดในการแยกชั้นต่างกัน
- ไม่มีสถาปัตยกรรมไหนดีที่สุดเสมอ — เลือกตามความซับซ้อนของ Business Logic และแนวโน้มการเปลี่ยนแปลงเทคโนโลยีในอนาคตของโปรเจกต์นั้นๆ
- บทถัดไปจะเรียน Microservices vs Monolith, Event Driven, DDD ซึ่งขยายจากสถาปัตยกรรมภายในโปรเจกต์เดียว ไปสู่การออกแบบระบบที่ประกอบด้วยหลายบริการทำงานร่วมกัน ปิดท้าย Part 12: Software Engineering

---
*บทต่อไป (บทที่ 49): Microservices vs Monolith, Event Driven, DDD — พิมพ์ "Next" เพื่อดำเนินการต่อ*
