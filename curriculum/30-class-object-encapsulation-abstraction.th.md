# บทที่ 30: Class & Object, Encapsulation, Abstraction

## แนวคิดหลัก (Concept)

ตลอด Part 1-5 เราเขียนโปรแกรมด้วยฟังก์ชันและโครงสร้างข้อมูล (array, dict, list) แยกจากกัน — ข้อมูลอยู่ที่หนึ่ง ฟังก์ชันที่จัดการข้อมูลอยู่อีกที่หนึ่ง บทนี้เริ่มต้น **Part 6: Object-Oriented Programming (OOP)** ซึ่งเป็นแนวคิดการออกแบบโค้ดที่**รวมข้อมูลและฟังก์ชันที่จัดการข้อมูลนั้นเข้าไว้ด้วยกัน**เป็นหน่วยเดียว ทำให้โค้ดจัดการง่ายขึ้นเมื่อโปรแกรมมีขนาดใหญ่และซับซ้อน

- **Class (คลาส)** — พิมพ์เขียว (blueprint) ที่นิยามว่า object ประเภทนี้จะมี**ข้อมูล** (attributes/properties) และ**พฤติกรรม** (methods) อะไรบ้าง
- **Object (อ็อบเจกต์) / Instance** — สิ่งที่ถูกสร้างขึ้นจริงจาก Class หนึ่งๆ แต่ละ object มีข้อมูลของตัวเอง แต่ใช้พฤติกรรม (methods) ร่วมกันตามที่ Class กำหนด
- **Encapsulation (การห่อหุ้ม)** — การซ่อนรายละเอียดภายในของ object ไว้ ไม่ให้โค้ดภายนอกเข้าถึงหรือแก้ไขข้อมูลโดยตรง ต้องผ่าน method ที่กำหนดไว้เท่านั้น
- **Abstraction (การนามธรรม)** — การซ่อน**ความซับซ้อนของการทำงานภายใน** ไว้เบื้องหลัง interface ที่เรียบง่าย ผู้ใช้เห็นแค่ "ทำอะไรได้" ไม่ต้องรู้ว่า "ทำอย่างไรข้างใน"

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **มีไว้จัดการความซับซ้อนเมื่อโปรแกรมมีขนาดใหญ่ขึ้น**: การเขียนฟังก์ชันแยกจากข้อมูล (procedural programming ที่ใช้มาตลอด Part 1-5) ใช้ได้ดีกับโปรแกรมขนาดเล็ก แต่เมื่อระบบใหญ่ขึ้น (เช่น ระบบธนาคารที่มีบัญชี, ผู้ใช้, ธุรกรรมนับพันจุด) การรวมข้อมูลกับพฤติกรรมที่เกี่ยวข้องไว้ด้วยกันทำให้โค้ดมีโครงสร้างที่สะท้อนโลกจริงและดูแลรักษาง่ายกว่ามาก
- **Encapsulation มีไว้ป้องกันข้อมูลเสียหายจากการแก้ไขที่ไม่ถูกต้อง**: ถ้าข้อมูลเข้าถึงได้อย่างอิสระจากทุกที่ในโปรแกรม (เหมือน global variable) โค้ดส่วนไหนก็แก้ไขมันได้โดยไม่ผ่านการตรวจสอบ ทำให้เกิด bug ที่ยากตามหา — การบังคับให้แก้ไขผ่าน method เท่านั้น เปิดโอกาสให้ตรวจสอบความถูกต้อง (validation) ได้ทุกครั้งที่มีการเปลี่ยนแปลง
- **Abstraction มีไว้ลดภาระทางความคิดของผู้ใช้โค้ด**: เมื่อเรียกใช้ `list.sort()` (บทที่ 21) เราไม่ต้องรู้ว่าข้างในใช้ Timsort ทำงานอย่างไร แค่รู้ว่าเรียกแล้วได้ผลลัพธ์ที่เรียงแล้ว — หลักการเดียวกันนี้ประยุกต์ใช้กับ Class ที่เราออกแบบเอง ทำให้คนอื่นใช้โค้ดของเราได้โดยไม่ต้องเข้าใจรายละเอียดภายในทั้งหมด

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Class vs Object** เหมือน **พิมพ์เขียวบ้าน vs บ้านจริงที่สร้างขึ้น** — พิมพ์เขียวบ้านหนึ่งแบบสร้างบ้านจริงได้หลายหลัง แต่ละหลังมีที่อยู่ สีทาบ้าน เฟอร์นิเจอร์ต่างกัน (ข้อมูลของแต่ละ object ต่างกัน) แต่ทุกหลังมีโครงสร้างพื้นฐานเดียวกันตามพิมพ์เขียว (methods เดียวกันตาม Class)
- **Encapsulation** เหมือน **เครื่อง ATM ที่ห่อหุ้มกลไกภายในธนาคารไว้** — ผู้ใช้กดปุ่ม "ถอนเงิน" ได้ แต่เข้าถึงฐานข้อมูลบัญชีโดยตรงไม่ได้เลย ต้องผ่านกระบวนการตรวจสอบ (ใส่ PIN, เช็คยอดคงเหลือ) ที่เครื่อง ATM กำหนดไว้เท่านั้น ป้องกันไม่ให้ใครก็ได้มาแก้ยอดเงินตรงๆ
- **Abstraction** เหมือน **การขับรถยนต์โดยไม่ต้องเข้าใจกลไกเครื่องยนต์สันดาปภายใน** — คนขับรู้แค่ว่ากดคันเร่งแล้วรถวิ่งเร็วขึ้น เหยียบเบรกแล้วรถหยุด ไม่ต้องรู้ว่าระบบหัวฉีดน้ำมันหรือวาล์วทำงานอย่างไรข้างใน

## แผนภาพอธิบาย (Visual explanation)

### Class เป็นพิมพ์เขียว สร้าง Object หลายตัวจาก Class เดียว

```
class BankAccount (พิมพ์เขียว):
    attributes: owner, balance
    methods: deposit(), withdraw(), get_balance()

              |
              | สร้าง object จาก Class เดียวกัน
              v

account1 = BankAccount("Alice", 1000)      account2 = BankAccount("Bob", 500)

  owner: "Alice"                             owner: "Bob"
  balance: 1000                              balance: 500
  (methods เดียวกันจาก Class)                 (methods เดียวกันจาก Class)

account1 กับ account2 เป็นคนละ object กัน มีข้อมูล (state) แยกจากกันโดยสิ้นเชิง
แต่ใช้พฤติกรรม (methods) ชุดเดียวกันตามที่ Class กำหนดไว้
```

### Encapsulation — เข้าถึง balance ได้ผ่าน method เท่านั้น ไม่ใช่ตรงๆ

```
ไม่มี Encapsulation (อันตราย!):              มี Encapsulation (ปลอดภัย):

account.balance = -500   ✗                  account.withdraw(99999)   ✗
(แก้ค่าตรงๆ ได้ทันที                          (method ตรวจสอบก่อนเสมอ:
 ไม่มีการตรวจสอบใดๆ                            ถ้าถอนเกินยอดคงเหลือ
 ยอดติดลบได้อย่างไม่สมเหตุสมผล)                  -> ปฏิเสธ แจ้ง error กลับ)

                                             account.balance          ✗ (private, เข้าถึงตรงๆ ไม่ได้)
                                             account.get_balance()    ✓ (ต้องผ่าน method ที่กำหนดไว้)
```

### Abstraction — ผู้ใช้เห็นแค่ interface เรียบง่าย ไม่เห็นความซับซ้อนภายใน

```
สิ่งที่ผู้ใช้เห็น (Public Interface):          สิ่งที่ซ่อนอยู่ข้างใน (Implementation Details):

account.withdraw(200)                        def withdraw(self, amount):
                                                  if amount <= 0:
    "แค่เรียกแล้วได้ผล"                              raise ValueError(...)
                                                  if amount > self._balance:
                                                      raise ValueError("ยอดไม่พอ")
                                                  self._balance -= amount
                                                  self._log_transaction("withdraw", amount)
                                                  self._notify_if_low_balance()

ผู้ใช้ไม่ต้องรู้เรื่อง validation, logging, notification ที่ซ่อนอยู่ข้างในเลย
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== Class & Object พื้นฐาน =====
class BankAccount:
    def __init__(self, owner, initial_balance=0):    # Constructor - เรียกอัตโนมัติตอนสร้าง object
        self.owner = owner                              # attribute (public)
        self._balance = initial_balance                  # attribute (convention: _ นำหน้า = "private" ตามธรรมเนียม)
        self._transaction_log = []                        # ข้อมูลภายในที่ผู้ใช้ไม่ควรยุ่งตรงๆ

    def deposit(self, amount):                          # method (พฤติกรรม)
        if amount <= 0:
            raise ValueError("จำนวนฝากต้องมากกว่า 0")
        self._balance += amount
        self._transaction_log.append(f"deposit: +{amount}")

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("จำนวนถอนต้องมากกว่า 0")
        if amount > self._balance:                        # Encapsulation: ตรวจสอบก่อนแก้ไขข้อมูลเสมอ
            raise ValueError("ยอดเงินไม่เพียงพอ")
        self._balance -= amount
        self._transaction_log.append(f"withdraw: -{amount}")

    def get_balance(self):                               # ทางเดียวที่ผู้ใช้ภายนอกเข้าถึง balance ได้
        return self._balance

# ===== การใช้งาน =====
account1 = BankAccount("Alice", 1000)     # สร้าง object ตัวที่ 1
account2 = BankAccount("Bob", 500)        # สร้าง object ตัวที่ 2 (แยกข้อมูลกันโดยสิ้นเชิง)

account1.deposit(200)
account1.withdraw(150)
print(account1.get_balance())              # 1050
print(account2.get_balance())              # 500 (ไม่ถูกกระทบจาก account1 เลย)

try:
    account1.withdraw(999999)               # เกินยอดคงเหลือ -> Encapsulation ป้องกันไว้
except ValueError as e:
    print(e)                                 # "ยอดเงินไม่เพียงพอ"

# ===== Abstraction: Interface เรียบง่าย ซ่อนความซับซ้อนของ get_interest_rate ไว้ข้างใน =====
class SavingsAccount(BankAccount):
    def __init__(self, owner, initial_balance=0, years_as_member=0):
        super().__init__(owner, initial_balance)
        self.years_as_member = years_as_member

    def _calculate_interest_rate(self):                  # รายละเอียดภายในที่ซ่อนไว้ (ขึ้นต้นด้วย _)
        if self.years_as_member >= 5:
            return 0.03
        elif self.years_as_member >= 1:
            return 0.015
        return 0.005

    def apply_annual_interest(self):                      # Public method ง่ายๆ ที่ผู้ใช้เรียกได้
        rate = self._calculate_interest_rate()               # ผู้ใช้ไม่ต้องรู้สูตรคำนวณข้างใน
        interest = self._balance * rate
        self.deposit(interest)
        return interest

savings = SavingsAccount("Charlie", 10000, years_as_member=6)
print(savings.apply_annual_interest())      # 300.0 -> ผู้ใช้แค่เรียก 1 บรรทัด ไม่ต้องรู้สูตรดอกเบี้ยเลย
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `account1 = BankAccount("Alice", 1000)` — เรียก `__init__` (Constructor) โดยอัตโนมัติทันทีที่สร้าง object ใหม่ `self` คือตัวแปรที่อ้างอิงถึง object ตัวเองที่กำลังถูกสร้าง (ทุก method ต้องมี `self` เป็นพารามิเตอร์ตัวแรกเสมอในภาษา Python) `self.owner = owner` และ `self._balance = initial_balance` คือการเก็บข้อมูลที่ส่งเข้ามาไว้ใน object นี้โดยเฉพาะ
2. `self._balance` (ขึ้นต้นด้วย underscore) — เป็น**ธรรมเนียม (convention)** ในภาษา Python ที่บอกว่า "นี่คือข้อมูลภายในที่ไม่ควรเข้าถึงจากภายนอกโดยตรง" (ไม่ใช่การบังคับจริงทางเทคนิคเหมือนภาษา Java/C++ ที่มีคีย์เวิร์ด `private` แท้จริง) — โปรแกรมเมอร์ Python ที่ดีจะเคารพธรรมเนียมนี้และเข้าถึงผ่าน `get_balance()` เสมอ แทนที่จะเขียน `account1._balance` ตรงๆ
3. `account1.withdraw(999999)` — ก่อนแก้ไข `self._balance` เสมอ method ตรวจสอบเงื่อนไข `if amount > self._balance` ก่อน ถ้าไม่ผ่านจะ `raise ValueError(...)` ทันทีโดยไม่แตะต้องข้อมูลเลย — นี่คือ Encapsulation ที่ทำงานจริง: **ทุกการเปลี่ยนแปลงข้อมูลต้องผ่านการตรวจสอบก่อนเสมอ** ไม่มีทางแก้ `_balance` ให้ติดลบได้จากภายนอก
4. `savings.apply_annual_interest()` — ผู้ใช้เรียกแค่ 1 บรรทัด แต่ข้างในมีการเรียก `self._calculate_interest_rate()` (ซึ่งมีเงื่อนไข if/elif/else ซับซ้อนตามจำนวนปีเป็นสมาชิก) และ `self.deposit(interest)` (ซึ่งมีการตรวจสอบและบันทึก log) ทำงานต่อกันอัตโนมัติ — ผู้ใช้ **ไม่จำเป็นต้องรู้รายละเอียดเหล่านี้เลย** นี่คือ Abstraction: interface เรียบง่าย (`apply_annual_interest()`) ซ่อนความซับซ้อนทั้งหมดไว้ข้างใน

## Time Complexity

- การสร้าง Object (`__init__`): **O(1)** โดยทั่วไป (เว้นแต่ constructor มี loop หรือการคำนวณที่ขึ้นกับขนาดข้อมูล)
- การเรียก method (`deposit`, `withdraw`, `get_balance`): **O(1)** สำหรับตัวอย่างในบทนี้ เพราะแค่เข้าถึงและแก้ไขค่าตัวแปรตรงๆ ไม่มี loop หรือ recursion — Time Complexity ของ method จริงๆ แล้วขึ้นอยู่กับสิ่งที่ method นั้นทำภายใน (เชื่อมโยงกับหลักการวิเคราะห์ Big O จากบทที่ 20)

## Space Complexity

- แต่ละ Object ใช้พื้นที่ **O(k)** โดย k คือจำนวน attribute ที่ Class นั้นมี (คงที่ ไม่ขึ้นกับขนาดข้อมูลภายนอก) — ถ้าสร้าง n objects จาก Class เดียวกัน ใช้พื้นที่รวม **O(n × k)**

## แนวทางปฏิบัติที่ดี (Best Practices)

- ตั้งชื่อ attribute ที่เป็นข้อมูลภายใน (ไม่ควรเข้าถึงจากภายนอกโดยตรง) ด้วย underscore นำหน้าเสมอ (`self._balance`) เพื่อสื่อสารเจตนาให้คนอื่นที่อ่านโค้ดเข้าใจตรงกัน แม้ Python จะไม่บังคับจริงทางเทคนิคก็ตาม
- ออกแบบ method สาธารณะ (public interface) ให้**เรียบง่ายและมีความหมายชัดเจน** (`deposit()`, `withdraw()`) แทนที่จะเปิดให้แก้ไข attribute ตรงๆ เพราะเปิดโอกาสให้ตรวจสอบความถูกต้องได้ทุกครั้ง
- ทุก method ที่แก้ไขข้อมูลภายในควร**ตรวจสอบเงื่อนไขก่อนแก้ไขเสมอ** (validation) และแจ้ง error ที่ชัดเจนถ้าเงื่อนไขไม่ผ่าน แทนที่จะปล่อยให้ข้อมูลเสียหายแบบเงียบๆ

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- เข้าถึงและแก้ไข attribute ที่ควรเป็น private โดยตรงจากภายนอก (`account._balance = -999`) ทำลายหลักการ Encapsulation ทั้งหมด แม้ Python จะไม่ห้ามทางเทคนิคก็ตาม
- ลืมตรวจสอบเงื่อนไข (validation) ใน method ที่แก้ไขข้อมูล ทำให้ object เข้าสู่สถานะที่ไม่สมเหตุสมผลได้ (เช่น `balance` ติดลบ)
- สร้าง Class ที่มี method สาธารณะมากเกินไปโดยไม่จำเป็น (เปิดเผยรายละเอียดภายในมากเกินไป) ทำให้เสียประโยชน์ของ Abstraction และทำให้แก้ไข implementation ภายในทีหลังยากขึ้น เพราะโค้ดภายนอกอาจพึ่งพา method เหล่านั้นไปแล้ว
- สับสนระหว่าง Class กับ Object — พูดว่า "สร้าง Class" ทั้งที่จริงๆ หมายถึง "สร้าง Object จาก Class" (Class นิยามไว้ครั้งเดียว แต่สร้าง Object จากมันได้หลายตัว)

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง Class และ Object พร้อมยกตัวอย่างจากชีวิตจริง
2. Encapsulation คืออะไร ทำไมถึงสำคัญในการออกแบบซอฟต์แวร์ที่มีขนาดใหญ่
3. Abstraction ต่างจาก Encapsulation อย่างไร ทั้งสองแนวคิดนี้เกี่ยวข้องกันหรือไม่
4. ทำไม Python ถึงใช้แค่ "ธรรมเนียม" (underscore) แทนคีย์เวิร์ด `private` จริงๆ เหมือนภาษาอื่น (Java, C++)? ข้อดี-ข้อเสียของแนวทางนี้คืออะไร
5. ยกตัวอย่างสถานการณ์จริงที่การไม่มี Encapsulation ทำให้เกิดบั๊กที่ตามหาได้ยาก

## แบบฝึกหัด (Practice Exercises)

1. เขียน Class `Student` ที่มี attribute `name`, `_grades` (list ของคะแนน เป็น private) พร้อม method `add_grade(score)` (ตรวจสอบว่าคะแนนอยู่ระหว่าง 0-100), `get_average()`, และ `get_letter_grade()` (แปลงคะแนนเฉลี่ยเป็นเกรด A/B/C/D/F)
2. เขียน Class `Rectangle` ที่มี attribute `_width`, `_height` (private) พร้อม method `get_area()`, `get_perimeter()`, และ `resize(factor)` ที่ตรวจสอบว่า `factor` ต้องมากกว่า 0 เท่านั้น
3. ปรับ `BankAccount` ในตัวอย่างให้เพิ่ม method `get_transaction_history()` ที่คืนค่า **สำเนา** (copy) ของ `_transaction_log` แทนที่จะคืน reference ตรงๆ พร้อมอธิบายว่าทำไมการคืน reference ตรงๆ ถึงทำลายหลักการ Encapsulation

## Mini Project

**"ระบบจัดการบัญชีธนาคารจำลอง (Mini Banking System)"**: เขียนโปรแกรมที่:
1. Implement `BankAccount` และ `SavingsAccount` (ตามตัวอย่างในบทนี้) พร้อมเพิ่ม `CheckingAccount` ที่มีวงเงินเบิกเกินบัญชี (overdraft limit) ได้
2. สร้างระบบจัดการหลายบัญชีพร้อมกัน (list ของ objects) รองรับคำสั่ง: ฝากเงิน, ถอนเงิน, โอนเงินระหว่างบัญชี (method ที่เรียก `withdraw` จากบัญชีหนึ่งและ `deposit` เข้าอีกบัญชี)
3. ทุก method ที่แก้ไขยอดเงินต้องมีการตรวจสอบ (validation) และบันทึก transaction log ที่เข้าถึงได้ผ่าน method สาธารณะเท่านั้น (ห้ามเข้าถึง `_balance` หรือ `_transaction_log` ตรงๆ จากภายนอก Class เด็ดขาด)
4. เขียนเทสต์ทดลองว่าการพยายามเข้าถึง private attribute ตรงๆ ยังคงทำได้ทางเทคนิค (เพราะ Python ไม่บังคับจริง) แต่ทำให้โค้ดเสี่ยงต่อข้อผิดพลาดอย่างไร เปรียบเทียบกับการใช้ method ที่ถูกต้อง

## ข้อคิดสำคัญ (Key Takeaways)

- Class คือพิมพ์เขียวที่นิยามข้อมูล (attributes) และพฤติกรรม (methods) ส่วน Object คือสิ่งที่ถูกสร้างขึ้นจริงจาก Class นั้น แต่ละ Object มีข้อมูลเป็นของตัวเอง
- Encapsulation ห่อหุ้มข้อมูลไว้ภายใน บังคับให้เข้าถึง/แก้ไขผ่าน method ที่กำหนดเท่านั้น เปิดโอกาสให้ตรวจสอบความถูกต้องได้ทุกครั้งที่มีการเปลี่ยนแปลง
- Abstraction ซ่อนความซับซ้อนของการทำงานภายในไว้เบื้องหลัง interface ที่เรียบง่าย ทำให้ผู้ใช้โค้ดไม่ต้องเข้าใจรายละเอียดทั้งหมด
- Python ใช้ธรรมเนียม (underscore นำหน้า) แทนการบังคับจริงทางเทคนิคสำหรับ private attribute ต่างจากภาษาอื่นที่มีคีย์เวิร์ดบังคับจริง
- OOP ทำให้โค้ดที่ซับซ้อนจัดการง่ายขึ้นด้วยการรวมข้อมูลกับพฤติกรรมที่เกี่ยวข้องไว้ด้วยกัน แทนที่จะแยกกันเหมือนใน procedural programming ที่ใช้มาตลอด Part 1-5
- บทถัดไปจะเรียน Inheritance และ Polymorphism ซึ่งขยายจากตัวอย่าง `SavingsAccount` ที่สืบทอดจาก `BankAccount` ในบทนี้ ไปสู่หลักการที่ทรงพลังกว่าในการนำโค้ดกลับมาใช้ซ้ำและออกแบบระบบที่ยืดหยุ่น

---
*บทต่อไป (บทที่ 31): Inheritance & Polymorphism — พิมพ์ "Next" เพื่อดำเนินการต่อ*
