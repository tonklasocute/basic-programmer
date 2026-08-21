# บทที่ 31: Inheritance & Polymorphism

## แนวคิดหลัก (Concept)

บทที่แล้วสร้าง `SavingsAccount` โดยใช้ `class SavingsAccount(BankAccount):` และ `super().__init__(...)` โดยยังไม่ได้อธิบายหลักการเบื้องหลังอย่างเป็นทางการ บทนี้เจาะลึกสองเสาหลักที่เหลือของ OOP (ต่อจาก Encapsulation และ Abstraction ในบทที่แล้ว): **Inheritance** (การสืบทอด) ที่ทำให้นำโค้ดกลับมาใช้ซ้ำได้โดยไม่ต้องเขียนใหม่ทั้งหมด และ **Polymorphism** (การมีหลายรูปแบบ) ที่ทำให้ object ต่างชนิดกันตอบสนองต่อคำสั่งเดียวกันได้ต่างกันตามธรรมชาติของตัวเอง

- **Inheritance (การสืบทอด)** — กลไกที่ Class หนึ่ง (**Child/Subclass**) รับเอา attributes และ methods ทั้งหมดจาก Class อื่น (**Parent/Superclass**) มาใช้โดยอัตโนมัติ แล้วสามารถเพิ่มเติมหรือปรับแก้เฉพาะส่วนที่ต่างออกไปได้
- **Method Overriding (การเขียนทับ method)** — การที่ Child Class นิยาม method ชื่อเดียวกับที่มีอยู่ใน Parent Class ใหม่ เพื่อเปลี่ยนพฤติกรรมให้เหมาะกับ Child Class นั้นโดยเฉพาะ
- **Polymorphism (การมีหลายรูปแบบ)** — ความสามารถที่ object ต่างชนิดกัน (แต่สืบทอดมาจาก Parent เดียวกัน) ตอบสนองต่อการเรียก method ชื่อเดียวกันด้วยพฤติกรรมที่ต่างกันไปตามชนิดของตัวเอง โดยโค้ดที่เรียกใช้ไม่จำเป็นต้องรู้ว่ากำลังเรียก object ชนิดไหนอยู่

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Inheritance มีไว้กำจัดการเขียนโค้ดซ้ำ (DRY — Don't Repeat Yourself)**: ถ้า `SavingsAccount` และ `CheckingAccount` (จาก Mini Project บทที่แล้ว) ต่างต้องมี `deposit()`, `withdraw()`, `get_balance()` เหมือนกันทุกตัวอักษร การเขียนแยกกันทำให้เมื่อแก้บั๊กในฟังก์ชันหนึ่งต้องไปแก้ซ้ำในทุก Class ที่ก็อปปี้โค้ดไว้ — Inheritance ทำให้เขียน logic ร่วมไว้ที่เดียวใน Parent Class แล้ว Child Class ทุกตัวได้ประโยชน์ทันที
- **Polymorphism มีไว้เขียนโค้ดที่ทำงานกับ object หลายชนิดได้โดยไม่ต้องรู้ชนิดที่แน่นอนล่วงหน้า**: ลองนึกภาพระบบที่ต้องคำนวณดอกเบี้ยของบัญชีนับพันบัญชีที่มีทั้ง `SavingsAccount` และ `CheckingAccount` ปนกัน — ถ้าไม่มี Polymorphism ต้องเขียน `if/elif` ตรวจสอบชนิดทุกครั้งก่อนเรียกฟังก์ชันที่ถูกต้อง (เชื่อมโยงกับ Conditionals บทที่ 10) แต่ Polymorphism ทำให้แค่เรียก `account.apply_interest()` เหมือนกันหมด แล้วแต่ละ object จัดการ logic ของตัวเองถูกต้องอัตโนมัติ
- **ทั้งสองแนวคิดมีไว้สร้างระบบที่ขยายได้ง่าย (extensible)**: เมื่อต้องเพิ่มบัญชีชนิดใหม่ในอนาคต (เช่น `FixedDepositAccount`) แค่สร้าง Child Class ใหม่ที่สืบทอดจาก Parent เดิม โดยไม่ต้องแก้โค้ดเก่าที่มีอยู่แล้วเลย — หลักการนี้จะกลับมาเป็นหัวใจของ Open/Closed Principle ใน SOLID Principles (บทถัดๆ ไป)

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Inheritance** เหมือน **ลูกที่สืบทอด DNA และนามสกุลจากพ่อแม่** — ลูกได้รับลักษณะพื้นฐานมาโดยอัตโนมัติ (สีตา, กรุ๊ปเลือด) แต่ก็มีลักษณะเฉพาะตัวเพิ่มเติมหรือแตกต่างไปจากพ่อแม่ได้ (นิสัย, ความสามารถพิเศษ) โดยไม่ต้อง "สร้างใหม่ตั้งแต่ศูนย์"
- **Method Overriding** เหมือน **สูตรอาหารประจำครอบครัวที่แต่ละรุ่นปรับปรุงเฉพาะบางขั้นตอน** — สูตรพื้นฐาน (ส่วนผสมหลัก) เหมือนกันทุกรุ่น แต่ลูกอาจปรับสูตรเผ็ดน้อยลงหรือใส่วัตถุดิบเสริมต่างจากพ่อแม่ในบางขั้นตอนเฉพาะ
- **Polymorphism** เหมือน **ปุ่ม "เล่น" บนรีโมททีวีที่ใช้ได้กับทุกอุปกรณ์ (ทีวี, เครื่องเล่น DVD, ลำโพงบลูทูธ)** — กดปุ่มเดียวกัน แต่ละอุปกรณ์ตอบสนองต่างกันตามธรรมชาติของตัวเอง (ทีวีเล่นภาพ, ลำโพงเล่นเสียง) โดยคนกดไม่ต้องรู้รายละเอียดว่าอุปกรณ์แต่ละชนิดทำงานอย่างไรข้างใน

## แผนภาพอธิบาย (Visual explanation)

### Inheritance Hierarchy — Animal เป็น Parent ของ Dog, Cat, Bird

```
                    Animal (Parent/Superclass)
                    - name
                    - make_sound()  <- นิยามไว้ทั่วไป
                       /      |      \
                      /       |       \
              Dog          Cat        Bird     (Child/Subclass ทั้งสาม)
        - make_sound()  - make_sound() - make_sound()   <- แต่ละตัว override เอง
          (เห่า)          (ร้องเหมียว)    (ร้องเจี๊ยบ)

Dog, Cat, Bird ทั้งหมดสืบทอด attribute "name" จาก Animal โดยอัตโนมัติ
แต่ต่าง override "make_sound()" ให้เป็นเสียงของตัวเอง
```

### Polymorphism — เรียก method เดียวกัน ได้พฤติกรรมต่างกันตามชนิดจริง

```
animals = [Dog("Rex"), Cat("Whiskers"), Bird("Tweety")]

for animal in animals:
    animal.make_sound()     <- เรียกเหมือนกันทุกตัว ไม่ต้องเช็คชนิดก่อนเลย!

ผลลัพธ์:
  Rex says: Woof!
  Whiskers says: Meow!
  Tweety says: Tweet!

โค้ดที่ loop ไม่จำเป็นต้องรู้เลยว่าแต่ละ object เป็น Dog, Cat หรือ Bird
แค่รู้ว่าทุกตัวมี make_sound() เพราะสืบทอดมาจาก Animal เดียวกัน
```

### super() — เรียก method ของ Parent จาก Child โดยไม่ต้องเขียนซ้ำ

```
class Animal:                          class Dog(Animal):
    def __init__(self, name):              def __init__(self, name, breed):
        self.name = name                       super().__init__(name)    <- เรียก Animal.__init__ ให้ทำงานก่อน
                                                self.breed = breed          <- แล้วค่อยเพิ่มส่วนที่ Dog มีเพิ่ม

Dog("Rex", "Golden Retriever")
        |
        v
super().__init__("Rex")  ทำงานก่อน -> self.name = "Rex"  (ได้จาก Animal ฟรี ไม่ต้องเขียนซ้ำ)
self.breed = "Golden Retriever"          (ส่วนเพิ่มเติมเฉพาะของ Dog)
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== Parent Class =====
class Animal:
    def __init__(self, name):
        self.name = name

    def make_sound(self):                          # method พื้นฐานที่ Child ทุกตัวจะ override
        return f"{self.name} makes a sound"

    def describe(self):                              # method ที่ไม่ต้อง override เลย ใช้ร่วมกันได้ทุก Child
        return f"{self.name} is an animal"

# ===== Child Classes ที่สืบทอดจาก Animal =====
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)                        # เรียก Animal.__init__ ก่อนเสมอ
        self.breed = breed                              # attribute เพิ่มเติมเฉพาะของ Dog

    def make_sound(self):                              # Method Overriding
        return f"{self.name} says: Woof!"

class Cat(Animal):
    def make_sound(self):                              # Cat ไม่มี __init__ ของตัวเอง -> ใช้ของ Animal ทันที
        return f"{self.name} says: Meow!"

class Bird(Animal):
    def make_sound(self):
        return f"{self.name} says: Tweet!"

# ===== Polymorphism ในทางปฏิบัติ =====
animals = [Dog("Rex", "Golden Retriever"), Cat("Whiskers"), Bird("Tweety")]

for animal in animals:
    print(animal.make_sound())          # เรียกเหมือนกันหมด แต่ได้ผลลัพธ์ต่างกันตามชนิดจริง
    print(animal.describe())              # method นี้ไม่ได้ override เลย ใช้ของ Animal ตรงๆ ทุกตัว

# ผลลัพธ์:
# Rex says: Woof!
# Rex is an animal
# Whiskers says: Meow!
# Whiskers is an animal
# Tweety says: Tweet!
# Tweety is an animal

print(isinstance(animals[0], Animal))     # True -> Dog เป็น Animal ด้วย (ผ่านการสืบทอด)
print(isinstance(animals[0], Dog))        # True
print(isinstance(animals[1], Dog))        # False -> Cat ไม่ใช่ Dog
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `class Dog(Animal):` — วงเล็บหลังชื่อ Class บอกว่า `Dog` สืบทอดจาก `Animal` ทำให้ `Dog` ได้รับ `describe()` มาใช้ได้ทันทีโดยไม่ต้องเขียนใหม่ และมี `self.name` ใช้ได้ (ถ้าเรียก `super().__init__()` ให้ตั้งค่าให้)
2. `super().__init__(name)` ใน `Dog.__init__` — เรียก `__init__` ของ `Animal` (Parent) ให้ทำงานก่อน เพื่อตั้งค่า `self.name` ตามที่ `Animal` กำหนดไว้ **โดยไม่ต้องเขียนบรรทัด `self.name = name` ซ้ำเอง** จากนั้นค่อยเพิ่ม `self.breed = breed` ซึ่งเป็น attribute ที่มีแค่ใน `Dog` เท่านั้น
3. `class Cat(Animal): def make_sound(self): ...` — สังเกตว่า `Cat` **ไม่ได้เขียน `__init__` ของตัวเองเลย** เมื่อสร้าง `Cat("Whiskers")` Python จะใช้ `__init__` ของ `Animal` (Parent) โดยอัตโนมัติเพราะ `Cat` ไม่ได้ override มันไว้ — นี่คือพฤติกรรมเริ่มต้นของ Inheritance: ถ้า Child ไม่นิยาม method ไหนเอง จะใช้ของ Parent โดยอัตโนมัติเสมอ
4. `for animal in animals: animal.make_sound()` — แม้ตัวแปร `animal` แต่ละรอบจะเป็นคนละ Class กัน (`Dog`, `Cat`, `Bird`) แต่โค้ดเรียก `animal.make_sound()` **เหมือนกันทุกรอบโดยไม่ต้องเช็คชนิด** เพราะ Python จะเลือก method ของ Class จริงของ object นั้นเสมอ (`Dog.make_sound`, `Cat.make_sound`, `Bird.make_sound` ตามลำดับ) — นี่คือ Polymorphism ที่ทำงานจริงในทางปฏิบัติ (เรียกว่า **Runtime Polymorphism** หรือ **Dynamic Dispatch**)
5. `isinstance(animals[0], Animal)` คืนค่า `True` — เพราะ `Dog` สืบทอดจาก `Animal` ทำให้ **object ของ Child Class ถือว่าเป็น object ของ Parent Class ด้วยเสมอ** (ความสัมพันธ์ "is-a": Dog is an Animal) แต่ `isinstance(animals[1], Dog)` คืนค่า `False` เพราะความสัมพันธ์นี้เป็นทางเดียว (Cat ไม่ใช่ Dog แม้ทั้งคู่จะสืบทอดจาก Animal เหมือนกัน)

## Time Complexity

- การเรียก method ที่ถูก override (Dynamic Dispatch): **O(1)** ในภาษาส่วนใหญ่ (Python ค้นหา method ผ่าน Method Resolution Order ซึ่งมีต้นทุนคงที่ในทางปฏิบัติ ไม่ขึ้นกับความลึกของ Inheritance chain อย่างมีนัยสำคัญสำหรับกรณีทั่วไป)
- `isinstance()` check: **O(d)** โดย d คือความลึกของ Inheritance hierarchy (จำนวนชั้น Parent ที่ต้องไล่ตรวจ) — ในทางปฏิบัติ d มีค่าน้อยมากเสมอ (ไม่กี่ชั้น) จึงถือเป็นเกือบคงที่

## Space Complexity

- แต่ละ Object ของ Child Class ใช้พื้นที่ **O(k_parent + k_child)** โดย k_parent คือจำนวน attribute ที่ได้จาก Parent และ k_child คือจำนวน attribute ที่ Child เพิ่มเติมเอง (เช่น `Dog` object มีทั้ง `name` จาก `Animal` และ `breed` ของตัวเอง)

## แนวทางปฏิบัติที่ดี (Best Practices)

- ใช้ Inheritance เมื่อความสัมพันธ์ระหว่าง Class เป็นแบบ **"is-a"** จริงๆ (Dog **is an** Animal) ไม่ใช่แค่ "มีบางอย่างเหมือนกัน" — ถ้าความสัมพันธ์เป็นแบบ "has-a" (เช่น รถ **has an** engine) ควรใช้ **Composition** (เก็บ object อื่นเป็น attribute) แทน ไม่ใช่ Inheritance
- เรียก `super().__init__(...)` ใน Child Class เสมอเมื่อต้องการให้ Parent ตั้งค่าเริ่มต้นให้ แทนที่จะเขียนโค้ดตั้งค่าเดิมซ้ำเอง เพื่อรักษาหลักการ DRY
- ออกแบบ Parent Class ให้มี method ที่ **Child ทุกตัวคาดว่าจะต้อง override** (เช่น `make_sound()`) ให้ชัดเจน เพื่อให้ Polymorphism ทำงานได้อย่างสมเหตุสมผล (จะเรียนรายละเอียดเรื่อง Abstract Method ใน SOLID Principles บทถัดๆ ไป)

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ใช้ Inheritance ทั้งที่ความสัมพันธ์จริงๆ เป็นแบบ "has-a" ไม่ใช่ "is-a" (เช่น ทำให้ `Car` สืบทอดจาก `Engine` ทั้งที่ควรจะเป็น `Car` **มี** `Engine` เป็น attribute แทน) ทำให้โครงสร้างโค้ดสับสนและขยายยากในระยะยาว
- ลืมเรียก `super().__init__()` ใน Child Class ที่มี `__init__` ของตัวเอง ทำให้ attribute ที่ควรได้จาก Parent ไม่ถูกตั้งค่าไว้เลย เกิด `AttributeError` เมื่อพยายามเข้าถึงทีหลัง
- สร้าง Inheritance hierarchy ที่ลึกเกินความจำเป็น (Parent -> Child -> Grandchild -> ...) ทำให้ยากต่อการติดตามว่า attribute/method หนึ่งมาจากชั้นไหนกันแน่
- เขียน `if isinstance(animal, Dog): ... elif isinstance(animal, Cat): ...` แทนที่จะปล่อยให้ Polymorphism ทำงานตามธรรมชาติ (`animal.make_sound()` ตรงๆ) — เป็นการเขียนโค้ดที่ขัดกับเจตนาของ Polymorphism และทำให้เพิ่ม Child Class ใหม่ในอนาคตต้องมาแก้ `if/elif` นี้ทุกที่ที่มันปรากฏ

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง Inheritance และ Composition พร้อมยกตัวอย่างว่าเมื่อไหร่ควรเลือกใช้แบบไหน
2. Method Overriding คืออะไร ต่างจาก Method Overloading (การมี method ชื่อเดียวกันแต่รับพารามิเตอร์ต่างกัน) อย่างไร
3. อธิบายว่า Polymorphism ช่วยลดการใช้ `if/elif` ตรวจสอบชนิด object ได้อย่างไร ยกตัวอย่างประกอบ
4. `super()` ทำหน้าที่อะไร ทำไมถึงควรเรียกใน `__init__` ของ Child Class เกือบทุกครั้ง
5. ทำไม `isinstance(dog_object, Animal)` ถึงคืนค่า True แต่ `isinstance(cat_object, Dog)` คืนค่า False? อธิบายด้วยหลักการ "is-a"

## แบบฝึกหัด (Practice Exercises)

1. เขียน Parent Class `Shape` ที่มี method `area()` และ `perimeter()` (คืนค่า `NotImplementedError` เป็นค่าเริ่มต้น) แล้วสร้าง Child Class `Circle`, `Rectangle`, `Triangle` ที่ override ทั้งสอง method ตามสูตรของแต่ละรูปทรง
2. เขียน list ของ `Shape` objects หลายชนิดปนกัน แล้วใช้ Polymorphism คำนวณ**ผลรวมพื้นที่ทั้งหมด**โดยไม่ต้องเช็คชนิดของแต่ละ object เลย
3. ปรับ `BankAccount`/`SavingsAccount`/`CheckingAccount` จาก Mini Project บทที่แล้ว ให้ทุก Child Class override method `apply_monthly_fee()` ที่คำนวณค่าธรรมเนียมรายเดือนต่างกันตามชนิดบัญชี แล้วเขียน loop ที่เรียก method นี้กับทุกบัญชีโดยไม่ต้องเช็คชนิด

## Mini Project

**"ระบบจำลองสวนสัตว์ (Zoo Management System)"**: เขียนโปรแกรมที่:
1. สร้าง Parent Class `Animal` ที่มี attribute พื้นฐาน (`name`, `age`) และ method `make_sound()`, `eat()` เป็นค่าเริ่มต้น
2. สร้าง Child Class อย่างน้อย 4 ชนิด (เช่น `Lion`, `Elephant`, `Penguin`, `Snake`) ที่ override `make_sound()` และ `eat()` ให้เหมาะกับสัตว์แต่ละชนิด (เช่น เนื้อสัตว์ vs พืช)
3. สร้างระบบ "ตารางให้อาหารประจำวัน" ที่ loop ผ่านสัตว์ทุกตัวในสวนสัตว์ (list ที่มีสัตว์หลายชนิดปนกัน) แล้วเรียก `feed_all()` โดยใช้ Polymorphism ล้วนๆ ไม่มี `if/elif` เช็คชนิดเลย
4. เพิ่มฟีเจอร์ตรวจสอบด้วย `isinstance()` ว่าสัตว์ตัวไหนเป็นสัตว์เลี้ยงลูกด้วยนม (สร้าง Class กลาง `Mammal` คั่นระหว่าง `Animal` และ `Lion`/`Elephant`) เพื่อสร้างรายงานแยกกลุ่มสัตว์เลี้ยงลูกด้วยนมกับไม่ใช่

## ข้อคิดสำคัญ (Key Takeaways)

- Inheritance ทำให้ Child Class ได้รับ attributes และ methods จาก Parent Class โดยอัตโนมัติ ลดการเขียนโค้ดซ้ำ (DRY) และทำให้ระบบขยายง่าย
- Method Overriding ให้ Child Class ปรับพฤติกรรมของ method ที่สืบทอดมาให้เหมาะกับตัวเองได้ โดยไม่กระทบ Parent หรือ Child อื่นๆ
- Polymorphism ทำให้เรียก method ชื่อเดียวกันกับ object ต่างชนิดได้ผลลัพธ์ต่างกันตามธรรมชาติของแต่ละชนิด โค้ดที่เรียกใช้ไม่ต้องรู้ชนิดที่แน่นอนล่วงหน้า ลดการเขียน `if/elif` ตรวจสอบชนิด
- `super()` ใช้เรียก method ของ Parent จาก Child โดยเฉพาะใน `__init__` เพื่อให้ Parent ตั้งค่าพื้นฐานให้โดยไม่ต้องเขียนซ้ำ
- ใช้ Inheritance เมื่อความสัมพันธ์เป็น "is-a" จริงๆ เท่านั้น ถ้าเป็น "has-a" ให้ใช้ Composition แทน
- บทถัดไปจะเรียน SOLID Principles ซึ่งเป็นชุดหลักการออกแบบ OOP ที่สร้างบนพื้นฐานทั้ง 4 เสาหลัก (Encapsulation, Abstraction, Inheritance, Polymorphism) ที่เรียนมาแล้วทั้งสองบทนี้ เพื่อทำให้ระบบยืดหยุ่นและดูแลรักษาง่ายยิ่งขึ้นไปอีก

---
*บทต่อไป (บทที่ 32): SOLID Principles — พิมพ์ "Next" เพื่อดำเนินการต่อ*
