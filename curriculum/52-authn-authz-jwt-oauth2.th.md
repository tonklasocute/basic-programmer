# บทที่ 52: AuthN/AuthZ, JWT, OAuth2

## แนวคิดหลัก (Concept)

บทที่ 51 สอนวิธียืนยันว่า**โค้ด**ทำงานถูกต้อง แต่ในระบบจริงยังต้องตอบคำถามที่สำคัญไม่แพ้กัน: **ยืนยันว่า "ผู้ใช้" คนนี้เป็นใครจริงๆ** และ **เขามีสิทธิ์ทำสิ่งที่ขอมาหรือไม่** บทนี้เปิด **Part 15: Security** ด้วยสองแนวคิดพื้นฐานที่มักสับสนกัน และเครื่องมือมาตรฐานที่ใช้ implement มันในระบบจริง

- **Authentication (AuthN)** — กระบวนการ**ยืนยันตัวตน**ว่าผู้ใช้คนนี้เป็นใครจริงๆ (ตอบคำถาม "คุณคือใคร?") เช่น ล็อกอินด้วยรหัสผ่าน
- **Authorization (AuthZ)** — กระบวนการ**ตรวจสอบสิทธิ์**ว่าผู้ใช้ที่ยืนยันตัวตนแล้วมีสิทธิ์ทำสิ่งที่ขอมาหรือไม่ (ตอบคำถาม "คุณทำสิ่งนี้ได้ไหม?") เช่น ตรวจว่าผู้ใช้เป็น admin ก่อนอนุญาตให้ลบข้อมูล
- **JWT (JSON Web Token)** — รูปแบบ token ที่เข้ารหัสข้อมูลผู้ใช้ (เช่น user id, role) ไว้ในตัวเอง พร้อมลายเซ็นดิจิทัลที่ยืนยันว่าไม่ถูกแก้ไข ทำให้ server ตรวจสอบได้โดยไม่ต้องถามฐานข้อมูลทุกครั้ง
- **OAuth2** — โปรโตคอลมาตรฐานที่อนุญาตให้แอปหนึ่ง**เข้าถึงข้อมูลของผู้ใช้บนอีกระบบหนึ่งได้โดยไม่ต้องรู้รหัสผ่านของผู้ใช้** (เช่น ปุ่ม "Login with Google")

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **AuthN/AuthZ ต้องแยกจากกันเพราะเป็นคำถามคนละคำถาม**: ผู้ใช้อาจยืนยันตัวตนสำเร็จ (รู้ว่าเป็นใคร) แต่ไม่มีสิทธิ์ทำบางอย่าง (เช่น พนักงานทั่วไปล็อกอินสำเร็จแต่ไม่มีสิทธิ์เข้าหน้า admin) — ถ้าปนสองแนวคิดนี้เข้าด้วยกันจะออกแบบระบบสิทธิ์ผิดพลาดได้ง่าย เช่น คิดว่า "ล็อกอินได้แล้วต้องทำได้ทุกอย่าง"
- **JWT มีไว้แก้ปัญหา Scalability ของการเก็บ Session ที่ฝั่ง Server**: วิธีดั้งเดิม (Session-based) ต้องเก็บสถานะการล็อกอินไว้ในฐานข้อมูล/หน่วยความจำฝั่ง server ทุกครั้งที่มีผู้ใช้ล็อกอิน — เมื่อระบบมีหลาย server (Microservices จากบทที่ 49) การเช็ค session ทุกครั้งต้องถามฐานข้อมูลกลาง ทำให้ช้าและเป็นคอขวด JWT ให้ server ตรวจสอบความถูกต้องได้ทันทีจากตัว token เองโดยไม่ต้องถามที่ไหนเลย (Stateless — เชื่อมโยงกับ Pure Function แนวคิดจากบทที่ 34)
- **OAuth2 มีไว้ให้ผู้ใช้ไม่ต้องแจกรหัสผ่านของตัวเองให้แอปอื่น**: ถ้าต้องการให้แอป A เข้าถึงรูปภาพใน Google Drive ของผู้ใช้ วิธีที่ไม่ปลอดภัยคือให้ผู้ใช้พิมพ์รหัสผ่าน Google ให้แอป A โดยตรง (แอป A จะรู้รหัสผ่านทั้งหมด เสี่ยงมาก) — OAuth2 ให้ Google ออก "ใบอนุญาตชั่วคราวที่จำกัดสิทธิ์" ให้แอป A แทน โดยผู้ใช้ไม่ต้องเปิดเผยรหัสผ่านให้แอป A เห็นเลย

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Authentication** เหมือน **การแสดงบัตรประชาชนที่ประตูสนามบิน** — พิสูจน์ว่าคุณเป็นคนที่มีชื่อตรงตามบัตรจริง
- **Authorization** เหมือน **การตรวจตั๋วเครื่องบินว่าคุณนั่งชั้นไหนได้** — แม้ยืนยันตัวตนผ่านแล้ว (ผ่านตรวจบัตรประชาชน) ก็ไม่ได้แปลว่าเข้าไปนั่ง First Class ได้ถ้าตั๋วเป็น Economy
- **JWT** เหมือน **ริสแบนด์เข้างานคอนเสิร์ตที่มีลายน้ำพิเศษพิมพ์ไว้ล่วงหน้า** — ยาม (server) แค่ดูริสแบนด์ก็รู้ทันทีว่าคนนี้ซื้อตั๋ว VIP หรือทั่วไป โดยไม่ต้องวิทยุถามฝ่ายขายตั๋ว (ฐานข้อมูล) ทุกครั้งที่มีคนเดินผ่าน
- **OAuth2** เหมือน **การให้บัตรพนักงานชั่วคราวแก่ช่างซ่อมแอร์ที่มาทำงานในบริษัท** — ช่างเข้าได้เฉพาะพื้นที่ที่กำหนด (จำกัดสิทธิ์) และบัตรหมดอายุเมื่องานเสร็จ โดยไม่ต้องรู้รหัสผ่านเข้าออกจริงของพนักงานประจำเลย

## แผนภาพอธิบาย (Visual explanation)

### AuthN vs AuthZ — สองคำถามที่แยกจากกันชัดเจน

```
Request: DELETE /users/5

ขั้นที่ 1 - Authentication (คุณคือใคร?)
   ตรวจ token/session -> "คุณคือ Bob, user_id=42"   ✓ ยืนยันตัวตนสำเร็จ

ขั้นที่ 2 - Authorization (Bob ลบ user คนอื่นได้ไหม?)
   ตรวจ role ของ Bob -> "Bob เป็น regular user ไม่ใช่ admin"
   -> ปฏิเสธคำขอ (403 Forbidden) แม้ Authentication จะผ่านแล้วก็ตาม!

สองขั้นตอนนี้ต้องผ่านทั้งคู่ถึงจะอนุญาตให้ทำ action ได้ ขาดข้อใดข้อหนึ่งไม่ได้
```

### JWT Structure — สามส่วนที่แยกด้วยจุด

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0Miwicm9sZSI6ImFkbWluIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
└──────── Header ────────┘└──────────── Payload ────────────┘└──────────── Signature ────────────┘

Header (base64):           Payload (base64):                Signature:
{"alg": "HS256"}           {"user_id": 42, "role": "admin"}  HMAC(header + payload, secret_key)

Header + Payload อ่านได้ทุกคนที่มี token (ไม่ได้เข้ารหัสลับ แค่ encode)
Signature เท่านั้นที่ป้องกันการปลอมแปลง -- ถ้าใครแก้ payload โดยไม่มี secret_key
signature เดิมจะไม่ตรงกับข้อมูลใหม่ -> server ตรวจจับได้ทันทีว่าถูกแก้ไข
```

### JWT Flow — ทำไมถึงไม่ต้องถามฐานข้อมูลทุกครั้ง

```
1. Login สำเร็จ:
   Client -> POST /login (username, password) -> Server
   Server ตรวจสอบกับฐานข้อมูล (ครั้งเดียว) -> สร้าง JWT -> ส่งกลับให้ Client

2. Request ถัดไปทุกครั้ง (ไม่ต้องถามฐานข้อมูลอีกเลย):
   Client -> GET /orders (แนบ JWT ใน Header) -> Server
   Server ตรวจ signature ของ JWT ด้วย secret_key ที่มีอยู่แล้ว -> ยืนยันตัวตนได้ทันที (Stateless)
   ถ้า signature ถูกต้อง -> เชื่อถือข้อมูลใน payload ได้เลย (user_id, role) โดยไม่ต้อง query DB
```

### OAuth2 Flow — "Login with Google" ทำงานอย่างไร

```
App A (ต้องการรูปจาก Google Drive)          Google (Resource Owner ยืนยันสิทธิ์)

1. Client คลิก "Login with Google" -> App A ส่งไปที่ Google
2. Google ถามผู้ใช้: "อนุญาตให้ App A เข้าถึง Google Drive ของคุณไหม?"
3. ผู้ใช้กด "อนุญาต" (ล็อกอิน Google โดยตรง ไม่ผ่าน App A เลย!)
4. Google ส่ง Authorization Code กลับไปที่ App A
5. App A ใช้ Code แลก Access Token กับ Google (เบื้องหลัง, server-to-server)
6. App A ใช้ Access Token เรียก Google Drive API แทนผู้ใช้

App A ไม่เคยเห็นรหัสผ่าน Google ของผู้ใช้เลยตลอดกระบวนการ!
Access Token มีขอบเขตจำกัด (scope) และหมดอายุได้ ต่างจากรหัสผ่านที่ใช้ได้ตลอดไปทุกที่
```

## ตัวอย่างโค้ด (Code example)

```python
import jwt                                              # pip install pyjwt
import datetime
from functools import wraps

SECRET_KEY = "your-secret-key-keep-this-safe"           # ต้องเก็บเป็นความลับ (ไม่ commit ลง Git!)

# ===== Authentication: ตรวจสอบรหัสผ่านแล้วออก JWT =====
def login(username, password, user_db):
    user = user_db.get(username)
    if user is None or user["password"] != password:      # ในระบบจริงต้อง hash รหัสผ่าน ไม่เก็บ plain text
        return None
    payload = {
        "user_id": user["id"],
        "role": user["role"],
        "exp": datetime.datetime.utcnow() + datetime.timedelta(hours=1)   # หมดอายุใน 1 ชั่วโมง
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")
    return token

# ===== Authentication: ตรวจสอบ JWT ที่แนบมากับ request =====
def authenticate(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])  # ตรวจ signature อัตโนมัติ
        return payload                                                    # {"user_id": 42, "role": "admin", ...}
    except jwt.ExpiredSignatureError:
        return None                                                       # token หมดอายุแล้ว
    except jwt.InvalidSignatureError:
        return None                                                       # signature ไม่ตรง -> ถูกปลอมแปลง!

# ===== Authorization: ตรวจสอบสิทธิ์แยกจาก Authentication เสมอ =====
def require_role(required_role):                          # Decorator (ทบทวนหลักการ HOF จากบทที่ 35)
    def decorator(func):
        @wraps(func)
        def wrapper(token, *args, **kwargs):
            payload = authenticate(token)                     # ขั้นที่ 1: Authentication
            if payload is None:
                return {"error": "Unauthorized", "status": 401}
            if payload["role"] != required_role:                # ขั้นที่ 2: Authorization (แยกกันชัดเจน)
                return {"error": "Forbidden", "status": 403}
            return func(payload, *args, **kwargs)
        return wrapper
    return decorator

@require_role("admin")
def delete_user(payload, target_user_id):
    return {"message": f"User {target_user_id} deleted by admin {payload['user_id']}"}

# ===== ทดสอบ =====
users_db = {"alice": {"id": 1, "password": "secret123", "role": "admin"},
            "bob": {"id": 2, "password": "pass456", "role": "user"}}

admin_token = login("alice", "secret123", users_db)
user_token = login("bob", "pass456", users_db)

print(delete_user(admin_token, target_user_id=99))       # สำเร็จ -- alice เป็น admin
print(delete_user(user_token, target_user_id=99))          # {"error": "Forbidden", "status": 403} -- bob ไม่ใช่ admin
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `login(username, password, user_db)` — ตรวจสอบ credential กับฐานข้อมูล**เพียงครั้งเดียว**ตอนล็อกอิน แล้วสร้าง `payload` ที่มีข้อมูลจำเป็น (`user_id`, `role`) พร้อม `exp` (เวลาหมดอายุ) — `jwt.encode(...)` สร้างลายเซ็นดิจิทัลด้วย `SECRET_KEY` ที่มีแค่ server เท่านั้นที่รู้ ทำให้ token ที่ได้**ไม่มีทางถูกปลอมแปลงได้โดยไม่มี `SECRET_KEY` นี้**
2. `authenticate(token)` — เรียก `jwt.decode(...)` ซึ่งตรวจสอบ**ลายเซ็นก่อนเสมอ**: ถ้า token ถูกแก้ไขแม้แต่ตัวอักษรเดียว (เช่น เปลี่ยน `"role": "user"` เป็น `"role": "admin"`) signature ที่คำนวณได้จะไม่ตรงกับที่แนบมา ทำให้ `jwt.decode` โยน `InvalidSignatureError` ทันที — **ไม่ต้องถามฐานข้อมูลเลยแม้แต่ครั้งเดียว**ในขั้นตอนนี้ เพราะข้อมูลทั้งหมดที่ต้องการอยู่ใน token เองแล้วและตรวจสอบความถูกต้องได้ในเครื่อง
3. `require_role("admin")` — เป็น **Decorator** (HOF จากบทที่ 35) ที่ครอบฟังก์ชัน `delete_user` ไว้ ก่อนเรียกฟังก์ชันจริง จะทำ **Authentication ก่อน** (`authenticate(token)`) แล้วจึงทำ **Authorization** (`payload["role"] != required_role`) **แยกกันเป็นสองขั้นตอนที่ชัดเจน** — สังเกตว่าแม้ `bob` จะผ่าน Authentication (มี token ที่ signature ถูกต้อง) แต่ยังคงถูกปฏิเสธที่ขั้น Authorization เพราะ role ไม่ตรง (403 Forbidden ไม่ใช่ 401 Unauthorized — สอง status code นี้สื่อความหมายต่างกันตามหลักการ HTTP จากบทที่ 42)
4. ผลลัพธ์ที่ต่างกันระหว่าง `admin_token` และ `user_token` — พิสูจน์ให้เห็นชัดว่า **การมี token ที่ valid (ผ่าน Authentication) ไม่ได้แปลว่าทำทุกอย่างได้เสมอไป** (ผ่าน Authorization ด้วย) ตรงตามหลักการที่อธิบายไว้ในหัวข้อ Concept

## Time Complexity

- **Session-based Authentication** (วิธีดั้งเดิม): ต้อง**ค้นหาในฐานข้อมูล/แคชกลาง**ทุกครั้งที่มี request — O(1) ถ้าใช้ Hash Table (บทที่ 15) แต่ยังคงมีต้นทุนของ network I/O ไปยังฐานข้อมูลกลางทุกครั้ง
- **JWT Authentication**: ตรวจสอบ signature ด้วยการคำนวณ HMAC ในเครื่อง — **O(1)** โดยไม่มี network I/O เลย เร็วกว่า Session-based มากเมื่อระบบมีหลาย server กระจายกัน

## Space Complexity

- Session-based: server ต้องเก็บสถานะของทุก session ที่ active อยู่ — **O(จำนวนผู้ใช้ที่ล็อกอินพร้อมกัน)**
- JWT: server**ไม่ต้องเก็บอะไรเลย** (Stateless) เพราะข้อมูลทั้งหมดอยู่ใน token ที่ client ถือไว้เอง — ประหยัดพื้นที่ฝั่ง server มาก แต่ token มีขนาดใหญ่กว่า session ID ธรรมดา (ต้องส่งข้อมูล payload ไปกลับทุกครั้ง)

## แนวทางปฏิบัติที่ดี (Best Practices)

- **แยก Authentication และ Authorization ออกจากกันเสมอ** ทั้งในการออกแบบโค้ดและการคิดเรื่องความปลอดภัย — อย่าสมมติว่า "ล็อกอินได้แล้วทำอะไรก็ได้"
- ตั้งเวลาหมดอายุ (`exp`) ของ JWT ให้สั้นพอสมควร (เช่น 15 นาทีถึง 1 ชั่วโมง) แล้วใช้ **Refresh Token** แยกต่างหาก (อายุยาวกว่าแต่ใช้ได้แค่ขอ Access Token ใหม่) เพื่อลดความเสียหายถ้า token หลุดไปถึงมือผู้ไม่ประสงค์ดี
- ไม่เก็บข้อมูลที่ละเอียดอ่อน (รหัสผ่าน, เลขบัตรเครดิต) ใน JWT payload เพราะ payload **อ่านได้โดยไม่ต้องมี secret key** (แค่ encode ไม่ได้เข้ารหัสลับ) เก็บได้แค่ข้อมูลที่เปิดเผยได้ (user_id, role, email)

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- คิดว่า Authentication ผ่านแล้วเท่ากับมีสิทธิ์ทำทุกอย่าง (ลืมทำ Authorization แยกต่างหาก) เปิดช่องให้ผู้ใช้ทั่วไปทำสิ่งที่ควรสงวนไว้สำหรับ admin เท่านั้น
- เก็บข้อมูลละเอียดอ่อนไว้ใน JWT payload โดยคิดว่า "เข้ารหัสแล้วปลอดภัย" ทั้งที่ payload แค่ถูก encode (base64) ไม่ได้เข้ารหัสลับ ใครก็ตามที่มี token อ่านเนื้อหาได้ทันที
- ไม่ตั้งเวลาหมดอายุของ JWT (หรือตั้งนานเกินไป) ทำให้ token ที่หลุดไปมีอายุใช้งานได้นานเกินความจำเป็น เพิ่มความเสี่ยงถ้าถูกขโมย
- ใช้ `SECRET_KEY` ที่คาดเดาง่ายหรือ commit ลง Git โดยไม่ตั้งใจ (เชื่อมโยงกับคำเตือนเรื่องตรวจสอบไฟล์ก่อน commit ในหลักการทำงานทั่วไป) ทำให้ใครก็ตามที่รู้ key นี้สร้าง JWT ปลอมที่มี signature ถูกต้องได้

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง Authentication และ Authorization พร้อมยกตัวอย่างที่แสดงว่าทำไมต้องแยกจากกัน
2. JWT ประกอบด้วยสามส่วนอะไรบ้าง แต่ละส่วนทำหน้าที่อะไร
3. ทำไม JWT ถึงเรียกว่า "Stateless Authentication" ต่างจาก Session-based Authentication อย่างไร
4. อธิบาย OAuth2 Flow ของ "Login with Google" ว่าทำไมแอปที่ขอสิทธิ์ถึงไม่เห็นรหัสผ่านของผู้ใช้เลย
5. ทำไมการเก็บข้อมูลละเอียดอ่อนไว้ใน JWT payload ถึงเป็นความเสี่ยงด้านความปลอดภัย

## แบบฝึกหัด (Practice Exercises)

1. เขียนระบบ Authorization ที่รองรับหลาย role (`admin`, `editor`, `viewer`) พร้อม decorator ที่ตรวจสอบว่า role ต้องอยู่ใน list ที่อนุญาต ไม่ใช่แค่ตรงกับ role เดียว
2. เขียนระบบ Refresh Token: เมื่อ Access Token หมดอายุ ให้ client ใช้ Refresh Token (อายุยาวกว่า) ขอ Access Token ใหม่โดยไม่ต้องล็อกอินซ้ำ
3. ทดลองแก้ไข JWT payload ด้วยมือ (เปลี่ยน role เป็น "admin") โดยไม่มี secret key แล้วพิสูจน์ว่า `jwt.decode` ตรวจจับการปลอมแปลงนี้ได้

## Mini Project

**"ระบบ API ที่มี Authentication และ Authorization ครบวงจร (Secure API with AuthN/AuthZ)"**: เขียนโปรแกรมที่:
1. สร้างระบบล็อกอินที่ออก JWT พร้อม role (`admin`, `manager`, `employee`) และ Refresh Token แยกต่างหาก
2. สร้าง endpoint จำลองหลายตัวที่ต้องการสิทธิ์ต่างกัน (เช่น `/reports` ต้องเป็น `manager` ขึ้นไป, `/admin/settings` ต้องเป็น `admin` เท่านั้น) โดยใช้ decorator `require_role` ที่รองรับหลาย role
3. Implement ระบบ Token Refresh ที่ปลอดภัย (ตรวจสอบว่า Refresh Token ยังไม่หมดอายุและยังไม่ถูกเพิกถอน)
4. เขียนเทสต์ (ทบทวนจากบทที่ 51) ที่ครอบคลุมทุกกรณี: ล็อกอินสำเร็จ/ล้มเหลว, เข้าถึง endpoint ด้วย role ที่ถูกต้อง/ไม่ถูกต้อง, token หมดอายุ, token ถูกปลอมแปลง

## ข้อคิดสำคัญ (Key Takeaways)

- Authentication ตอบคำถาม "คุณคือใคร" ส่วน Authorization ตอบคำถาม "คุณทำสิ่งนี้ได้ไหม" — สองคำถามที่ต้องแยกจากกันชัดเจนเสมอในการออกแบบระบบความปลอดภัย
- JWT เข้ารหัสข้อมูลผู้ใช้พร้อมลายเซ็นดิจิทัลไว้ในตัวเอง ทำให้ server ตรวจสอบได้แบบ Stateless โดยไม่ต้องถามฐานข้อมูลทุกครั้ง เหมาะกับระบบ Microservices ที่กระจายหลาย server
- OAuth2 ให้แอปหนึ่งเข้าถึงข้อมูลของผู้ใช้บนอีกระบบได้โดยไม่ต้องรู้รหัสผ่านจริง ผ่านการออก Access Token ที่มีขอบเขตจำกัดและหมดอายุได้
- JWT payload อ่านได้โดยไม่ต้องมี secret key (แค่ encode ไม่ใช่เข้ารหัสลับ) จึงห้ามเก็บข้อมูลละเอียดอ่อนไว้ในนั้น
- Decorator (HOF จากบทที่ 35) เป็นเครื่องมือที่เหมาะมากในการ implement การตรวจสอบ Authentication/Authorization แยกจาก business logic หลัก
- บทถัดไปจะเรียน Encryption, Hashing ซึ่งเป็นรากฐานทางคณิตศาสตร์ที่ทำให้ JWT signature และการเก็บรหัสผ่านอย่างปลอดภัย (ที่กล่าวถึงในบทนี้) เป็นไปได้จริง

---
*บทต่อไป (บทที่ 53): Encryption, Hashing — พิมพ์ "Next" เพื่อดำเนินการต่อ*
