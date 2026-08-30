# บทที่ 43: REST, GraphQL, WebSocket, gRPC

## แนวคิดหลัก (Concept)

บทที่แล้วสอน HTTP ในระดับโปรโตคอลพื้นฐาน (request/response, method, status code) แต่ยังไม่ได้ตอบว่า "จะออกแบบ API ให้เป็นระบบอย่างไร" บทนี้เป็นบทปิดท้าย **Part 10: Networking** โดยแนะนำ 4 สถาปัตยกรรม/โปรโตคอลที่สร้างขึ้นบน HTTP (และ TCP โดยตรงในบางกรณี) เพื่อออกแบบการสื่อสารระหว่างระบบให้เหมาะกับสถานการณ์ต่างๆ

- **REST (Representational State Transfer)** — สถาปัตยกรรม API ที่มองทุกอย่างเป็น **Resource** (ทรัพยากร เช่น ผู้ใช้, สินค้า) เข้าถึงผ่าน URL ที่มีโครงสร้างชัดเจน และใช้ HTTP Method (GET/POST/PUT/DELETE จากบทที่แล้ว) สื่อความหมายของการกระทำ
- **GraphQL** — ภาษา query สำหรับ API ที่ให้ **client เลือกได้เองว่าต้องการข้อมูลฟิลด์ไหนบ้าง** ในคำขอเดียว แก้ปัญหาที่ REST มักเจอ (ได้ข้อมูลเกินความจำเป็นหรือต้องยิงหลายคำขอ)
- **WebSocket** — โปรโตคอลที่เปิด**การเชื่อมต่อสองทางค้างไว้ (persistent connection)** ระหว่าง client กับ server ทำให้ทั้งสองฝั่งส่งข้อมูลหากันได้ทันทีโดยไม่ต้องเปิดการเชื่อมต่อใหม่ทุกครั้ง (ต่างจาก HTTP ปกติที่แต่ละ request เปิด-ปิดการเชื่อมต่อของตัวเอง)
- **gRPC** — เฟรมเวิร์ก RPC (Remote Procedure Call) ที่ให้เรียกฟังก์ชันบนเครื่องอื่นได้เหมือนเรียกฟังก์ชันในเครื่องตัวเอง ใช้ **Protocol Buffers** (รูปแบบข้อมูลแบบไบนารีที่กระชับกว่า JSON มาก) วิ่งบน HTTP/2

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **REST มีไว้เป็นมาตรฐานที่เข้าใจง่ายและใช้ประโยชน์จาก HTTP เต็มที่**: ก่อนมี REST นักพัฒนาต้องคิดค้นรูปแบบ API ของตัวเองไม่มีมาตรฐานกลาง ทำให้แต่ละระบบสื่อสารกันยาก REST ใช้แนวคิดที่คุ้นเคยอยู่แล้ว (URL, HTTP Method, Status Code) มาจัดระเบียบ API ให้คาดเดาได้ (เช่น `GET /users/5` ย่อมหมายถึง "ขอข้อมูลผู้ใช้คนที่ 5" โดยสัญชาตญาณ)
- **GraphQL มีไว้แก้ปัญหา Over-fetching และ Under-fetching ของ REST**: ถ้า endpoint REST คืนข้อมูลผู้ใช้มาทั้งหมด 20 ฟิลด์ แต่ mobile app ต้องการแค่ `name` กับ `avatar` การได้ข้อมูลเกินความจำเป็น (Over-fetching) สิ้นเปลือง bandwidth — หรือถ้าต้องยิง 3 endpoint แยกกันเพื่อรวมข้อมูลที่ต้องการ (Under-fetching) ก็ช้าเพราะต้องรอ round-trip หลายครั้ง — GraphQL ให้ client ระบุเองว่าต้องการฟิลด์ไหนในคำขอเดียว แก้ปัญหาทั้งสองแบบพร้อมกัน
- **WebSocket มีไว้สำหรับข้อมูลที่ต้องอัปเดตแบบเรียลไทม์**: HTTP ปกติเป็นแบบ "client ถามก่อนเสมอ server ถึงจะตอบได้" (request-response) ทำให้ไม่เหมาะกับสถานการณ์ที่ server ต้อง**ส่งข้อมูลใหม่ให้ client ทันทีที่มีการเปลี่ยนแปลง** (เช่น ข้อความแชทใหม่, ราคาหุ้นที่เปลี่ยนตลอดเวลา) — ถ้าไม่มี WebSocket ต้องใช้เทคนิค polling (ถามซ้ำๆ ทุกไม่กี่วินาที) ซึ่งสิ้นเปลืองทรัพยากรมาก
- **gRPC มีไว้สำหรับการสื่อสารระหว่างเซิร์ฟเวอร์กับเซิร์ฟเวอร์ (service-to-service) ที่ต้องการความเร็วสูงสุด**: ในสถาปัตยกรรม Microservices (จะเรียนในบทถัดๆ ไป) เซิร์ฟเวอร์หลายตัวต้องคุยกันบ่อยมาก JSON ของ REST อ่านง่ายแต่ขนาดใหญ่และแปลงข้อมูล (parse) ช้ากว่า Protocol Buffers ของ gRPC มาก — เมื่อความเร็วสำคัญกว่าความสามารถอ่านง่ายด้วยตา gRPC จึงเหมาะกว่า

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **REST** เหมือน **เมนูอาหารตามสั่งที่มีจานสำเร็จรูปตายตัว** — สั่ง "ข้าวผัดหมู" ได้ข้าวผัดหมูมาทั้งจาน จะเอาแค่ข้าวไม่เอาหมูไม่ได้ ต้องสั่งเมนูอื่นแยกถ้าต้องการแค่บางส่วน
- **GraphQL** เหมือน **บุฟเฟ่ต์ที่ตักได้เฉพาะสิ่งที่ต้องการ** — บอกครัวว่าต้องการแค่ข้าวกับผักจากจาน "ข้าวผัดหมู" (ไม่เอาหมู) ได้ในคำขอเดียว ไม่ต้องสั่งใหม่หรือทิ้งส่วนเกิน
- **WebSocket** เหมือน **การคุยโทรศัพท์ที่สายเปิดค้างไว้ตลอด** — ทั้งสองฝั่งพูดคุยโต้ตอบกันได้ทันทีโดยไม่ต้องกดโทรออกใหม่ทุกครั้งที่มีอะไรจะพูด (ต่างจาก HTTP ที่เหมือนการส่งจดหมายทีละฉบับ — ต้องส่งใหม่ทุกครั้ง)
- **gRPC** เหมือน **การพูดคุยกับเพื่อนร่วมงานในห้องเดียวกันด้วยภาษามือรหัสเฉพาะที่รวดเร็ว** — เข้าใจกันเร็วมากเพราะทั้งสองฝ่ายรู้รหัสเดียวกันล่วงหน้า (Protocol Buffers ที่นิยามไว้ชัดเจน) ต่างจากการเขียนจดหมายอธิบายยาวๆ (JSON) ที่ชัดเจนกว่าแต่ช้ากว่า

## แผนภาพอธิบาย (Visual explanation)

### REST — URL และ Method สื่อความหมายชัดเจน

```
GET    /users           -> ดึงรายชื่อผู้ใช้ทั้งหมด
GET    /users/5         -> ดึงข้อมูลผู้ใช้คนที่ 5
POST   /users           -> สร้างผู้ใช้ใหม่ (ข้อมูลอยู่ใน request body)
PUT    /users/5         -> อัปเดตผู้ใช้คนที่ 5 ทั้งหมด
DELETE /users/5         -> ลบผู้ใช้คนที่ 5

Response ของ GET /users/5 (Over-fetching — ได้ทุกฟิลด์แม้ไม่ต้องการหมด):
{
  "id": 5, "name": "Alice", "email": "...", "address": "...",
  "phone": "...", "created_at": "...", "updated_at": "..." (อีก 15 ฟิลด์)
}
```

### GraphQL — Client เลือกฟิลด์ที่ต้องการเองในคำขอเดียว

```
Query ที่ client ส่งไป:                        Response ที่ได้กลับมา (ตรงตามที่ขอเป๊ะ):

query {                                        {
  user(id: 5) {                                  "user": {
    name                                            "name": "Alice",
    avatar                                          "avatar": "https://..."
  }                                               }
}                                               }

ไม่มี email, address, phone ฯลฯ ที่ไม่ได้ขอเลย -> แก้ Over-fetching
ถ้าต้องการข้อมูลจาก resource อื่นด้วย (เช่น posts ของ user) ขอในคำขอเดียวกันได้เลย -> แก้ Under-fetching
```

### HTTP Request-Response เทียบกับ WebSocket

```
HTTP (แบบ REST/GraphQL):                    WebSocket:

Client -> Request 1 -> Server                Client <--- Handshake ---> Server
Client <- Response 1 <- Server                        (เชื่อมต่อค้างไว้)
(ปิดการเชื่อมต่อ)                             Client ---> "Hello" ------> Server
Client -> Request 2 -> Server                Client <--- "Hi back" <---- Server
Client <- Response 2 <- Server                Client <--- "New msg!" <-- Server   (Server ส่งเองได้โดยไม่ต้องรอถาม!)
(เปิด-ปิดการเชื่อมต่อใหม่ทุกครั้ง)              (สื่อสารสองทางตลอดเวลาผ่านการเชื่อมต่อเดียว)
```

### gRPC — เรียกฟังก์ชันข้ามเครื่องเหมือนเรียกในเครื่องตัวเอง

```
.proto file (นิยาม service ล่วงหน้า):

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}

Client Code (ดูเหมือนเรียกฟังก์ชันธรรมดา):        เบื้องหลังจริง:
response = client.GetUser(request)              -> แปลงเป็น Protocol Buffers (ไบนารี, กระชับ)
                                                 -> ส่งผ่าน HTTP/2
                                                 -> Server แปลงกลับ ประมวลผล ส่งกลับ
                                                 -> Client ได้ response เหมือนเรียกฟังก์ชัน local
```

## ตัวอย่างโค้ด (Code example)

```python
import requests

# ===== REST API — เรียกตาม HTTP Method มาตรฐาน =====
response = requests.get("https://api.example.com/users/5")
user = response.json()                             # ได้ข้อมูลผู้ใช้ทุกฟิลด์ (Over-fetching)

requests.post("https://api.example.com/users", json={"name": "Alice", "email": "a@mail.com"})
requests.put("https://api.example.com/users/5", json={"name": "Alice Updated"})
requests.delete("https://api.example.com/users/5")

# ===== GraphQL — ส่ง query เดียวระบุฟิลด์ที่ต้องการ =====
graphql_query = """
query {
  user(id: 5) {
    name
    avatar
    posts {
      title
    }
  }
}
"""
response = requests.post(
    "https://api.example.com/graphql",
    json={"query": graphql_query}                     # GraphQL ใช้ endpoint เดียวเสมอ ต่างจาก REST หลาย URL
)
data = response.json()["data"]
print(data["user"]["name"], data["user"]["posts"])      # ได้เฉพาะฟิลด์ที่ขอ ไม่มี email/address ปนมา

# ===== WebSocket — เชื่อมต่อค้างไว้ สื่อสารสองทาง =====
import websocket

def on_message(ws, message):                          # ทำงานทุกครั้งที่ server ส่งข้อมูลมาโดยไม่ต้องถาม
    print("Received:", message)

def on_open(ws):
    ws.send("Hello from client!")                       # ส่งข้อความแรกหลังเชื่อมต่อสำเร็จ

ws = websocket.WebSocketApp(
    "wss://echo.websocket.org",
    on_open=on_open,
    on_message=on_message
)
ws.run_forever()                                        # การเชื่อมต่อค้างไว้ตลอดจนกว่าจะปิดเอง

# ===== gRPC (ตัวอย่างแนวคิด — ต้อง generate code จาก .proto ก่อนใช้งานจริง) =====
"""
# user.proto
service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}
message UserRequest { int32 id = 1; }
message UserResponse { string name = 1; string email = 2; }
"""
# หลัง generate code จาก .proto แล้ว ใช้งานเหมือนเรียกฟังก์ชันธรรมดา:
# channel = grpc.insecure_channel('localhost:50051')
# stub = user_pb2_grpc.UserServiceStub(channel)
# response = stub.GetUser(user_pb2.UserRequest(id=5))    # ดูเหมือนเรียกฟังก์ชัน local ทั้งที่วิ่งข้ามเครื่อง
# print(response.name)
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `requests.get("https://api.example.com/users/5")` — ตาม REST เต็มรูปแบบ URL `/users/5` สื่อถึง **Resource** (ผู้ใช้คนที่ 5) และ Method `GET` สื่อถึง "ขอข้อมูล" — เซิร์ฟเวอร์คืนค่าโครงสร้างข้อมูลที่**กำหนดไว้ล่วงหน้าตายตัว**สำหรับ endpoint นี้เสมอ (ทุกฟิลด์ ไม่ว่า client จะต้องการหมดหรือไม่ก็ตาม)
2. `graphql_query` ที่ส่งผ่าน `requests.post` — สังเกตว่า **GraphQL ใช้ endpoint เดียว** (`/graphql`) เสมอไม่ว่าจะขอข้อมูลอะไรก็ตาม (ต่างจาก REST ที่แต่ละ Resource มี URL ของตัวเอง) รายละเอียดของสิ่งที่ต้องการอยู่ใน **query string** ที่ส่งไปในตัว request body แทน — เซิร์ฟเวอร์ฝั่ง GraphQL จะแปล query นี้แล้วดึงเฉพาะฟิลด์ที่ระบุไว้เท่านั้น (`name`, `avatar`, `posts.title`) มาคืนกลับ
3. `websocket.WebSocketApp(..., on_open=on_open, on_message=on_message)` — `on_open` ทำงานครั้งเดียวทันทีที่การเชื่อมต่อสำเร็จ (คล้าย TCP Handshake ในบทที่แล้ว แต่เป็นระดับ WebSocket) ส่วน `on_message` เป็น **callback ที่ถูกเรียกทุกครั้งที่ server ส่งข้อมูลมาใหม่** โดยที่ client **ไม่ได้ร้องขอ (request) ก่อนเลย** — นี่คือความต่างสำคัญจาก HTTP ที่ server ตอบได้เฉพาะเมื่อมี request เข้ามาเท่านั้น `ws.run_forever()` ทำให้โปรแกรมรอฟังข้อความใหม่ตลอดไปจนกว่าการเชื่อมต่อจะถูกปิด
4. ไฟล์ `.proto` ใน gRPC — นิยาม**โครงสร้างของ service และข้อความ (message) ล่วงหน้าอย่างเข้มงวด** (คล้าย type ใน static typing) จากนั้นเครื่องมือของ gRPC จะ **generate โค้ด client และ server** ให้อัตโนมัติจากไฟล์นี้ — เมื่อเรียก `stub.GetUser(...)` โค้ดที่ generate มาจะแปลงคำขอเป็น **Protocol Buffers** (รูปแบบไบนารีที่กระชับกว่า JSON มาก เพราะไม่ต้องเก็บชื่อฟิลด์เป็นข้อความซ้ำทุกครั้งเหมือน JSON) ส่งผ่าน HTTP/2 ไปยังเซิร์ฟเวอร์ ทำให้เร็วกว่า REST+JSON อย่างมีนัยสำคัญในสถานการณ์ที่เรียกบ่อยและข้อมูลมีโครงสร้างซับซ้อน

## Time Complexity

REST/GraphQL/WebSocket/gRPC เป็นเรื่องของ**สถาปัตยกรรมการสื่อสาร** ไม่ใช่อัลกอริทึม จึงไม่มี Big O โดยตรง แต่เปรียบเทียบ**เวลาแฝง (latency) และปริมาณข้อมูล**ได้ดังนี้:

| แนวทาง | จำนวน Round-Trip สำหรับข้อมูลซับซ้อน | ขนาดข้อมูลต่อคำขอ |
|---|---|---|
| REST | อาจต้องหลายครั้ง (Under-fetching) | มักใหญ่กว่าที่จำเป็น (Over-fetching) |
| GraphQL | ครั้งเดียว (รวมทุกอย่างที่ต้องการ) | พอดีตามที่ขอ |
| WebSocket | 1 ครั้งสำหรับเปิดการเชื่อมต่อ แล้วส่งได้ไม่จำกัดโดยไม่มี round-trip ใหม่ | ขึ้นกับข้อความ |
| gRPC | เหมือน REST แต่ overhead ต่อคำขอน้อยกว่ามาก (Protocol Buffers + HTTP/2) | เล็กกว่า JSON อย่างมีนัยสำคัญ |

## Space Complexity

- Protocol Buffers ของ gRPC ใช้พื้นที่**น้อยกว่า JSON** อย่างมีนัยสำคัญสำหรับข้อมูลเดียวกัน เพราะเข้ารหัสเป็นไบนารีที่กระชับแทนข้อความที่มนุษย์อ่านได้
- WebSocket ที่เปิดค้างไว้ใช้หน่วยความจำสำหรับรักษาสถานะการเชื่อมต่อตลอดเวลาที่เปิดอยู่ (ต่างจาก HTTP ที่ปิดการเชื่อมต่อทันทีหลัง response)

## แนวทางปฏิบัติที่ดี (Best Practices)

- เลือก REST เป็นค่าเริ่มต้นสำหรับ API สาธารณะทั่วไป เพราะเข้าใจง่าย มีเครื่องมือรองรับเยอะ (caching ของ HTTP ทำงานได้ดีกับ REST) และนักพัฒนาส่วนใหญ่คุ้นเคย
- เลือก GraphQL เมื่อ client มีความหลากหลายสูง (เว็บ, มือถือ, IoT) ที่ต้องการข้อมูลต่างกันมากจาก resource เดียวกัน หรือเมื่อข้อมูลที่ต้องแสดงผลเชื่อมโยงกันซับซ้อนหลาย resource
- เลือก WebSocket เมื่อต้องการข้อมูลแบบเรียลไทม์จริงๆ (แชท, การแจ้งเตือนสด, เกมออนไลน์) ไม่ใช่ใช้ WebSocket กับทุกอย่างเพราะดูทันสมัย — ถ้าข้อมูลไม่ได้เปลี่ยนบ่อยจริง REST/GraphQL แบบ request-response ยังคงง่ายกว่าและเพียงพอ
- เลือก gRPC สำหรับการสื่อสารระหว่าง microservice ภายในองค์กรเดียวกันที่ต้องการความเร็วสูงสุด แต่ไม่เหมาะกับ API สาธารณะที่ client เป็นเบราว์เซอร์ทั่วไป (เพราะเบราว์เซอร์รองรับ gRPC ได้จำกัดกว่า REST/GraphQL)

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ใช้ GraphQL ทั้งที่ API มีความต้องการเรียบง่ายและ client มีแค่ประเภทเดียว ทำให้เพิ่มความซับซ้อนโดยไม่ได้ประโยชน์คุ้มค่า (over-engineering เหมือนที่เตือนไว้ในบทที่ 32-33)
- ใช้ WebSocket สำหรับข้อมูลที่ไม่จำเป็นต้องเรียลไทม์จริงๆ ทำให้ต้องดูแลการเชื่อมต่อค้างไว้จำนวนมากโดยไม่จำเป็น สิ้นเปลืองทรัพยากรเซิร์ฟเวอร์
- ออกแบบ REST API ที่ไม่ยึดหลัก Resource-based URL อย่างสม่ำเสมอ (เช่น `/getUser?id=5` แทนที่จะเป็น `/users/5`) ทำให้เสียประโยชน์ของความคาดเดาได้ที่ REST ควรมี
- พยายามใช้ gRPC เรียกจากเบราว์เซอร์โดยตรงโดยไม่รู้ว่ามีข้อจำกัดทางเทคนิค (ต้องผ่าน gRPC-Web หรือ proxy พิเศษ) ต่างจาก REST/GraphQL ที่เบราว์เซอร์เรียกได้ตรงๆ

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบาย Over-fetching และ Under-fetching ใน REST API คืออะไร GraphQL แก้ปัญหานี้อย่างไร
2. WebSocket ต่างจาก HTTP request-response ธรรมดาอย่างไร ยกตัวอย่างสถานการณ์ที่ต้องใช้ WebSocket
3. gRPC ใช้ Protocol Buffers แทน JSON เพราะเหตุผลอะไร มีข้อเสียอะไรบ้างเมื่อเทียบกับ JSON
4. ถ้าต้องออกแบบ API สำหรับแอปแชท ควรเลือกใช้ REST, GraphQL, หรือ WebSocket? อธิบายเหตุผล
5. ทำไม gRPC ถึงไม่เหมาะกับการเรียกจากเบราว์เซอร์โดยตรงเท่า REST/GraphQL

## แบบฝึกหัด (Practice Exercises)

1. ออกแบบ REST API สำหรับระบบจัดการห้องสมุด (books, members, loans) ให้ครบทุก Method (GET/POST/PUT/DELETE) พร้อม URL ที่ถูกต้องตามหลัก Resource-based
2. เขียน GraphQL schema (แนวคิด ไม่ต้องรันจริง) สำหรับข้อมูลเดียวกัน (books, members, loans) ที่ให้ client ขอข้อมูลหนังสือพร้อมชื่อสมาชิกที่ยืมอยู่ในคำขอเดียว
3. เขียนโปรแกรมทดสอบ WebSocket client ง่ายๆ ที่เชื่อมต่อไปยัง WebSocket echo server สาธารณะ ส่งข้อความและรับข้อความกลับมาแสดงผล

## Mini Project

**"ระบบแชทเรียลไทม์พร้อม REST API จัดการประวัติ (Real-time Chat with REST History API)"**: เขียนโปรแกรมที่:
1. ใช้ WebSocket สำหรับส่งและรับข้อความแชทแบบเรียลไทม์ระหว่างผู้ใช้หลายคน (broadcast ข้อความใหม่ไปยังทุกคนที่เชื่อมต่ออยู่ทันที)
2. ใช้ REST API แยกต่างหากสำหรับดึงประวัติข้อความเก่า (`GET /rooms/{id}/messages`) ที่ไม่จำเป็นต้องเรียลไทม์
3. เปรียบเทียบว่าถ้าใช้แค่ REST + polling (ถามซ้ำทุก 2 วินาที) แทน WebSocket จะมี delay และภาระเซิร์ฟเวอร์ต่างกันอย่างไร โดยวัดจำนวน request ทั้งหมดที่เกิดขึ้นในทั้งสองแนวทางเมื่อรัน 5 นาที
4. เขียนรายงานสรุปว่าทำไมการผสมทั้ง WebSocket (สำหรับเรียลไทม์) และ REST (สำหรับข้อมูลที่ไม่เปลี่ยนบ่อย) ในระบบเดียวกันถึงเป็นแนวทางที่นิยมใช้จริงในระบบแชทสมัยใหม่

## ข้อคิดสำคัญ (Key Takeaways)

- REST ใช้ URL แทน Resource และ HTTP Method แทนการกระทำ เข้าใจง่ายและเป็นมาตรฐานที่นิยมที่สุด แต่เสี่ยง Over-fetching/Under-fetching
- GraphQL ให้ client เลือกฟิลด์ที่ต้องการเองในคำขอเดียวผ่าน endpoint เดียว แก้ปัญหา Over-fetching/Under-fetching ของ REST แต่เพิ่มความซับซ้อนของระบบ
- WebSocket เปิดการเชื่อมต่อสองทางค้างไว้ ทำให้ server ส่งข้อมูลหา client ได้ทันทีโดยไม่ต้องรอถาม เหมาะกับข้อมูลเรียลไทม์
- gRPC ใช้ Protocol Buffers (ไบนารี กระชับ) วิ่งบน HTTP/2 เหมาะกับการสื่อสารระหว่าง microservice ที่ต้องการความเร็วสูง แต่ไม่เหมาะกับเบราว์เซอร์โดยตรง
- ไม่มีสถาปัตยกรรมไหนที่ "ดีที่สุดเสมอ" — ต้องเลือกให้เหมาะกับสถานการณ์จริง (ผู้ใช้เป็นใคร, ต้องการเรียลไทม์ไหม, ประสิทธิภาพสำคัญแค่ไหน) เหมือนหลักการเลือกอัลกอริทึม/โครงสร้างข้อมูลที่เรียนมาตลอดหลักสูตร
- นี่คือบทปิดท้าย **Part 10: Networking** — บทถัดไปจะเข้าสู่ **Part 11: Operating Systems** ซึ่งเจาะลึกกลไกภายในของระบบปฏิบัติการที่รองรับทุกสิ่งที่เรียนมา (Process, Thread, Memory) โดยเริ่มจาก File System และ Scheduling

---
*บทต่อไป (บทที่ 44): File System, Scheduling — พิมพ์ "Next" เพื่อดำเนินการต่อ*
