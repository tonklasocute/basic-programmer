# บทที่ 59: CI/CD, Logging, Monitoring

## แนวคิดหลัก (Concept)

นี่คือบทสุดท้ายของหลักสูตร บทที่แล้วสอนวิธีบรรจุโปรแกรมด้วย Docker และจัดการด้วย Kubernetes แต่ยังไม่ได้ตอบคำถามที่ปิดวงจรการพัฒนาซอฟต์แวร์อย่างสมบูรณ์: **โค้ดที่ผ่านเทสต์ (บทที่ 51) แล้วจะถูกนำไปรันจริงใน production ได้อย่างไรอย่างปลอดภัยและอัตโนมัติ** และ **เมื่อระบบรันอยู่จริง จะรู้ได้อย่างไรว่ามันทำงานถูกต้องหรือมีปัญหาเกิดขึ้น**

- **CI (Continuous Integration)** — แนวทางที่นักพัฒนารวมโค้ด (merge — ทบทวนบทที่ 50) เข้า branch หลักบ่อยๆ โดยมีระบบอัตโนมัติรันเทสต์ (บทที่ 51) ทุกครั้งที่มีการเปลี่ยนแปลง เพื่อจับปัญหาให้เร็วที่สุด
- **CD (Continuous Deployment/Delivery)** — แนวทางที่นำโค้ดที่ผ่านการทดสอบไป**ปล่อยใช้งานจริง (deploy)** โดยอัตโนมัติ ลดขั้นตอนที่ต้องทำด้วยมือซึ่งเสี่ยงต่อความผิดพลาด
- **Logging (การบันทึก log)** — การบันทึกเหตุการณ์ที่เกิดขึ้นในระบบไว้เป็นลายลักษณ์อักษร (เช่น request ที่เข้ามา, error ที่เกิดขึ้น) เพื่อย้อนดูภายหลังเมื่อต้องสืบสวนปัญหา
- **Monitoring (การเฝ้าติดตาม)** — การเก็บและแสดงผลตัวชี้วัด (metrics) ของระบบแบบต่อเนื่อง (เช่น CPU usage, response time, error rate) พร้อมระบบแจ้งเตือน (alert) เมื่อค่าผิดปกติ

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **CI มีไว้จับปัญหาให้เร็วที่สุดเท่าที่จะเป็นไปได้**: ถ้านักพัฒนาหลายคนทำงานแยกกันนานหลายสัปดาห์แล้วค่อยมารวมโค้ดทีเดียว (แทนที่จะ merge บ่อยๆ) มักเจอ Merge Conflict (บทที่ 50) จำนวนมากและปัญหาที่สะสมมานาน — CI บังคับให้รันเทสต์ (บทที่ 51) ทุกครั้งที่มีการเปลี่ยนแปลงเล็กๆ ทำให้เจอบั๊กทันทีตั้งแต่จุดที่เกิดขึ้น ไม่ใช่หลังจากสะสมปัญหาไว้นาน
- **CD มีไว้ลดความเสี่ยงจากการ deploy ด้วยมือ**: การ deploy ด้วยมือ (พิมพ์คำสั่งเอง, copy ไฟล์เอง) เสี่ยงต่อความผิดพลาดของมนุษย์สูงมาก (ลืมขั้นตอน, พิมพ์ผิด) และไม่สม่ำเสมอ (แต่ละครั้งอาจทำต่างกันเล็กน้อย) — Pipeline อัตโนมัติทำตามขั้นตอนเดียวกันทุกครั้งเป๊ะ ลดความผิดพลาดและทำให้ deploy บ่อยขึ้นได้อย่างปลอดภัย (บางทีมสามารถ deploy ได้วันละหลายสิบครั้ง)
- **Logging มีไว้เพราะเมื่อระบบพังใน production เราไม่สามารถ debug ด้วยการรันโปรแกรมใหม่แบบ step-by-step ได้เหมือนตอนพัฒนา**: ต้องอาศัยหลักฐานที่บันทึกไว้ (log) เพื่อย้อนดูว่าเกิดอะไรขึ้นก่อนที่ปัญหาจะเกิด — ไม่มี log ที่ดีเท่ากับสืบสวนคดีโดยไม่มีหลักฐาน
- **Monitoring มีไว้ให้รู้ว่าระบบมีปัญหา "ก่อนที่ผู้ใช้จะมาบ่น"**: ถ้าไม่มีระบบเฝ้าติดตาม ทีมงานจะรู้ว่าระบบมีปัญหาก็ต่อเมื่อผู้ใช้ร้องเรียนเข้ามา (สายเกินไป) — Monitoring ที่ดีแจ้งเตือนทีมงานทันทีที่ตัวชี้วัดเริ่มผิดปกติ (เช่น error rate เพิ่มขึ้นกะทันหัน) ทำให้แก้ไขได้ก่อนที่ผู้ใช้จำนวนมากจะได้รับผลกระทบ

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **CI** เหมือน **การตรวจสอบคุณภาพชิ้นส่วนทุกชิ้นทันทีที่ผลิตเสร็จในโรงงาน** แทนที่จะรอตรวจทีเดียวตอนท้ายสายการผลิต — เจอชิ้นส่วนเสียได้เร็วกว่าและแก้ไขง่ายกว่ามาก (ไม่ต้องรื้อสินค้าสำเร็จรูปทั้งชุดกลับมาแก้)
- **CD** เหมือน **สายพานลำเลียงอัตโนมัติที่ส่งสินค้าจากโรงงานไปถึงชั้นวางในร้านโดยไม่ต้องมีคนยกด้วยมือทุกขั้นตอน** — ลดโอกาสทำสินค้าตกหล่นหรือส่งผิดที่ระหว่างทาง
- **Logging** เหมือน **กล้องวงจรปิดที่บันทึกเหตุการณ์ทุกอย่างไว้** — ถ้าเกิดเหตุการณ์ผิดปกติ สามารถย้อนดูภาพบันทึกเพื่อสืบสวนได้ ต่างจากการพยายามจำเหตุการณ์จากความทรงจำที่ไม่แม่นยำ
- **Monitoring** เหมือน **เครื่องวัดสัญญาณชีพในห้อง ICU ที่ส่งเสียงเตือนทันทีเมื่อค่าผิดปกติ** — พยาบาลไม่ต้องนั่งเฝ้าดูตัวเลขตลอดเวลา แต่จะรู้ทันทีเมื่อมีอะไรผิดปกติเกิดขึ้นและต้องเข้าไปดูแล

## แผนภาพอธิบาย (Visual explanation)

### CI/CD Pipeline — จากโค้ดถึง Production

```
Developer push code
        |
        v
┌───────────────────┐
│  1. Build           │  <- Docker build image (บทที่ 58)
└─────────┬──────────┘
          v
┌───────────────────┐
│  2. Test            │  <- รัน Unit Test + Integration Test (บทที่ 51) อัตโนมัติ
└─────────┬──────────┘
          v (ถ้าเทสต์ผ่านทั้งหมด)
┌───────────────────┐
│  3. Deploy Staging   │  <- ปล่อยขึ้นสภาพแวดล้อมทดสอบก่อน
└─────────┬──────────┘
          v (ถ้าทดสอบบน Staging ผ่าน)
┌───────────────────┐
│  4. Deploy Production│  <- ปล่อยขึ้นใช้งานจริง (อัตโนมัติหรือกดอนุมัติ 1 ครั้ง)
└───────────────────┘

ถ้าขั้นตอนไหนล้มเหลว -> Pipeline หยุดทันที ไม่ปล่อยโค้ดที่มีปัญหาไปถึงผู้ใช้จริงเด็ดขาด
```

### Log Levels — ระดับความสำคัญของข้อความ Log

```
DEBUG    - รายละเอียดสำหรับตอนพัฒนาเท่านั้น (เช่น ค่าตัวแปรระหว่างทาง)
INFO     - เหตุการณ์ปกติที่น่าสนใจ (เช่น "User 42 logged in")
WARNING  - สิ่งที่ผิดปกติแต่ยังไม่ทำให้ระบบพัง (เช่น "API response slower than usual")
ERROR    - เกิดข้อผิดพลาดที่กระทบการทำงาน (เช่น "Failed to connect to database")
CRITICAL - ระบบกำลังจะพังทั้งหมด (เช่น "Out of memory, shutting down")

Production มักตั้งค่าให้บันทึกแค่ INFO ขึ้นไป (ไม่บันทึก DEBUG เพราะข้อมูลเยอะเกินไปและไม่จำเป็น)
```

### Monitoring — Metrics, Dashboard, Alert

```
ระบบ Production                    Monitoring System (เช่น Prometheus + Grafana)

ส่ง metrics ทุก 10 วินาที:            เก็บ metrics เป็น time-series data
  - CPU usage: 45%                          |
  - Response time: 120ms                    v
  - Error rate: 0.1%              ┌─────────────────────┐
  - Request count: 1500/min       │   Dashboard (Grafana) │  <- แสดงกราฟให้ทีมดูภาพรวม
                                  └─────────────────────┘
                                            |
                                            v
                                  ┌─────────────────────┐
                                  │  Alert Rule:          │
                                  │  error_rate > 5%      │  <- ถ้าเงื่อนไขเป็นจริง
                                  │  -> ส่ง Slack/Email    │     แจ้งเตือนทีมทันที
                                  └─────────────────────┘
```

## ตัวอย่างโค้ด (Code example)

```yaml
# ===== CI/CD Pipeline: GitHub Actions (ตัวอย่าง) =====
name: CI/CD Pipeline
on:
  push:
    branches: [main]

jobs:
  test:                                                    # ขั้นตอน CI: รันเทสต์อัตโนมัติ
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3                            # ดึงโค้ดล่าสุด
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: python -m pytest tests/ -v                        # รัน Unit/Integration Test (บทที่ 51)

  build_and_deploy:                                          # ขั้นตอน CD: build และ deploy
    needs: test                                                # รอให้ test ผ่านก่อนเสมอ
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .          # ใช้ commit hash เป็น tag (บทที่ 50, 58)
      - name: Push to registry
        run: docker push myapp:${{ github.sha }}
      - name: Deploy to Kubernetes
        run: kubectl set image deployment/myapp-deployment myapp=myapp:${{ github.sha }}
```

```python
import logging
import time

# ===== Logging: บันทึกเหตุการณ์พร้อมระดับความสำคัญ =====
logging.basicConfig(
    level=logging.INFO,                                       # production ตั้งเป็น INFO ขึ้นไป (ไม่เอา DEBUG)
    format='%(asctime)s [%(levelname)s] %(message)s'
)
logger = logging.getLogger(__name__)

def process_order(order_id, amount):
    logger.info(f"Processing order {order_id}, amount={amount}")   # เหตุการณ์ปกติ
    try:
        if amount <= 0:
            logger.warning(f"Order {order_id} has suspicious amount: {amount}")
        # ... ประมวลผลจริง ...
        logger.info(f"Order {order_id} completed successfully")
    except ConnectionError as e:
        logger.error(f"Order {order_id} failed: database connection error - {e}")
        raise
    except Exception as e:
        logger.critical(f"Unexpected failure processing order {order_id}: {e}")
        raise

# ===== Monitoring: เก็บ Metrics แบบง่าย (แนวคิดของ Prometheus Client) =====
class MetricsCollector:
    def __init__(self):
        self.request_count = 0
        self.error_count = 0
        self.total_response_time = 0.0

    def record_request(self, response_time, is_error=False):
        self.request_count += 1
        self.total_response_time += response_time
        if is_error:
            self.error_count += 1

    def get_error_rate(self):
        if self.request_count == 0:
            return 0.0
        return (self.error_count / self.request_count) * 100

    def get_avg_response_time(self):
        if self.request_count == 0:
            return 0.0
        return self.total_response_time / self.request_count

metrics = MetricsCollector()

def handle_request(request_id):
    start_time = time.time()
    is_error = False
    try:
        # ... ประมวลผล request จริง ...
        pass
    except Exception:
        is_error = True
        raise
    finally:
        response_time = time.time() - start_time
        metrics.record_request(response_time, is_error)
        if metrics.get_error_rate() > 5.0:                        # Alert Rule: error rate เกิน 5%
            logger.critical(f"ALERT: Error rate is {metrics.get_error_rate():.1f}%! Notify team immediately.")
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `jobs: test` ใน GitHub Actions — ทำงานทุกครั้งที่มีการ `push` เข้า branch `main` (ตาม `on: push: branches: [main]`) โดยรันเทสต์ทั้งหมด (`pytest tests/`) **อัตโนมัติโดยไม่ต้องมีใครสั่งเอง** — ถ้าเทสต์ตัวใดล้มเหลว pipeline จะหยุดทันทีที่ขั้นตอนนี้ (สถานะ "failed" แสดงให้ทีมเห็น) และ**ไม่ไปถึงขั้นตอน `build_and_deploy` เลย**
2. `build_and_deploy: needs: test` — คีย์เวิร์ด `needs: test` บังคับว่าขั้นตอนนี้จะเริ่มทำงานได้ก็ต่อเมื่อขั้นตอน `test` **สำเร็จเท่านั้น** — นี่คือหัวใจของ CI/CD: **ไม่มีทางที่โค้ดซึ่งเทสต์ไม่ผ่านจะถูก deploy ไปถึงผู้ใช้จริงได้เลย** เพราะ pipeline บังคับลำดับขั้นตอนไว้อย่างเข้มงวด
3. `docker build -t myapp:${{ github.sha }}` — ใช้ **commit hash** (จาก Git บทที่ 50) เป็น tag ของ Docker image แทนการใช้ tag คงที่อย่าง `latest` — ทำให้**ทุก deploy สามารถย้อนกลับไปดูได้แน่นอนว่ามาจาก commit ไหน** และถ้าต้อง rollback (ย้อนกลับเวอร์ชัน) ก็แค่สั่ง deploy image ของ commit ก่อนหน้ากลับมา
4. `logger.warning(...)` เทียบกับ `logger.error(...)` เทียบกับ `logger.critical(...)` — เลือกระดับความสำคัญให้ตรงกับสถานการณ์จริงเสมอ: `warning` สำหรับสิ่งที่น่าสงสัยแต่ไม่กระทบการทำงาน, `error` สำหรับข้อผิดพลาดที่กระทบ request นี้แต่ระบบยังทำงานต่อได้, `critical` สำหรับสถานการณ์ที่ต้องการความสนใจทันที — การเลือกระดับที่ถูกต้องทำให้ทีมงานกรอง log ดูเฉพาะสิ่งที่สำคัญได้ (เช่น ตั้ง alert ให้แจ้งเตือนเฉพาะ `error` ขึ้นไป ไม่ใช่ทุกข้อความ)
5. `MetricsCollector.get_error_rate()` — คำนวณเปอร์เซ็นต์ error จากจำนวน request ทั้งหมดที่ผ่านมา แล้วในฟังก์ชัน `handle_request` เช็คว่า `error_rate > 5.0` หรือไม่**หลังจบทุก request** ถ้าเกินเกณฑ์ที่ตั้งไว้ (**Alert Rule**) จะบันทึก log ระดับ `critical` ทันที — ในระบบจริง Monitoring tool อย่าง Prometheus/Grafana ทำสิ่งนี้โดยอัตโนมัติและซับซ้อนกว่านี้มาก (ส่งแจ้งเตือนผ่าน Slack/Email/SMS จริง ไม่ใช่แค่ log)

## Time Complexity

CI/CD, Logging, Monitoring เป็นกระบวนการด้าน**การดำเนินงาน (operations)** ไม่ใช่อัลกอริทึม แต่มีผลกระทบด้านเวลาที่สำคัญ:
- CI Pipeline ที่มี test suite ใหญ่ (ทบทวน Testing Pyramid บทที่ 51): เวลารัน pipeline ทั้งหมดขึ้นกับสัดส่วน Unit Test (เร็ว) ต่อ Integration/E2E Test (ช้า) — Testing Pyramid ที่ถูกต้องทำให้ CI feedback เร็ว
- Logging: การเขียน log แต่ละบรรทัดมีต้นทุน I/O เล็กน้อย (**O(1)** ต่อบรรทัด) แต่ถ้า log มากเกินไป (เช่น log ทุก loop iteration) อาจกลายเป็นคอขวดที่ทำให้ระบบช้าลงอย่างมีนัยสำคัญ
- Metrics collection: **O(1)** ต่อ request สำหรับการอัปเดตตัวนับพื้นฐาน

## Space Complexity

- Log files สะสมขนาดตามเวลา ต้องมีนโยบาย **Log Rotation** (ลบ/บีบอัด log เก่าหลังจากช่วงเวลาหนึ่ง) ไม่เช่นนั้นจะเต็มพื้นที่ disk
- Metrics ที่เก็บเป็น time-series มักถูกบีบอัดและสรุปข้อมูลเก่าให้หยาบขึ้นเมื่อเวลาผ่านไป (เช่น ข้อมูลรายวินาทีของเมื่อวาน อาจถูกสรุปเป็นค่าเฉลี่ยรายนาทีหลังผ่านไป 30 วัน) เพื่อประหยัดพื้นที่ในระยะยาว

## แนวทางปฏิบัติที่ดี (Best Practices)

- ทำให้ CI Pipeline **รันเร็วที่สุดเท่าที่จะทำได้** (ไม่เกิน 10-15 นาที) เพื่อให้นักพัฒนาได้ feedback เร็ว — ถ้า pipeline ช้าเกินไป ทีมจะเริ่มข้ามขั้นตอนทดสอบหรือรอนานจนเสียประสิทธิภาพการทำงาน
- Log ข้อมูลที่**เป็นประโยชน์ต่อการสืบสวนปัญหาในอนาคต** เสมอ (request id, user id, timestamp) แต่**ห้าม log ข้อมูลละเอียดอ่อน** (รหัสผ่าน, เลขบัตรเครดิต — เชื่อมโยงกับบทที่ 53) แม้จะดูมีประโยชน์สำหรับ debug ก็ตาม
- ตั้ง Alert ให้ **แจ้งเตือนเฉพาะสิ่งที่ต้องการ action จริงๆ** ไม่ใช่ทุกความผิดปกติเล็กน้อย — ถ้า Alert ดังบ่อยเกินไปโดยไม่มีอะไรต้องทำจริง ทีมจะเริ่มเพิกเฉยต่อ Alert ทั้งหมด (Alert Fatigue) ซึ่งอันตรายกว่าไม่มี Alert เลย

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ข้าม CI (ไม่รันเทสต์ก่อน deploy) เพราะ "รีบ" ทำให้บั๊กที่ตรวจจับได้ง่ายหลุดรอดไปถึง production
- Log ข้อมูลละเอียดอ่อน (รหัสผ่าน, token) ในข้อความ log โดยไม่ตั้งใจ ทำให้ระบบ log กลายเป็นช่องโหว่ความปลอดภัยใหม่ (ถ้าใครเข้าถึงไฟล์ log ได้ก็เห็นข้อมูลลับทั้งหมด)
- ตั้ง Log Level เป็น `DEBUG` ใน production ทำให้ log มีปริมาณมหาศาลจนหาข้อมูลที่ต้องการยาก และสิ้นเปลืองพื้นที่จัดเก็บโดยไม่จำเป็น
- ไม่มี Log Rotation ทำให้ไฟล์ log โตจนเต็มพื้นที่ disk ของเซิร์ฟเวอร์ (อาจทำให้ระบบล่มทั้งที่ไม่เกี่ยวกับบั๊กในโค้ดเลย)
- ตั้ง Alert ที่ threshold ต่ำเกินไปจนแจ้งเตือนตลอดเวลาโดยไม่มีปัญหาจริง ทำให้ทีมเกิด Alert Fatigue และอาจพลาด Alert ที่สำคัญจริงๆ ไปเพราะเพิกเฉยจนเป็นนิสัย

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง Continuous Integration และ Continuous Deployment
2. ทำไม CI Pipeline ถึงควรรันเทสต์ก่อนที่จะ deploy เสมอ เชื่อมโยงกับ Testing Pyramid จากบทที่ 51
3. อธิบายระดับของ Log (DEBUG, INFO, WARNING, ERROR, CRITICAL) แต่ละระดับใช้เมื่อไหร่
4. Monitoring กับ Logging ต่างกันอย่างไร ทั้งสองอย่างเสริมกันอย่างไรในการดูแลระบบ production
5. Alert Fatigue คืออะไร เกิดขึ้นได้อย่างไร และป้องกันได้อย่างไร

## แบบฝึกหัด (Practice Exercises)

1. เขียน CI Pipeline (ใช้ GitHub Actions หรือจำลองด้วย script) ที่รันเทสต์อัตโนมัติทุกครั้งที่มีการ push โค้ด แล้วทดสอบว่า pipeline หยุดทำงานจริงเมื่อเทสต์ล้มเหลว
2. เขียนระบบ Logging ที่มีทั้ง 5 ระดับ (DEBUG ถึง CRITICAL) สำหรับแอปพลิเคชันง่ายๆ แล้วทดสอบว่าตั้งค่า level เป็น `INFO` แล้วข้อความ `DEBUG` ไม่ถูกบันทึกจริง
3. ขยาย `MetricsCollector` ในตัวอย่างให้เก็บ metrics แยกตาม endpoint (เช่น `/login`, `/orders`) แล้วสร้างรายงานสรุปว่า endpoint ไหนมี error rate หรือ response time สูงผิดปกติ

## Mini Project

**"ระบบ Deploy และเฝ้าติดตามแบบครบวงจร (Full CI/CD & Observability Pipeline)"** — นี่คือ Mini Project สุดท้ายของหลักสูตรที่รวมทุกอย่างที่เรียนมาทั้งหมด เขียนโปรแกรมที่:
1. สร้างแอปพลิเคชัน REST API (ต่อยอดจากบทที่ 43, 47-49) พร้อม Unit/Integration Test ครบถ้วน (บทที่ 51)
2. เขียน CI Pipeline ที่รันเทสต์อัตโนมัติทุกครั้งที่มีการเปลี่ยนแปลงโค้ด (บทที่ 50) แล้ว build Docker image (บทที่ 58) เฉพาะเมื่อเทสต์ผ่านทั้งหมด
3. เขียน Kubernetes manifest สำหรับ deploy แอปพลิเคชันนี้พร้อม Auto-scaling (บทที่ 58) และเชื่อมต่อกับ Load Balancer, Cache, Message Queue (บทที่ 55-56) ตามความเหมาะสม
4. เพิ่มระบบ Logging ที่บันทึกทุก request พร้อมระดับความสำคัญที่เหมาะสม และระบบ Monitoring ที่เก็บ metrics (response time, error rate) พร้อม Alert เมื่อค่าผิดปกติ
5. เขียนเอกสารสรุป (README) อธิบายสถาปัตยกรรมทั้งหมดของระบบที่สร้างขึ้น อ้างอิงกลับไปยังบทเรียนที่เกี่ยวข้องตลอดทั้งหลักสูตร ตั้งแต่ Big O (บทที่ 20) ไปจนถึง Monitoring (บทนี้) — เป็นการสรุปทบทวนความรู้ทั้งหมดที่เรียนมา

## ข้อคิดสำคัญ (Key Takeaways)

- CI รันเทสต์อัตโนมัติทุกครั้งที่มีการเปลี่ยนแปลงโค้ด จับปัญหาให้เร็วที่สุดก่อนที่จะสะสม
- CD ปล่อยโค้ดที่ผ่านการทดสอบไปใช้งานจริงโดยอัตโนมัติ ลดความเสี่ยงจากการ deploy ด้วยมือ
- Logging บันทึกเหตุการณ์ในระบบเป็นหลักฐานสำหรับสืบสวนปัญหาภายหลัง ต้องเลือกระดับความสำคัญให้เหมาะสมและไม่บันทึกข้อมูลละเอียดอ่อน
- Monitoring เฝ้าติดตามตัวชี้วัดของระบบแบบต่อเนื่องพร้อมระบบแจ้งเตือน ทำให้รู้ปัญหาก่อนผู้ใช้จะร้องเรียน
- ทั้งสี่เรื่องนี้ปิดวงจรการพัฒนาซอฟต์แวร์ที่สมบูรณ์: เขียนโค้ด -> ทดสอบ -> deploy -> เฝ้าติดตาม -> วนกลับไปแก้ไขปรับปรุง

**นี่คือบทสุดท้ายของหลักสูตร "Programming Curriculum — Beginner to Senior Backend Engineer"** ตลอด 59 บทที่ผ่านมา หลักสูตรนี้พาเดินทางจากพื้นฐานที่สุด (คอมพิวเตอร์ทำงานอย่างไร) ผ่านโครงสร้างข้อมูลและอัลกอริทึม, OOP และ Functional Programming, Concurrency, ฐานข้อมูล, เครือข่าย, ระบบปฏิบัติการ, วิศวกรรมซอฟต์แวร์, Version Control, การทดสอบ, ความปลอดภัย, การออกแบบระบบขนาดใหญ่, ไปจนถึง DevOps — ครบทุกด้านที่วิศวกรซอฟต์แวร์ Backend ระดับ Senior ควรเข้าใจ ยินดีด้วยกับการเรียนจบหลักสูตรนี้!

---
*จบหลักสูตร 59 บท — ขอให้สนุกกับการนำความรู้ไปประยุกต์ใช้ในงานจริง*
