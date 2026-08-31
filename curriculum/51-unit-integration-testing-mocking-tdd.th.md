# บทที่ 51: Unit / Integration Testing, Mocking, TDD

## แนวคิดหลัก (Concept)

บทที่แล้วสอน Git ในการจัดการประวัติของโค้ด แต่ไม่ได้ตอบว่า "จะรู้ได้อย่างไรว่าโค้ดที่แก้ไขล่าสุดยังทำงานถูกต้อง" บทนี้เปิด **Part 14: Testing** ด้วยแนวคิดพื้นฐานที่สุดของการทดสอบซอฟต์แวร์อย่างเป็นระบบ — แทนที่จะรันโปรแกรมด้วยมือทุกครั้งเพื่อเช็คว่ายังทำงานถูกต้อง เขียน**โค้ดที่ทดสอบโค้ด**ให้ทำงานอัตโนมัติแทน

- **Unit Test** — ทดสอบ**หน่วยเล็กที่สุด**ของโค้ด (มักเป็นฟังก์ชันเดียวหรือ method เดียว) แบบแยกโดดเดี่ยวจากส่วนอื่นของระบบ เพื่อยืนยันว่าหน่วยนั้นทำงานถูกต้องตามที่ออกแบบไว้
- **Integration Test** — ทดสอบว่า**หลายส่วนของระบบทำงานร่วมกันได้ถูกต้อง** (เช่น ฟังก์ชันที่คุยกับฐานข้อมูลจริง หรือ service ที่เรียก API อื่นจริง) ต่างจาก Unit Test ที่แยกทดสอบทีละส่วน
- **Mocking** — เทคนิคสร้าง**ของปลอม**แทนที่ dependency จริง (เช่น ฐานข้อมูล, API ภายนอก) เพื่อทดสอบ Unit Test โดยไม่ต้องพึ่งพาสิ่งเหล่านั้นจริง ทำให้ทดสอบเร็วและคาดเดาผลลัพธ์ได้แน่นอน
- **TDD (Test-Driven Development)** — แนวทางการพัฒนาที่**เขียนเทสต์ก่อนเขียนโค้ดจริง** ตามวงจร **Red-Green-Refactor**: เขียนเทสต์ที่ยังไม่ผ่าน (Red) -> เขียนโค้ดขั้นต่ำให้เทสต์ผ่าน (Green) -> ปรับปรุงโค้ดให้สะอาดขึ้นโดยเทสต์ยังผ่านเหมือนเดิม (Refactor)

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Unit Test มีไว้จับบั๊กได้เร็วและถูกจุด**: ถ้าไม่มีเทสต์อัตโนมัติ ทุกครั้งที่แก้โค้ดต้องรันโปรแกรมทั้งหมดด้วยมือเพื่อเช็คว่ายังทำงานถูกต้อง (ช้าและมักพลาดกรณีที่ไม่ได้ลองด้วยมือ) — Unit Test รันได้ในเสี้ยววินาทีและบอกทันทีว่าฟังก์ชันไหนพังจากการแก้ไขล่าสุด
- **Integration Test มีไว้จับปัญหาที่ Unit Test มองไม่เห็น**: ฟังก์ชันแต่ละตัวอาจผ่าน Unit Test ทุกตัว แต่เมื่อนำมาทำงานร่วมกันจริง (เช่น รูปแบบข้อมูลที่ service หนึ่งส่งไม่ตรงกับที่อีก service คาดหวัง) อาจเกิดปัญหาที่ Unit Test แยกส่วนตรวจไม่เจอ — Integration Test ยืนยันว่าส่วนต่างๆ "พูดภาษาเดียวกัน" จริง
- **Mocking มีไว้ทำให้ Unit Test เร็ว คาดเดาได้ และไม่ขึ้นกับปัจจัยภายนอก**: ถ้า Unit Test ต้องเชื่อมต่อฐานข้อมูลจริงทุกครั้ง จะช้า (network I/O) และผลลัพธ์อาจไม่แน่นอน (ข้อมูลในฐานข้อมูลเปลี่ยนได้ตลอดเวลา) — Mock แทนที่ dependency เหล่านั้นด้วยของปลอมที่ควบคุมได้เต็มที่ ทำให้ทดสอบ logic ล้วนๆ โดยไม่ปนกับปัจจัยภายนอก
- **TDD มีไว้บังคับให้คิดเรื่อง "จะรู้ได้อย่างไรว่าโค้ดถูกต้อง" ก่อนเขียนโค้ดจริง**: การเขียนเทสต์ทีหลังมักถูกมองข้ามเพราะรีบไปเรื่องอื่น (เชื่อมโยงกับปัญหาที่บทที่ 47 เตือนเรื่อง Small Functions ที่ทดสอบยากถ้าฟังก์ชันทำหลายอย่างปนกัน) TDD กลับลำดับให้เทสต์นำ ทำให้โค้ดที่ได้ถูกออกแบบมาให้ทดสอบง่ายตั้งแต่ต้น (testable by design)

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Unit Test** เหมือน **การตรวจสอบชิ้นส่วนแต่ละชิ้นก่อนประกอบรถยนต์** — เช็คว่าเบรกทำงานได้ดีแยกต่างหาก เช็คว่าไฟหน้าติดแยกต่างหาก ก่อนเอาทุกชิ้นมาประกอบรวมกัน
- **Integration Test** เหมือน **การทดลองขับรถทั้งคันหลังประกอบเสร็จ** — แม้แต่ละชิ้นส่วนจะผ่านการตรวจแยกแล้ว ต้องทดลองว่าทำงานร่วมกันจริงได้ไหม (เช่น เบรกกับระบบไฟฟ้าทำงานประสานกันถูกต้องหรือไม่)
- **Mocking** เหมือน **หุ่นจำลองผู้ป่วยที่นักศึกษาแพทย์ใช้ฝึกก่อนผ่าตัดจริง** — ทดสอบทักษะและขั้นตอนได้โดยไม่ต้องเสี่ยงกับผู้ป่วยจริง ควบคุมสถานการณ์ได้เต็มที่ (เช่น จำลองอาการฉุกเฉินได้ตามต้องการ)
- **TDD** เหมือน **การตั้งเป้าหมายของข้อสอบก่อนแล้วค่อยเรียนเนื้อหาให้ผ่านเป้าหมายนั้น** — รู้ชัดเจนตั้งแต่ต้นว่า "ต้องทำอะไรได้บ้างถึงจะถือว่าสำเร็จ" แทนที่จะเรียนไปเรื่อยๆ แล้วค่อยดูว่าครอบคลุมสิ่งที่ต้องรู้หรือไม่ทีหลัง

## แผนภาพอธิบาย (Visual explanation)

### Testing Pyramid — สัดส่วนของเทสต์แต่ละชนิดที่แนะนำ

```
                    /\
                   /  \      E2E Tests (น้อยที่สุด)
                  /----\     - ทดสอบทั้งระบบเหมือนผู้ใช้จริง ช้าที่สุด เปราะบางที่สุด
                 /      \
                / Integ- \   Integration Tests (ปานกลาง)
               /  ration  \  - ทดสอบหลายส่วนทำงานร่วมกัน ช้ากว่า Unit แต่เร็วกว่า E2E
              /------------\
             /              \
            /   Unit Tests   \  Unit Tests (มากที่สุด)
           /                  \ - ทดสอบทีละฟังก์ชัน เร็วที่สุด เขียนง่ายที่สุด
          /____________________\

ยิ่งฐานพีระมิดกว้าง (Unit Test เยอะ) ยิ่งได้ feedback เร็วและเทสต์ทั้งชุดรันเร็ว
```

### Mocking — แทนที่ Dependency จริงด้วยของปลอมที่ควบคุมได้

```
โค้ดจริง:                                   ทดสอบด้วย Mock:

def get_user_discount(user_id, db):         def test_get_user_discount():
    user = db.fetch_user(user_id)               mock_db = MockDatabase()
    if user.is_vip:                              mock_db.fetch_user = lambda id: User(is_vip=True)
        return 0.2                               result = get_user_discount(1, mock_db)
    return 0.0                                   assert result == 0.2

ไม่ต้องเชื่อมต่อฐานข้อมูลจริงเลย -> เร็วมาก, ผลลัพธ์แน่นอนเสมอ (ไม่ขึ้นกับข้อมูลจริงในฐานข้อมูล)
```

### TDD Cycle — Red, Green, Refactor

```
1. RED:      เขียนเทสต์ที่ยังไม่ผ่าน (เพราะยังไม่มีโค้ดจริง)
             def test_add(): assert add(2, 3) == 5    <- ยังไม่มีฟังก์ชัน add() เลย -> เทสต์ล้มเหลว (แดง)

2. GREEN:    เขียนโค้ดขั้นต่ำสุดให้เทสต์ผ่าน
             def add(a, b): return a + b               <- เทสต์ผ่านแล้ว (เขียว)

3. REFACTOR: ปรับปรุงโค้ดให้สะอาดขึ้น โดยเทสต์ต้องยังผ่านเหมือนเดิม
             (ในตัวอย่างนี้โค้ดง่ายมากแล้ว แต่ในเคสจริงอาจปรับโครงสร้าง/ตั้งชื่อใหม่ให้ดีขึ้น)

วนซ้ำ Red -> Green -> Refactor ไปเรื่อยๆ ทีละฟีเจอร์เล็กๆ
```

## ตัวอย่างโค้ด (Code example)

```python
import unittest
from unittest.mock import Mock

# ===== โค้ดจริงที่จะทดสอบ =====
def calculate_shipping_cost(weight_kg, is_express):
    base_cost = weight_kg * 10
    return base_cost * 2 if is_express else base_cost

class OrderService:
    def __init__(self, database, email_service):          # รับ dependency ผ่าน constructor (DI จากบทที่ 32)
        self.database = database
        self.email_service = email_service

    def place_order(self, user_id, total):
        user = self.database.get_user(user_id)               # dependency ภายนอก (จะ mock ตอนเทสต์)
        if user.balance < total:
            return {"success": False}
        self.database.deduct_balance(user_id, total)
        self.email_service.send(user.email, f"Order total: {total}")
        return {"success": True}

# ===== Unit Test: ทดสอบฟังก์ชันเดียวแบบแยกโดดเดี่ยว =====
class TestShippingCost(unittest.TestCase):
    def test_normal_shipping(self):
        self.assertEqual(calculate_shipping_cost(5, False), 50)

    def test_express_shipping_doubles_cost(self):
        self.assertEqual(calculate_shipping_cost(5, True), 100)

    def test_zero_weight(self):                              # ทดสอบ edge case เสมอ
        self.assertEqual(calculate_shipping_cost(0, False), 0)

# ===== Unit Test พร้อม Mocking: ทดสอบ OrderService โดยไม่ต้องมีฐานข้อมูล/email service จริง =====
class TestOrderService(unittest.TestCase):
    def test_successful_order(self):
        mock_db = Mock()
        mock_db.get_user.return_value = Mock(balance=1000, email="test@mail.com")
        mock_email = Mock()

        service = OrderService(database=mock_db, email_service=mock_email)
        result = service.place_order(user_id=1, total=500)

        self.assertTrue(result["success"])
        mock_db.deduct_balance.assert_called_once_with(1, 500)     # ตรวจว่าถูกเรียกจริงพร้อมค่าที่ถูกต้อง
        mock_email.send.assert_called_once()                          # ตรวจว่ามีการส่งอีเมลจริง

    def test_insufficient_balance(self):
        mock_db = Mock()
        mock_db.get_user.return_value = Mock(balance=100, email="test@mail.com")
        mock_email = Mock()

        service = OrderService(database=mock_db, email_service=mock_email)
        result = service.place_order(user_id=1, total=500)

        self.assertFalse(result["success"])
        mock_db.deduct_balance.assert_not_called()                    # ตรวจว่าไม่มีการหักเงินเลยเมื่อยอดไม่พอ

# ===== TDD ตัวอย่าง: เขียนเทสต์ก่อน แล้วค่อยเขียนโค้ด =====
class TestDiscountCalculator(unittest.TestCase):
    def test_vip_gets_20_percent_discount(self):              # RED: เขียนเทสต์นี้ก่อนที่ calculate_discount จะมีอยู่จริง
        self.assertEqual(calculate_discount(100, is_vip=True), 20)

    def test_regular_gets_no_discount(self):
        self.assertEqual(calculate_discount(100, is_vip=False), 0)

def calculate_discount(amount, is_vip):                        # GREEN: เขียนโค้ดขั้นต่ำให้เทสต์ผ่าน
    return amount * 0.2 if is_vip else 0

if __name__ == "__main__":
    unittest.main()
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `TestShippingCost` — แต่ละเมทอด (`test_normal_shipping`, `test_express_shipping_doubles_cost`, `test_zero_weight`) ทดสอบ**พฤติกรรมเดียว**ของ `calculate_shipping_cost` โดยไม่ต้องพึ่งพาสิ่งใดนอกจากตัวฟังก์ชันเอง — เพราะ `calculate_shipping_cost` เป็น **Pure Function** (ทบทวนจากบทที่ 34) การทดสอบจึงทำได้ง่ายมาก แค่ป้อน input แล้วเช็ค output ตรงๆ โดยไม่ต้องตั้งค่า state ใดๆ ก่อน
2. `Mock()` ใน `test_successful_order` — สร้าง**object ปลอม**ที่แสร้งทำตัวเป็น database/email service จริง `mock_db.get_user.return_value = Mock(balance=1000, ...)` กำหนดว่า**เมื่อไหร่ก็ตามที่มีการเรียก `mock_db.get_user(...)`** (ไม่ว่าจะส่งพารามิเตอร์อะไรมา) ให้คืนค่า object ที่มี `balance=1000` เสมอ — ทำให้ `OrderService.place_order()` ทำงานได้เต็มที่โดยไม่ต้องมีฐานข้อมูลจริงอยู่เลย
3. `mock_db.deduct_balance.assert_called_once_with(1, 500)` — ไม่ได้แค่เช็คว่า `result["success"]` เป็น `True` แต่ยัง**ยืนยันว่าฟังก์ชัน `deduct_balance` ถูกเรียกจริงพร้อมพารามิเตอร์ที่ถูกต้อง** (`user_id=1, total=500`) — นี่คือความสามารถพิเศษของ Mock ที่ทำให้ตรวจสอบ**พฤติกรรมการเรียกใช้** ไม่ใช่แค่ค่าผลลัพธ์สุดท้ายเท่านั้น
4. `test_insufficient_balance` — `mock_db.deduct_balance.assert_not_called()` ยืนยันว่า**ไม่มีการหักเงินเกิดขึ้นเลย**เมื่อยอดเงินไม่พอ ซึ่งเป็นการทดสอบที่สำคัญไม่แพ้กรณีสำเร็จ (ทดสอบทั้งเส้นทาง "happy path" และ "error path" เสมอ)
5. `TestDiscountCalculator` — สังเกตลำดับการเขียน: **เทสต์ (`test_vip_gets_20_percent_discount`) ถูกเขียนขึ้นก่อน** ที่ `calculate_discount` จะมีอยู่จริงด้วยซ้ำ (ตามหลัก TDD) ถ้ารันเทสต์นี้ก่อนเขียนฟังก์ชัน จะได้ error ทันที (**Red**) — เมื่อเขียน `calculate_discount` แบบง่ายที่สุดที่ทำให้เทสต์ผ่าน (**Green**) แล้ว จึงค่อยพิจารณาว่าจะปรับปรุงโค้ดให้ดีขึ้นอีกหรือไม่ (**Refactor**) โดยรันเทสต์ซ้ำทุกครั้งเพื่อยืนยันว่ายังผ่านเหมือนเดิม

## Time Complexity

Testing เป็นเรื่องของ**กระบวนการพัฒนาซอฟต์แวร์** ไม่ใช่อัลกอริทึม แต่มีผลกระทบด้านเวลาที่วัดได้ชัดเจน:
- Unit Test: รันเร็วมาก (**milliseconds** ต่อเทสต์) เพราะไม่มี I/O ภายนอกเลย (โดยเฉพาะเมื่อใช้ Mock แทน dependency จริงทั้งหมด)
- Integration Test: รันช้ากว่า Unit Test มาก (**อาจเป็นวินาที**ต่อเทสต์) เพราะต้องเชื่อมต่อฐานข้อมูล/service จริง (network I/O ตามบทที่ 42)
- E2E Test: ช้าที่สุด (**หลายวินาทีถึงนาที**) เพราะจำลองการใช้งานทั้งระบบเหมือนผู้ใช้จริง

นี่คือเหตุผลที่ Testing Pyramid แนะนำให้มี Unit Test **เยอะที่สุด** เพื่อให้ชุดเทสต์ทั้งหมดรันเร็วและให้ feedback ทันทีเมื่อแก้โค้ด

## Space Complexity

- Mock object ใช้พื้นที่หน่วยความจำน้อยมาก (**O(1)** โดยประมาณ) เพราะไม่มีการเชื่อมต่อจริงหรือเก็บข้อมูลจริงใดๆ เลย เป็นแค่ object ที่จำลองพฤติกรรมตามที่กำหนด

## แนวทางปฏิบัติที่ดี (Best Practices)

- เขียน Unit Test ให้ครอบคลุมทั้ง **happy path** (กรณีที่ทำงานถูกต้อง) และ **edge case**/**error path** (กรณีขอบเขต เช่น ค่าว่าง, ค่าติดลบ, และกรณีที่ควรล้มเหลว) เสมอ อย่าทดสอบแค่กรณีที่คาดว่าจะสำเร็จ
- Mock เฉพาะ**dependency ภายนอก** (ฐานข้อมูล, API, ไฟล์ระบบ) ไม่ใช่ mock ทุกอย่างจนทดสอบแค่ mock กับ mock คุยกันเอง — ถ้า mock มากเกินไปเทสต์จะไม่ได้ยืนยันอะไรจริงเกี่ยวกับโค้ดเลย
- ออกแบบ dependency ด้วย Dependency Injection (บทที่ 32) เสมอ (รับผ่าน constructor เหมือนตัวอย่าง `OrderService`) เพื่อให้ mock ได้ง่ายตอนเทสต์ — ถ้าสร้าง dependency ขึ้นเองข้างในฟังก์ชันโดยตรง จะ mock ไม่ได้เลย

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- เขียนเทสต์ที่ทดสอบแค่กรณีสำเร็จเสมอ (happy path) โดยลืมทดสอบกรณีล้มเหลวหรือ edge case ทำให้บั๊กในกรณีพิเศษหลุดรอดไปถึง production
- Mock มากเกินไปจนเทสต์ไม่ได้ยืนยันพฤติกรรมจริงของโค้ดเลย (เช่น mock ทุกอย่างรวมถึง logic ที่ควรทดสอบจริง)
- เขียน Unit Test ที่พึ่งพากันเอง (test หนึ่งต้องรันก่อนอีก test ถึงจะผ่าน) ทำให้เทสต์ไม่เป็นอิสระจากกัน รันลำดับผิดแล้วพังทั้งชุด
- ทำ TDD แบบผิวเผิน (เขียนเทสต์หลังเขียนโค้ดเสร็จแล้วค่อยมาปรับให้ผ่าน) ซึ่งเสียประโยชน์หลักของ TDD ที่ต้องการให้เทสต์เป็นตัวขับเคลื่อนการออกแบบตั้งแต่ต้น
- ลืมทดสอบว่า mock ถูกเรียกด้วยพารามิเตอร์ที่ถูกต้อง (แค่เช็คผลลัพธ์สุดท้ายอย่างเดียว) ทำให้พลาดบั๊กที่ฟังก์ชันเรียก dependency ผิดวิธีแต่บังเอิญได้ผลลัพธ์ที่ดูถูกต้อง

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง Unit Test และ Integration Test พร้อมยกตัวอย่างแต่ละแบบ
2. Mocking คืออะไร ทำไมถึงจำเป็นสำหรับการเขียน Unit Test ที่ดี
3. อธิบายวงจร Red-Green-Refactor ของ TDD แต่ละขั้นตอนทำอะไรบ้าง
4. ทำไม Testing Pyramid ถึงแนะนำให้มี Unit Test เยอะกว่า Integration Test และ E2E Test มาก
5. Dependency Injection (บทที่ 32) เกี่ยวข้องกับความสามารถในการเขียน Unit Test ที่ดีอย่างไร

## แบบฝึกหัด (Practice Exercises)

1. เขียน Unit Test ให้ครบทุก edge case สำหรับฟังก์ชัน `calculate_discount` ในตัวอย่าง (เช่น amount เป็น 0, amount ติดลบ)
2. เขียน `PaymentService` ที่รับ `payment_gateway` เป็น dependency ผ่าน constructor แล้วเขียน Unit Test ที่ mock `payment_gateway` เพื่อทดสอบทั้งกรณีชำระเงินสำเร็จและล้มเหลว
3. ฝึก TDD: เขียนเทสต์ก่อนสำหรับฟังก์ชัน `is_valid_password(password)` ที่ต้องมีอย่างน้อย 8 ตัวอักษร มีตัวเลขอย่างน้อย 1 ตัว แล้วค่อยเขียนฟังก์ชันให้ผ่านเทสต์ทั้งหมด

## Mini Project

**"ระบบตะกร้าสินค้าที่พัฒนาด้วย TDD (Shopping Cart Built with TDD)"**: เขียนโปรแกรมที่:
1. ใช้ TDD พัฒนา Class `ShoppingCart` ตั้งแต่ต้น — เขียนเทสต์ก่อนเสมอสำหรับทุกฟีเจอร์ (เพิ่มสินค้า, ลบสินค้า, คำนวณราคารวม, ใส่โค้ดส่วนลด) ตามวงจร Red-Green-Refactor
2. เขียน Unit Test ที่ครอบคลุม edge case ทั้งหมด (ตะกร้าว่าง, สินค้าจำนวนติดลบ, โค้ดส่วนลดไม่ถูกต้อง)
3. สร้าง `OrderProcessor` ที่พึ่งพา `PaymentGateway` และ `InventorySystem` ผ่าน Dependency Injection แล้วเขียน Unit Test ที่ mock ทั้งสอง dependency เพื่อทดสอบ logic การประมวลผลออเดอร์แยกจากระบบภายนอกจริง
4. เขียน Integration Test อย่างน้อย 1 ตัวที่ทดสอบ `ShoppingCart` และ `OrderProcessor` ทำงานร่วมกันจริง (ไม่ mock) เพื่อยืนยันว่าทั้งสองส่วน "พูดภาษาเดียวกัน" ถูกต้อง

## ข้อคิดสำคัญ (Key Takeaways)

- Unit Test ทดสอบหน่วยเล็กที่สุดของโค้ดแบบแยกโดดเดี่ยว รันเร็วและควรมีจำนวนมากที่สุดตาม Testing Pyramid
- Integration Test ยืนยันว่าหลายส่วนของระบบทำงานร่วมกันได้จริง จับปัญหาที่ Unit Test แยกส่วนมองไม่เห็น
- Mocking แทนที่ dependency ภายนอกด้วยของปลอมที่ควบคุมได้ ทำให้ Unit Test เร็วและคาดเดาผลลัพธ์ได้แน่นอน โดยยังตรวจสอบพฤติกรรมการเรียกใช้ได้ด้วย (ไม่ใช่แค่ผลลัพธ์สุดท้าย)
- TDD กลับลำดับให้เขียนเทสต์ก่อนโค้ดจริงตามวงจร Red-Green-Refactor ทำให้โค้ดที่ได้ถูกออกแบบให้ทดสอบง่ายตั้งแต่ต้น
- Dependency Injection (บทที่ 32) เป็นกุญแจสำคัญที่ทำให้ mock dependency ได้ง่าย เชื่อมโยงหลักการ OOP/SOLID เข้ากับความสามารถในการทดสอบโดยตรง
- บทถัดไปจะเข้าสู่ **Part 15: Security** ซึ่งเป็นอีกทักษะที่ต้องคำนึงถึงคู่กับ Testing เสมอในการพัฒนาซอฟต์แวร์จริง โดยเริ่มจาก AuthN/AuthZ, JWT, OAuth2

---
*บทต่อไป (บทที่ 52): AuthN/AuthZ, JWT, OAuth2 — พิมพ์ "Next" เพื่อดำเนินการต่อ*
