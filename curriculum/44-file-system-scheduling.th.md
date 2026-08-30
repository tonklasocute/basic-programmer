# บทที่ 44: File System, Scheduling

## แนวคิดหลัก (Concept)

ตลอดหลักสูตรที่ผ่านมาใช้คำว่า "process", "thread", "disk" อยู่บ่อยครั้ง (บทที่ 1, 3, 36-38) โดยยังไม่ได้อธิบายว่าระบบปฏิบัติการ (Operating System) จัดการสิ่งเหล่านี้อย่างไรจริงๆ บทนี้เปิด **Part 11: Operating Systems** โดยเจาะลึกสองหน้าที่พื้นฐานที่สุดของ OS: **File System** (จัดการว่าข้อมูลบน disk เก็บที่ไหนและเข้าถึงอย่างไร) และ **Scheduling** (ตัดสินใจว่า process/thread ไหนได้ใช้ CPU เมื่อไหร่)

- **File System** — ระบบที่ OS ใช้จัดระเบียบและติดตามตำแหน่งของไฟล์บนอุปกรณ์จัดเก็บข้อมูล (HDD/SSD) ทำหน้าที่แปลง "ชื่อไฟล์ที่มนุษย์เข้าใจ" ให้เป็น "ตำแหน่งจริงบนดิสก์" ที่ฮาร์ดแวร์เข้าใจ
- **Inode** — โครงสร้างข้อมูลที่เก็บ**ข้อมูลเกี่ยวกับไฟล์** (metadata: ขนาด, สิทธิ์การเข้าถึง, เวลาที่แก้ไขล่าสุด, และที่สำคัญที่สุดคือ**ตำแหน่งของข้อมูลจริงบนดิสก์**) แยกจากชื่อไฟล์และเนื้อหาไฟล์
- **Process Scheduling** — กลไกที่ OS ตัดสินใจว่า process/thread ตัวไหนจะได้ใช้ CPU ในช่วงเวลาถัดไป เมื่อมีหลาย process แข่งกันขอใช้ CPU ที่มีจำนวนคอร์จำกัด
- **Scheduling Algorithm** — อัลกอริทึมเฉพาะที่ OS ใช้ตัดสินใจลำดับ เช่น **FCFS** (First-Come-First-Served), **Round Robin** (แบ่งเวลาเท่าๆ กันหมุนเวียน), **Priority Scheduling** (ให้ process ที่สำคัญกว่าได้ CPU ก่อน)

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **File System มีไว้ซ่อนความซับซ้อนของฮาร์ดแวร์จัดเก็บข้อมูลจากผู้ใช้**: ฮาร์ดดิสก์จริงๆ เป็นแค่ลำดับของ block ข้อมูลดิบๆ ที่ไม่มีความหมายในตัวเอง — ถ้าไม่มี File System ผู้ใช้ต้องจำเลขที่อยู่ block ที่แน่นอนเพื่อเปิดไฟล์แต่ละไฟล์ (แทนที่จะพิมพ์ชื่อไฟล์อย่าง `document.txt`) File System สร้างชั้นนามธรรม (Abstraction — ทบทวนบทที่ 30) ที่ทำให้ "ไฟล์และโฟลเดอร์" เป็นแนวคิดที่ใช้งานง่าย
- **Inode มีไว้แยกชื่อไฟล์ออกจากข้อมูลจริง เพื่อให้จัดการไฟล์ได้ยืดหยุ่นกว่า**: การแยกนี้ทำให้ไฟล์เดียวกันมีหลายชื่อ (hard link) ได้, การเปลี่ยนชื่อไฟล์ทำได้เร็วมาก (แค่แก้ mapping ไม่ต้องย้ายข้อมูลจริงบนดิสก์เลย)
- **Scheduling มีไว้เพราะจำนวน process ที่ต้องการทำงานมักมากกว่าจำนวนคอร์ CPU ที่มีจริง**: คอมพิวเตอร์ทั่วไปรัน process หลายสิบหลายร้อยตัวพร้อมกัน (เบราว์เซอร์, เพลง, แอปแชท) แต่ CPU มีแค่ไม่กี่คอร์ — Scheduler ต้องตัดสินใจว่าใครได้ใช้ CPU เมื่อไหร่ เพื่อให้ทุก process รู้สึกว่า "ทำงานพร้อมกัน" ได้ (ผ่านการสลับกันเร็วมากจนมนุษย์ไม่รู้สึก)
- **Scheduling Algorithm ที่ต่างกันมีไว้ให้เหมาะกับเป้าหมายต่างกัน**: เหมือนที่ Part 5 สอนว่าไม่มีอัลกอริทึม sort ตัวเดียวที่ดีที่สุดเสมอ การ schedule ก็เช่นกัน — บาง OS เน้นให้ process โต้ตอบกับผู้ใช้ (เช่น เมาส์, คีย์บอร์ด) ตอบสนองเร็ว บาง OS เน้นความยุติธรรมระหว่าง process ทั้งหมด

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **File System** เหมือน **ระบบห้องสมุดที่มีบัตรรายการหนังสือ (card catalog)** — ผู้ใช้ค้นหาด้วยชื่อหนังสือ (ชื่อไฟล์) บัตรรายการบอกว่าหนังสือเล่มนั้นอยู่ชั้นไหน แถวไหน (ตำแหน่งบนดิสก์) โดยไม่ต้องเดินหาทีละชั้นเอง
- **Inode** เหมือน **ป้ายทะเบียนรถที่แยกจากตัวรถจริง** — ป้ายทะเบียน (ชื่อไฟล์) เปลี่ยนได้โดยไม่ต้องเปลี่ยนตัวรถ (ข้อมูลจริง) เลย และรถคันเดียวอาจมีการจดทะเบียนซ้อนในหลายประเทศได้ (คล้าย hard link หลายชื่อชี้ไปที่ข้อมูลเดียวกัน)
- **Scheduling** เหมือน **พนักงานเสิร์ฟคนเดียวที่ต้องดูแลหลายโต๊ะพร้อมกัน** — เดินวนไปแต่ละโต๊ะทีละนิด (สลับ process) เร็วมากจนลูกค้าแต่ละโต๊ะรู้สึกเหมือนได้รับการดูแลตลอดเวลา ทั้งที่จริงๆ พนักงานทำได้แค่ทีละโต๊ะในแต่ละขณะ
- **Round Robin** เหมือน **การแบ่งเวลาพูดในที่ประชุมให้ทุกคนพูดได้คนละ 2 นาทีเท่ากันหมด สลับวนไปเรื่อยๆ** — ยุติธรรม ไม่มีใครถูกเมินไปตลอด แม้บางคนอาจต้องการเวลามากกว่าจริง

## แผนภาพอธิบาย (Visual explanation)

### File System — จากชื่อไฟล์ถึงข้อมูลจริงบนดิสก์

```
ผู้ใช้เปิดไฟล์ "report.pdf"
        |
        v
Directory Entry: "report.pdf" -> Inode #12345      (ชื่อไฟล์ map ไปยัง inode)
        |
        v
Inode #12345:
  - ขนาด: 2.5 MB
  - สิทธิ์: read-write
  - เวลาแก้ไขล่าสุด: 2026-08-15
  - ตำแหน่งข้อมูลจริง: [Block 500, Block 501, Block 780, ...]   <- ชี้ไปยัง block จริงบนดิสก์
        |
        v
Disk Blocks: อ่านข้อมูลจริงจาก Block 500, 501, 780, ... มาประกอบกันเป็นไฟล์ที่สมบูรณ์
```

### Hard Link — สองชื่อไฟล์ชี้ไปยัง Inode เดียวกัน

```
"report.pdf"     ---\
                      ---> Inode #12345 ---> Disk Blocks [500, 501, 780]
"final_report.pdf" --/

ทั้งสองชื่อคือไฟล์เดียวกันจริงๆ (ไม่ใช่สำเนา) แก้ไขผ่านชื่อไหนก็เห็นการเปลี่ยนแปลงในอีกชื่อทันที
ลบชื่อใดชื่อหนึ่งออก -> อีกชื่อยังคงเข้าถึงข้อมูลได้ปกติ (inode จะถูกลบจริงเมื่อไม่มีชื่อใดชี้มาแล้วเท่านั้น)
```

### FCFS (First-Come-First-Served) — เรียงคิวตามลำดับที่มาถึง

```
Process มาถึงตามลำดับ:  P1(ใช้เวลา 10ms)  P2(ใช้เวลา 1ms)  P3(ใช้เวลา 1ms)

Timeline: |---P1 (10ms)---|-P2-|-P3-|
Waiting time: P1=0, P2=10, P3=11    เฉลี่ย = 7ms

ปัญหา: P2, P3 ใช้เวลาสั้นมากแต่ต้องรอ P1 ที่ใช้เวลานานจบก่อน (Convoy Effect)
```

### Round Robin — แบ่งเวลาเท่ากันหมุนเวียน (time quantum = 4ms)

```
Process: P1(10ms), P2(1ms), P3(1ms)

Timeline: |-P1(4ms)-|-P2(1ms)-|-P3(1ms)-|-P1(4ms)-|-P1(2ms)-|
                      (จบ)      (จบ)                  (จบ)

P2, P3 ที่ใช้เวลาสั้นได้ทำงานเสร็จเร็วกว่ามาก แม้ P1 จะมาถึงก่อนก็ตาม
Waiting time: P1=0+5(รอตอน P2,P3 ทำงาน)=5, P2=4, P3=5    เฉลี่ยดีขึ้นกว่า FCFS อย่างเห็นได้ชัด
```

### Priority Scheduling — งานสำคัญกว่าได้ CPU ก่อนเสมอ

```
Process: P1(priority=3, low), P2(priority=1, high), P3(priority=2, medium)

Timeline: |---P2 (priority สูงสุดทำก่อน)---|---P3---|---P1---|

ปัญหาที่อาจเกิดขึ้น: Starvation - ถ้ามี process priority สูงมาเรื่อยๆ ไม่หยุด
                     P1 (priority ต่ำ) อาจไม่ได้ใช้ CPU เลยตลอดไป!
```

## ตัวอย่างโค้ด (Code example)

```python
import os

# ===== File System: ดูข้อมูล inode (metadata) ของไฟล์ผ่าน Python =====
stat_info = os.stat("example.txt")
print(f"Size: {stat_info.st_size} bytes")           # ขนาดไฟล์
print(f"Inode number: {stat_info.st_ino}")             # หมายเลข inode
print(f"Last modified: {stat_info.st_mtime}")           # เวลาที่แก้ไขล่าสุด

# ===== Hard Link: สองชื่อไฟล์ชี้ไปยัง inode เดียวกัน =====
os.link("example.txt", "example_alias.txt")           # สร้าง hard link
print(os.stat("example.txt").st_ino == os.stat("example_alias.txt").st_ino)   # True -> inode เดียวกัน!

# ===== จำลอง FCFS Scheduling =====
def fcfs_scheduling(processes):                        # processes = [(name, burst_time)]
    current_time = 0
    schedule = []
    for name, burst_time in processes:                    # ทำตามลำดับที่มาถึงเป๊ะๆ ไม่สลับ
        waiting_time = current_time
        schedule.append((name, current_time, current_time + burst_time))
        current_time += burst_time
        print(f"{name}: waiting={waiting_time}ms, runs from {current_time - burst_time} to {current_time}")
    return schedule

# ===== จำลอง Round Robin Scheduling =====
def round_robin_scheduling(processes, quantum):        # processes = [(name, remaining_time)]
    from collections import deque
    queue = deque(processes)
    current_time = 0
    while queue:
        name, remaining = queue.popleft()
        run_time = min(quantum, remaining)                # ทำงานได้แค่ quantum เดียว หรือน้อยกว่าถ้าใกล้เสร็จ
        print(f"{name}: runs from {current_time} to {current_time + run_time}")
        current_time += run_time
        remaining -= run_time
        if remaining > 0:                                    # ยังทำไม่เสร็จ -> กลับไปต่อคิวท้ายสุด
            queue.append((name, remaining))
        else:
            print(f"{name}: finished at {current_time}")

print("=== FCFS ===")
fcfs_scheduling([("P1", 10), ("P2", 1), ("P3", 1)])

print("=== Round Robin (quantum=4) ===")
round_robin_scheduling([("P1", 10), ("P2", 1), ("P3", 1)], quantum=4)
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `os.stat("example.txt")` — เรียก system call ไปยัง OS เพื่อขอดู **inode metadata** ของไฟล์นั้นโดยตรง (ไม่ต้องเปิดอ่านเนื้อหาไฟล์เลย) `st_ino` คือหมายเลข inode ที่ไฟล์นี้ผูกอยู่ — ถ้าไฟล์สองชื่อมี `st_ino` เดียวกัน แปลว่าทั้งคู่คือไฟล์เดียวกันจริงๆ (hard link) ไม่ใช่แค่มีเนื้อหาเหมือนกันบังเอิญ
2. `os.link("example.txt", "example_alias.txt")` — สร้าง**ชื่อใหม่ที่ชี้ไปยัง inode เดิม** ไม่ใช่การคัดลอกข้อมูล — ทั้งสองชื่อ**แชร์ข้อมูลเดียวกันบนดิสก์**อย่างแท้จริง การแก้ไขผ่านชื่อไหนก็ตามจะเห็นผลในอีกชื่อทันที เพราะทั้งคู่อ่าน/เขียนไปที่ block เดียวกันบนดิสก์
3. `fcfs_scheduling` — วนลูปตาม**ลำดับที่ process เข้ามาในคิวเป๊ะๆ** ไม่มีการสลับลำดับเลย `waiting_time` ของแต่ละ process คือเวลาที่ต้องรอก่อนได้เริ่มทำงานจริง (เท่ากับผลรวมเวลาทำงานของ process ก่อนหน้าทั้งหมด) — สังเกตว่า `P2` และ `P3` ที่ใช้เวลาสั้นมากต้องรอ `P1` ที่ใช้เวลานาน (10ms) จบก่อนเสมอ แม้จะมาถึงทีหลังแค่นิดเดียว (**Convoy Effect**)
4. `round_robin_scheduling` — ใช้ `deque` (ทบทวนจากบทที่ 14: Queue) เป็นคิวหมุนเวียน แต่ละรอบดึง process ตัวแรกออกมาทำงานได้แค่ `quantum` (4ms ในตัวอย่าง) เท่านั้น ถ้ายังทำไม่เสร็จ (`remaining > 0`) จะถูกส่งกลับไปต่อท้ายคิวเพื่อรอรอบถัดไป — เพราะเหตุนี้ `P2` และ `P3` ที่ใช้เวลาแค่ 1ms จึงได้ทำงานและเสร็จสิ้นเร็วกว่าการรอ `P1` จบก่อนแบบ FCFS มาก แม้ `P1` จะมาถึงก่อนก็ตาม

## Time Complexity

| Scheduling Algorithm | Time Complexity ต่อการตัดสินใจ |
|---|---|
| FCFS | O(1) — แค่ดึงจากคิวตามลำดับ (FIFO เหมือนบทที่ 14) |
| Round Robin | O(1) — ดึงจากคิวและใส่กลับท้ายคิว (Queue operations) |
| Priority Scheduling | O(log n) ถ้าใช้ Priority Queue/Heap (ทบทวนบทที่ 17) เพื่อดึง process priority สูงสุดออกมาเสมอ |

- File System — การเข้าถึง inode จากชื่อไฟล์ (Directory Lookup): ขึ้นอยู่กับโครงสร้างที่ใช้เก็บ directory entry มักเป็น **Hash Table หรือ B-Tree** (บทที่ 15, 40) ทำให้ได้ **O(1)** หรือ **O(log n)** โดยเฉลี่ย แทนที่จะเป็น O(n) จากการไล่หาทีละชื่อ

## Space Complexity

- Inode ใช้พื้นที่คงที่ **O(1)** ต่อไฟล์หนึ่งไฟล์ (ไม่ขึ้นกับขนาดไฟล์จริง เพราะเก็บแค่ metadata และตัวชี้ไปยัง block) แต่ระบบไฟล์ต้องจองพื้นที่ไว้ล่วงหน้าสำหรับ inode ทั้งหมดที่รองรับได้ ทำให้มี **ขีดจำกัดจำนวนไฟล์สูงสุด** แม้ดิสก์จะยังมีพื้นที่ว่างเหลือมากก็ตาม
- Scheduling Queue ใช้พื้นที่ **O(n)** โดย n คือจำนวน process ที่รอคิวอยู่ ณ ขณะนั้น

## แนวทางปฏิบัติที่ดี (Best Practices)

- ใช้ **soft link (symbolic link)** แทน hard link เมื่อต้องการอ้างอิงข้ามไดรฟ์/พาร์ติชัน หรือต้องการให้ชื่ออ้างอิงชี้ตามไฟล์ต้นทางแม้ไฟล์ต้นทางจะถูกย้าย (hard link ทำแบบนี้ไม่ได้เพราะผูกกับ inode บนพาร์ติชันเดียวกันเท่านั้น)
- เลือก Scheduling Algorithm ให้ตรงกับเป้าหมายของระบบ: Round Robin เหมาะกับระบบที่ต้องการการตอบสนองที่ยุติธรรม (เช่น ระบบ interactive ทั่วไป) ส่วน Priority Scheduling เหมาะกับระบบที่มีงานสำคัญชัดเจนต้องทำก่อน (เช่น real-time system) แต่ต้องระวัง Starvation เสมอ
- ป้องกัน **Starvation** ใน Priority Scheduling ด้วยเทคนิค **Aging** (ค่อยๆ เพิ่ม priority ของ process ที่รอนานขึ้นเรื่อยๆ เพื่อให้ในที่สุดได้ทำงานแน่นอน)

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- สับสนระหว่าง hard link กับการคัดลอกไฟล์ธรรมดา — คิดว่าลบไฟล์ต้นฉบับแล้วสำเนา (hard link) จะหายไปด้วย ทั้งที่จริงข้อมูลยังอยู่ตราบใดที่ยังมีชื่ออื่นชี้ไปยัง inode เดียวกันอยู่
- ตั้งค่า `quantum` ของ Round Robin **เล็กเกินไป** ทำให้ OS เสียเวลาสลับ process (context switch) บ่อยเกินจำเป็น ซึ่งมีต้นทุนจริง (คล้ายการล็อก Critical Section เล็กเกินไปในบทที่ 36 ที่ยังพอรับได้ แต่การสลับ context บ่อยเกินไปกลับเสียประสิทธิภาพ)
- ใช้ Priority Scheduling โดยไม่มีกลไกป้องกัน Starvation ทำให้ process priority ต่ำบางตัวไม่ได้ทำงานเลยเป็นเวลานานผิดปกติ
- เข้าใจผิดว่า Scheduling Algorithm ที่ "ยุติธรรมที่สุด" (Round Robin) ดีที่สุดเสมอ ทั้งที่บางสถานการณ์ (เช่น งาน background ที่ไม่ต้องการตอบสนองเร็ว) FCFS อาจเพียงพอและมี overhead น้อยกว่า

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายว่า Inode คืออะไร แยกออกจากชื่อไฟล์และเนื้อหาไฟล์อย่างไร ทำไมการแยกนี้ถึงมีประโยชน์
2. Hard Link ต่างจาก Symbolic Link (Soft Link) อย่างไร แต่ละแบบมีข้อจำกัดอะไรบ้าง
3. อธิบาย Convoy Effect ใน FCFS Scheduling คืออะไร Round Robin แก้ปัญหานี้อย่างไร
4. Starvation ใน Priority Scheduling คืออะไร มีวิธีป้องกันอย่างไร (ใบ้: Aging)
5. ทำไม `quantum` ของ Round Robin ที่เล็กเกินไปถึงเป็นปัญหา แม้จะดูเหมือนทำให้ทุก process ได้ทำงานเร็วขึ้น

## แบบฝึกหัด (Practice Exercises)

1. เขียนโปรแกรมจำลอง Scheduling Algorithm เพิ่มอีกแบบคือ **Shortest Job First (SJF)** ที่เลือก process ที่ใช้เวลาสั้นที่สุดทำก่อนเสมอ แล้วเปรียบเทียบ average waiting time กับ FCFS และ Round Robin ด้วยชุดข้อมูลเดียวกัน
2. เขียนโปรแกรมที่ใช้ `os.stat()` เปรียบเทียบ inode number ของไฟล์ก่อนและหลังเปลี่ยนชื่อไฟล์ (`os.rename()`) เพื่อพิสูจน์ว่าการเปลี่ยนชื่อไม่ได้สร้างไฟล์ใหม่ (inode number เดิม)
3. Implement Priority Scheduling ด้วย Heap (ทบทวนจากบทที่ 17) พร้อมกลไก Aging ที่เพิ่ม priority ของ process ที่รอเกิน 10ms ขึ้น 1 ระดับทุก 5ms แล้วพิสูจน์ว่า process priority ต่ำไม่ถูก starve

## Mini Project

**"เครื่องมือจำลองและเปรียบเทียบ CPU Scheduler (CPU Scheduler Simulator & Visualizer)"**: เขียนโปรแกรมที่:
1. Implement Scheduling Algorithm อย่างน้อย 4 แบบ: FCFS, Round Robin, Priority Scheduling (พร้อม Aging), และ Shortest Job First
2. รับชุดข้อมูล process จำลอง (ชื่อ, เวลาที่มาถึง, เวลาที่ต้องใช้ CPU, priority) แล้วรันผ่านทุกอัลกอริทึม
3. แสดง Gantt Chart แบบข้อความ (คล้ายแผนภาพในบทเรียน) ของแต่ละอัลกอริทึม พร้อมคำนวณ average waiting time และ average turnaround time
4. เขียนบทสรุปเปรียบเทียบว่าอัลกอริทึมไหนเหมาะกับสถานการณ์ไหน (เช่น ระบบที่มีงานหลากหลายขนาด, ระบบที่ต้องการความยุติธรรม, ระบบ real-time) โดยอ้างอิงผลการทดลองจริงประกอบ

## ข้อคิดสำคัญ (Key Takeaways)

- File System แปลง "ชื่อไฟล์" ที่มนุษย์เข้าใจให้เป็นตำแหน่งข้อมูลจริงบนดิสก์ผ่าน Inode ที่เก็บ metadata แยกจากเนื้อหาไฟล์
- Hard Link ทำให้ไฟล์เดียวกันมีหลายชื่อได้ (ชี้ไปยัง inode เดียวกัน) ต่างจากการคัดลอกไฟล์ที่สร้างข้อมูลใหม่แยกกันโดยสิ้นเชิง
- Scheduling ตัดสินใจว่า process ไหนได้ใช้ CPU เมื่อไหร่ เพราะจำนวน process มักมากกว่าจำนวนคอร์ CPU ที่มีจริง
- FCFS เรียบง่ายแต่เกิด Convoy Effect (process สั้นต้องรอ process ยาวจบก่อน) Round Robin แก้ปัญหานี้ด้วยการแบ่งเวลาเท่ากันหมุนเวียน ส่วน Priority Scheduling เสี่ยง Starvation ถ้าไม่มี Aging ป้องกัน
- การเลือก Scheduling Algorithm ที่เหมาะสมขึ้นอยู่กับเป้าหมายของระบบ (ความยุติธรรม vs ความสำคัญของงาน vs overhead ในการสลับ process) เหมือนหลักการเลือกอัลกอริทึม/โครงสร้างข้อมูลที่เรียนมาตลอดหลักสูตร
- บทถัดไปจะเจาะลึก Memory Management, Virtual Memory, และ Paging ซึ่งอธิบายว่า OS จัดการหน่วยความจำ (RAM) ให้กับหลาย process พร้อมกันอย่างปลอดภัยและมีประสิทธิภาพได้อย่างไร ต่อยอดจาก Memory Model ในบทที่ 3

---
*บทต่อไป (บทที่ 45): Memory Management, Virtual Memory, Paging — พิมพ์ "Next" เพื่อดำเนินการต่อ*
