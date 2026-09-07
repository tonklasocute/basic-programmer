# บทที่ 58: Docker, Kubernetes

## แนวคิดหลัก (Concept)

ตลอดหลักสูตรเราเขียนโค้ดและออกแบบระบบที่ซับซ้อนขึ้นเรื่อยๆ แต่ยังไม่ได้ตอบคำถามพื้นฐานที่สุดข้อหนึ่ง: **จะเอาโค้ดที่เขียนเสร็จแล้วไปรันบนเครื่องอื่น (เซิร์ฟเวอร์จริง) ให้ทำงานเหมือนกับที่รันบนเครื่องพัฒนาได้อย่างไร** บทนี้เปิด **Part 17: DevOps** ด้วยสองเครื่องมือที่เป็นมาตรฐานของวงการในการแก้ปัญหานี้ — **Docker** (บรรจุโปรแกรมให้พกพาไปรันที่ไหนก็ได้เหมือนกันเป๊ะ) และ **Kubernetes** (จัดการ container จำนวนมากให้ทำงานเป็นระบบเดียวกันโดยอัตโนมัติ)

- **Container (คอนเทนเนอร์)** — หน่วยบรรจุโปรแกรมที่รวมโค้ด, dependency (library ทุกตัวที่ต้องการ), และการตั้งค่าทั้งหมดไว้ในแพ็กเกจเดียว ทำให้รันได้เหมือนกันทุกที่ไม่ว่าเครื่องปลายทางจะติดตั้งอะไรไว้ก่อนหน้านี้
- **Docker Image** — พิมพ์เขียว (blueprint) ที่ใช้สร้าง container นิยามด้วยไฟล์ `Dockerfile` ที่บอกขั้นตอนการติดตั้งทีละขั้น
- **Docker vs Virtual Machine** — Container**แชร์ OS Kernel เดียวกัน**กับเครื่อง host (เบากว่า เร็วกว่า) ต่างจาก Virtual Machine ที่จำลอง**ทั้งระบบปฏิบัติการ**แยกต่างหาก (หนักกว่ามาก)
- **Kubernetes (K8s)** — ระบบจัดการ (orchestration) container จำนวนมากโดยอัตโนมัติ: เริ่ม container ใหม่เมื่อพัง, กระจาย container ไปหลายเครื่อง, ปรับจำนวน container ตามโหลด (Auto-scaling — เชื่อมโยงกับ Horizontal Scaling บทที่ 55)

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Container มีไว้แก้ปัญหาคลาสสิก "It works on my machine"**: โปรแกรมที่รันได้ปกติบนเครื่องพัฒนา อาจพังบนเซิร์ฟเวอร์จริงเพราะเวอร์ชันของ library ไม่ตรงกัน, ตัวแปรสภาพแวดล้อมต่างกัน, หรือ OS ต่างกัน — Container บรรจุทุกอย่างที่โปรแกรมต้องการไว้ในแพ็กเกจเดียวกัน ทำให้ "รันที่ไหนก็เหมือนกัน" จริงๆ ไม่ว่าจะเป็นเครื่องนักพัฒนา, staging server, หรือ production
- **Docker เบากว่า Virtual Machine เพราะไม่ต้องจำลอง OS ทั้งระบบ**: Virtual Machine ต้องรัน Guest OS เต็มรูปแบบ (ใช้ RAM/CPU มาก, เริ่มต้นช้า) แต่ Container แชร์ Kernel ของ host OS จึงเบากว่ามาก (ใช้ RAM น้อยกว่ามาก, เริ่มต้นเร็วกว่ามาก — เป็นเหตุผลว่าทำไมสร้าง container ใหม่ได้เร็วเป็นวินาทีในขณะที่ VM ใช้เวลาเป็นนาที)
- **Kubernetes มีไว้จัดการ container จำนวนมากที่ไม่สามารถดูแลด้วยมือได้อีกต่อไป**: เมื่อระบบเป็น Microservices (บทที่ 49) ที่มี container นับสิบนับร้อยตัวกระจายอยู่หลายเครื่อง การเริ่ม/หยุด/ตรวจสอบสุขภาพแต่ละตัวด้วยมือเป็นไปไม่ได้ในทางปฏิบัติ — Kubernetes ทำหน้าที่นี้อัตโนมัติ: ถ้า container ตัวหนึ่งล่ม Kubernetes เริ่มตัวใหม่ทันที (Self-healing), ถ้าโหลดสูงขึ้น Kubernetes เพิ่มจำนวน container อัตโนมัติ (Auto-scaling)

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Container** เหมือน **ตู้คอนเทนเนอร์ขนส่งสินค้าทางเรือมาตรฐาน** — ไม่ว่าข้างในจะเป็นเสื้อผ้า อาหาร หรือเครื่องจักร ตู้คอนเทนเนอร์มีขนาดมาตรฐานเดียวกัน ทำให้เรือ, รถบรรทุก, และเครนที่ท่าเรือทุกแห่งจัดการมันได้เหมือนกันหมด โดยไม่ต้องสนใจว่าข้างในบรรจุอะไร
- **Docker Image vs Container** เหมือน **พิมพ์เขียวบ้าน (Image) เทียบกับบ้านที่สร้างขึ้นจริง (Container)** — พิมพ์เขียวเดียวสร้างบ้านได้หลายหลัง (สร้าง container หลายตัวจาก image เดียว) แต่ละหลังทำงานเป็นอิสระจากกัน
- **Container vs Virtual Machine** เหมือน **ห้องเช่าในอพาร์ตเมนต์ (Container ที่แชร์โครงสร้างตึกเดียวกัน) เทียบกับบ้านเดี่ยวแยกทั้งหลัง (VM ที่มีระบบสาธารณูปโภคของตัวเองครบชุด)** — ห้องเช่าสร้างเร็วกว่า ประหยัดกว่า เพราะแชร์โครงสร้างพื้นฐานร่วมกัน
- **Kubernetes** เหมือน **ผู้จัดการโรงงานที่คอยตรวจสอบสายการผลิตทุกสาย** — ถ้าเครื่องจักรตัวหนึ่งเสีย (container ล่ม) ผู้จัดการสั่งเปิดเครื่องสำรองทันทีโดยอัตโนมัติ ถ้าออเดอร์เยอะขึ้น ผู้จัดการสั่งเปิดสายการผลิตเพิ่มเอง โดยเจ้าของโรงงานไม่ต้องมาคอยสั่งการทีละขั้นตอนเอง

## แผนภาพอธิบาย (Visual explanation)

### Container vs Virtual Machine — สถาปัตยกรรมที่ต่างกัน

```
Virtual Machine:                          Container:

┌─────────┐┌─────────┐┌─────────┐         ┌─────────┐┌─────────┐┌─────────┐
│  App A  ││  App B  ││  App C  │         │  App A  ││  App B  ││  App C  │
├─────────┤├─────────┤├─────────┤         ├─────────┤├─────────┤├─────────┤
│Guest OS ││Guest OS ││Guest OS │         │  (แยกกันแค่ process/filesystem) │
├─────────┴┴─────────┴┴─────────┤         ├─────────────────────────────────┤
│         Hypervisor              │         │      Docker Engine                │
├─────────────────────────────────┤         ├─────────────────────────────────┤
│         Host OS                 │         │         Host OS (Kernel เดียวกัน)  │
├─────────────────────────────────┤         ├─────────────────────────────────┤
│         Hardware                │         │         Hardware                  │
└─────────────────────────────────┘         └─────────────────────────────────┘

แต่ละ VM มี Guest OS เต็มรูปแบบ                Container แชร์ Host OS Kernel เดียวกัน
(หนัก, เริ่มต้นช้า, ใช้ RAM มาก)                (เบา, เริ่มต้นเร็ว, ใช้ RAM น้อยกว่ามาก)
```

### Dockerfile — สูตรสร้าง Image ทีละขั้นตอน

```
FROM python:3.11-slim              <- เริ่มจาก base image ที่มี Python ติดตั้งไว้แล้ว
WORKDIR /app                        <- กำหนดโฟลเดอร์ทำงานภายใน container
COPY requirements.txt .             <- คัดลอกไฟล์ dependency เข้ามาก่อน
RUN pip install -r requirements.txt  <- ติดตั้ง dependency (layer นี้ cache ได้ถ้าไม่เปลี่ยน)
COPY . .                             <- คัดลอกโค้ดทั้งหมดเข้ามา (แยกจาก dependency เพื่อ cache ดีกว่า)
CMD ["python", "app.py"]             <- คำสั่งที่รันเมื่อ container เริ่มทำงาน

docker build -t myapp .    <- สร้าง Image จาก Dockerfile นี้
docker run myapp             <- สร้างและรัน Container จาก Image
```

### Kubernetes Architecture — จัดการ Container จำนวนมากอัตโนมัติ

```
                    ┌─────────────────────┐
                    │   Kubernetes Control  │   <- ตัดสินใจว่า Pod ควรอยู่เครื่องไหน, กี่ตัว
                    │   Plane (Master)      │
                    └──────────┬──────────┘
                               │ สั่งการ
          ┌────────────────────┼────────────────────┐
          v                    v                     v
    ┌──────────┐        ┌──────────┐          ┌──────────┐
    │  Node 1  │        │  Node 2  │          │  Node 3  │    (แต่ละ Node = เครื่องจริง 1 เครื่อง)
    │ [Pod A]  │        │ [Pod A]  │          │ [Pod B]  │
    │ [Pod B]  │        │ [Pod C]  │          │ [Pod C]  │
    └──────────┘        └──────────┘          └──────────┘

Pod A มี 2 instance (Node 1, Node 2) -> ถ้า Node 1 ล่ม ยังมี Pod A ที่ Node 2 ทำงานต่อได้
Kubernetes ตรวจ Health Check ตลอดเวลา -> Pod ไหนล่ม สร้างตัวใหม่ทดแทนทันที (Self-healing)
```

### Auto-scaling — Kubernetes ปรับจำนวน Pod ตามโหลดอัตโนมัติ

```
โหลดต่ำ (10:00):          [Pod][Pod]                    2 instance พอ

โหลดสูงขึ้น (12:00):        [Pod][Pod][Pod][Pod][Pod]      Kubernetes เห็น CPU usage สูง
                                                          -> เพิ่ม Pod อัตโนมัติเป็น 5 ตัว

โหลดลดลง (15:00):          [Pod][Pod]                    Kubernetes ลด Pod กลับเหลือ 2 ตัว
                                                          (ประหยัดทรัพยากรเมื่อไม่จำเป็น)
```

## ตัวอย่างโค้ด (Code example)

```dockerfile
# ===== Dockerfile: นิยามวิธีสร้าง Image ของแอปพลิเคชัน Python =====
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000                                          # ประกาศว่า container นี้เปิด port 8000
CMD ["python", "app.py"]
```

```yaml
# ===== docker-compose.yml: รันหลาย container พร้อมกัน (app + database) =====
version: '3'
services:
  web:
    build: .
    ports:
      - "8000:8000"                                    # map port host:container
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb

  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - db_data:/var/lib/postgresql/data                # persist ข้อมูลแม้ container restart

volumes:
  db_data:
```

```yaml
# ===== Kubernetes Deployment: กำหนดว่าต้องการ Pod กี่ตัวและใช้ Image ไหน =====
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3                                          # ต้องการ 3 instance เสมอ
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 8000
          resources:
            limits:
              cpu: "0.5"
              memory: "512Mi"
---
# ===== HorizontalPodAutoscaler: ปรับจำนวน Pod อัตโนมัติตาม CPU usage =====
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70                    # ถ้า CPU เฉลี่ยเกิน 70% -> เพิ่ม Pod อัตโนมัติ
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `Dockerfile` — แต่ละบรรทัดสร้าง **layer** ใหม่ซ้อนกันขึ้นไป Docker **cache แต่ละ layer แยกกัน**: ถ้าแก้แค่โค้ด (`COPY . .`) แต่ `requirements.txt` ไม่เปลี่ยน การ build ครั้งถัดไปจะข้าม `RUN pip install ...` ไปเลย (ใช้ layer cache เดิม) ทำให้ build เร็วขึ้นมาก — นี่คือเหตุผลที่แนะนำให้ `COPY requirements.txt` **ก่อน** `COPY . .` เสมอ (แยก dependency ออกจากโค้ดที่เปลี่ยนบ่อยกว่า)
2. `docker-compose.yml` — `depends_on: - db` บอกว่า service `web` ต้องรอให้ `db` เริ่มทำงานก่อน (แม้จะไม่รับประกันว่า database "พร้อมรับ connection" จริง แค่ container เริ่มทำงานแล้ว) — `volumes: - db_data:/var/lib/postgresql/data` ทำให้ข้อมูลของฐานข้อมูล**ไม่หายไปแม้ container จะถูกลบและสร้างใหม่** เพราะข้อมูลจริงถูกเก็บไว้ที่ **Volume** ซึ่งแยกอิสระจาก container (container คือของที่สร้างและทำลายได้ตลอดเวลา แต่ข้อมูลสำคัญต้องคงอยู่)
3. `replicas: 3` ใน Kubernetes Deployment — บอก Kubernetes ว่า**ต้องการให้มี Pod ของแอปนี้ทำงานอยู่ 3 ตัวเสมอ** ไม่ว่าจะเกิดอะไรขึ้น ถ้า Pod ตัวหนึ่งล่ม (เช่น เครื่องที่ Pod นั้นรันอยู่มีปัญหา) Kubernetes Control Plane จะตรวจพบทันทีและ**สร้าง Pod ใหม่ทดแทนโดยอัตโนมัติ** เพื่อให้จำนวนกลับมาเป็น 3 เสมอ (Self-healing ตามที่อธิบายในแผนภาพ)
4. `HorizontalPodAutoscaler` — ตรวจสอบ `targetCPUUtilizationPercentage: 70` **ตลอดเวลา** ถ้า CPU เฉลี่ยของทุก Pod เกิน 70% Kubernetes จะเพิ่มจำนวน Pod อัตโนมัติ (ไม่เกิน `maxReplicas: 10`) เพื่อกระจายภาระ — เมื่อโหลดลดลง Kubernetes จะลด Pod กลับลงมา (ไม่ต่ำกว่า `minReplicas: 2`) เพื่อประหยัดทรัพยากร — นี่คือ **Horizontal Scaling (บทที่ 55) ที่ทำงานอัตโนมัติเต็มรูปแบบโดยไม่ต้องมีมนุษย์มาคอยปรับเอง**

## Time Complexity

Docker/Kubernetes เป็นเครื่องมือด้าน infrastructure ไม่ใช่อัลกอริทึม แต่มีผลกระทบด้านเวลาที่วัดได้ชัดเจน:
- การเริ่ม Container ใหม่: **วินาที** (เพราะแชร์ Kernel กับ host แล้ว ไม่ต้องบูต OS ใหม่)
- การเริ่ม Virtual Machine ใหม่: **นาที** (ต้องบูต Guest OS เต็มรูปแบบ)
- Kubernetes ตรวจ Health Check และตัดสินใจ scale/restart: เกิดขึ้นเป็นระยะ (มักตั้งค่าได้ เช่น ทุก 10-30 วินาที)

## Space Complexity

- Container ใช้พื้นที่**เฉพาะโค้ดและ dependency ของแอป** (ไม่รวม OS เต็มรูปแบบ) — image ขนาดเล็กมักมีขนาดแค่หลักสิบ MB ถึงร้อย MB
- Virtual Machine ต้องมีพื้นที่สำหรับ Guest OS เต็มรูปแบบ — มักมีขนาดหลาย GB

## แนวทางปฏิบัติที่ดี (Best Practices)

- ใช้ **base image ที่เล็กที่สุดเท่าที่ทำได้** (เช่น `python:3.11-slim` แทน `python:3.11` เต็มรูปแบบ) เพื่อลดขนาด image และพื้นผิวการโจมตี (attack surface — เชื่อมโยงกับความปลอดภัยจาก Part 15)
- จัดลำดับคำสั่งใน Dockerfile ให้สิ่งที่เปลี่ยนบ่อยที่สุด (โค้ด) อยู่**ท้ายสุด** และสิ่งที่เปลี่ยนน้อย (dependency) อยู่**ก่อน** เพื่อใช้ประโยชน์จาก layer cache ให้เต็มที่
- ไม่เก็บข้อมูลสำคัญไว้**ภายใน container โดยตรง** เสมอ ใช้ **Volume** แยกต่างหากสำหรับข้อมูลที่ต้องคงอยู่ถาวร เพราะ container ถูกออกแบบมาให้ "สร้างและทำลายได้ตลอดเวลา" (ephemeral)
- กำหนด `resources.limits` (CPU, memory) ให้ทุก container ใน Kubernetes เสมอ เพื่อป้องกันไม่ให้ container ตัวหนึ่งใช้ทรัพยากรจนกระทบ container อื่นบนเครื่องเดียวกัน

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- เก็บข้อมูลสำคัญ (เช่น ไฟล์ที่ผู้ใช้อัปโหลด) ไว้ในไฟล์ระบบของ container โดยตรง แล้วข้อมูลหายไปเมื่อ container ถูกลบ/สร้างใหม่ (ลืมใช้ Volume)
- จัดลำดับ `COPY` ใน Dockerfile ผิด (`COPY . .` ก่อน `COPY requirements.txt`) ทำให้ทุกครั้งที่แก้โค้ดแม้แต่นิดเดียว ต้องติดตั้ง dependency ใหม่ทั้งหมด (เสียประโยชน์ของ layer cache ไปหมด)
- ไม่กำหนด `resources.limits` ใน Kubernetes ทำให้ container หนึ่งใช้ CPU/memory จนกระทบ Pod อื่นบนเครื่องเดียวกัน (Noisy Neighbor Problem)
- ตั้งค่า `replicas` เพียง 1 ตัวสำหรับบริการสำคัญ ทำให้ไม่มี High Availability เลย (ถ้า Pod ตัวเดียวนั้นล่ม บริการหยุดทำงานทันทีจนกว่า Kubernetes จะสร้างตัวใหม่เสร็จ)

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง Container และ Virtual Machine ทั้งในแง่สถาปัตยกรรมและประสิทธิภาพ
2. Docker Layer Cache ทำงานอย่างไร ทำไมลำดับคำสั่งใน Dockerfile ถึงมีผลต่อความเร็วในการ build
3. Kubernetes แก้ปัญหาอะไรที่ Docker เพียงอย่างเดียวทำไม่ได้
4. อธิบาย Self-healing และ Auto-scaling ใน Kubernetes ทำงานอย่างไร เชื่อมโยงกับ Horizontal Scaling บทที่ 55
5. ทำไมข้อมูลสำคัญถึงไม่ควรเก็บไว้ภายใน container โดยตรง ควรใช้อะไรแทน

## แบบฝึกหัด (Practice Exercises)

1. เขียน Dockerfile สำหรับแอปพลิเคชัน Python ง่ายๆ ที่มี dependency จาก `requirements.txt` แล้ว build และรันด้วย `docker build`/`docker run` จริง
2. เขียน `docker-compose.yml` ที่รันแอปพลิเคชัน 2 ตัวพร้อมกัน (web + Redis cache จากบทที่ 55) แล้วทดสอบว่าทั้งสอง service สื่อสารกันได้
3. เขียน Kubernetes Deployment YAML สำหรับแอปที่มี 3 replica พร้อม HorizontalPodAutoscaler ที่ปรับตาม CPU usage (ถ้ามี cluster ทดสอบ เช่น minikube ให้ลองรันจริง)

## Mini Project

**"ระบบ Deploy แอปพลิเคชันแบบครบวงจร (Full Deployment Pipeline Simulator)"**: เขียนโปรแกรมที่:
1. สร้างแอปพลิเคชัน REST API ง่ายๆ (ต่อยอดจากบทที่ 43) พร้อม Dockerfile ที่ build image ได้สำเร็จ
2. เขียน `docker-compose.yml` ที่รวมแอปพลิเคชัน, ฐานข้อมูล, และ Redis cache เข้าด้วยกัน ทดสอบว่าทั้งระบบทำงานร่วมกันได้จริงด้วย `docker-compose up`
3. เขียน Kubernetes manifest (Deployment + Service + HorizontalPodAutoscaler) สำหรับ deploy แอปพลิเคชันนี้ พร้อมจำลองสถานการณ์ Pod ล่ม (ลบ Pod ด้วยมือ) แล้วสังเกตว่า Kubernetes สร้างตัวใหม่ทดแทนอัตโนมัติ
4. เขียนรายงานเปรียบเทียบขนาดและเวลาเริ่มต้นระหว่าง Docker Image กับ Virtual Machine image ขนาดเทียบเท่า (ถ้าเข้าถึงเครื่องมือทดสอบทั้งสองแบบได้) เพื่อพิสูจน์ความแตกต่างที่อธิบายไว้ในบทเรียน

## ข้อคิดสำคัญ (Key Takeaways)

- Container บรรจุโค้ด, dependency, และการตั้งค่าไว้ในแพ็กเกจเดียว แก้ปัญหา "It works on my machine" ทำให้รันได้เหมือนกันทุกที่
- Container เบากว่า Virtual Machine มากเพราะแชร์ OS Kernel เดียวกันกับ host แทนที่จะจำลองทั้งระบบปฏิบัติการแยกต่างหาก
- Docker ใช้ Layer Cache เร่งการ build โดยจัดลำดับคำสั่งให้สิ่งที่เปลี่ยนน้อย (dependency) อยู่ก่อน สิ่งที่เปลี่ยนบ่อย (โค้ด) อยู่หลัง
- Kubernetes จัดการ container จำนวนมากโดยอัตโนมัติ: Self-healing (สร้าง Pod ใหม่ทดแทนเมื่อล่ม) และ Auto-scaling (ปรับจำนวน Pod ตามโหลด) ต่อยอดจาก Horizontal Scaling บทที่ 55
- ข้อมูลสำคัญต้องเก็บใน Volume แยกต่างหากเสมอ เพราะ container ถูกออกแบบมาให้สร้างและทำลายได้ตลอดเวลา (ephemeral)
- บทถัดไปเป็นบทสุดท้ายของหลักสูตร: CI/CD, Logging, Monitoring ซึ่งปิดวงจรการพัฒนาซอฟต์แวร์ตั้งแต่เขียนโค้ดจนถึงการดูแลระบบที่รันอยู่จริงใน production

---
*บทต่อไป (บทที่ 59): CI/CD, Logging, Monitoring — พิมพ์ "Next" เพื่อดำเนินการต่อ*
