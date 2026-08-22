# บทที่ 33: Design Patterns

## แนวคิดหลัก (Concept)

บทที่แล้วปิดท้ายด้วยการชี้ให้เห็นว่า `DiscountStrategy` ที่เขียนไว้เป็นตัวอย่างของ OCP นั้น แท้จริงคือรูปแบบการออกแบบที่มีชื่อเรียกอย่างเป็นทางการอยู่แล้ว — **Design Pattern** คือ**วิธีแก้ปัญหาการออกแบบที่พบซ้ำๆ บ่อยครั้ง**ในงานซอฟต์แวร์ ซึ่งถูกตั้งชื่อและบันทึกไว้เป็นมาตรฐานให้วิศวกรทั่วโลกสื่อสารกันได้โดยไม่ต้องอธิบายรายละเอียดใหม่ทุกครั้ง บทนี้เป็นบทปิดท้าย **Part 6: OOP** โดยแนะนำ Design Pattern คลาสสิกที่พบบ่อยที่สุด 3 กลุ่ม

- **Design Pattern** — เทมเพลตวิธีแก้ปัญหาการออกแบบที่นำกลับมาใช้ซ้ำได้ ไม่ใช่โค้ดสำเร็จรูป แต่เป็น "แนวคิดโครงสร้าง" ที่ปรับใช้ได้กับสถานการณ์ต่างๆ
- **Creational Patterns (รูปแบบการสร้าง)** — เกี่ยวกับ**วิธีสร้าง object** ให้ยืดหยุ่นและควบคุมได้ เช่น **Singleton** (รับประกันว่ามี object แค่ตัวเดียวในทั้งโปรแกรม) และ **Factory** (มอบหมายการสร้าง object ให้ฟังก์ชัน/Class กลางจัดการแทนการเรียก constructor ตรงๆ)
- **Structural Patterns (รูปแบบโครงสร้าง)** — เกี่ยวกับ**วิธีจัดองค์ประกอบของ Class ให้ทำงานร่วมกัน** เช่น **Adapter** (แปลง interface หนึ่งให้เข้ากับอีก interface หนึ่งที่โค้ดคาดหวัง)
- **Behavioral Patterns (รูปแบบพฤติกรรม)** — เกี่ยวกับ**วิธีที่ object สื่อสารและกระจายความรับผิดชอบกัน** เช่น **Strategy** (สลับพฤติกรรม/อัลกอริทึมได้ที่ runtime — ที่เห็นแล้วในบทที่แล้ว) และ **Observer** (แจ้งเตือน object หลายตัวโดยอัตโนมัติเมื่อมีเหตุการณ์เกิดขึ้น)

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **มีไว้เป็นภาษากลางระหว่างวิศวกรซอฟต์แวร์**: เมื่อบอกเพื่อนร่วมทีมว่า "ใช้ Singleton ตรงนี้" หรือ "ทำเป็น Observer Pattern" ทุกคนเข้าใจโครงสร้างและ trade-off ทันทีโดยไม่ต้องอธิบายรายละเอียดยาวๆ ใหม่ทุกครั้ง — เหมือนคำศัพท์เฉพาะทางในวิชาชีพอื่น (เช่น สถาปนิกพูดว่า "cantilever" ก็เข้าใจตรงกันทันที)
- **แต่ละ pattern คือคำตอบที่ผ่านการพิสูจน์แล้วจากปัญหาที่เกิดซ้ำในหลายโปรเจกต์**: แทนที่จะคิดค้นวิธีแก้ปัญหาการออกแบบใหม่ทุกครั้ง (ซึ่งมักนำไปสู่ข้อผิดพลาดที่คนอื่นเคยเจอมาแล้ว) การใช้ pattern ที่ผ่านการทดสอบมานานทำให้หลีกเลี่ยงกับดักที่รู้จักกันดีอยู่แล้ว
- **สร้างขึ้นบนพื้นฐานของ SOLID Principles (บทที่แล้ว) โดยตรง**: Strategy Pattern คือการนำ OCP ไปปฏิบัติจริง, Dependency Injection ที่ใช้ใน DIP มักปรากฏในรูปแบบของ Factory Pattern — การเข้าใจ pattern ช่วยให้เห็นภาพว่า SOLID Principles ถูกนำไปใช้แก้ปัญหาจริงอย่างไร ไม่ใช่แค่ทฤษฎีลอยๆ

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Singleton** เหมือน **ตำแหน่งนายกรัฐมนตรีของประเทศ** — ไม่ว่าใครต้องการติดต่อ "นายกรัฐมนตรี" กี่ครั้ง ก็มีคนเดียวกันเสมอที่ดำรงตำแหน่งนี้ ไม่มีทางมีนายกฯ สองคนพร้อมกันในเวลาเดียวกัน
- **Factory** เหมือน **โรงงานผลิตรถยนต์ที่รับออเดอร์แล้วเลือกสายการผลิตให้เอง** — ลูกค้าสั่ง "รถ SUV" โดยไม่ต้องรู้รายละเอียดว่าโรงงานประกอบรถรุ่นนั้นอย่างไร โรงงาน (Factory) จัดการเลือกกระบวนการที่ถูกต้องให้เองตามคำสั่งที่ได้รับ
- **Adapter** เหมือน **หัวปลั๊กแปลงไฟสำหรับนักท่องเที่ยวต่างประเทศ** — อุปกรณ์ไฟฟ้าจากประเทศหนึ่ง (interface หนึ่ง) เสียบเข้ากับปลั๊กไฟของอีกประเทศ (interface ที่ระบบคาดหวัง) ไม่ได้โดยตรง ต้องมีตัวแปลง (Adapter) คั่นกลางเพื่อให้ทำงานร่วมกันได้
- **Strategy** เหมือน **แอปนำทางที่เลือกโหมดเดินทางได้ (รถยนต์, เดินเท้า, ขนส่งสาธารณะ)** — ปลายทางเดียวกัน แต่สลับ "วิธี" คำนวณเส้นทางได้ตามต้องการโดยไม่ต้องเปลี่ยนแอป
- **Observer** เหมือน **ระบบแจ้งเตือนของช่อง YouTube ที่เราติดตาม (subscribe)** — เมื่อเจ้าของช่องอัปโหลดวิดีโอใหม่ (เหตุการณ์เกิดขึ้น) ผู้ติดตามทุกคนได้รับการแจ้งเตือนอัตโนมัติ โดยเจ้าของช่องไม่ต้องรู้เลยว่าใครติดตามอยู่บ้างกี่คน

## แผนภาพอธิบาย (Visual explanation)

### Singleton — รับประกันมี instance เดียวเสมอในทั้งโปรแกรม

```
config1 = AppConfig()          config2 = AppConfig()          config3 = AppConfig()
     |                              |                              |
     v                              v                              v
     ทั้งสามตัวแปรชี้ไปที่ object เดียวกันจริงๆ ในหน่วยความจำ (ไม่ใช่สร้างใหม่ทุกครั้ง)

config1 is config2   ->  True
config1 is config3   ->  True

ถ้า config1.set("debug", True) -> config2.get("debug") ก็เห็นค่า True ด้วย (object เดียวกัน)
```

### Factory — ซ่อนตรรกะการเลือกสร้าง object ไว้ในที่เดียว

```
ไม่ใช้ Factory (โค้ดผู้เรียกต้องรู้รายละเอียดการสร้างเอง):

if payment_type == "credit_card":
    payment = CreditCardPayment(gateway_config, retry_count, timeout)
elif payment_type == "bank_transfer":
    payment = BankTransferPayment(bank_api_key, account_number)
   (ผู้เรียกต้องรู้พารามิเตอร์การสร้างของทุกชนิด!)

ใช้ Factory (ผู้เรียกแค่บอกชนิด ไม่ต้องรู้รายละเอียดการสร้าง):

payment = PaymentFactory.create(payment_type)     <- แค่บรรทัดเดียว
   (รายละเอียดการสร้างทั้งหมดถูกซ่อนไว้ใน PaymentFactory)
```

### Strategy — สลับอัลกอริทึมที่ runtime (ทบทวนจากบทที่แล้ว)

```
context = ShoppingCart(discount_strategy=RegularDiscount())
context.checkout(1000)     -> ใช้ RegularDiscount

context.discount_strategy = VipDiscount()      <- สลับ strategy ที่ runtime ได้ทันที
context.checkout(1000)     -> ใช้ VipDiscount แทน โดยไม่ต้องแก้โค้ด ShoppingCart เลย
```

### Observer — แจ้งเตือนหลาย object อัตโนมัติเมื่อมีเหตุการณ์

```
YouTubeChannel (Subject)              Subscribers (Observers)
     |                                    - Alice
     | upload_video("New tutorial!")      - Bob
     v                                    - Charlie
notify_all()  ---------------------->  ทุกคนได้รับ update("New tutorial!") พร้อมกัน

ถ้า Dave subscribe เพิ่มทีหลัง -> เข้าร่วม list ของ Observers โดย Subject ไม่ต้องแก้โค้ดเลย
ถ้า Alice unsubscribe -> ออกจาก list โดย Observer อื่นไม่ถูกกระทบ
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== Singleton — รับประกันมี instance เดียวเสมอ =====
class AppConfig:
    _instance = None                              # เก็บ instance เดียวไว้ระดับ Class

    def __new__(cls):
        if cls._instance is None:                   # ถ้ายังไม่เคยสร้าง -> สร้างครั้งแรกเท่านั้น
            cls._instance = super().__new__(cls)
            cls._instance.settings = {}
        return cls._instance                          # ครั้งต่อไป -> คืน instance เดิมเสมอ

config1 = AppConfig()
config2 = AppConfig()
config1.settings["debug"] = True
print(config2.settings["debug"])          # True -> config1 กับ config2 คือ object เดียวกันจริงๆ
print(config1 is config2)                   # True

# ===== Factory — มอบหมายการสร้าง object ให้ Class กลาง =====
class CreditCardPayment:
    def pay(self, amount):
        return f"Paid {amount} via Credit Card"

class BankTransferPayment:
    def pay(self, amount):
        return f"Paid {amount} via Bank Transfer"

class PaymentFactory:
    @staticmethod
    def create(payment_type):                     # ผู้เรียกไม่ต้องรู้ constructor ของแต่ละชนิดเลย
        if payment_type == "credit_card":
            return CreditCardPayment()
        elif payment_type == "bank_transfer":
            return BankTransferPayment()
        raise ValueError(f"Unknown payment type: {payment_type}")

payment = PaymentFactory.create("credit_card")
print(payment.pay(500))                     # "Paid 500 via Credit Card"

# ===== Strategy — สลับอัลกอริทึมที่ runtime (ต่อยอดจากบทที่แล้ว) =====
class ShoppingCart:
    def __init__(self, discount_strategy):
        self.discount_strategy = discount_strategy

    def checkout(self, price):
        return self.discount_strategy.calculate(price)

# ===== Observer — แจ้งเตือนหลาย object อัตโนมัติ =====
class YouTubeChannel:
    def __init__(self, name):
        self.name = name
        self._subscribers = []                      # เก็บรายชื่อ observer ทั้งหมด

    def subscribe(self, observer):
        self._subscribers.append(observer)

    def unsubscribe(self, observer):
        self._subscribers.remove(observer)

    def upload_video(self, title):
        for subscriber in self._subscribers:           # แจ้งเตือนทุกคนโดยอัตโนมัติ
            subscriber.update(self.name, title)

class Subscriber:
    def __init__(self, name):
        self.name = name

    def update(self, channel_name, video_title):
        print(f"{self.name} got notified: {channel_name} uploaded '{video_title}'")

channel = YouTubeChannel("CodeWithThai")
alice = Subscriber("Alice")
bob = Subscriber("Bob")
channel.subscribe(alice)
channel.subscribe(bob)
channel.upload_video("Design Patterns Explained")
# Alice got notified: CodeWithThai uploaded 'Design Patterns Explained'
# Bob got notified: CodeWithThai uploaded 'Design Patterns Explained'
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `AppConfig.__new__(cls)` — Python เรียก `__new__` ก่อน `__init__` เสมอเมื่อสร้าง object ใหม่ (เชื่อมโยงบทที่ 30) โค้ดเช็ค `cls._instance is None` ก่อนเสมอ: **ถ้ายังไม่เคยสร้างมาก่อน** (`None`) จึงสร้างจริงด้วย `super().__new__(cls)` แล้วเก็บไว้ที่ `cls._instance` (attribute ระดับ Class ที่ทุก instance เข้าถึงร่วมกัน) **ถ้าเคยสร้างแล้ว** จะคืนค่า `cls._instance` เดิมทันที โดยไม่สร้างใหม่เลย — นี่คือกลไกที่รับประกันว่ามี instance เดียวเสมอไม่ว่าจะเรียก `AppConfig()` กี่ครั้งก็ตาม
2. `PaymentFactory.create(payment_type)` — ใช้ `@staticmethod` (method ที่ไม่ต้องพึ่ง `self` เพราะไม่เกี่ยวกับ instance ใดโดยเฉพาะ) รับ string บอกชนิด แล้วตัดสินใจว่าจะสร้าง object ชนิดไหนคืนกลับไป — ผู้เรียกโค้ด (`PaymentFactory.create("credit_card")`) **ไม่จำเป็นต้องรู้เลยว่า `CreditCardPayment` มี constructor รับพารามิเตอร์อะไรบ้าง** เพราะรายละเอียดทั้งหมดถูกซ่อนไว้ใน Factory
3. `ShoppingCart(discount_strategy)` — เก็บ `discount_strategy` ไว้เป็น attribute แล้วเมื่อ `checkout()` ถูกเรียก จะส่งต่อการคำนวณให้ `self.discount_strategy.calculate(price)` ทำงาน — ถ้าต้องการเปลี่ยนพฤติกรรมการคิดส่วนลด **แค่เปลี่ยนค่า `discount_strategy` (สลับ object) โดยไม่ต้องแก้โค้ดใน `ShoppingCart` เลยแม้แต่บรรทัดเดียว** ตรงตามหลัก OCP จากบทที่แล้ว
4. `channel.upload_video("...")` — loop ผ่าน `self._subscribers` (list ที่เก็บ observer ทุกตัวที่ subscribe ไว้) แล้วเรียก `subscriber.update(...)` กับทุกตัวโดยอัตโนมัติ — `YouTubeChannel` (Subject) **ไม่จำเป็นต้องรู้ว่ามี Subscriber กี่คนหรือใครบ้าง** แค่รู้ว่าต้องเรียก `update()` กับทุกตัวใน list เมื่อมีเหตุการณ์เกิดขึ้น — การเพิ่ม/ลบ subscriber ทำได้อิสระโดยไม่กระทบ `YouTubeChannel` เลย

## Time Complexity

- **Singleton**: การเรียก `__new__` ครั้งถัดๆ ไปคือ **O(1)** (แค่เช็คเงื่อนไขและคืนค่าเดิม)
- **Factory**: **O(1)** หรือ **O(k)** โดย k คือจำนวนชนิดที่ต้องเช็คใน `if/elif` (ในทางปฏิบัติ k มีค่าน้อยและคงที่)
- **Strategy**: การสลับ strategy คือ **O(1)** (แค่เปลี่ยน reference ไปยัง object อื่น) ส่วนความเร็วของการคำนวณจริงขึ้นอยู่กับ strategy ที่เลือกใช้แต่ละตัว
- **Observer**: `upload_video()` คือ **O(n)** โดย n คือจำนวน subscriber ทั้งหมด (ต้อง loop แจ้งเตือนทุกคน)

## Space Complexity

- **Singleton**: **O(1)** เพิ่มเติม — ประหยัดกว่าการสร้าง object ใหม่ทุกครั้งเพราะใช้ instance เดียวซ้ำตลอดโปรแกรม
- **Observer**: **O(n)** สำหรับเก็บรายชื่อ subscriber ทั้งหมดใน `_subscribers`

## แนวทางปฏิบัติที่ดี (Best Practices)

- ใช้ Singleton **เท่าที่จำเป็นจริงๆ** เท่านั้น (เช่น การตั้งค่าระบบส่วนกลาง, connection pool ของฐานข้อมูล) เพราะการมี global state ที่ทุกส่วนของโปรแกรมเข้าถึงร่วมกันทำให้เทสต์ยากขึ้น (เชื่อมโยงกับปัญหาการทดสอบที่จะเรียนในบท Testing)
- ใช้ Factory เมื่อ logic การสร้าง object ซับซ้อนพอที่ควรแยกออกจากโค้ดที่เรียกใช้ — ถ้าการสร้างง่ายมาก (แค่เรียก constructor ตรงๆ) การใช้ Factory อาจเป็นการเพิ่มความซับซ้อนโดยไม่จำเป็น
- เลือก pattern ให้เหมาะกับปัญหาจริง**หลังจาก**เห็นปัญหาเกิดขึ้นแล้ว (ไม่ใช่ยัด pattern เข้าไปล่วงหน้าโดยไม่มีปัญหาจริงรองรับ) — เชื่อมโยงกับคำเตือนเรื่อง over-engineering ในบทที่แล้ว

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ใช้ Singleton พร่ำเพรื่อจนโปรแกรมเต็มไปด้วย global state ที่ตามรอยยาก ทำให้เทสต์หน่วย (Unit Test) ยากขึ้นมากเพราะ state จาก test หนึ่งอาจรั่วไปกระทบ test อื่น (เชื่อมโยงกับบท Testing)
- สร้าง Factory ที่ซับซ้อนเกินจำเป็นสำหรับการสร้าง object ที่ง่ายมากอยู่แล้ว (over-engineering เหมือนที่เตือนไว้ในบทที่แล้ว)
- ลืมว่า Strategy Pattern ต้องการให้ Class ทุกตัวที่ implement strategy มี interface เดียวกัน (method ชื่อเดียวกัน, พารามิเตอร์แบบเดียวกัน) — ถ้าไม่สอดคล้องกันจะทำให้ `context.checkout()` พังเมื่อสลับ strategy บางตัว (ละเมิด LSP จากบทที่แล้ว)
- ลืม `unsubscribe()` observer ที่ไม่ใช้แล้วออกจาก list ทำให้เกิด **memory leak** (object ที่ควรถูกทิ้งแต่ยังถูกอ้างอิงอยู่ใน `_subscribers` ตลอดไป)
- ท่องจำชื่อ pattern โดยไม่เข้าใจปัญหาที่มันแก้จริงๆ แล้วนำไปใช้ผิดที่ผิดสถานการณ์ (pattern ไม่ใช่สูตรสำเร็จที่ใช้ได้ทุกที่)

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบาย Singleton Pattern คืออะไร ทำไมถึงต้องระวังการใช้มากเกินไป
2. Factory Pattern แก้ปัญหาอะไร ต่างจากการเรียก constructor ตรงๆ อย่างไร
3. Strategy Pattern เกี่ยวข้องกับ Open/Closed Principle (บทที่แล้ว) อย่างไร
4. Observer Pattern คืออะไร ยกตัวอย่างระบบจริงที่ใช้ pattern นี้ (นอกเหนือจากตัวอย่างในบท)
5. Design Pattern ต่างจาก Algorithm (Part 5) อย่างไร ทำไมถึงจัดอยู่ในหมวดการออกแบบ ไม่ใช่การแก้ปัญหาเชิงคำนวณ

## แบบฝึกหัด (Practice Exercises)

1. เขียน Adapter Pattern ที่แปลง interface ของ Class `OldPrinter` (มี method `print_old(text)`) ให้ทำงานร่วมกับระบบใหม่ที่คาดหวัง interface `print(text)` ได้ โดยไม่แก้โค้ดของ `OldPrinter` เลย
2. เขียน Factory Pattern สำหรับสร้าง `Shape` (จากแบบฝึกหัดบทที่ 31: `Circle`, `Rectangle`, `Triangle`) ที่รับ dictionary ของพารามิเตอร์และชนิดรูปทรง แล้วคืน object ที่ถูกต้อง
3. เขียน Observer Pattern จำลองระบบ "แจ้งเตือนราคาหุ้น" ที่นักลงทุนหลายคน (Observer) subscribe หุ้นตัวเดียวกัน (Subject) แล้วได้รับแจ้งเตือนอัตโนมัติเมื่อราคาเปลี่ยนแปลงเกินเปอร์เซ็นต์ที่กำหนด

## Mini Project

**"ระบบจัดการออเดอร์ร้านกาแฟ (Coffee Shop Order System)"**: เขียนโปรแกรมที่ใช้ Design Pattern อย่างน้อย 3 แบบร่วมกัน:
1. **Singleton**: สร้าง `InventoryManager` ที่มี instance เดียวทั้งระบบ เก็บจำนวนวัตถุดิบคงเหลือ (เมล็ดกาแฟ, นม, น้ำตาล) ที่ทุกส่วนของโปรแกรมเข้าถึงร่วมกัน
2. **Factory**: สร้าง `DrinkFactory` ที่รับชื่อเมนู (เช่น "latte", "americano", "cappuccino") แล้วคืน object เครื่องดื่มที่ถูกต้อง พร้อมสูตรผสมของแต่ละเมนู
3. **Observer**: สร้างระบบแจ้งเตือนเมื่อวัตถุดิบเหลือน้อย (`InventoryManager` เป็น Subject) โดยมีทั้ง `BaristaDisplay` (แสดงบนจอในร้าน) และ `ManagerNotifier` (ส่งแจ้งเตือนไปหาผู้จัดการ) เป็น Observer ที่ได้รับแจ้งพร้อมกันอัตโนมัติ
4. เขียนรายงานสรุปว่าแต่ละ pattern ที่ใช้ช่วยแก้ปัญหาอะไร และถ้าไม่ใช้ pattern นั้นจะเกิดปัญหาอะไรตามมา

## ข้อคิดสำคัญ (Key Takeaways)

- Design Pattern คือวิธีแก้ปัญหาการออกแบบที่พบซ้ำๆ ถูกตั้งชื่อไว้เป็นมาตรฐานให้วิศวกรสื่อสารกันได้เร็วขึ้น ไม่ใช่โค้ดสำเร็จรูปที่ก็อปวางได้ตรงๆ
- Singleton รับประกันมี instance เดียวในทั้งโปรแกรม, Factory ซ่อนตรรกะการสร้าง object ไว้ในที่เดียว, Strategy สลับอัลกอริทึมได้ที่ runtime (สร้างบน OCP), Observer แจ้งเตือนหลาย object อัตโนมัติเมื่อมีเหตุการณ์เกิดขึ้น
- Design Pattern จำนวนมากสร้างขึ้นบนพื้นฐานของ SOLID Principles (บทที่แล้ว) โดยตรง โดยเฉพาะ OCP และ DIP
- ต้องเลือกใช้ pattern ให้เหมาะกับปัญหาจริงที่เกิดขึ้น ไม่ใช่ยัดเข้าไปล่วงหน้าโดยไม่มีความจำเป็น (over-engineering)
- นี่คือบทสุดท้ายของ **Part 6: Object-Oriented Programming** — ตลอด 4 บทที่ผ่านมา (Class/Object ถึง Design Patterns) วางรากฐานการออกแบบซอฟต์แวร์เชิงวัตถุครบถ้วน บทถัดไปจะเข้าสู่ **Part 7: Functional Programming** ซึ่งเป็นแนวคิดการเขียนโปรแกรมอีกแบบที่ต่างจาก OOP โดยสิ้นเชิง โดยเริ่มจาก Pure Functions และ Immutability

---
*บทต่อไป (บทที่ 34): Pure Functions, Immutability — พิมพ์ "Next" เพื่อดำเนินการต่อ*
