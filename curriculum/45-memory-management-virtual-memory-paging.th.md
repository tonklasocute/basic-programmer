# บทที่ 45: Memory Management, Virtual Memory, Paging

## แนวคิดหลัก (Concept)

บทที่ 3 สอน Stack กับ Heap ในฐานะ "พื้นที่หน่วยความจำของโปรแกรมหนึ่ง" โดยยังไม่ได้อธิบายว่าเมื่อมีหลายโปรแกรมรันพร้อมกัน (บทที่ 44 พูดถึงหลาย process แข่งกันขอ CPU) แต่ละโปรแกรมได้หน่วยความจำ (RAM) มาจากไหน และทำไมโปรแกรมหนึ่งจึงไม่สามารถเข้าไปยุ่งกับหน่วยความจำของอีกโปรแกรมได้โดยบังเอิญ บทนี้อธิบายกลไกเบื้องหลังที่ทำให้เรื่องนี้เป็นไปได้ — **Virtual Memory** และ **Paging**

- **Physical Memory (หน่วยความจำจริง)** — RAM จริงที่ติดตั้งอยู่ในเครื่อง มีขนาดจำกัดตายตัว (เช่น 16 GB) ที่ต้องแบ่งปันให้ทุก process ที่รันอยู่พร้อมกัน
- **Virtual Memory (หน่วยความจำเสมือน)** — ภาพลวงตาที่ OS สร้างขึ้นให้**แต่ละ process เชื่อว่าตัวเองมีหน่วยความจำเป็นของตัวเองทั้งหมด** (เช่น เห็นพื้นที่แอดเดรสเต็ม 4GB หรือมากกว่า) แม้ RAM จริงจะเล็กกว่าหรือถูกใช้ร่วมกับ process อื่นอยู่ก็ตาม
- **Paging** — เทคนิคที่แบ่งทั้ง Virtual Memory และ Physical Memory ออกเป็นชิ้นเล็กๆ ขนาดเท่ากัน (**Page** ในฝั่ง virtual, **Frame** ในฝั่ง physical) แล้วใช้ **Page Table** เป็นตัวจับคู่ระหว่างสองฝั่ง
- **Page Fault** — เหตุการณ์ที่ process พยายามเข้าถึง page ที่ยังไม่ถูกโหลดเข้า RAM จริง (อาจถูกเก็บไว้ที่ disk แทน) ทำให้ OS ต้องหยุดโปรแกรมชั่วคราวเพื่อโหลดข้อมูลจาก disk เข้า RAM ก่อน

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **Virtual Memory มีไว้ให้แต่ละ process ทำงานราวกับว่าตัวเองเป็นเจ้าของเครื่องคนเดียว**: ถ้าไม่มี Virtual Memory ทุก process ต้องรู้จักและจัดการ address ของ RAM จริงร่วมกัน (แบบเดียวกับ Race Condition ในบทที่ 37 แต่ระดับหน่วยความจำทั้งเครื่อง) — เสี่ยงต่อการเขียนทับข้อมูลของ process อื่นโดยบังเอิญหรือโดยเจตนาร้าย Virtual Memory แยก address space ของแต่ละ process ออกจากกันอย่างสมบูรณ์ ทำให้ process หนึ่งไม่มีทางเข้าถึงหน่วยความจำของอีก process ได้เลยนอกจากผ่านกลไกที่ OS อนุญาตชัดเจน
- **Paging มีไว้ให้จัดสรรหน่วยความจำได้ยืดหยุ่นโดยไม่ต้องการพื้นที่ต่อเนื่องกัน**: ถ้าต้องจัดสรรหน่วยความจำเป็นก้อนใหญ่ต่อเนื่องกันเสมอ (แบบเดียวกับการหาพื้นที่ว่างต่อเนื่องในบทที่ 12: Arrays) จะเกิดปัญหา **External Fragmentation** (มีพื้นที่ว่างรวมกันพอ แต่กระจัดกระจายจนหาก้อนต่อเนื่องไม่ได้) — Paging แบ่งเป็นชิ้นเล็กๆ เท่ากันทำให้จัดสรรที่ไหนก็ได้โดยไม่ต้องต่อเนื่องกันจริง
- **Page Fault + การเก็บข้อมูลบางส่วนไว้ที่ disk มีไว้ทำให้โปรแกรมใช้หน่วยความจำได้มากกว่า RAM จริงที่มี**: เหมือน `Least Recently Used Cache` ที่ต้องเลือกว่าจะเก็บอะไรไว้ใน cache ที่มีจำกัด OS ก็เลือกเก็บเฉพาะ page ที่กำลังถูกใช้งานบ่อยไว้ใน RAM จริง ส่วนที่เหลือพักไว้ที่ disk (**Swap Space**) ทำให้รันโปรแกรมที่ต้องการหน่วยความจำมากกว่า RAM จริงได้ (แม้จะช้าลงเมื่อต้องสลับไปมาบ่อย)

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Virtual Memory** เหมือน **ห้องพักโรงแรมที่แขกแต่ละคนเห็นแค่ห้องเลขที่ของตัวเอง (เช่น "ห้อง 305")** — แขกไม่จำเป็นต้องรู้ว่าจริงๆ แล้วโรงแรมจัดสรรพื้นที่จริงให้ห้องนั้นอย่างไร (อาจอยู่ชั้นไหนก็ได้) และแขกห้อง 305 ไม่มีทางเดินเข้าไปในห้อง 306 ของคนอื่นได้โดยบังเอิญ (แยก address space)
- **Paging** เหมือน **ตู้ล็อกเกอร์ที่แบ่งเป็นช่องขนาดเท่ากันทั้งหมด** — แทนที่จะต้องหาพื้นที่เก็บของต่อเนื่องกันเป็นก้อนใหญ่ (ซึ่งหายากถ้าตู้ล็อกเกอร์เต็มไปด้วยของกระจัดกระจาย) แบ่งของใส่หลายช่องเล็กๆ ที่ไหนก็ได้ในตู้ แล้วจดบันทึกไว้ว่าของแต่ละชิ้นอยู่ช่องไหนบ้าง (Page Table)
- **Page Fault** เหมือน **การไปหยิบของจากโกดังเก็บของนอกสถานที่เมื่อของที่ต้องการไม่ได้อยู่ในตู้ล็อกเกอร์ที่ใกล้มือ** — ใช้เวลานานกว่าการหยิบจากตู้ล็อกเกอร์ (RAM) มาก เพราะต้องเดินทางไปโกดัง (disk) ก่อน

## แผนภาพอธิบาย (Visual explanation)

### Virtual Address Space — แต่ละ Process เห็นหน่วยความจำเป็นของตัวเอง

```
Process A เห็น:                Process B เห็น:                RAM จริง (Physical Memory):

Virtual Address 0x1000          Virtual Address 0x1000          Physical Address 0x5000 (ของ A)
  |                               |                             Physical Address 0x8000 (ของ B)
  v (แปลผ่าน Page Table ของ A)      v (แปลผ่าน Page Table ของ B)     ...

ทั้งสอง process เห็น address "0x1000" เหมือนกัน แต่ถูกแปลไปยังตำแหน่งจริงที่ต่างกันโดยสิ้นเชิง
-> A ไม่มีทางเข้าถึงหน่วยความจำจริงของ B ได้เลยผ่าน address นี้ (แยกกันอย่างสมบูรณ์)
```

### Paging — แบ่ง Virtual Memory เป็น Page, Physical Memory เป็น Frame

```
Virtual Memory ของ Process (แบ่งเป็น Page ขนาดเท่ากัน):
[Page 0][Page 1][Page 2][Page 3][Page 4]

Page Table (จับคู่ Page -> Frame):
Page 0 -> Frame 5
Page 1 -> Frame 2      <- ไม่จำเป็นต้องเรียงติดกันในหน่วยความจำจริงเลย!
Page 2 -> (อยู่ที่ Disk/Swap, ยังไม่โหลดเข้า RAM)
Page 3 -> Frame 9
Page 4 -> Frame 1

Physical Memory (RAM จริง แบ่งเป็น Frame):
[Frame 0][Frame 1=P4][Frame 2=P1][Frame 3][Frame 4][Frame 5=P0]...[Frame 9=P3]

Page 1, 4, 0, 3 ของ process นี้กระจัดกระจายอยู่คนละที่ใน RAM จริง แต่ process มองว่าต่อเนื่องกัน
```

### Page Fault — เมื่อเข้าถึง Page ที่ยังไม่อยู่ใน RAM

```
Process พยายามอ่านข้อมูลที่ Virtual Address ใน Page 2

CPU ตรวจ Page Table -> Page 2 ไม่มีอยู่ใน Frame ไหนเลย (อยู่ที่ Disk) -> เกิด PAGE FAULT!

ขั้นตอนที่ OS ทำ:
1. หยุด process ชั่วคราว
2. หา Frame ว่างใน RAM (ถ้าไม่มีว่าง ต้องเลือก Frame ใดตัวหนึ่งมา "สลับออก" ไปที่ Disk ก่อน)
3. โหลดข้อมูล Page 2 จาก Disk เข้า Frame ที่เตรียมไว้
4. อัปเดต Page Table: Page 2 -> Frame ที่เพิ่งโหลดเข้าไป
5. ให้ process ทำงานต่อ (ตอนนี้เข้าถึง Page 2 ได้แล้ว)

Page Fault ใช้เวลานานกว่าการเข้าถึง RAM ปกติมาก (เพราะต้องรอ disk I/O) -> ควรเกิดให้น้อยที่สุด
```

## ตัวอย่างโค้ด (Code example)

```python
import resource
import sys

# ===== ดูการใช้หน่วยความจำจริงของโปรแกรม Python (ผ่าน OS) =====
usage = resource.getrusage(resource.RUSAGE_SELF)
print(f"Peak memory usage: {usage.ru_maxrss} KB")        # หน่วยความจำสูงสุดที่โปรแกรมนี้เคยใช้จริง

# ===== จำลอง Page Table แบบง่าย (แนวคิด ไม่ใช่ของจริงระดับ OS) =====
class SimplePageTable:
    def __init__(self, num_pages, num_frames):
        self.page_table = {}                              # dict: page_number -> frame_number หรือ None
        self.frames_in_use = [None] * num_frames             # จำลอง RAM จริง (Frame ว่าง = None)
        self.disk_storage = {}                               # จำลอง Disk/Swap สำหรับ page ที่ไม่ได้อยู่ใน RAM

    def access_page(self, page_number):
        if page_number in self.page_table and self.page_table[page_number] is not None:
            frame = self.page_table[page_number]
            print(f"Page {page_number}: HIT -> Frame {frame} (เร็ว, อ่านจาก RAM ตรงๆ)")
            return
        # Page Fault: page นี้ไม่อยู่ใน RAM
        print(f"Page {page_number}: PAGE FAULT! ต้องโหลดจาก Disk...")
        free_frame = self._find_free_frame()
        if free_frame is None:                                # RAM เต็ม -> ต้องเลือก page เก่ามาสลับออก (evict)
            free_frame = self._evict_page()
        self.page_table[page_number] = free_frame
        self.frames_in_use[free_frame] = page_number
        print(f"Page {page_number}: โหลดเข้า Frame {free_frame} เรียบร้อย")

    def _find_free_frame(self):
        for i, occupant in enumerate(self.frames_in_use):
            if occupant is None:
                return i
        return None

    def _evict_page(self):                                    # เลือก Frame แรกสุดมาสลับออก (FIFO อย่างง่าย)
        victim_frame = 0
        victim_page = self.frames_in_use[victim_frame]
        print(f"RAM เต็ม! สลับ Page {victim_page} ออกไปที่ Disk")
        self.page_table[victim_page] = None                     # page นั้นไม่อยู่ใน RAM แล้ว
        return victim_frame

pt = SimplePageTable(num_pages=5, num_frames=3)               # RAM เล็กกว่าจำนวน page ที่ต้องการ (จำลอง Page Fault)
pt.access_page(0)     # Page Fault -> โหลดเข้า Frame 0
pt.access_page(1)     # Page Fault -> โหลดเข้า Frame 1
pt.access_page(0)     # HIT! อยู่ใน RAM แล้ว
pt.access_page(2)     # Page Fault -> โหลดเข้า Frame 2 (RAM เต็มพอดี 3/3)
pt.access_page(3)     # RAM เต็ม! ต้อง evict page 0 ออกก่อนถึงจะโหลด page 3 ได้
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `access_page(page_number)` เช็คก่อนว่า page นั้นมี entry ใน `page_table` และมี frame ที่ใช้งานได้จริงหรือไม่ — ถ้ามี (**HIT**) แปลว่าข้อมูลอยู่ใน RAM แล้ว เข้าถึงได้เร็วทันที ไม่ต้องทำอะไรเพิ่ม
2. ถ้าไม่มี (**Page Fault**) โปรแกรมจำลองการทำงานของ OS จริง: หา frame ว่างก่อน (`_find_free_frame`) ถ้าไม่มี frame ว่างเลย (RAM เต็ม) ต้องเรียก `_evict_page()` เพื่อเลือก page หนึ่งที่อยู่ใน RAM มา "เขี่ยออก" (สมมติว่าส่งกลับไปที่ disk) ก่อนถึงจะมีที่ว่างให้ page ใหม่เข้ามาได้
3. `_evict_page()` ในตัวอย่างนี้ใช้กลยุทธ์**อย่างง่ายที่สุด**คือเลือก frame แรกสุดเสมอ (คล้าย FIFO) — ในระบบจริง OS ใช้อัลกอริทึมที่ซับซ้อนกว่านี้มาก เช่น **LRU (Least Recently Used)** ที่เลือก page ที่ไม่ได้ถูกใช้งานนานที่สุดมาสลับออกก่อน (หลักการเดียวกับ LRU Cache ที่มักถูกถามในสัมภาษณ์งาน ซึ่งใช้ Hash Table + Doubly Linked List จากบทที่ 13, 15 ร่วมกัน)
4. `pt.access_page(3)` ในตัวอย่างสุดท้าย — เพราะ RAM (3 frames) เต็มไปด้วย Page 0, 1, 2 แล้ว การเข้าถึง Page 3 บังคับให้เกิดการ **evict** ก่อนเสมอ — นี่คือสถานการณ์ **Thrashing** ในรูปแบบย่อ: ถ้าโปรแกรมสลับไปมาระหว่าง page ที่มากกว่าจำนวน frame ที่มีอยู่บ่อยเกินไป ระบบจะเสียเวลาส่วนใหญ่ไปกับการโหลด/สลับ page แทนที่จะทำงานจริง

## Time Complexity

- การเข้าถึง page ที่อยู่ใน RAM (**Page Table Hit**): **O(1)** โดยประมาณ (ฮาร์ดแวร์ช่วยเร่งด้วย **TLB — Translation Lookaside Buffer** ซึ่งเป็น cache พิเศษสำหรับการแปล address)
- **Page Fault**: ช้ากว่า Page Hit **หลายพันถึงหลายหมื่นเท่า** เพราะต้องรอ disk I/O ซึ่งช้ากว่า RAM มาก (ทบทวนจากบทที่ 1: speed hierarchy ของ CPU/RAM/Disk)
- อัลกอริทึมเลือก page ที่จะ evict (เช่น LRU) มี complexity ขึ้นกับการ implement: แบบ Hash Table + Doubly Linked List ให้ **O(1)** ต่อการเข้าถึงและอัปเดต

## Space Complexity

- Page Table ใช้พื้นที่ **O(จำนวน page ทั้งหมดของ process)** — process ที่มี virtual address space ใหญ่มากต้องการ Page Table ที่ใหญ่ตามไปด้วย (ระบบจริงใช้ **Multi-level Page Table** เพื่อประหยัดพื้นที่เมื่อ process ใช้ address space แค่บางส่วน)

## แนวทางปฏิบัติที่ดี (Best Practices)

- เมื่อเขียนโปรแกรมที่ประมวลผลข้อมูลขนาดใหญ่ ควรเข้าถึงข้อมูลแบบ**ต่อเนื่อง (sequential access)** มากกว่าการกระโดดไปมาแบบสุ่ม (random access) เพราะข้อมูลที่อยู่ติดกันมักถูกจัดให้อยู่ใน page เดียวกันหรือใกล้กัน ลดโอกาสเกิด Page Fault
- หลีกเลี่ยงการสร้างโปรแกรมที่ใช้หน่วยความจำมากเกินกว่า RAM จริงอย่างมีนัยสำคัญ เพราะจะเกิด **Thrashing** (สลับ page ไปมาบ่อยจนประสิทธิภาพตกลงมาก) — ถ้าจำเป็นต้องประมวลผลข้อมูลใหญ่กว่า RAM ให้พิจารณาประมวลผลเป็นส่วนๆ (chunk) แทนการโหลดทั้งหมดพร้อมกัน
- เข้าใจว่า Virtual Memory ทำให้โปรแกรมทำงานได้แม้ RAM ไม่พอ แต่ **ไม่ใช่ทางแก้ปัญหาประสิทธิภาพ** — ถ้าโปรแกรมพึ่งพา swap บ่อยเกินไป วิธีแก้ที่แท้จริงคือลดการใช้หน่วยความจำหรือเพิ่ม RAM จริง ไม่ใช่พึ่งพา Virtual Memory ตลอดไป

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- เข้าใจผิดว่า Virtual Memory คือหน่วยความจำที่ "ไม่จำกัด" จริงๆ — ที่จริงมันแค่ทำให้แต่ละ process มองเห็น address space ที่ใหญ่กว่า RAM จริง แต่ประสิทธิภาพยังถูกจำกัดด้วย RAM จริงและความเร็วของ disk เมื่อต้อง swap บ่อย
- เขียนโปรแกรมที่เข้าถึงข้อมูลแบบสุ่มกระจัดกระจายในชุดข้อมูลขนาดใหญ่ โดยไม่ตระหนักว่าจะทำให้เกิด Page Fault บ่อยกว่าการเข้าถึงแบบต่อเนื่อง ส่งผลให้ช้ากว่าที่คาดคิดมาก
- คิดว่า Page Fault คือข้อผิดพลาด (error) ของโปรแกรม — ที่จริงเป็นกลไกปกติของระบบ Virtual Memory ที่เกิดขึ้นเสมอเมื่อเริ่มเข้าถึง page ใหม่ที่ยังไม่เคยโหลด (First-time access) ปัญหาจะเกิดก็ต่อเมื่อเกิดถี่เกินไปจนกลายเป็น Thrashing
- สับสนระหว่าง Physical Address กับ Virtual Address — คิดว่า address ที่เห็นในโปรแกรม debugger คือตำแหน่งจริงบน RAM ทั้งที่จริงๆ มันคือ Virtual Address ที่ต้องแปลผ่าน Page Table ก่อนเสมอ

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบาย Virtual Memory คืออะไร แก้ปัญหาอะไรที่การใช้ Physical Memory ตรงๆ ทำไม่ได้
2. Paging คืออะไร ทำไมถึงช่วยแก้ปัญหา External Fragmentation ได้
3. Page Fault คืออะไร เกิดขึ้นเมื่อไหร่ และทำไมถึงช้ากว่าการเข้าถึง RAM ปกติมาก
4. อธิบาย Thrashing คืออะไร เกิดขึ้นได้อย่างไร และมีวิธีป้องกันอย่างไร
5. LRU (Least Recently Used) เป็นอัลกอริทึมที่ใช้ในบริบทไหนของ Memory Management เชื่อมโยงกับโครงสร้างข้อมูลที่เคยเรียนมาอย่างไร

## แบบฝึกหัด (Practice Exercises)

1. ปรับ `SimplePageTable` ในตัวอย่างให้ใช้อัลกอริทึม **LRU** แทน FIFO ในการเลือก page ที่จะ evict (ใบ้: เก็บลำดับการเข้าถึงล่าสุดของแต่ละ page ไว้)
2. เขียนโปรแกรมทดสอบเปรียบเทียบเวลาที่ใช้อ่านข้อมูลจาก array ขนาดใหญ่แบบ**ต่อเนื่อง** (index 0, 1, 2, 3, ...) กับแบบ**สุ่ม** (index สุ่มไม่ซ้ำ) แล้ววิเคราะห์ว่าทำไมแบบต่อเนื่องมักเร็วกว่า (เชื่อมโยงกับ Page Fault และ Cache)
3. คำนวณด้วยมือ: ถ้า process มี virtual address space 4GB และแต่ละ page มีขนาด 4KB จะมี page ทั้งหมดกี่ page และถ้า Page Table Entry แต่ละตัวใช้ 4 bytes จะต้องใช้พื้นที่เก็บ Page Table ทั้งหมดกี่ MB

## Mini Project

**"เครื่องมือจำลอง Virtual Memory และเปรียบเทียบ Page Replacement Algorithm (Virtual Memory & Page Replacement Simulator)"**: เขียนโปรแกรมที่:
1. จำลอง Page Table และ RAM (จำนวน frame จำกัด) ตามตัวอย่างในบทเรียน แต่ implement Page Replacement Algorithm 3 แบบ: FIFO, LRU, และ **Optimal** (เลือก page ที่จะไม่ถูกใช้นานที่สุดในอนาคต — ใช้เป็นมาตรฐานเปรียบเทียบทางทฤษฎี แม้ในโลกจริงจะทำนายอนาคตไม่ได้)
2. รับลำดับการเข้าถึง page (page reference string) จำลอง แล้วรันผ่านทั้ง 3 อัลกอริทึม นับจำนวน Page Fault ที่เกิดขึ้นในแต่ละแบบ
3. ทดลองกับจำนวน frame ต่างกัน (3, 5, 10 frames) เพื่อดูว่าจำนวน Page Fault ลดลงอย่างไรเมื่อ RAM มีมากขึ้น
4. เขียนรายงานเปรียบเทียบว่าทำไม LRU มักใกล้เคียงกับ Optimal มากกว่า FIFO และอธิบายว่าทำไมในระบบจริงถึงเลือกใช้ LRU (หรือตัวแปรของมัน) แทนที่จะพยายามใช้ Optimal ตรงๆ

## ข้อคิดสำคัญ (Key Takeaways)

- Virtual Memory สร้างภาพลวงตาให้แต่ละ process เห็นหน่วยความจำเป็นของตัวเองทั้งหมด แยกจาก process อื่นอย่างสมบูรณ์ผ่านการแปล address
- Paging แบ่งทั้ง Virtual Memory (เป็น Page) และ Physical Memory (เป็น Frame) เป็นชิ้นเล็กๆ ขนาดเท่ากัน แก้ปัญหา External Fragmentation ที่เกิดจากการต้องการพื้นที่ต่อเนื่องกัน
- Page Fault เกิดขึ้นเมื่อเข้าถึง page ที่ยังไม่อยู่ใน RAM จริง ต้องโหลดจาก disk ซึ่งช้ากว่าการเข้าถึง RAM มาก
- เมื่อ RAM เต็มต้องเลือก page มา evict ออก อัลกอริทึมที่นิยมคือ LRU (เลือก page ที่ไม่ได้ใช้นานที่สุด) ซึ่งใกล้เคียงกับ Optimal มากกว่า FIFO
- Thrashing เกิดเมื่อโปรแกรมสลับ page ไปมาบ่อยเกินไปจนประสิทธิภาพตกลงอย่างมาก การเข้าถึงข้อมูลแบบต่อเนื่องช่วยลดโอกาสนี้ได้
- บทถัดไปจะเรียน Process Communication (IPC) ซึ่งปิดท้าย Part 11: Operating Systems โดยอธิบายว่าเมื่อ process ถูกแยก address space ออกจากกันอย่างสมบูรณ์แล้ว (ตามที่เรียนในบทนี้) จะสื่อสารแลกเปลี่ยนข้อมูลกันได้อย่างไร

---
*บทต่อไป (บทที่ 46): Process Communication — พิมพ์ "Next" เพื่อดำเนินการต่อ*
