# บทที่ 57: Rate Limiting, Sharding, Replication

## แนวคิดหลัก (Concept)

สองบทที่ผ่านมาสอนวิธีขยายระบบให้รองรับโหลดมาก (Scaling, Caching, Message Queue) บทนี้เป็นบทปิดท้าย **Part 16: System Design** ด้วยสามเทคนิคที่ปกป้องระบบจากการใช้งานเกินขีดจำกัดและกระจายข้อมูลในระดับฐานข้อมูล — คำถามที่ต่อเนื่องจากบทที่แล้ว: ถ้าระบบรองรับโหลดได้มากขึ้นแล้ว จะ**ป้องกันไม่ให้ผู้ใช้คนเดียวถล่มระบบ**ได้อย่างไร และเมื่อข้อมูลใหญ่เกินกว่าเครื่องเดียวจะเก็บไหว จะ**แบ่งข้อมูลออกไปหลายเครื่อง**อย่างไร

- **Rate Limiting (การจำกัดอัตรา)** — เทคนิคจำกัดจำนวน request ที่ผู้ใช้/client หนึ่งรายทำได้ในช่วงเวลาหนึ่ง เพื่อป้องกันการใช้งานเกินขีดจำกัดหรือการโจมตี (เชื่อมโยงกับความปลอดภัยจาก Part 15)
- **Sharding (การแบ่งฐานข้อมูล)** — เทคนิคแบ่งข้อมูลในฐานข้อมูลเดียวออกเป็นหลายส่วน (shard) กระจายไปเก็บในหลายเครื่อง โดยแต่ละเครื่องเก็บแค่**บางส่วน**ของข้อมูลทั้งหมด (ต่างจาก Replication ที่แต่ละเครื่องเก็บ**ข้อมูลเดียวกันทั้งหมด**)
- **Replication (การทำสำเนา)** — เทคนิคสร้างสำเนาของข้อมูลเดียวกันไว้หลายเครื่อง (มักเป็น **Primary/Leader** ที่รับการเขียน และ **Replica/Follower** หลายตัวที่รับการอ่าน) เพื่อความทนทานต่อความล้มเหลวและกระจายภาระการอ่าน

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Rate Limiting มีไว้ป้องกันทั้งการโจมตีและการใช้งานผิดพลาดโดยไม่ตั้งใจ**: แม้จะมี Load Balancer และ Cache (บทที่ 55) ช่วยรองรับโหลดแล้ว แต่ถ้าผู้ใช้คนเดียว (หรือบอทที่เขียนโค้ดผิดพลาด) ยิง request รัวๆ ไม่หยุด ระบบยังคงล่มได้ — Rate Limiting ตัดปัญหานี้ตั้งแต่ต้นทาง ก่อนที่ request จะไปถึงส่วนอื่นของระบบเลย
- **Sharding มีไว้แก้ปัญหาที่ Replication แก้ไม่ได้: ข้อมูลใหญ่เกินกว่าเครื่องเดียวจะเก็บไหว**: Replication ช่วยเรื่องการอ่าน (มีหลายสำเนาให้กระจายอ่าน) แต่ถ้าข้อมูลทั้งหมดใหญ่กว่าพื้นที่ disk ของเครื่องเดียว (เช่น ฐานข้อมูลผู้ใช้ Facebook หลายพันล้านคน) การมีสำเนาซ้ำก็ยังคงเก็บไม่พอ — Sharding แบ่งข้อมูลออกเป็นส่วนย่อยที่แต่ละเครื่องเก็บแค่บางส่วน ทำให้ขนาดข้อมูลรวมไม่มีเพดานตายตัวอีกต่อไป
- **Replication มีไว้ป้องกันข้อมูลสูญหายและกระจายภาระการอ่าน**: ถ้ามีข้อมูลอยู่ที่เดียว เครื่องนั้นล่มเท่ากับข้อมูลหายหมด (Single Point of Failure) — Replication ทำให้มีสำเนาสำรองพร้อมใช้งานทันทีถ้า Primary ล่ม (High Availability) และยังช่วยกระจายภาระ query แบบอ่านอย่างเดียว (read-heavy workload) ไปยัง Replica หลายตัวได้ด้วย (ต่อยอดจากแนวคิด Load Balancer บทที่ 55)

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Rate Limiting** เหมือน **ประตูหมุนที่ผ่านได้ทีละคนตามจังหวะที่กำหนด** — ไม่ว่าจะมีคนแห่กันมากี่คนพร้อมกัน ประตูก็ยอมให้ผ่านได้แค่อัตราที่กำหนดไว้เท่านั้น ป้องกันไม่ให้ฝูงชนแห่เข้ามาพร้อมกันจนโกลาหล
- **Sharding** เหมือน **การแบ่งหนังสือในห้องสมุดขนาดใหญ่ไปเก็บตามสาขาต่างๆ ตามหมวดหมู่ (A-M ที่สาขา 1, N-Z ที่สาขา 2)** — แต่ละสาขาเก็บแค่บางส่วนของหนังสือทั้งหมด รวมกันแล้วเก็บได้มากกว่าสาขาเดียวเก็บทุกเล่ม
- **Replication** เหมือน **การถ่ายเอกสารสำคัญเก็บไว้หลายที่ (บ้าน, ที่ทำงาน, กล่องนิรภัยธนาคาร)** — ถ้าที่หนึ่งเกิดไฟไหม้ ยังมีสำเนาที่อื่นให้ใช้ได้ทันที และถ้าหลายคนอยากอ่านเอกสารพร้อมกัน แต่ละคนก็ไปอ่านจากสำเนาคนละที่ได้โดยไม่ต้องแย่งกัน

## แผนภาพอธิบาย (Visual explanation)

### Rate Limiting — Token Bucket Algorithm (อัลกอริทึมที่นิยมที่สุด)

```
Bucket ความจุ 5 token, เติม token ใหม่ 1 ตัวทุก 1 วินาที

เวลา 0s: [●●●●●] (5 token เต็ม)
Request 1 มา -> ใช้ 1 token -> [●●●●○] (เหลือ 4) -> อนุญาต
Request 2 มา -> ใช้ 1 token -> [●●●○○] (เหลือ 3) -> อนุญาต
Request 3,4,5 มาพร้อมกัน -> ใช้ token จนหมด -> [○○○○○] -> อนุญาตทั้งหมด (มี token พอ)
Request 6 มาทันที -> ไม่มี token เหลือ -> ปฏิเสธ! (429 Too Many Requests)

เวลา 1s: เติม token ใหม่ 1 ตัว -> [●○○○○] -> Request ถัดไปผ่านได้ 1 ครั้ง

Token Bucket ยอมให้ "burst" (ยิงรัวๆ ช่วงสั้น) ได้ถ้ามี token สะสมไว้ แต่ไม่เกินอัตราเฉลี่ยระยะยาว
```

### Sharding — แบ่งข้อมูลตาม Shard Key

```
ข้อมูลผู้ใช้ทั้งหมด (user_id 1 ถึง 1,000,000)

Shard Key: user_id % 3   (แบ่งตามเศษเหลือจากการหาร - Hash-based Sharding)

Shard 0 (user_id % 3 == 0): user 3, 6, 9, ... 999999      <- เครื่อง A
Shard 1 (user_id % 3 == 1): user 1, 4, 7, ... 999997      <- เครื่อง B
Shard 2 (user_id % 3 == 2): user 2, 5, 8, ... 999998      <- เครื่อง C

Query "หา user_id=7" -> คำนวณ 7 % 3 = 1 -> ไปที่ Shard 1 (เครื่อง B) โดยตรง ไม่ต้องถามเครื่องอื่นเลย
แต่ละเครื่องเก็บแค่ 1/3 ของข้อมูลทั้งหมด -> รวมกันเก็บได้มากกว่าเครื่องเดียวเก็บทั้งหมด
```

### Replication — Primary-Replica Architecture

```
                     Write Request
                          |
                          v
                    [Primary/Leader]  <- รับการเขียนเท่านั้น (single source of truth)
                    /      |       \
              (sync/async replication)
                  /        |         \
                 v         v          v
          [Replica 1] [Replica 2] [Replica 3]  <- รับการอ่านเท่านั้น กระจายภาระ query

Read Request ---------> Load Balancer -----> กระจายไปยัง Replica ตัวใดตัวหนึ่ง

ถ้า Primary ล่ม -> เลื่อน Replica ตัวหนึ่งขึ้นเป็น Primary ใหม่ (Failover) -> ระบบยังทำงานต่อได้
```

## ตัวอย่างโค้ด (Code example)

```python
import time
import hashlib

# ===== Rate Limiting: Token Bucket Algorithm =====
class TokenBucket:
    def __init__(self, capacity, refill_rate):                # refill_rate = token ต่อวินาที
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate
        self.last_refill = time.time()

    def _refill(self):
        now = time.time()
        elapsed = now - self.last_refill
        new_tokens = elapsed * self.refill_rate
        self.tokens = min(self.capacity, self.tokens + new_tokens)   # เติมแต่ไม่เกินความจุ
        self.last_refill = now

    def allow_request(self):
        self._refill()
        if self.tokens >= 1:
            self.tokens -= 1
            return True                                             # มี token พอ -> อนุญาต
        return False                                                 # token หมด -> ปฏิเสธ (429)

buckets = {}                                                       # เก็บ bucket แยกตาม client (เช่น ตาม API key)
def rate_limit_middleware(client_id):
    if client_id not in buckets:
        buckets[client_id] = TokenBucket(capacity=5, refill_rate=1)   # 5 request burst, เติม 1/วินาที
    return buckets[client_id].allow_request()

# ===== Sharding: กระจายข้อมูลตาม Hash ของ Shard Key =====
class ShardedDatabase:
    def __init__(self, num_shards):
        self.shards = [{} for _ in range(num_shards)]                 # จำลอง num_shards เครื่องด้วย dict
        self.num_shards = num_shards

    def _get_shard(self, key):
        shard_index = int(hashlib.md5(str(key).encode()).hexdigest(), 16) % self.num_shards
        return self.shards[shard_index]                                 # Hash-based Sharding (กระจายสม่ำเสมอกว่า % ตรงๆ)

    def put(self, user_id, data):
        shard = self._get_shard(user_id)
        shard[user_id] = data

    def get(self, user_id):
        shard = self._get_shard(user_id)                                 # คำนวณ hash ครั้งเดียว รู้ทันทีว่าอยู่ shard ไหน
        return shard.get(user_id)

db = ShardedDatabase(num_shards=3)
db.put(user_id=42, data={"name": "Alice"})
print(db.get(user_id=42))                                                  # {"name": "Alice"} -- หาเจอทันทีจาก shard ที่ถูกต้อง

# ===== Replication: Primary รับเขียน, Replica รับอ่าน =====
class ReplicatedDatabase:
    def __init__(self, num_replicas):
        self.primary = {}
        self.replicas = [{} for _ in range(num_replicas)]
        self.replica_index = 0

    def write(self, key, value):
        self.primary[key] = value                                          # เขียนที่ Primary เท่านั้น
        for replica in self.replicas:                                       # จำลอง replication ไปยังทุก replica
            replica[key] = value

    def read(self, key):                                                     # กระจายการอ่านไปยัง replica แบบ round robin
        replica = self.replicas[self.replica_index]
        self.replica_index = (self.replica_index + 1) % len(self.replicas)
        return replica.get(key)                                              # อ่านจาก replica ไม่ใช่ primary

repl_db = ReplicatedDatabase(num_replicas=3)
repl_db.write("config", "production")
print(repl_db.read("config"))    # อ่านจาก replica ตัวที่ 1
print(repl_db.read("config"))    # อ่านจาก replica ตัวที่ 2 (กระจายภาระ)
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `TokenBucket._refill()` — คำนวณเวลาที่ผ่านไปตั้งแต่เติม token ครั้งล่าสุด (`elapsed`) แล้วคำนวณจำนวน token ใหม่ที่ควรได้รับ (`elapsed * refill_rate`) — เทคนิคนี้เรียกว่า **Lazy Refill**: ไม่ต้องมี timer แยกต่างหากคอยเติม token ทุกวินาที แต่คำนวณจาก**เวลาที่ผ่านไปจริง**ทุกครั้งที่มี request เข้ามาแทน ประหยัดทรัพยากรกว่าการรัน background job
2. `allow_request()` — เช็คว่า `self.tokens >= 1` ก่อนเสมอ ถ้ามีพอจะหักออก 1 หน่วยแล้วคืนค่า `True` (อนุญาต) ถ้าไม่พอคืนค่า `False` (ปฏิเสธ) — สังเกตว่า bucket ที่มี token สะสมไว้เต็ม (`capacity=5`) ยอมให้ request พุ่งเข้ามาทีเดียว 5 ครั้งได้ (**burst**) แต่หลังจากนั้นต้องรอให้ token เติมใหม่ตามอัตรา `refill_rate` เท่านั้น — นี่คือความยืดหยุ่นของ Token Bucket เทียบกับอัลกอริทึมที่จำกัดอัตราคงที่ตรงๆ โดยไม่ยอมให้ burst เลย
3. `ShardedDatabase._get_shard(key)` — ใช้ `hashlib.md5(...)` แปลง `key` (เช่น `user_id`) เป็นค่า hash แล้ว `% self.num_shards` เพื่อกำหนดว่าข้อมูลนี้ควรอยู่ shard ไหน — การใช้ **Hash function** แทนการ `% ` ตรงๆ กับ `key` (ถ้า key เป็นตัวเลขต่อเนื่องกัน) ทำให้ข้อมูลกระจายไปยังแต่ละ shard ได้**สม่ำเสมอกว่ามาก** แม้ key จะไม่กระจายตัวสม่ำเสมอโดยธรรมชาติ (เช่น user_id ส่วนใหญ่อาจเป็นเลขที่ลงท้ายด้วย 0 พร้อมกันเยอะๆ)
4. `ReplicatedDatabase.write(key, value)` — เขียนที่ `self.primary` ก่อน แล้ว**คัดลอกข้อมูลเดียวกันไปยังทุก replica** (ในระบบจริง การ replicate นี้อาจเป็นแบบ **synchronous** (รอให้ replica เขียนเสร็จก่อนถือว่า write สำเร็จ — ปลอดภัยกว่าแต่ช้ากว่า) หรือ **asynchronous** (ไม่รอ replica เลย เร็วกว่าแต่เสี่ยงข้อมูลไม่ตรงกันชั่วขณะถ้า Primary ล่มก่อน replicate เสร็จ) — trade-off นี้เชื่อมโยงโดยตรงกับ CAP Theorem จากบทที่ 41
5. `ReplicatedDatabase.read(key)` — **ไม่เคยอ่านจาก `self.primary` เลย** อ่านจาก `self.replicas` แบบ Round Robin (เทคนิคเดียวกับ Load Balancer บทที่ 55) เสมอ — การแยกทางอ่าน (ไปที่ Replica) ออกจากทางเขียน (ไปที่ Primary) ทำให้กระจายภาระของระบบที่มีการอ่านมากกว่าเขียนมาก (read-heavy) ได้อย่างมีประสิทธิภาพ

## Time Complexity

- Token Bucket check: **O(1)** ต่อ request
- Sharding: การหา shard ที่ถูกต้องด้วย hash function ใช้ **O(1)** (หรือ O(k) ตามความยาวของ key ที่ใช้คำนวณ hash) — เร็วกว่าการค้นหาทีละเครื่องมาก
- Replication write: **O(r)** โดย r คือจำนวน replica ที่ต้อง sync ข้อมูลไปให้ (แต่ในทางปฏิบัติมักทำแบบขนานพร้อมกัน ไม่ใช่ทีละตัว)

## Space Complexity

- Sharding: ข้อมูลรวม **O(n)** ทั้งหมด แต่ **กระจายเป็น O(n/k)** ต่อเครื่อง โดย k คือจำนวน shard — นี่คือประโยชน์หลักของ Sharding เมื่อเทียบกับ Replication ที่แต่ละเครื่องยังคงต้องเก็บ **O(n)** เต็มจำนวน (สำเนาทั้งหมด)

## แนวทางปฏิบัติที่ดี (Best Practices)

- เลือก **Rate Limiting Algorithm** ให้เหมาะกับสถานการณ์: Token Bucket เหมาะกับ API ที่ยอมให้ burst ได้บ้าง (เช่น ผู้ใช้ทั่วไป), Fixed Window/Sliding Window เหมาะกับการจำกัดที่เข้มงวดกว่า (เชื่อมโยงกับ Sliding Window บทที่ 28)
- เลือก **Shard Key** ที่กระจายข้อมูลสม่ำเสมอและตรงกับรูปแบบ query ที่ใช้บ่อยที่สุด (เช่น ถ้า query ส่วนใหญ่ค้นด้วย `user_id` ให้ shard ตาม `user_id` ไม่ใช่ตามคอลัมน์อื่น) เพื่อหลีกเลี่ยง **Hot Shard** (shard หนึ่งรับภาระมากกว่าตัวอื่นมาก)
- ใช้ **Asynchronous Replication** เมื่อความเร็วสำคัญกว่าความสอดคล้องแบบทันที (เช่น social media feed) และใช้ **Synchronous Replication** เมื่อข้อมูลต้องถูกต้องแน่นอนเสมอ (เช่น ธุรกรรมทางการเงิน) — trade-off นี้สะท้อน CAP Theorem จากบทที่ 41 โดยตรง

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ตั้งค่า Rate Limit ที่เข้มงวดเกินไปจนผู้ใช้ปกติถูกบล็อกโดยไม่ตั้งใจ หรือหลวมเกินไปจนป้องกันการโจมตีไม่ได้จริง
- เลือก Shard Key ที่ไม่เหมาะสม (เช่น shard ตาม "วันที่สมัคร" ทำให้ข้อมูลของผู้ใช้ใหม่ล่าสุดกระจุกอยู่ shard เดียว) ทำให้เกิด Hot Shard ที่รับภาระมากกว่าตัวอื่นอย่างไม่สมดุล
- อ่านข้อมูลจาก Replica ทันทีหลังเขียนที่ Primary โดยไม่รู้ว่า Replication อาจยังไม่เสร็จ (โดยเฉพาะ Asynchronous Replication) ทำให้เห็นข้อมูลเก่าชั่วขณะ (**Replication Lag**) ซึ่งอาจสร้างความสับสนถ้าไม่ได้ออกแบบระบบให้รองรับสถานการณ์นี้
- คิดว่า Replication แก้ปัญหาข้อมูลใหญ่เกินเครื่องเดียวได้ (ที่จริง Replication ทำให้แต่ละเครื่องมีข้อมูลเท่ากันทั้งหมด ไม่ได้ลดขนาดข้อมูลต่อเครื่องเลย ต้องใช้ Sharding ต่างหากสำหรับปัญหานี้)

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบาย Token Bucket Algorithm ทำงานอย่างไร ทำไมถึงยอมให้ "burst" ได้
2. Sharding ต่างจาก Replication อย่างไร แก้ปัญหาคนละแบบอะไรบ้าง
3. อธิบายว่า Hot Shard คืออะไร เกิดจากการเลือก Shard Key ที่ไม่ดีอย่างไร
4. Synchronous Replication ต่างจาก Asynchronous Replication อย่างไร แต่ละแบบมี trade-off อะไรบ้าง เชื่อมโยงกับ CAP Theorem
5. ทำไมการอ่านข้อมูลจาก Replica ทันทีหลังเขียนที่ Primary อาจได้ข้อมูลเก่า? ปัญหานี้เรียกว่าอะไร

## แบบฝึกหัด (Practice Exercises)

1. Implement Rate Limiting แบบ **Sliding Window** (นับจำนวน request ในช่วงเวลาที่เลื่อนไปเรื่อยๆ ต่อยอดจากบทที่ 28) แล้วเปรียบเทียบพฤติกรรมกับ Token Bucket
2. เขียนโปรแกรมทดสอบว่า Hash-based Sharding กระจายข้อมูล 100,000 user_id ไปยัง 5 shard ได้สม่ำเสมอแค่ไหน (นับจำนวนข้อมูลในแต่ละ shard แล้วเปรียบเทียบ)
3. จำลอง Replication Lag: เขียน Primary-Replica ที่ replica อัปเดตข้อมูลช้ากว่า primary 2 วินาที (ใช้ `time.sleep`) แล้วทดสอบว่าอ่านจาก replica ทันทีหลังเขียนจะได้ข้อมูลเก่าจริงหรือไม่

## Mini Project

**"ระบบฐานข้อมูลกระจายจำลองพร้อม Rate Limiting (Distributed Database Simulator with Rate Limiting)"**: เขียนโปรแกรมที่:
1. สร้าง `ShardedDatabase` ที่รองรับข้อมูลผู้ใช้จำนวนมาก (จำลอง 1,000,000 รายการ) กระจายไปยังหลาย shard ด้วย Hash-based Sharding
2. เพิ่ม `ReplicatedDatabase` ให้แต่ละ shard มี replica ของตัวเอง (Sharding + Replication ผสมกัน ตามที่ระบบจริงส่วนใหญ่ทำ) พร้อมระบบ Failover ที่เลื่อน replica ขึ้นเป็น primary อัตโนมัติเมื่อ primary จำลองล่ม
3. เพิ่ม Rate Limiting middleware ที่ป้องกันไม่ให้ client เดียวยิง request เกินขีดจำกัดไปยังระบบฐานข้อมูลนี้
4. ทดสอบระบบทั้งหมดด้วยการจำลอง traffic จำนวนมากพร้อมกัน วัดว่าข้อมูลกระจายสม่ำเสมอหรือไม่ (ไม่มี Hot Shard) และ Rate Limiting ทำงานถูกต้องหรือไม่ (ปฏิเสธ request ที่เกินขีดจำกัดจริง)

## ข้อคิดสำคัญ (Key Takeaways)

- Rate Limiting ป้องกันระบบจากการใช้งานเกินขีดจำกัดหรือการโจมตี โดย Token Bucket เป็นอัลกอริทึมที่นิยมเพราะยอมให้ burst ได้ในขณะที่ยังจำกัดอัตราเฉลี่ยระยะยาว
- Sharding แบ่งข้อมูลออกเป็นส่วนย่อยกระจายไปหลายเครื่อง แก้ปัญหาข้อมูลใหญ่เกินกว่าเครื่องเดียวจะเก็บไหว โดยแต่ละเครื่องเก็บแค่บางส่วนของข้อมูลทั้งหมด
- Replication สร้างสำเนาของข้อมูลเดียวกันไว้หลายเครื่อง แก้ปัญหาความทนทานต่อความล้มเหลว (Failover) และกระจายภาระการอ่าน
- Sharding กับ Replication แก้ปัญหาคนละแบบและมักใช้ร่วมกันในระบบจริง (แต่ละ shard มี replica ของตัวเอง)
- การเลือก Shard Key ที่ดีและการเลือกระหว่าง Synchronous/Asynchronous Replication ล้วนเป็น trade-off ที่เชื่อมโยงกับ CAP Theorem จากบทที่ 41 โดยตรง
- นี่คือบทปิดท้าย **Part 16: System Design** — ตลอด 3 บทที่ผ่านมา (Scalability/Caching/CDN, Message Queues, Rate Limiting/Sharding/Replication) วางรากฐานการออกแบบระบบขนาดใหญ่ที่รองรับผู้ใช้จำนวนมากได้อย่างเสถียรและปลอดภัย บทถัดไปจะเข้าสู่ **Part 17: DevOps** ซึ่งเป็นบทสุดท้ายของหลักสูตร โดยเริ่มจาก Docker, Kubernetes

---
*บทต่อไป (บทที่ 58): Docker, Kubernetes — พิมพ์ "Next" เพื่อดำเนินการต่อ*
