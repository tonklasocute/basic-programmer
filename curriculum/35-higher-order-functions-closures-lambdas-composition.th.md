# บทที่ 35: Higher Order Functions, Closures, Lambdas, Composition

## แนวคิดหลัก (Concept)

บทที่แล้วสอนให้เขียน Pure Function แยกกันเป็นชิ้นเล็กๆ ที่ไม่มี Side Effect แต่ยังไม่ได้สอนว่าจะ**เอาฟังก์ชันเหล่านั้นมาประกอบกัน**เป็นโปรแกรมที่ใหญ่ขึ้นได้อย่างไร บทนี้เป็นบทปิดท้าย **Part 7: Functional Programming** โดยแนะนำเครื่องมือหลักที่ทำให้ทำแบบนั้นได้ — หัวใจสำคัญคือแนวคิดที่ว่า **ฟังก์ชันคือ "ค่า" ชนิดหนึ่งเหมือนตัวเลขหรือ string** ส่งต่อไปมา เก็บในตัวแปร หรือส่งเป็นพารามิเตอร์ให้ฟังก์ชันอื่นได้เหมือนข้อมูลทั่วไป

- **First-Class Function** — คุณสมบัติของภาษาที่ปฏิบัติต่อฟังก์ชันเหมือนเป็นค่าชนิดหนึ่ง สามารถเก็บในตัวแปร ส่งเป็นพารามิเตอร์ หรือคืนค่าออกจากฟังก์ชันอื่นได้ (Python มีคุณสมบัตินี้)
- **Higher Order Function (HOF)** — ฟังก์ชันที่**รับฟังก์ชันอื่นเป็นพารามิเตอร์** หรือ**คืนค่าเป็นฟังก์ชัน** (หรือทั้งสองอย่าง) เช่น `map()`, `filter()`, `sorted(key=...)` ที่คุ้นเคยมาตลอด
- **Lambda (ฟังก์ชันนิรนาม)** — วิธีเขียนฟังก์ชันสั้นๆ แบบไม่ต้องตั้งชื่อ ใช้เมื่อต้องการฟังก์ชันเล็กๆ แบบใช้ครั้งเดียวโดยไม่คุ้มที่จะเขียน `def` แยก
- **Closure (การปิดล้อม)** — ฟังก์ชันที่ "จำ" ตัวแปรจาก scope ภายนอกที่มันถูกสร้างขึ้นมาไว้ได้ แม้ scope นั้นจะจบการทำงานไปแล้วก็ตาม
- **Function Composition (การประกอบฟังก์ชัน)** — การนำฟังก์ชันหลายตัวมาเรียงต่อกัน โดยผลลัพธ์ของฟังก์ชันหนึ่งเป็น input ของฟังก์ชันถัดไป สร้างเป็น pipeline การประมวลผล

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **HOF มีไว้ลด boilerplate code ของ loop ที่ทำซ้ำบ่อยๆ**: แทนที่จะเขียน `for` loop (บทที่ 11) เพื่อแปลงข้อมูลทุกตัวใน list ทุกครั้ง `map()` ห่อหุ้ม pattern นี้ไว้ให้เรียกใช้ซ้ำได้ ทำให้โค้ดสั้นลงและสื่อเจตนา ("แปลงข้อมูลทุกตัวด้วยฟังก์ชันนี้") ชัดเจนกว่า loop ที่เขียนเอง
- **Closure มีไว้สร้างฟังก์ชันที่ "จำสถานะ" ได้โดยไม่ต้องใช้ Class**: บางครั้งต้องการฟังก์ชันที่เก็บค่าบางอย่างไว้ใช้ในการเรียกครั้งถัดไป (เช่น ตัวนับ, ค่า config ที่ตั้งไว้ล่วงหน้า) — Closure ทำสิ่งนี้ได้โดยไม่ต้องสร้าง Class ทั้ง object (ทางเลือกที่เบากว่า OOP จาก Part 6 สำหรับปัญหาบางประเภท)
- **Function Composition มีไว้สร้าง pipeline การประมวลผลข้อมูลที่อ่านง่ายและทดสอบทีละส่วนได้**: เมื่อต้องประมวลผลข้อมูลผ่านหลายขั้นตอน (กรอง -> แปลง -> เรียง) การประกอบ Pure Function (บทที่แล้ว) หลายตัวเข้าด้วยกันเป็น pipeline ทำให้แต่ละขั้นตอนแยกทดสอบได้อิสระ และอ่านโค้ดแล้วเข้าใจลำดับการทำงานได้ทันที

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Higher Order Function** เหมือน **เครื่องซักผ้าที่รับ "โปรแกรมซัก" เป็นอินพุต** — เครื่องเดียวกัน (`map()`) ทำงานต่างกันได้ตามโปรแกรมที่เลือกใส่เข้าไป (ฟังก์ชันที่ส่งเข้ามา) โดยไม่ต้องซื้อเครื่องใหม่ทุกครั้งที่อยากซักผ้าแบบต่างกัน
- **Closure** เหมือน **กล่องเก็บของส่วนตัวที่ล็อกด้วยกุญแจเฉพาะของแต่ละคน** — แม้ห้อง (scope) ที่สร้างกล่องจะถูกปิดไปแล้ว กล่องยังคงจำเนื้อหาข้างในไว้ได้และเปิดดูได้เมื่อไหร่ก็ตาม (ผ่านฟังก์ชันที่ถูกสร้างขึ้นมาพร้อมกัน)
- **Function Composition** เหมือน **สายพานการผลิตในโรงงาน** — วัตถุดิบผ่านสถานีที่ 1 (ตัดแต่ง) แล้วส่งต่อไปสถานีที่ 2 (ประกอบ) แล้วสถานีที่ 3 (บรรจุภัณฑ์) โดยแต่ละสถานีทำหน้าที่เดียวและรับผลลัพธ์จากสถานีก่อนหน้ามาทำงานต่อ

## แผนภาพอธิบาย (Visual explanation)

### First-Class Function — ฟังก์ชันเก็บในตัวแปร ส่งเป็นพารามิเตอร์ได้เหมือนค่าทั่วไป

```
def greet(name):
    return f"Hello, {name}!"

say_hello = greet              <- เก็บฟังก์ชันไว้ในตัวแปร (ไม่ใช่เรียกใช้ ไม่มีวงเล็บ)
print(say_hello("Alice"))       -> "Hello, Alice!"     (say_hello ทำงานเหมือน greet ทุกประการ)

def apply_function(func, value):    <- รับฟังก์ชันเป็นพารามิเตอร์ (นี่คือ HOF)
    return func(value)

print(apply_function(greet, "Bob"))  -> "Hello, Bob!"
```

### map(), filter() — HOF ที่ใช้บ่อยที่สุด

```
numbers = [1, 2, 3, 4, 5]

map(square, numbers):              filter(is_even, numbers):
  1 -> square(1)=1                    1 -> is_even(1)=False -> ตัดทิ้ง
  2 -> square(2)=4                    2 -> is_even(2)=True  -> เก็บไว้
  3 -> square(3)=9                    3 -> is_even(3)=False -> ตัดทิ้ง
  4 -> square(4)=16                   4 -> is_even(4)=True  -> เก็บไว้
  5 -> square(5)=25                   5 -> is_even(5)=False -> ตัดทิ้ง

ผลลัพธ์: [1,4,9,16,25]              ผลลัพธ์: [2,4]

map "แปลง" ทุกตัว (input กับ output จำนวนเท่ากันเสมอ)
filter "คัดกรอง" (output อาจน้อยกว่า input)
```

### Closure — ฟังก์ชันจำตัวแปรจาก scope ภายนอกไว้ได้

```
def make_counter():
    count = 0                          <- ตัวแปรใน scope ของ make_counter
    def increment():
        nonlocal count                    <- บอกว่าจะแก้ไข count จาก scope ภายนอก (ไม่ใช่สร้างใหม่)
        count += 1
        return count
    return increment                    <- คืนค่าเป็นฟังก์ชัน (ไม่ใช่เรียกใช้)

counter1 = make_counter()             <- สร้าง closure ตัวที่ 1 (มี count ของตัวเอง)
counter2 = make_counter()             <- สร้าง closure ตัวที่ 2 (มี count แยกต่างหาก)

counter1()  -> 1        counter2()  -> 1     <- แต่ละตัวจำ count ของตัวเองแยกกันโดยสิ้นเชิง
counter1()  -> 2        counter2()  -> 1
counter1()  -> 3        counter2()  -> 2

แม้ make_counter() จะทำงานจบไปแล้ว (return แล้ว) แต่ increment ยังคงจำ "count" ของ scope นั้นไว้ได้
```

### Function Composition — ต่อฟังก์ชันเป็น pipeline

```
data = [-2, 3, -5, 8, -1, 4]

pipeline: data -> remove_negatives -> double_values -> sum_all

remove_negatives([-2,3,-5,8,-1,4]) -> [3,8,4]
double_values([3,8,4])              -> [6,16,8]
sum_all([6,16,8])                    -> 30

แต่ละขั้นตอนเป็น Pure Function แยกกัน (บทที่แล้ว) ทดสอบทีละตัวได้อิสระ
รวมกันเป็น pipeline เดียวที่อ่านลำดับการทำงานได้ชัดเจนจากซ้ายไปขวา
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== Higher Order Function: map, filter, sorted(key=...) =====
numbers = [1, 2, 3, 4, 5]

squared = list(map(lambda x: x ** 2, numbers))            # Lambda ใช้ตรงๆ ไม่ต้องตั้งชื่อฟังก์ชัน
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(squared)    # [1, 4, 9, 16, 25]
print(evens)        # [2, 4]

students = [("Alice", 85), ("Bob", 72), ("Charlie", 90)]
sorted_by_score = sorted(students, key=lambda s: s[1], reverse=True)   # key รับฟังก์ชันเป็นพารามิเตอร์
print(sorted_by_score)   # [('Charlie', 90), ('Alice', 85), ('Bob', 72)]

# ===== HOF ที่คืนค่าเป็นฟังก์ชัน =====
def make_multiplier(factor):                 # HOF ที่คืนค่าเป็นฟังก์ชันใหม่
    def multiply(x):
        return x * factor                      # "จำ" factor ไว้ได้ -> นี่คือ Closure
    return multiply

double = make_multiplier(2)                  # double คือ closure ที่จำ factor=2 ไว้
triple = make_multiplier(3)                    # triple คือ closure ที่จำ factor=3 ไว้ (แยกจาก double)
print(double(5))    # 10
print(triple(5))     # 15

# ===== Closure: ตัวนับที่จำ state ของตัวเองได้โดยไม่ต้องใช้ Class =====
def make_counter():
    count = 0
    def increment():
        nonlocal count                          # บอกว่าจะแก้ไข count จาก enclosing scope
        count += 1
        return count
    return increment

counter1 = make_counter()
counter2 = make_counter()                       # counter1 กับ counter2 มี count แยกกันคนละชุด
print(counter1())    # 1
print(counter1())    # 2
print(counter2())    # 1  (ไม่ถูกกระทบจาก counter1 เลย)

# ===== Function Composition: ต่อ Pure Function เป็น pipeline =====
def remove_negatives(nums):
    return [n for n in nums if n >= 0]

def double_values(nums):
    return [n * 2 for n in nums]

def sum_all(nums):
    return sum(nums)

def compose(*functions):                        # HOF ทั่วไปสำหรับประกอบฟังก์ชันหลายตัวเข้าด้วยกัน
    def pipeline(data):
        for func in functions:
            data = func(data)                      # ผลลัพธ์ของฟังก์ชันก่อนหน้า -> input ของฟังก์ชันถัดไป
        return data
    return pipeline

process = compose(remove_negatives, double_values, sum_all)
print(process([-2, 3, -5, 8, -1, 4]))          # 30
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `list(map(lambda x: x ** 2, numbers))` — `map()` รับฟังก์ชัน (`lambda x: x**2`) และ iterable (`numbers`) แล้วเรียกฟังก์ชันนั้นกับสมาชิกทุกตัวทีละตัว คืนค่าเป็น **map object** (lazy iterator ที่ยังไม่คำนวณจริงจนกว่าจะถูกวนใช้) จึงต้องห่อด้วย `list()` เพื่อบังคับให้คำนวณและเก็บผลลัพธ์เป็น list จริง
2. `make_multiplier(factor)` — เมื่อเรียก `make_multiplier(2)` ฟังก์ชันชั้นในสุด `multiply` ถูกสร้างขึ้นพร้อม**จำค่า `factor=2` ไว้ในตัวมันเอง** แล้ว `make_multiplier` คืนค่า `multiply` (ยังไม่เรียกใช้ ไม่มีวงเล็บ) ออกมา — แม้ `make_multiplier(2)` จะทำงานเสร็จสิ้นไปแล้ว (return แล้ว) ตัวแปร `factor` ยังคง "อยู่" ในหน่วยความจำที่ `double` เข้าถึงได้เสมอ เพราะ `double` คือ **Closure** ที่ปิดล้อม (close over) ตัวแปร `factor` ไว้
3. `nonlocal count` ใน `increment()` — คีย์เวิร์ด `nonlocal` (ต่างจาก `global` ในบทที่แล้วที่อ้างถึงตัวแปรระดับโมดูล) บอก Python ว่า `count` ที่จะแก้ไขคือตัวแปรจาก **enclosing scope** (scope ของ `make_counter` ที่ห่อ `increment` อยู่) ไม่ใช่การสร้างตัวแปร `count` ใหม่ในขอบเขตของ `increment` เอง — ถ้าไม่มี `nonlocal` บรรทัด `count += 1` จะโยน error ทันทีเพราะ Python จะพยายามอ่านตัวแปร local ที่ยังไม่ถูกกำหนดค่า
4. `compose(*functions)` — ใช้ `*functions` (unpacking พารามิเตอร์แบบไม่จำกัดจำนวน) รับฟังก์ชันกี่ตัวก็ได้ แล้วคืนค่าฟังก์ชัน `pipeline` ที่วนลูปเรียกแต่ละฟังก์ชันตามลำดับ โดยส่ง**ผลลัพธ์ของฟังก์ชันก่อนหน้าเป็น input ของฟังก์ชันถัดไปเสมอ** (`data = func(data)`) — `process = compose(remove_negatives, double_values, sum_all)` จึงเทียบเท่ากับการเขียน `sum_all(double_values(remove_negatives(data)))` แต่**อ่านลำดับการทำงานจากซ้ายไปขวาได้ชัดเจนกว่ามาก**

## Time Complexity

- `map()`, `filter()`: **O(n)** โดย n คือจำนวนสมาชิกใน iterable (เรียกฟังก์ชันที่ส่งเข้ามาหนึ่งครั้งต่อสมาชิกหนึ่งตัว)
- Closure (การเรียก `increment()` แต่ละครั้ง): **O(1)** — แค่เข้าถึงและแก้ไขตัวแปรที่จำไว้
- `compose()` ที่มี k ฟังก์ชันในลำดับ: **O(k × cost ของแต่ละฟังก์ชัน)** รวมกัน — ในตัวอย่างข้างต้นแต่ละฟังก์ชันเป็น O(n) จึงรวมเป็น O(k × n)

## Space Complexity

- Closure ใช้พื้นที่เพิ่ม **O(1)** ต่อตัวแปรที่ถูกจดจำไว้ (เช่น `factor`, `count`) — แต่ถ้าสร้าง closure จำนวนมาก (เช่น `make_counter()` นับพันครั้ง) แต่ละตัวจะมีพื้นที่เก็บ state ของตัวเองแยกกัน รวมเป็น O(จำนวน closure ที่สร้าง)
- `compose()` ที่สร้าง list ใหม่ทุกขั้นตอน (ตามหลัก Immutability บทที่แล้ว) ใช้พื้นที่ **O(n × k)** ในกรณีที่แย่ที่สุด เพราะแต่ละขั้นตอนสร้างข้อมูลชุดใหม่แยกจากกัน (trade-off เดียวกับที่กล่าวถึงในบทที่แล้ว)

## แนวทางปฏิบัติที่ดี (Best Practices)

- ใช้ Lambda เฉพาะกับฟังก์ชันที่ **สั้นมากและใช้ครั้งเดียว** (เช่น `key=` ใน `sorted()`) ถ้าฟังก์ชันมี logic ซับซ้อนหรือใช้ซ้ำหลายที่ ควรเขียนเป็น `def` ที่มีชื่อชัดเจนแทน เพื่อความอ่านง่าย
- ใช้ Closure เมื่อต้องการฟังก์ชันที่ "จำ" ค่าคงที่หรือ state เล็กๆ น้อยๆ โดยไม่ต้องการความซับซ้อนของการสร้าง Class เต็มรูปแบบ — แต่ถ้า state ที่ต้องจำมีความซับซ้อนมาก (หลาย attribute, หลาย method ที่เกี่ยวข้องกัน) ควรกลับไปใช้ Class (Part 6) แทน
- ออกแบบฟังก์ชันที่จะนำมา compose กันให้เป็น Pure Function เสมอ (ทบทวนจากบทที่แล้ว) เพราะถ้ามี Side Effect แทรกอยู่ตรงกลาง pipeline การเรียงลำดับหรือทดสอบแต่ละขั้นตอนจะยากขึ้นมาก

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ลืมใส่ `nonlocal` เมื่อต้องการแก้ไขตัวแปรจาก enclosing scope ภายใน Closure ทำให้เกิด `UnboundLocalError` เพราะ Python จะถือว่าตัวแปรนั้นเป็นตัวแปร local ใหม่ทันทีที่เห็นการ assign ค่าให้มัน
- ใช้ `map()`/`filter()` แล้วลืมห่อด้วย `list()` เมื่อต้องการผลลัพธ์เป็น list จริงๆ (ไม่ใช่ lazy iterator object) ทำให้พิมพ์ผลลัพธ์ออกมาเป็น `<map object at 0x...>` โดยไม่เข้าใจว่าเกิดอะไรขึ้น
- สร้าง Closure ใน loop โดยอ้างอิงตัวแปร loop โดยตรง (กับดัก "late binding" ใน Python) เช่น `funcs = [lambda: i for i in range(3)]` ทำให้ทุกฟังก์ชันใน list คืนค่า `2` เหมือนกันหมด (ค่าสุดท้ายของ `i`) แทนที่จะเป็น `0, 1, 2` ตามที่คาดหวัง — ต้องแก้ด้วยการส่ง `i` เป็นค่าเริ่มต้นของพารามิเตอร์แทน (`lambda i=i: i`)
- เขียน Lambda ที่ซับซ้อนเกินไป (มีเงื่อนไขหลายชั้นซ้อนกันในบรรทัดเดียว) ทำให้อ่านยากกว่าการเขียน `def` ที่มีชื่อและตัวแปรอธิบายตัวเองชัดเจน

## คำถามสัมภาษณ์งาน (Interview Questions)

1. First-Class Function คืออะไร ทำไมถึงเป็นเงื่อนไขที่จำเป็นสำหรับ Higher Order Function
2. อธิบาย Closure คืออะไร ยกตัวอย่างสถานการณ์ที่ Closure มีประโยชน์กว่าการใช้ Class
3. `nonlocal` กับ `global` ต่างกันอย่างไร แต่ละคำสั่งใช้ในสถานการณ์ไหน
4. อธิบายกับดัก "late binding" เมื่อสร้าง Closure หลายตัวใน loop พร้อมวิธีแก้ไข
5. Function Composition มีประโยชน์อย่างไรเมื่อเทียบกับการเขียนโค้ดประมวลผลข้อมูลแบบ nested function call ยาวๆ (`f(g(h(x)))`)

## แบบฝึกหัด (Practice Exercises)

1. เขียน HOF ชื่อ `retry(func, times)` ที่รับฟังก์ชันและจำนวนครั้ง แล้วคืนค่าฟังก์ชันใหม่ที่พยายามเรียก `func` ซ้ำสูงสุด `times` ครั้งถ้าเกิด exception (ใบ้: ใช้ `try/except` ร่วมกับ Closure)
2. เขียนฟังก์ชัน `compose2(f, g)` ที่รับฟังก์ชัน 2 ตัว แล้วคืนค่าฟังก์ชันใหม่ที่เทียบเท่ากับ `f(g(x))` จากนั้นทดสอบว่า `compose(f, g, h)` (จากตัวอย่างในบทเรียน) ให้ผลลัพธ์เดียวกันกับการเรียก `compose2` ซ้อนกันหลายชั้นหรือไม่
3. แก้กับดัก "late binding" ในโค้ดตัวอย่าง `funcs = [lambda: i for i in range(3)]` ให้ได้ผลลัพธ์ที่ถูกต้อง (`0, 1, 2`) แทนที่จะได้ `2` ซ้ำกันหมด

## Mini Project

**"เครื่องมือประมวลผลข้อมูลแบบ Pipeline (Functional Data Processing Toolkit)"**: เขียนโปรแกรมที่:
1. เขียนชุดฟังก์ชัน Pure Function สำหรับแปลงข้อมูลพนักงาน (list ของ dict ที่มี `name`, `salary`, `department`) เช่น `filter_by_department()`, `apply_raise(percent)` (ใช้ Closure เก็บค่า percent), `sort_by_salary()`
2. เขียนฟังก์ชัน `compose()` (ตามตัวอย่างในบทเรียน) เพื่อประกอบฟังก์ชันเหล่านี้เข้าด้วยกันเป็น pipeline ที่ปรับแต่งได้ตามต้องการ (สลับลำดับ, เพิ่ม/ลดขั้นตอนได้ง่าย)
3. ใช้ `make_multiplier`-style Closure สร้างฟังก์ชันคำนวณโบนัสที่ปรับเปอร์เซ็นต์ได้ตามแผนก (เช่น `make_bonus_calculator(0.1)` สำหรับแผนกขาย, `make_bonus_calculator(0.05)` สำหรับแผนกอื่น)
4. เขียนรายงานเปรียบเทียบโค้ดเวอร์ชัน Functional (ใช้ HOF และ compose) กับเวอร์ชันที่เขียนด้วย loop และ if/else ตรงๆ แบบดั้งเดิม เพื่อดูว่าอ่านง่ายขึ้นหรือไม่ในสถานการณ์นี้

## ข้อคิดสำคัญ (Key Takeaways)

- First-Class Function ทำให้ฟังก์ชันเก็บในตัวแปร ส่งเป็นพารามิเตอร์ หรือคืนค่าออกจากฟังก์ชันอื่นได้เหมือนข้อมูลทั่วไป เป็นรากฐานของทุกแนวคิดในบทนี้
- Higher Order Function (`map`, `filter`, ฟังก์ชันที่คืนค่าเป็นฟังก์ชัน) ลด boilerplate ของ loop และสื่อเจตนาของโค้ดได้ชัดเจนกว่า
- Closure ทำให้ฟังก์ชัน "จำ" ตัวแปรจาก scope ภายนอกไว้ได้ แม้ scope นั้นจะทำงานจบไปแล้ว เป็นทางเลือกที่เบากว่า Class สำหรับการเก็บ state ขนาดเล็ก
- Function Composition ต่อ Pure Function หลายตัว (จากบทที่แล้ว) เป็น pipeline ที่อ่านง่ายและทดสอบทีละขั้นตอนได้
- ต้องระวังกับดักคลาสสิกอย่าง "late binding" ใน Closure ที่สร้างภายใน loop และการลืม `nonlocal`/`list()` ที่พบบ่อยเมื่อเริ่มเขียน FP ใน Python
- นี่คือบทปิดท้าย **Part 7: Functional Programming** — บทถัดไปจะเข้าสู่ **Part 8: Concurrency** ซึ่งเชื่อมโยงกลับไปที่ประโยชน์ของ Immutability (บทที่แล้ว) โดยตรงในการแก้ปัญหา Race Condition เมื่อหลาย thread ทำงานพร้อมกัน โดยเริ่มจาก Thread, Mutex, Semaphore, Atomic

---
*บทต่อไป (บทที่ 36): Thread, Mutex, Semaphore, Atomic — พิมพ์ "Next" เพื่อดำเนินการต่อ*
