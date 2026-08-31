# บทที่ 49: Microservices vs Monolith, Event Driven, DDD

## แนวคิดหลัก (Concept)

สองบทที่ผ่านมาสอนวิธีจัดโครงสร้าง**ภายในโปรเจกต์เดียว** (Clean Code, MVC, Layered, Hexagonal) บทนี้เป็นบทปิดท้าย **Part 12: Software Engineering** โดยขยับมุมมองไปที่คำถามระดับใหญ่กว่า: **ควรสร้างระบบเป็นโปรแกรมเดียวก้อนใหญ่ หรือแยกเป็นหลายบริการย่อย?** และถ้าแยกเป็นหลายบริการ **จะสื่อสารกันอย่างไรและแบ่งขอบเขตความรับผิดชอบอย่างไร**

- **Monolith (สถาปัตยกรรมก้อนเดียว)** — สร้างระบบทั้งหมดเป็น**โปรแกรมเดียว** ที่ deploy พร้อมกันทั้งหมด แม้ภายในจะแบ่งเป็นชั้น (Layered/Hexagonal จากบทที่แล้ว) ก็ตาม แต่สุดท้ายรันเป็น process เดียว
- **Microservices (สถาปัตยกรรมหลายบริการย่อย)** — แยกระบบเป็น**หลายบริการอิสระ** ที่แต่ละตัว deploy แยกกันได้ มีฐานข้อมูลของตัวเอง และสื่อสารกันผ่านเครือข่าย (มักผ่าน REST/gRPC จากบทที่ 43)
- **Event-Driven Architecture** — รูปแบบการสื่อสารที่บริการต่างๆ ไม่เรียกกันโดยตรง แต่ส่ง**เหตุการณ์ (Event)** ผ่านตัวกลาง (Message Broker) แล้วบริการที่สนใจเหตุการณ์นั้นจะรับไปประมวลผลเอง
- **DDD (Domain-Driven Design)** — แนวทางออกแบบซอฟต์แวร์ที่**แบ่งขอบเขตของระบบตามขอบเขตทางธุรกิจจริง (Bounded Context)** แทนที่จะแบ่งตามเทคนิค ทำให้แต่ละส่วนของระบบสะท้อนวิธีที่ธุรกิจจริงทำงาน

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Monolith มีไว้เป็นจุดเริ่มต้นที่เรียบง่ายและพัฒนาได้เร็ว**: การ deploy, debug, และทดสอบระบบเดียวง่ายกว่าหลายระบบมาก ไม่ต้องจัดการเรื่อง network latency, การสื่อสารข้าม service, หรือ distributed transaction (เชื่อมโยงกับความซับซ้อนที่เพิ่มขึ้นของ CAP Theorem จากบทที่ 41) — โปรเจกต์ใหม่ส่วนใหญ่ควรเริ่มจาก Monolith ก่อนเสมอ
- **Microservices มีไว้แก้ปัญหาที่ Monolith เจอเมื่อระบบและทีมงานใหญ่ขึ้นมาก**: เมื่อ Monolith ใหญ่จนแก้ไขส่วนหนึ่งต้องทดสอบและ deploy ทั้งระบบใหม่ทุกครั้ง (ช้าและเสี่ยง), หลายทีมทำงานในโค้ดเดียวกันชนกันบ่อย, หรือบางส่วนของระบบต้องการ scale มากกว่าส่วนอื่น (เช่น ระบบค้นหาต้องการ CPU มากกว่าระบบแจ้งเตือน) — Microservices ให้แต่ละทีม deploy และ scale ส่วนของตัวเองอย่างอิสระ
- **Event-Driven Architecture มีไว้ลดการผูกติดกันแน่นเกินไปข้าม service (tight coupling)**: ถ้า Service A ต้องเรียก Service B, C, D ตรงๆ ทุกครั้งที่มีเหตุการณ์เกิดขึ้น (คล้าย Observer Pattern บทที่ 33 แต่ข้าม service) การเพิ่ม Service E ในอนาคตต้องแก้โค้ดของ A ทุกครั้ง — Event-Driven ให้ A แค่ "ประกาศ" เหตุการณ์ออกไป โดยไม่ต้องรู้ว่าใครฟังอยู่บ้าง (ตรงตามหลัก OCP จากบทที่ 32)
- **DDD มีไว้ป้องกันการแบ่ง Microservices ผิดขอบเขต**: การแบ่ง service ตามเทคนิค (เช่น "service สำหรับ database queries ทั้งหมด") มักนำไปสู่ service ที่ทุกอย่างต้องคุยกับมันตลอดเวลา (ผูกติดกันแน่น) — DDD สอนให้แบ่งตาม "ขอบเขตทางธุรกิจ" (เช่น "Order Service", "Inventory Service", "Payment Service") ซึ่งมักเปลี่ยนแปลงพร้อมกันภายในตัวเองมากกว่าข้ามขอบเขต ทำให้แต่ละ service เป็นอิสระจากกันจริงๆ

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Monolith** เหมือน **ร้านอาหารครบวงจรที่มีครัว แคชเชียร์ และที่นั่งอยู่ในอาคารเดียว** — จัดการง่าย ทุกอย่างอยู่ในที่เดียว แต่ถ้าต้องขยายแค่ส่วนครัว ก็ต้องต่อเติมทั้งอาคาร
- **Microservices** เหมือน **ศูนย์อาหารที่แต่ละร้านเป็นอิสระต่อกัน** — ร้านก๋วยเตี๋ยวปิดปรับปรุงไม่กระทบร้านข้าวมันไก่เลย แต่ละร้านจ้างพนักงานและสั่งวัตถุดิบของตัวเอง (ฐานข้อมูลแยกกัน) ลูกค้าเดินไปสั่งแต่ละร้านแยกกัน (เรียก API แยกกัน)
- **Event-Driven Architecture** เหมือน **ประกาศเสียงตามสายในห้างสรรพสินค้า** — พนักงานประกาศ "ลดราคาแผนกเสื้อผ้า!" โดยไม่รู้ว่าใครได้ยินบ้าง ลูกค้าที่สนใจก็เดินไปแผนกนั้นเอง (ต่างจากการเดินไปบอกลูกค้าแต่ละคนตรงๆ ทีละคน)
- **DDD** เหมือน **การแบ่งแผนกในบริษัทตามหน้าที่ทางธุรกิจจริง (ฝ่ายขาย, ฝ่ายบัญชี, ฝ่ายคลังสินค้า)** แทนที่จะแบ่งตามเทคนิค (เช่น "แผนกที่ใช้ Excel", "แผนกที่ใช้อีเมล") — การแบ่งตามธุรกิจจริงทำให้แต่ละแผนกมีความรับผิดชอบชัดเจนและทำงานเป็นอิสระจากแผนกอื่นได้มากกว่า

## แผนภาพอธิบาย (Visual explanation)

### Monolith vs Microservices

```
Monolith:                                    Microservices:

┌─────────────────────────────┐              ┌──────────┐  ┌──────────┐  ┌──────────┐
│                               │              │  Order   │  │Inventory │  │ Payment  │
│   Order + Inventory +         │              │ Service  │  │ Service  │  │ Service  │
│   Payment + Notification      │              │ (own DB) │  │ (own DB) │  │ (own DB) │
│   (โค้ดเดียว, DB เดียว)         │              └────┬─────┘  └────┬─────┘  └────┬─────┘
│                               │                    │             │             │
└─────────────────────────────┘                    └──── เรียกกันผ่าน REST/gRPC ────┘

Deploy: ครั้งเดียวทั้งระบบ                        Deploy: แยกกันอิสระ แก้ Order ไม่กระทบ Payment เลย
Scale: ต้อง scale ทั้งระบบพร้อมกัน                 Scale: scale เฉพาะ service ที่ต้องการมากกว่าได้
```

### Event-Driven Architecture — สื่อสารผ่าน Event แทนการเรียกตรง

```
แบบเรียกตรง (Tight Coupling):                 แบบ Event-Driven (Loose Coupling):

OrderService                                  OrderService
    │                                              │
    ├──> เรียก InventoryService โดยตรง                ├──> publish("OrderPlaced", order_data)
    ├──> เรียก PaymentService โดยตรง                        │
    └──> เรียก NotificationService โดยตรง                    v
                                                    [Message Broker เช่น Kafka/RabbitMQ]
เพิ่ม service ใหม่ -> ต้องแก้ OrderService เพิ่ม        │        │           │
                                                    v        v           v
                                            Inventory  Payment    Notification
                                            (subscribe) (subscribe) (subscribe)

เพิ่ม service ใหม่ -> แค่ subscribe event เดิม ไม่ต้องแก้ OrderService เลย!
```

### DDD — แบ่งขอบเขตตามธุรกิจจริง (Bounded Context)

```
แบ่งผิด (ตามเทคนิค):                            แบ่งถูก (ตาม Bounded Context ทางธุรกิจ):

- Database Service (ทุกอย่างที่คุย DB)             - Order Context (ทุกอย่างเกี่ยวกับคำสั่งซื้อ)
- API Service (ทุกอย่างที่รับ request)             - Inventory Context (ทุกอย่างเกี่ยวกับสต็อกสินค้า)
- Validation Service (ทุกอย่างที่ตรวจสอบ)          - Payment Context (ทุกอย่างเกี่ยวกับการชำระเงิน)
   (ทุก service ต้องคุยกับ Database Service           (แต่ละ context มี model, DB, logic ของตัวเอง
    ตลอดเวลา -> ผูกติดกันแน่นมาก)                      คำว่า "Order" ใน context หนึ่งอาจมีความหมาย
                                                     ต่างจากอีก context ก็ได้ - เป็นเรื่องปกติ)
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== Monolith: ทุกอย่างอยู่ในโปรแกรมเดียว เรียกฟังก์ชันตรงๆ =====
class MonolithOrderSystem:
    def place_order(self, items):
        total = self.calculate_total(items)              # เรียกฟังก์ชันในโปรแกรมเดียวกัน
        self.check_inventory(items)                          # ไม่มี network call เลย เร็วมาก
        self.process_payment(total)
        self.send_notification("Order placed!")
        return total

    def calculate_total(self, items):
        return sum(item.price for item in items)

    def check_inventory(self, items):
        print("Checking inventory (same process)...")

    def process_payment(self, total):
        print(f"Processing payment of {total} (same process)...")

    def send_notification(self, message):
        print(f"Notification: {message} (same process)")

# ===== Microservices: แต่ละ service แยกกัน สื่อสารผ่าน HTTP (จำลอง) =====
import requests

class OrderMicroservice:
    def place_order(self, items):
        total = sum(item.price for item in items)
        # ในระบบจริง แต่ละบรรทัดคือ network call ข้าม service ที่แยก deploy กัน
        requests.post("http://inventory-service/check", json={"items": items})
        requests.post("http://payment-service/charge", json={"amount": total})
        requests.post("http://notification-service/send", json={"message": "Order placed!"})
        return total

# ===== Event-Driven Architecture: ใช้ Message Broker จำลอง (Publish/Subscribe) =====
class EventBus:                                            # จำลอง Message Broker แบบง่าย (ในระบบจริงใช้ Kafka/RabbitMQ)
    def __init__(self):
        self.subscribers = {}                                 # event_name -> list ของ handler functions

    def subscribe(self, event_name, handler):
        self.subscribers.setdefault(event_name, []).append(handler)

    def publish(self, event_name, data):
        for handler in self.subscribers.get(event_name, []):     # แจ้งทุกคนที่ subscribe (คล้าย Observer บทที่ 33)
            handler(data)

event_bus = EventBus()

def inventory_handler(order_data):
    print(f"[Inventory] Reserving items: {order_data['items']}")

def payment_handler(order_data):
    print(f"[Payment] Charging: {order_data['total']}")

def notification_handler(order_data):
    print(f"[Notification] Order confirmed: {order_data['total']}")

event_bus.subscribe("OrderPlaced", inventory_handler)         # แต่ละ service subscribe เอง
event_bus.subscribe("OrderPlaced", payment_handler)
event_bus.subscribe("OrderPlaced", notification_handler)

class EventDrivenOrderService:
    def place_order(self, items, total):
        event_bus.publish("OrderPlaced", {"items": items, "total": total})  # ไม่รู้จักใครฟังอยู่บ้างเลย

# ===== DDD: แบ่ง Bounded Context ชัดเจน แต่ละ context มี model ของตัวเอง =====
class OrderContext:                                          # "Product" ใน context นี้หมายถึงสิ่งที่ถูกสั่งซื้อ
    class Product:
        def __init__(self, sku, quantity):
            self.sku = sku
            self.quantity = quantity

class InventoryContext:                                       # "Product" ใน context นี้หมายถึงสต็อกที่มีอยู่จริง
    class Product:
        def __init__(self, sku, warehouse_location, stock_level):
            self.sku = sku
            self.warehouse_location = warehouse_location         # ข้อมูลนี้ไม่มีความหมายใน OrderContext เลย
            self.stock_level = stock_level
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `MonolithOrderSystem.place_order()` — ทุกเมทอด (`check_inventory`, `process_payment`, `send_notification`) เป็นแค่**การเรียกฟังก์ชันธรรมดาในโปรแกรมเดียวกัน** ไม่มี network call เลย ทำให้เร็วและไม่มีความเสี่ยงจากปัญหาเครือข่าย (latency, connection failure จากบทที่ 42) — แต่ทั้งหมดนี้รันอยู่ใน**process เดียว** ถ้าโค้ดส่วนใดพัง (เช่น memory leak ใน `send_notification`) มีโอกาสกระทบทั้งระบบ
2. `OrderMicroservice.place_order()` — แต่ละบรรทัดที่เรียก `requests.post(...)` คือ**การสื่อสารข้ามเครือข่ายไปยัง service อื่นที่ deploy แยกกันอย่างสมบูรณ์** (เชื่อมโยงกับความรู้เรื่อง HTTP บทที่ 42-43 โดยตรง) — ถ้า `payment-service` ล่ม ระบบยังทำงานส่วนอื่นได้ (ความทนทานต่อความล้มเหลวบางส่วน) แต่ต้องจัดการกรณี network เชื่อมต่อไม่ได้เพิ่มเติม (สิ่งที่ Monolith ไม่ต้องกังวลเลย)
3. `event_bus.publish("OrderPlaced", ...)` ใน `EventDrivenOrderService` — สังเกตว่า `EventDrivenOrderService` **ไม่มีการอ้างอิงถึง `inventory_handler`, `payment_handler`, หรือ `notification_handler` เลยแม้แต่น้อย** มันแค่ "ประกาศ" ว่าเกิดเหตุการณ์ `OrderPlaced` ขึ้น ส่วนใครจะฟังและทำอะไรต่อเป็นเรื่องของฝั่งนั้น — การเพิ่ม handler ใหม่ (เช่น `analytics_handler` สำหรับเก็บสถิติ) ทำได้แค่เพิ่ม `event_bus.subscribe("OrderPlaced", analytics_handler)` โดย**ไม่ต้องแก้ `EventDrivenOrderService` เลยแม้แต่บรรทัดเดียว** (ตรงตามหลัก OCP บทที่ 32 อย่างสมบูรณ์)
4. `OrderContext.Product` เทียบกับ `InventoryContext.Product` — ทั้งสอง Class ชื่อ `Product` เหมือนกัน แต่มีความหมายและ attribute ต่างกันโดยสิ้นเชิงตาม **Bounded Context** ของตัวเอง `OrderContext.Product` สนใจแค่ "สั่งอะไร จำนวนเท่าไหร่" ในขณะที่ `InventoryContext.Product` สนใจ "อยู่ที่ไหน เหลือเท่าไหร่จริง" — DDD ยอมรับว่า**คำเดียวกันมีความหมายต่างกันได้ในแต่ละขอบเขตทางธุรกิจ** และไม่พยายามบังคับให้ทุก context ใช้ model เดียวกันแบบเดียวกันทั้งหมด (ต่างจากแนวคิดดั้งเดิมที่มักพยายามสร้าง "Product model กลางที่ใช้ได้ทุกที่" ซึ่งมักบวมและซับซ้อนเกินจำเป็นในทางปฏิบัติ)

## Time Complexity

Microservices, Event-Driven, และ DDD เป็นการตัดสินใจด้านสถาปัตยกรรมระดับองค์กร ไม่ใช่อัลกอริทึม แต่มีผลกระทบด้าน latency ที่วัดได้:
- Monolith: การเรียกฟังก์ชันภายในโปรแกรมเดียวกัน — **เร็วมาก (nanoseconds ถึง microseconds)**
- Microservices (synchronous, เช่น REST): แต่ละการเรียกข้าม service มีต้นทุนของ network round-trip (บทที่ 42) — **ช้ากว่า Monolith หลายเท่า** (milliseconds)
- Event-Driven (asynchronous): ผู้ส่ง Event ไม่ต้องรอผลลัพธ์จาก subscriber เลย (`publish()` คืนค่าทันที) — **เร็วกว่า synchronous microservices ในมุมมองของผู้ส่ง** แต่ subscriber ประมวลผลช้ากว่าแบบ synchronous (ต้องรอคิวของ message broker)

## Space Complexity

- Monolith ใช้ทรัพยากร (RAM, CPU) รวมกันในเครื่องเดียวหรือกลุ่มเครื่องที่ scale พร้อมกันทั้งหมด
- Microservices แต่ละตัวใช้ทรัพยากรแยกกัน ทำให้ scale เฉพาะส่วนที่ต้องการได้ (เช่น เพิ่ม instance ของ `payment-service` อย่างเดียวโดยไม่ต้องเพิ่ม `notification-service`) — ยืดหยุ่นกว่าแต่ก็มี overhead จากการรันหลาย process/container แยกกัน

## แนวทางปฏิบัติที่ดี (Best Practices)

- **เริ่มต้นด้วย Monolith เสมอ** สำหรับโปรเจกต์ใหม่ (แม้จะออกแบบภายในให้เป็น Layered/Hexagonal อย่างดีตามบทที่แล้วก็ตาม) แล้วค่อยแยกเป็น Microservices ทีหลังเมื่อพิสูจน์แล้วว่าจำเป็นจริง (เช่น ทีมใหญ่ขึ้นมาก, บางส่วนต้อง scale ต่างกันมาก) — นี่คือหลักการที่เรียกว่า "Monolith First"
- ใช้ Event-Driven Architecture เมื่อการกระทำหนึ่งต้องแจ้งหลายฝ่ายที่**ไม่จำเป็นต้องรอผลลัพธ์ทันที** (เช่น ส่งอีเมลยืนยัน, บันทึก analytics) แต่ใช้ synchronous call (REST/gRPC) เมื่อต้องการผลลัพธ์ทันที (เช่น ตรวจสอบว่ามีสต็อกพอก่อนยืนยันคำสั่งซื้อ)
- ใช้หลักการ DDD กำหนดขอบเขตของแต่ละ Microservice **ก่อน**ตัดสินใจแบ่งจริง อย่าแบ่งตามความสะดวกทางเทคนิค (เช่น "service ที่ใช้ภาษาเดียวกัน") เพราะมักนำไปสู่ service ที่ผูกติดกันแน่นเกินไป

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- เริ่มโปรเจกต์ใหม่ด้วย Microservices ทันทีเพราะ "บริษัทใหญ่ๆ ใช้กัน" โดยไม่พิจารณาว่าทีมและระบบยังเล็กเกินไปที่จะได้ประโยชน์จากความซับซ้อนที่เพิ่มขึ้น (network, distributed transaction, monitoring หลาย service) — เป็น over-engineering รูปแบบหนึ่งที่พบบ่อยมาก
- แบ่ง Microservices ตามชั้นทางเทคนิค (เช่น "Database Service", "API Service") แทนที่จะแบ่งตาม Bounded Context ทางธุรกิจ ทำให้ทุก service ต้องคุยกันตลอดเวลาจนแทบไม่ต่างจาก Monolith แต่มี overhead ของ network เพิ่มเข้ามา
- ใช้ Event-Driven Architecture กับทุกการสื่อสาร แม้กรณีที่ต้องการผลลัพธ์ทันที ทำให้ต้องเขียนโค้ดซับซ้อนขึ้นมากเพื่อรอผลลัพธ์แบบ asynchronous ทั้งที่ synchronous call ตรงไปตรงมากว่ามาก
- ไม่มี Bounded Context ที่ชัดเจน ทำให้ Model เดียวกัน (เช่น `User`) ถูกใช้ทั่วทั้งระบบจนบวมด้วย attribute ที่ไม่เกี่ยวข้องกันจากหลาย context ปนกัน

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายข้อดี-ข้อเสียของ Monolith เทียบกับ Microservices พร้อมยกตัวอย่างสถานการณ์ที่เหมาะกับแต่ละแบบ
2. ทำไมถึงแนะนำให้เริ่มโปรเจกต์ใหม่ด้วย Monolith ก่อนเสมอ (แนวคิด "Monolith First")
3. Event-Driven Architecture ลดปัญหา tight coupling ระหว่าง service ได้อย่างไร เชื่อมโยงกับ Observer Pattern (บทที่ 33) และ OCP (บทที่ 32)
4. Bounded Context ใน DDD คืออะไร ทำไมการแบ่ง Microservices ตามขอบเขตทางเทคนิคถึงมักเป็นความผิดพลาด
5. Synchronous call (REST) ต่างจาก Event-Driven (asynchronous) อย่างไร ควรเลือกใช้แบบไหนเมื่อไหร่

## แบบฝึกหัด (Practice Exercises)

1. ออกแบบระบบร้านค้าออนไลน์แบบ Monolith ก่อน แล้วระบุว่าถ้าต้องแยกเป็น Microservices ควรแบ่งเป็นกี่ service และแต่ละ service ควรรับผิดชอบอะไรบ้างตามหลัก DDD (Bounded Context)
2. เขียนโปรแกรมจำลอง EventBus (ตามตัวอย่างในบทเรียน) แล้วเพิ่ม subscriber ใหม่ (`analytics_handler`) โดยไม่แก้ไข `EventDrivenOrderService` เลย พิสูจน์หลักการ OCP
3. เปรียบเทียบเวลาที่ใช้ (จำลองด้วย `time.sleep`) ระหว่างการเรียก 3 service แบบ synchronous เรียงกัน กับการ publish event เดียวที่ทั้ง 3 service รับไปประมวลผลแบบ asynchronous พร้อมกัน

## Mini Project

**"ระบบสั่งซื้อสินค้าออนไลน์แบบ Event-Driven Microservices (Event-Driven E-commerce Simulation)"**: เขียนโปรแกรมที่:
1. ออกแบบระบบด้วย DDD โดยแบ่งเป็น 3 Bounded Context ชัดเจน: Order Context, Inventory Context, Payment Context พร้อม Model ของตัวเองในแต่ละ context (ไม่แชร์ Class เดียวกันข้าม context)
2. Implement แต่ละ context เป็น "service" แยกกัน (จำลองด้วย Class/module แยกไฟล์ ไม่จำเป็นต้องรันเป็น process จริงแยกกัน) สื่อสารกันผ่าน EventBus ที่เขียนเอง (ตามตัวอย่างในบทเรียน)
3. จำลองสถานการณ์ที่ Payment Context ล้มเหลว (จงใจโยน error) แล้วออกแบบให้ระบบส่ง Compensating Event (เช่น `PaymentFailed` ที่ทำให้ Inventory Context คืนสต็อกกลับ) แทนที่จะปล่อยให้ระบบอยู่ในสถานะไม่สอดคล้องกัน
4. เขียนรายงานเปรียบเทียบว่าถ้าออกแบบระบบนี้เป็น Monolith แทน จะง่ายหรือยากกว่าอย่างไรในการจัดการสถานการณ์ Payment ล้มเหลวนี้

## ข้อคิดสำคัญ (Key Takeaways)

- Monolith เรียบง่ายและเหมาะกับการเริ่มต้นโปรเจกต์ ส่วน Microservices ให้ความยืดหยุ่นในการ deploy/scale แยกกันแต่เพิ่มความซับซ้อนของระบบอย่างมาก — ควรเริ่มจาก Monolith เสมอแล้วค่อยแยกทีหลังเมื่อจำเป็นจริง
- Event-Driven Architecture ลดการผูกติดกันแน่นระหว่าง service ด้วยการสื่อสารผ่านเหตุการณ์แทนการเรียกตรง สร้างบนหลักการเดียวกับ Observer Pattern และ OCP ที่เรียนมาแล้ว
- DDD แบ่งขอบเขตของระบบตาม Bounded Context ทางธุรกิจจริง ไม่ใช่ตามความสะดวกทางเทคนิค ทำให้แต่ละส่วนของระบบเป็นอิสระจากกันจริงๆ
- คำเดียวกัน (เช่น "Product") สามารถมีความหมายต่างกันในแต่ละ Bounded Context ได้ ซึ่งเป็นเรื่องปกติและถูกต้องตามหลัก DDD ไม่ใช่ความผิดพลาดที่ต้องแก้ไข
- นี่คือบทปิดท้าย **Part 12: Software Engineering** — ตลอด 3 บทที่ผ่านมา (Clean Code/Architecture, MVC/Layered/Hexagonal, Microservices/Event-Driven/DDD) วางรากฐานตั้งแต่ระดับบรรทัดโค้ดไปจนถึงระดับสถาปัตยกรรมองค์กรทั้งหมด บทถัดไปจะเข้าสู่ **Part 13: Version Control** ซึ่งเป็นเครื่องมือที่ใช้ทุกวันในการทำงานร่วมกันของทีมพัฒนา โดยเริ่มจาก Git — branch, merge, rebase, cherry-pick, conflicts

---
*บทต่อไป (บทที่ 50): Git — branch, merge, rebase, cherry-pick, conflicts — พิมพ์ "Next" เพื่อดำเนินการต่อ*
