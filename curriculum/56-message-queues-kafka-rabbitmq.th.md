# บทที่ 56: Message Queues, Kafka, RabbitMQ

## แนวคิดหลัก (Concept)

บทที่ 49 แนะนำ Event-Driven Architecture ด้วย `EventBus` ที่เขียนขึ้นเองแบบง่ายๆ (แค่ dict เก็บ subscriber ในหน่วยความจำ) บทนี้เจาะลึกว่าในระบบจริงการ "ส่ง Event" ต้องใช้ **Message Queue** ซึ่งเป็นระบบที่ซับซ้อนและทนทานกว่ามาก — รับประกันว่าข้อความจะไม่หายแม้ระบบล่ม, ควบคุมความเร็วระหว่างผู้ส่งกับผู้รับที่ไม่เท่ากันได้ และรองรับการประมวลผลแบบกระจาย

- **Message Queue** — ระบบกลางที่เก็บข้อความ (message) ไว้ชั่วคราวระหว่างที่ผู้ส่ง (Producer) ส่งเข้ามาจนกว่าผู้รับ (Consumer) จะมาดึงไปประมวลผล ทำงานแบบ FIFO เหมือน Queue (บทที่ 14) แต่เพิ่มความทนทานและการกระจายงาน
- **Producer / Consumer** — Producer คือฝั่งที่ส่งข้อความเข้าคิว (เช่น เว็บไซต์ที่รับคำสั่งซื้อ) Consumer คือฝั่งที่ดึงข้อความออกมาประมวลผล (เช่น service ที่ส่งอีเมลยืนยัน)
- **RabbitMQ** — Message Broker แบบดั้งเดิมที่เน้นการ**ส่งมอบข้อความที่แน่นอน** (message delivery guarantee) เหมาะกับงานที่ต้องการให้แต่ละข้อความถูกประมวลผล**เพียงครั้งเดียว**โดยผู้รับที่เจาะจง (Task Queue)
- **Kafka** — Message Broker ที่เน้น**ปริมาณงานสูงมาก (high throughput)** และเก็บประวัติข้อความไว้เป็น **Log** ที่หลาย Consumer อ่านซ้ำได้ เหมาะกับ Event Streaming ที่ข้อมูลเดียวกันต้องถูกใช้โดยหลายระบบพร้อมกัน

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Message Queue มีไว้แก้ปัญหาที่ `EventBus` แบบง่ายในบทที่ 49 ทำไม่ได้**: `EventBus` ที่เขียนเองเก็บ subscriber ไว้ในหน่วยความจำเท่านั้น — ถ้าโปรแกรมล่มระหว่างประมวลผล ข้อความที่ยังไม่เสร็จจะ**หายไปตลอดกาล** และถ้า Consumer ทำงานช้ากว่า Producer ส่ง ข้อความจะกองพะเนินจนโปรแกรมหน่วยความจำเต็ม — Message Queue จริงแก้ทั้งสองปัญหาด้วยการเก็บข้อความแบบ persistent (บันทึกลง disk) และมี buffer ที่จัดการความเร็วที่ไม่เท่ากันระหว่าง Producer/Consumer
- **การแยก Producer และ Consumer ออกจากกันด้วยคิวมีไว้ให้แต่ละฝั่ง scale อิสระจากกันได้**: ถ้า Consumer ทำงานช้า (เช่น ส่งอีเมลใช้เวลานาน) สามารถเพิ่มจำนวน Consumer instance ได้โดยไม่ต้องแก้ Producer เลย (เชื่อมโยงกับ Microservices บทที่ 49 และ Horizontal Scaling บทที่ 55)
- **RabbitMQ มีไว้สำหรับงานที่ต้องการความแน่นอนว่า "งานนี้ถูกทำแล้ว" (Task Queue)**: เช่น การประมวลผลคำสั่งซื้อ — แต่ละคำสั่งควรถูกประมวลผล**เพียงครั้งเดียว**โดย worker ตัวใดตัวหนึ่ง ไม่ใช่ทุก worker ทำซ้ำ RabbitMQ ออกแบบมาให้กระจายงานแบบนี้ได้ดี (Competing Consumers Pattern)
- **Kafka มีไว้สำหรับข้อมูลปริมาณมหาศาลที่ต้องแชร์ให้หลายระบบอ่านพร้อมกัน**: เช่น log กิจกรรมผู้ใช้ที่ทั้ง analytics, recommendation system, และ fraud detection ต้องการอ่านข้อมูลชุดเดียวกันแยกกัน — Kafka เก็บข้อความเป็น log ที่**อ่านซ้ำได้หลายครั้งโดยหลาย Consumer Group** ต่างจาก RabbitMQ ที่ข้อความมักถูกลบทิ้งหลังมีคนรับไปแล้ว

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Message Queue** เหมือน **ตู้ล็อกเกอร์รับพัสดุหน้าหมู่บ้าน** — พนักงานส่งพัสดุ (Producer) วางพัสดุไว้ในตู้ล็อกเกอร์แล้วไปต่อได้เลย ไม่ต้องรอเจ้าของบ้าน (Consumer) มารับ ณ ตอนนั้น เจ้าของบ้านมารับเมื่อสะดวก
- **RabbitMQ (Task Queue)** เหมือน **ระบบคิวธนาคารที่เรียกเลขคิวแล้วมีแค่เคาน์เตอร์เดียวที่ให้บริการลูกค้าคนนั้น** — แต่ละใบคิวถูกบริการแค่ครั้งเดียวโดยพนักงานคนใดคนหนึ่ง ไม่มีทางที่สองเคาน์เตอร์จะบริการลูกค้าคนเดียวกันซ้ำ
- **Kafka (Event Streaming)** เหมือน **การถ่ายทอดสดข่าวที่หลายช่องทีวีรับสัญญาณไปออกอากาศพร้อมกัน** — ข่าวเดียวกัน (message เดียวกัน) ถูกหลายช่อง (Consumer Group หลายกลุ่ม) รับไปใช้งานแยกกันได้พร้อมกัน โดยแต่ละช่องไม่กระทบกันเลย และข่าวเก่ายังดูย้อนหลังได้ (เก็บเป็น log)

## แผนภาพอธิบาย (Visual explanation)

### Producer-Consumer ผ่าน Message Queue

```
Producer (Web Server)              Message Queue              Consumer (Email Worker)

รับคำสั่งซื้อ                    [msg1][msg2][msg3]              ดึงทีละข้อความ
   |                                    ^                              |
   +--- publish("OrderPlaced") --------+                              |
                                        |                              v
                                        +------- consume() -----> ส่งอีเมลยืนยัน

Producer ไม่ต้องรอ Consumer ทำงานเสร็จเลย (ส่งแล้วไปทำงานต่อทันที - Asynchronous)
ถ้า Consumer ล่มชั่วคราว ข้อความยังคงอยู่ในคิว (ไม่หาย) รอจนกว่า Consumer จะกลับมาทำงาน
```

### RabbitMQ — Task Queue: แต่ละข้อความถูกประมวลผลครั้งเดียว (Competing Consumers)

```
Producer ---> [Queue: task1, task2, task3, task4] <--- Worker A (ดึง task1, task3)
                                                    <--- Worker B (ดึง task2, task4)

แต่ละ task ถูกประมวลผลโดย worker แค่ตัวเดียวเท่านั้น ไม่มีทางถูกทำซ้ำโดยสอง worker
เพิ่ม Worker C -> งานถูกกระจายเฉลี่ยมากขึ้น (scale การประมวลผลได้อิสระจาก Producer)
```

### Kafka — Event Streaming: หลาย Consumer Group อ่าน Log เดียวกันได้พร้อมกัน

```
Producer ---> Kafka Topic: "user_activity"
              [event1][event2][event3][event4][event5]   <- เก็บเป็น log เรียงลำดับ ไม่ลบทิ้งทันที

              Consumer Group A (Analytics)     อ่านตั้งแต่ event1 -> ปัจจุบันอ่านถึง event3
              Consumer Group B (Fraud Detect)  อ่านตั้งแต่ event1 -> ปัจจุบันอ่านถึง event5 (เร็วกว่า)
              Consumer Group C (ใหม่ เข้าร่วมทีหลัง)  อ่านย้อนกลับไปตั้งแต่ event1 ได้เลย!

แต่ละ Consumer Group มี "ตำแหน่งอ่านล่าสุด" (offset) ของตัวเอง ไม่กระทบกัน
ข้อมูลเดิมยังอยู่ใน log จนกว่าจะครบเวลาที่ตั้งไว้ (retention period) ทำให้ Consumer ใหม่อ่านย้อนหลังได้
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== RabbitMQ: Task Queue (ใช้ library pika) =====
import pika
import json

def rabbitmq_producer(order_data):
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='order_processing', durable=True)   # durable=True: คิวยังอยู่แม้ broker restart

    channel.basic_publish(
        exchange='',
        routing_key='order_processing',
        body=json.dumps(order_data),
        properties=pika.BasicProperties(delivery_mode=2)              # delivery_mode=2: บันทึกข้อความลง disk ด้วย
    )
    connection.close()

def rabbitmq_consumer():
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='order_processing', durable=True)
    channel.basic_qos(prefetch_count=1)                                # แต่ละ worker รับทีละ 1 งาน ไม่แย่งงานเกินตัว

    def callback(ch, method, properties, body):
        order = json.loads(body)
        print(f"Processing order: {order}")
        # ... ประมวลผลคำสั่งซื้อจริง ...
        ch.basic_ack(delivery_tag=method.delivery_tag)                   # ยืนยันว่างานเสร็จแล้ว -> ลบออกจากคิว

    channel.basic_consume(queue='order_processing', on_message_callback=callback)
    channel.start_consuming()                                            # รอรับงานต่อไปเรื่อยๆ

# ===== Kafka: Event Streaming (ใช้ library kafka-python) =====
from kafka import KafkaProducer, KafkaConsumer

def kafka_producer(user_activity):
    producer = KafkaProducer(
        bootstrap_servers='localhost:9092',
        value_serializer=lambda v: json.dumps(v).encode()
    )
    producer.send('user_activity', value=user_activity)                 # ส่งเข้า topic "user_activity"
    producer.flush()

def kafka_consumer_analytics():                                          # Consumer Group: "analytics"
    consumer = KafkaConsumer(
        'user_activity',
        bootstrap_servers='localhost:9092',
        group_id='analytics',                                             # ระบุ Consumer Group ของตัวเอง
        value_deserializer=lambda v: json.loads(v.decode())
    )
    for message in consumer:                                              # วนอ่านข้อความใหม่ที่เข้ามาเรื่อยๆ
        print(f"[Analytics] Processing: {message.value}")

def kafka_consumer_fraud_detection():                                    # Consumer Group: "fraud-detection"
    consumer = KafkaConsumer(
        'user_activity',
        bootstrap_servers='localhost:9092',
        group_id='fraud-detection',                                        # คนละ group -> อ่าน topic เดียวกันแยกอิสระ
        value_deserializer=lambda v: json.loads(v.decode())
    )
    for message in consumer:
        print(f"[Fraud Detection] Checking: {message.value}")
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `rabbitmq_producer` — `queue_declare(..., durable=True)` บอก RabbitMQ ให้**บันทึกคิวนี้ไว้แม้ broker จะ restart** และ `delivery_mode=2` บอกให้**บันทึกข้อความแต่ละชิ้นลง disk ด้วย** (ไม่ใช่แค่เก็บใน RAM) — สองการตั้งค่านี้ร่วมกันรับประกันว่าข้อความจะไม่หายแม้ RabbitMQ จะล่มกะทันหันระหว่างที่ยังไม่มี Consumer มารับ
2. `channel.basic_qos(prefetch_count=1)` — บอก RabbitMQ ว่า worker ตัวนี้ **รับงานได้ทีละ 1 ชิ้นเท่านั้น** จนกว่าจะ ack งานปัจจุบันก่อน — ถ้าไม่ตั้งค่านี้ RabbitMQ อาจส่งงานหลายชิ้นให้ worker ตัวเดียวพร้อมกันโดยไม่สนใจว่า worker จะทำทันหรือไม่ ทำให้งานกระจายไม่เท่าเทียมกันระหว่าง worker หลายตัว
3. `ch.basic_ack(delivery_tag=method.delivery_tag)` — **ยืนยันกับ RabbitMQ ว่างานนี้ประมวลผลเสร็จสมบูรณ์แล้ว** RabbitMQ จึงจะลบข้อความนี้ออกจากคิวจริง — ถ้า worker ล่มก่อนเรียก `basic_ack` (เช่น เกิด exception ระหว่างประมวลผล) RabbitMQ จะเห็นว่ายังไม่ได้ ack และ**ส่งข้อความเดิมนี้ให้ worker ตัวอื่นทำใหม่โดยอัตโนมัติ** — นี่คือกลไกที่รับประกันว่างานจะไม่หายแม้ worker ล่มกลางคัน (แต่ต้องออกแบบให้งานทำซ้ำได้อย่างปลอดภัย หรือที่เรียกว่า **Idempotent**)
4. `kafka_consumer_analytics` และ `kafka_consumer_fraud_detection` — ทั้งสองฟังก์ชันอ่านจาก topic **`user_activity` เดียวกัน** แต่ระบุ `group_id` **ต่างกัน** (`analytics` vs `fraud-detection`) — Kafka ติดตาม**ตำแหน่งอ่านล่าสุด (offset) แยกกันตาม group_id** ทำให้ทั้งสอง Consumer Group อ่านข้อความชุดเดียวกันได้อย่างอิสระโดยไม่กระทบกันเลย (Consumer หนึ่งอ่านเร็วหรือช้ากว่าอีกฝั่งก็ไม่เป็นไร) — ต่างจาก RabbitMQ ที่ข้อความหนึ่งชิ้นถูก consume ไปแล้วจะหายจากคิวทันที ไม่มีทาง consume ซ้ำได้อีก

## Time Complexity

- Message Queue operations (publish/consume): **O(1)** ต่อข้อความหนึ่งชิ้น (โครงสร้างพื้นฐานเป็น Queue — FIFO จากบทที่ 14)
- Kafka: การอ่าน log แบบเรียงลำดับ (sequential read) ทำให้เร็วกว่าการอ่านแบบสุ่มมาก แม้จะเก็บข้อมูลบน disk (เชื่อมโยงกับหลักการ Page Fault/sequential access จากบทที่ 45 — sequential I/O เร็วกว่า random I/O เสมอ) นี่คือเหตุผลหลักที่ Kafka มี throughput สูงมากแม้จะเขียนข้อมูลลง disk จริง

## Space Complexity

- Message Queue ใช้พื้นที่ตามจำนวนข้อความที่ค้างอยู่ในคิว (ยังไม่ถูก consume หรือยังไม่หมด retention period)
- Kafka เก็บ log ไว้นานตามที่ตั้งค่า **retention period** (เช่น 7 วัน) ไม่ว่าจะมี Consumer อ่านไปแล้วหรือไม่ ทำให้ใช้พื้นที่ disk มากกว่า RabbitMQ ที่มักลบข้อความทันทีหลัง consume สำเร็จ

## แนวทางปฏิบัติที่ดี (Best Practices)

- เลือก **RabbitMQ** เมื่อต้องการ Task Queue ที่แต่ละงานถูกประมวลผลครั้งเดียวแน่นอน (เช่น ส่งอีเมล, ประมวลผลคำสั่งซื้อ) และต้องการ routing ที่ซับซ้อน (ส่งงานไปคิวต่างกันตามเงื่อนไข)
- เลือก **Kafka** เมื่อต้องการ Event Streaming ที่ข้อมูลปริมาณมากต้องถูกใช้โดยหลายระบบพร้อมกัน หรือต้องการเก็บประวัติข้อมูลไว้ให้ระบบใหม่ในอนาคตอ่านย้อนหลังได้
- ออกแบบ Consumer ให้เป็น **Idempotent เสมอ** (ประมวลผลซ้ำได้โดยผลลัพธ์ไม่เปลี่ยน) เพราะทั้ง RabbitMQ และ Kafka อาจส่งข้อความซ้ำได้ในบางสถานการณ์ (เช่น worker ล่มก่อน ack แต่ประมวลผลเสร็จไปแล้วบางส่วน)

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ลืมตั้งค่า `durable=True` และ `delivery_mode=2` ใน RabbitMQ ทำให้ข้อความหายไปถ้า broker restart หรือล่มกะทันหัน
- ลืม `basic_ack` หรือ ack ก่อนประมวลผลเสร็จจริง (ack ทันทีที่ได้รับงานแทนที่จะ ack หลังทำเสร็จ) ทำให้ถ้า worker ล่มระหว่างทำงาน ข้อความนั้นหายไปเลยโดยไม่มีใครทำงานนั้นซ้ำ
- ออกแบบ Consumer ที่ไม่ Idempotent (เช่น เพิ่มยอดเงินโดยตรงทุกครั้งที่ได้รับข้อความ) ทำให้ถ้าข้อความถูกส่งซ้ำ (duplicate delivery) เกิดผลลัพธ์ผิดพลาด (เพิ่มยอดเงินซ้ำสองครั้ง)
- ใช้ Kafka สำหรับงานที่ต้องการแค่ Task Queue ธรรมดา (งานเดียวทำครั้งเดียวโดย worker ตัวเดียว) ทำให้ระบบซับซ้อนเกินความจำเป็นเมื่อเทียบกับ RabbitMQ ที่เหมาะกับงานนี้มากกว่า

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง RabbitMQ และ Kafka พร้อมยกตัวอย่างสถานการณ์ที่เหมาะกับแต่ละแบบ
2. Message Queue แก้ปัญหาอะไรที่การเรียก service ตรงๆ แบบ synchronous (บทที่ 43) ทำไม่ได้
3. อธิบาย Consumer Group ใน Kafka คืออะไร ทำไมหลาย Consumer Group ถึงอ่าน topic เดียวกันได้โดยไม่กระทบกัน
4. Idempotent Consumer คืออะไร ทำไมถึงสำคัญเมื่อใช้ Message Queue
5. อธิบายว่า `basic_ack` ใน RabbitMQ ทำงานอย่างไร และเกิดอะไรขึ้นถ้า worker ล่มก่อน ack

## แบบฝึกหัด (Practice Exercises)

1. เขียน Producer-Consumer ด้วย RabbitMQ ที่จำลองระบบส่งอีเมล โดยมี worker หลายตัวแข่งกันรับงานจากคิวเดียวกัน (Competing Consumers) แล้วพิสูจน์ว่าแต่ละอีเมลถูกส่งแค่ครั้งเดียว
2. เขียน Kafka Producer ที่ส่ง event การซื้อสินค้า แล้วสร้าง Consumer Group 2 กลุ่มที่อ่าน topic เดียวกัน (กลุ่มหนึ่งคำนวณยอดขายรวม อีกกลุ่มตรวจจับการซื้อผิดปกติ) พิสูจน์ว่าทั้งสองกลุ่มทำงานอิสระจากกัน
3. เขียนฟังก์ชัน Consumer ที่ไม่ Idempotent แล้วจำลองการส่งข้อความซ้ำ (duplicate delivery) เพื่อดูผลลัพธ์ที่ผิดพลาด จากนั้นแก้ไขให้เป็น Idempotent (เช่น เช็ค order_id ที่เคยประมวลผลไปแล้วก่อนทำซ้ำ)

## Mini Project

**"ระบบประมวลผลคำสั่งซื้อแบบกระจายด้วย Message Queue (Distributed Order Processing with Message Queue)"**: เขียนโปรแกรมที่:
1. ใช้ RabbitMQ (หรือจำลองด้วย library ในเครื่อง) สร้างระบบ Task Queue สำหรับประมวลผลคำสั่งซื้อ พร้อม worker หลายตัวที่แข่งกันรับงาน (Competing Consumers) และ Idempotent design ป้องกันการประมวลผลซ้ำ
2. ใช้ Kafka (หรือจำลอง) ส่ง event ทุกครั้งที่มีคำสั่งซื้อสำเร็จ ไปยัง 2 Consumer Group แยกกัน: หนึ่งสำหรับสรุปยอดขายรายวัน อีกหนึ่งสำหรับตรวจจับรูปแบบการซื้อที่น่าสงสัย
3. จำลองสถานการณ์ worker ล่มกลางคันขณะประมวลผล (จงใจโยน exception ก่อน ack) แล้วพิสูจน์ว่างานนั้นถูกส่งให้ worker ตัวอื่นทำต่อโดยอัตโนมัติ โดยไม่มีคำสั่งซื้อไหนหายไปหรือถูกประมวลผลซ้ำผิดพลาด
4. เขียนรายงานเปรียบเทียบว่าถ้าใช้ Synchronous REST call แทน Message Queue (ตามบทที่ 43) จะมีปัญหาอะไรบ้างเมื่อ worker/service ปลายทางล่มหรือทำงานช้า

## ข้อคิดสำคัญ (Key Takeaways)

- Message Queue เก็บข้อความไว้ชั่วคราวระหว่าง Producer และ Consumer ทำให้ทั้งสองฝั่งทำงานแบบ asynchronous และ scale อิสระจากกันได้ ต่อยอดจาก Event-Driven Architecture ในบทที่ 49
- RabbitMQ เหมาะกับ Task Queue ที่แต่ละงานถูกประมวลผลครั้งเดียวโดย worker ตัวใดตัวหนึ่ง (Competing Consumers Pattern)
- Kafka เหมาะกับ Event Streaming ที่เก็บข้อความเป็น log ให้หลาย Consumer Group อ่านซ้ำและอิสระจากกันได้
- `durable`/`delivery_mode` ใน RabbitMQ และ `basic_ack` รับประกันว่าข้อความจะไม่หายแม้ broker หรือ worker ล่มกะทันหัน
- Consumer ควรออกแบบให้ Idempotent เสมอ เพราะ Message Queue อาจส่งข้อความซ้ำได้ในบางสถานการณ์
- นี่คือเครื่องมือสำคัญที่ทำให้ระบบขนาดใหญ่ประมวลผลงานจำนวนมากได้อย่างเสถียรและทนทานต่อความล้มเหลวบางส่วน บทถัดไปจะปิดท้าย Part 16: System Design ด้วย Rate Limiting, Sharding, Replication ซึ่งเป็นเทคนิคปกป้องระบบและกระจายข้อมูลในระดับฐานข้อมูล

---
*บทต่อไป (บทที่ 57): Rate Limiting, Sharding, Replication — พิมพ์ "Next" เพื่อดำเนินการต่อ*
