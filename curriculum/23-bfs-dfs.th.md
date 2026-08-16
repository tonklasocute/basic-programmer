# บทที่ 23: BFS / DFS

## แนวคิดหลัก (Concept)

บทที่ 19 สอนวิธี**เก็บ** Graph ไว้ในโปรแกรม (Adjacency List/Matrix) แต่ยังไม่ได้สอนวิธี**เดิน**ไปทั่วทั้ง Graph นั้น บทนี้แนะนำสองอัลกอริทึมพื้นฐานที่สุดในการ traverse ทั้ง Graph และ Tree — **BFS** และ **DFS** — ซึ่งเป็นรากฐานของอัลกอริทึมขั้นสูงเกือบทุกตัวที่เกี่ยวกับ Graph (หาเส้นทางสั้นที่สุด, ตรวจ cycle, topological sort, ฯลฯ)

- **BFS (Breadth-First Search)** — เดินสำรวจ**ทีละชั้น** (level by level) จากจุดเริ่มต้น ไปเยี่ยมเพื่อนบ้านทุกตัวที่อยู่ห่าง 1 ก้าวก่อน แล้วค่อยไปห่าง 2 ก้าว ใช้ **Queue** (FIFO จากบทที่ 14) เป็นตัวช่วยจำลำดับ
- **DFS (Depth-First Search)** — เดินสำรวจ**ลึกที่สุดก่อน** ไปตามเส้นทางเดียวจนสุดทาง แล้วค่อยถอยกลับ (backtrack) มาลองเส้นทางอื่น ใช้ **Stack** (LIFO จากบทที่ 14) หรือ **Recursion** (บทที่ 9 — เพราะ call stack ก็คือ stack) เป็นตัวช่วยจำลำดับ
- **Visited Set** — โครงสร้างข้อมูล (ปกติใช้ Hash Set จากบทที่ 15) ที่เก็บว่าเคยไป vertex ไหนมาแล้วบ้าง จำเป็นเสมอเมื่อ Graph มี cycle (ตามที่เตือนไว้ในบทที่ 19) เพื่อไม่ให้วนซ้ำไม่รู้จบ

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **มีไว้ตอบคำถามพื้นฐานที่สุดของ Graph: "จากจุด A ไปจุด B ได้ไหม และไปอย่างไร"**: ถ้าไม่มีวิธี traverse ที่เป็นระบบ การมี Graph เก็บไว้เฉยๆ ก็ใช้ประโยชน์อะไรไม่ได้เลย ทั้ง BFS และ DFS คือวิธี "เดินให้ครบทุกจุดที่ไปถึงได้" อย่างเป็นระบบและไม่วนซ้ำ
- **BFS มีไว้หาเส้นทางที่ "สั้นที่สุด" (จำนวน edge น้อยที่สุด) ใน Unweighted Graph**: เพราะ BFS สำรวจทีละชั้นเสมอ จุดแรกที่เจอเป้าหมายจึงรับประกันว่ามาจากเส้นทางที่สั้นที่สุด (เช่น "ระดับความสัมพันธ์" บน LinkedIn: เพื่อนของเพื่อนคือระดับ 2 เสมอ ไม่มีทางเจอระดับ 2 ก่อนระดับ 1)
- **DFS มีไว้สำรวจ "ให้ครบทุกความเป็นไปได้" อย่างมีประสิทธิภาพด้านหน่วยความจำ**: เหมาะกับปัญหาที่ต้องลองทุกเส้นทางจนสุด เช่น ตรวจสอบว่า Graph มี cycle หรือไม่, หา connected component, หรือแก้ปัญหา Backtracking (จะเรียนในบทถัดๆ ไป) เพราะ DFS ใช้หน่วยความจำแค่ตามความลึกของเส้นทาง ไม่ใช่ตามความกว้างเหมือน BFS
- **เป็นรากฐานของอัลกอริทึมขั้นสูงแทบทุกตัวบน Graph**: Dijkstra's Algorithm (shortest path แบบมีน้ำหนัก), Topological Sort, การตรวจ cycle, การหา connected components — ล้วนดัดแปลงมาจาก BFS หรือ DFS ทั้งสิ้น การเข้าใจสองอัลกอริทึมนี้ให้แน่นคือกุญแจสำคัญของทั้ง Part 5

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **BFS** เหมือน **หยดสีลงในน้ำ** — สีจะกระจายออกเป็นวงกลมทีละชั้นรอบจุดที่หยด ชั้นในสุดกระจายเต็มก่อนจะขยายไปชั้นถัดไปเสมอ ไม่มีทางที่ชั้นนอกจะกระจายไปถึงก่อนชั้นใน
- **DFS** เหมือน **การเดินเข้าเขาวงกต (maze) โดยใช้มือแตะกำแพงด้านขวาตลอดเวลา** — เดินลึกเข้าไปเรื่อยๆ ตามทางเดียวจนสุดทางตัน แล้วค่อยถอยหลังกลับมาจุดแยกล่าสุดเพื่อลองทางที่ยังไม่ได้ไป ทำซ้ำจนครบทุกทาง
- **Visited Set** เหมือน **การขีดกากบาทบนแผนที่ทุกจุดที่เคยเดินผ่าน** — ป้องกันไม่ให้เดินวนกลับมาที่เดิมซ้ำๆ ไม่รู้จบ

## แผนภาพอธิบาย (Visual explanation)

### Graph ตัวอย่างที่จะใช้ตลอดบทนี้

```
        A
       / \
      B   C
      |   |
      D   E
       \ /
        F

Adjacency List:
A: [B, C]
B: [A, D]
C: [A, E]
D: [B, F]
E: [C, F]
F: [D, E]
```

### BFS จาก A — เดินทีละชั้น ใช้ Queue

```
Queue เริ่มต้น: [A]          Visited: {A}

รอบ 1: ดึง A ออก -> เยี่ยม A
       เพื่อนบ้านของ A: B, C (ยังไม่เคยไป) -> ใส่ Queue
       Queue: [B, C]          Visited: {A, B, C}

รอบ 2: ดึง B ออก -> เยี่ยม B
       เพื่อนบ้านของ B: A(ไปแล้ว), D -> ใส่ Queue
       Queue: [C, D]          Visited: {A, B, C, D}

รอบ 3: ดึง C ออก -> เยี่ยม C
       เพื่อนบ้านของ C: A(ไปแล้ว), E -> ใส่ Queue
       Queue: [D, E]          Visited: {A, B, C, D, E}

รอบ 4: ดึง D ออก -> เยี่ยม D
       เพื่อนบ้านของ D: B(ไปแล้ว), F -> ใส่ Queue
       Queue: [E, F]          Visited: {A, B, C, D, E, F}

รอบ 5: ดึง E ออก -> เยี่ยม E (เพื่อนบ้านไปแล้วหมด)
       Queue: [F]

รอบ 6: ดึง F ออก -> เยี่ยม F (เพื่อนบ้านไปแล้วหมด)
       Queue: []  -> จบ

ลำดับการเยี่ยม: A -> B -> C -> D -> E -> F   (ทีละ "ชั้น" จาก A: ชั้น0=A, ชั้น1=B,C, ชั้น2=D,E, ชั้น3=F)
```

### DFS จาก A — เดินลึกที่สุดก่อน ใช้ Stack/Recursion

```
เริ่มที่ A -> เยี่ยม A -> Visited: {A}
  ไป B (เพื่อนบ้านตัวแรกของ A) -> เยี่ยม B -> Visited: {A, B}
    ไป D (เพื่อนบ้านตัวแรกที่ยังไม่ไปของ B) -> เยี่ยม D -> Visited: {A, B, D}
      ไป F (เพื่อนบ้านตัวแรกที่ยังไม่ไปของ D) -> เยี่ยม F -> Visited: {A, B, D, F}
        เพื่อนบ้านของ F คือ D(ไปแล้ว), E -> ไป E -> เยี่ยม E -> Visited: {..., E}
          เพื่อนบ้านของ E คือ C(ยังไม่ไป), F(ไปแล้ว) -> ไป C -> เยี่ยม C
            เพื่อนบ้านของ C คือ A(ไปแล้ว), E(ไปแล้ว) -> ทางตัน ถอยกลับ (backtrack)
          กลับมาที่ E: ไปครบแล้ว -> backtrack
        กลับมาที่ F: ไปครบแล้ว -> backtrack
      กลับมาที่ D: ไปครบแล้ว -> backtrack
    กลับมาที่ B: ไปครบแล้ว -> backtrack
  กลับมาที่ A: เพื่อนบ้าน C ไปแล้ว(จากเส้นทางลึก) -> ไปครบแล้ว -> จบ

ลำดับการเยี่ยม: A -> B -> D -> F -> E -> C   (ลึกสุดก่อนเสมอ ต่างจาก BFS โดยสิ้นเชิง)
```

### เปรียบเทียบ BFS vs DFS ด้วยภาพเดียว

```
Graph เดียวกัน:        BFS จาก A (ทีละชั้น):     DFS จาก A (ลึกก่อน):
      A                    ชั้น 0: A                A
     / \                   ชั้น 1: B  C             └─ B
    B   C                  ชั้น 2: D  E                 └─ D
    |   |                  ชั้น 3: F                        └─ F
    D   E                                                        └─ E
     \ /                  โครงสร้าง: Queue                            └─ C
      F                   (FIFO) เข้าก่อนออกก่อน      โครงสร้าง: Stack/Recursion
                                                        (LIFO) เข้าหลังออกก่อน
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== Graph ด้วย Adjacency List (จากบทที่ 19) =====
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "E"],
    "D": ["B", "F"],
    "E": ["C", "F"],
    "F": ["D", "E"],
}

# ===== BFS: ใช้ Queue (collections.deque จากบทที่ 14 — O(1) popleft) =====
from collections import deque

def bfs(graph, start):
    visited = {start}                 # ใส่ start ลง visited ทันทีตอนเข้า queue (สำคัญมาก - ดู Common Mistakes)
    queue = deque([start])
    order = []                        # เก็บลำดับการเยี่ยมไว้ดู
    while queue:
        vertex = queue.popleft()      # FIFO: ดึงตัวที่เข้าคิวก่อนสุดออกมาก่อน
        order.append(vertex)
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                visited.add(neighbor)     # mark ทันทีตอนเพิ่มเข้า queue ไม่ใช่ตอนดึงออกมา
                queue.append(neighbor)
    return order

# ===== DFS แบบ Iterative: ใช้ Stack =====
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    order = []
    while stack:
        vertex = stack.pop()          # LIFO: ดึงตัวที่เพิ่งเข้าไปล่าสุดออกมาก่อน
        if vertex not in visited:      # mark ตอนดึงออกมา (ต่างจาก BFS! ดู Step-by-step)
            visited.add(vertex)
            order.append(vertex)
            for neighbor in graph[vertex]:
                if neighbor not in visited:
                    stack.append(neighbor)
    return order

# ===== DFS แบบ Recursive: ใช้ Call Stack (เชื่อมโยงบทที่ 9) =====
def dfs_recursive(graph, start, visited=None, order=None):
    if visited is None:
        visited = set()
        order = []
    visited.add(start)
    order.append(start)
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited, order)   # Recursive Case
    return order                                               # Base Case โดยนัย: ไม่มีเพื่อนบ้านใหม่ให้ไปต่อ

print("BFS:", bfs(graph, "A"))                # ['A', 'B', 'C', 'D', 'E', 'F']
print("DFS iterative:", dfs_iterative(graph, "A"))   # ['A', 'C', 'E', 'F', 'D', 'B'] (ลำดับขึ้นกับ stack)
print("DFS recursive:", dfs_recursive(graph, "A"))   # ['A', 'B', 'D', 'F', 'E', 'C']

# ===== การใช้งานจริง: หาเส้นทางสั้นที่สุดด้วย BFS (Unweighted Shortest Path) =====
def bfs_shortest_path(graph, start, target):
    if start == target:
        return [start]
    visited = {start}
    queue = deque([[start]])          # เก็บ "เส้นทางทั้งหมด" ไม่ใช่แค่ vertex เดียว
    while queue:
        path = queue.popleft()
        vertex = path[-1]
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                new_path = path + [neighbor]
                if neighbor == target:
                    return new_path         # เจอทันทีที่ระดับตื้นที่สุด = สั้นที่สุดเสมอ
                visited.add(neighbor)
                queue.append(new_path)
    return None                              # ไม่มีเส้นทางไปถึง

print(bfs_shortest_path(graph, "A", "F"))    # ['A', 'B', 'D', 'F'] หรือ ['A', 'C', 'E', 'F'] (ยาว 3 เท่ากัน)
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. **`visited.add(start)` ก่อนเข้า `queue` ใน BFS** — ต้อง mark ว่า "visited" **ทันทีตอนใส่เข้า queue** ไม่ใช่ตอนดึงออกมาประมวลผล เพราะ BFS สำรวจหลาย vertex "พร้อมกัน" ในแต่ละชั้น ถ้ารอ mark ตอนดึงออก vertex เดียวกันอาจถูกเพิ่มเข้า queue ซ้ำหลายครั้งจากเพื่อนบ้านหลายตัวในชั้นเดียวกัน ทำให้ประมวลผลซ้ำโดยไม่จำเป็น (ไม่ทำให้ผลลัพธ์ผิด แต่เสียประสิทธิภาพ)
2. **`visited.add(vertex)` หลังดึงออกจาก `stack` ใน DFS iterative** — ตรงข้ามกับ BFS! เพราะ DFS ใส่เพื่อนบ้านเข้า stack โดยยังไม่รู้ว่าจะถูกดึงมาใช้จริงเมื่อไหร่ (อาจถูก vertex อื่นแซงคิวก่อน) การ mark ตอนดึงออกมาจึงถูกต้องกว่า และต้องเช็ค `if vertex not in visited` ซ้ำอีกทีตอนดึงออก เผื่อ vertex นั้นถูกใส่ซ้ำหลายครั้งใน stack ไปแล้ว
3. **`dfs_recursive` ไม่ต้องใช้ stack เอง** — เพราะทุกครั้งที่เรียกฟังก์ชันตัวเอง (`dfs_recursive(graph, neighbor, ...)`) ภาษาโปรแกรมจะสร้าง stack frame ใหม่ให้อัตโนมัติใน **Call Stack** (ตามที่เรียนในบทที่ 3 และ 9) ซึ่งทำงานแบบ LIFO เหมือน stack ที่เขียนเองทุกประการ — นี่คือเหตุผลที่ DFS "เป็นธรรมชาติ" กับ recursion มากกว่า BFS
4. **`bfs_shortest_path`** — เก็บ "เส้นทางทั้งหมด" (`path`) แทนที่จะเก็บแค่ vertex เดียวใน queue เพื่อให้สามารถคืนค่าเส้นทางแบบเต็มได้เมื่อเจอเป้าหมาย เพราะ BFS รับประกันว่า vertex แรกที่เจอ target มาจากเส้นทางที่สั้นที่สุดเสมอ (สำรวจทีละชั้น) จึง `return` ได้ทันทีโดยไม่ต้องรอสำรวจ vertex ที่เหลือ

## Time Complexity

- **BFS**: **O(V + E)** — เยี่ยมทุก vertex ครั้งเดียว (O(V)) และไล่ดู edge ของแต่ละ vertex ครั้งเดียวรวมกันทั้งหมด (O(E))
- **DFS** (ทั้ง iterative และ recursive): **O(V + E)** — หลักการเดียวกับ BFS เพราะทั้งคู่เยี่ยมทุก vertex และทุก edge อย่างละครั้งเท่านั้น

## Space Complexity

- **BFS**: **O(V)** — Queue อาจเก็บได้มากที่สุดเท่ากับจำนวน vertex ในชั้นที่กว้างที่สุด (worst case คือทุก vertex อยู่ชั้นเดียวกัน) บวกกับ `visited` set ที่เก็บทุก vertex
- **DFS Iterative**: **O(V)** — Stack ในกรณีแย่ที่สุดเก็บได้มากที่สุดเท่ากับความลึกของเส้นทาง บวกกับ vertex ที่ยังไม่ได้ประมวลผลซ้ำใน stack
- **DFS Recursive**: **O(V)** ในกรณีแย่ที่สุด — ใช้ Call Stack ตามความลึกของการเรียกซ้ำ (เชื่อมโยงบทที่ 3 และ 22) ถ้า Graph เป็นเส้นตรงยาว (เหมือน Linked List) ความลึกจะเท่ากับ V พอดี ซึ่งอาจทำให้เกิด **Stack Overflow** ในภาษาที่ไม่มี tail-call optimization ถ้า Graph ใหญ่มาก — เป็นข้อเสียของ recursive เมื่อเทียบกับ iterative

## แนวทางปฏิบัติที่ดี (Best Practices)

- เลือก BFS เมื่อโจทย์ถามหา "เส้นทางสั้นที่สุด" หรือ "ระยะห่างน้อยที่สุด" ใน Unweighted Graph เสมอ เพราะ DFS ไม่รับประกันเรื่องนี้เลย
- เลือก DFS เมื่อโจทย์ต้องการสำรวจ "ให้ครบทุกความเป็นไปได้" เช่น ตรวจ cycle, หา connected component, หรือปัญหาที่เกี่ยวกับ Backtracking (จะเรียนในบทถัดๆ ไป)
- ใช้ DFS แบบ recursive เมื่อ Graph ไม่ลึกมากและโค้ดที่อ่านง่ายสำคัญกว่า ใช้แบบ iterative เมื่อกังวลเรื่อง Stack Overflow กับ Graph ขนาดใหญ่
- Mark `visited` **ทันทีตอนค้นพบ** (ก่อนใส่เข้า queue/stack) สำหรับ BFS เสมอ เพื่อป้องกันการใส่ vertex ซ้ำเข้า queue โดยไม่จำเป็น

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ลืมสร้าง `visited` set ทำให้ traverse Graph ที่มี cycle วนไม่รู้จบ (infinite loop) — ย้ำจากคำเตือนในบทที่ 19
- ใน BFS: mark `visited` ตอน**ดึงออกจาก queue** แทนที่จะ mark ตอน**ใส่เข้า queue** ทำให้ vertex เดียวกันถูกใส่ซ้ำหลายครั้งโดยไม่จำเป็น (ไม่ผิด แต่เปลืองหน่วยความจำและเวลาโดยใช่เหตุ)
- สับสนระหว่าง Queue กับ Stack — ใช้ `list.pop(0)` (ซึ่งเป็น O(n) ใน Python เพราะต้องขยับข้อมูลทุกตัว) แทน `deque.popleft()` (O(1)) ตอนเขียน BFS ทำให้ performance แย่ลงมากเมื่อข้อมูลใหญ่
- คิดว่า DFS หาเส้นทางสั้นที่สุดได้เหมือน BFS — DFS ไม่รับประกันเรื่องนี้เลย เพราะเดินลึกก่อนโดยไม่สนใจว่าเส้นทางนั้นสั้นหรือยาว
- ใช้ DFS recursive กับ Graph ขนาดใหญ่มากที่มีลักษณะเป็นเส้นตรงยาว (deep) โดยไม่ระวังเรื่อง Stack Overflow

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง BFS และ DFS ทั้งในแง่โครงสร้างข้อมูลที่ใช้และลำดับการเยี่ยม vertex
2. ทำไม BFS ถึงรับประกันว่าเจอเส้นทางสั้นที่สุดใน Unweighted Graph แต่ DFS ไม่รับประกัน?
3. เขียน BFS และ DFS ด้วยมือ (ไม่ดูโค้ด) ทั้งแบบ iterative
4. DFS แบบ recursive กับ iterative ต่างกันอย่างไร แบบไหนเสี่ยง Stack Overflow มากกว่า และทำไม?
5. ถ้าไม่ใช้ `visited` set เลย จะเกิดอะไรขึ้นเมื่อ traverse Graph ที่มี cycle เทียบกับ Tree ที่ไม่มี cycle?

## แบบฝึกหัด (Practice Exercises)

1. เขียนฟังก์ชันใช้ BFS หา**จำนวน connected component** ทั้งหมดใน Undirected Graph ที่อาจไม่เชื่อมกันทั้งหมด (เช่น Graph ที่แบ่งเป็นกลุ่มแยกกัน 3 กลุ่ม ต้องได้คำตอบ 3)
2. เขียนฟังก์ชันใช้ DFS ตรวจสอบว่า Directed Graph มี cycle หรือไม่ (ใบ้: ต้องใช้ visited 2 แบบ คือ "เคยไปแล้วแบบถาวร" กับ "อยู่ในเส้นทางปัจจุบัน" เพื่อแยกแยะ back edge ออกจาก cross edge)
3. ดัดแปลง `bfs_shortest_path` ให้ทำงานกับ Grid 2 มิติ (เช่น เขาวงกตที่เป็น array ของ 0/1 โดย 0=เดินได้ 1=กำแพง) หาเส้นทางสั้นที่สุดจากจุดเริ่มต้นไปจุดหมาย

## Mini Project

**"ระบบหาระดับความสัมพันธ์บนโซเชียลมีเดีย (Social Network Degree Finder)"**: เขียนโปรแกรมที่:
1. สร้าง Undirected Graph จำลองเครือข่ายเพื่อน (ต่อยอดจาก Mini Project บทที่ 19) ใช้ Adjacency List
2. ใช้ BFS หา "ระดับความสัมพันธ์" ระหว่างผู้ใช้สองคน (เพื่อนตรง = ระดับ 1, เพื่อนของเพื่อน = ระดับ 2, ...) พร้อมแสดงเส้นทางที่เชื่อมทั้งสองคน
3. ใช้ DFS หา "กลุ่มเพื่อนที่แยกตัวออกไป" (connected component) ทั้งหมดในเครือข่าย เพื่อดูว่ามีกลุ่มคนที่ไม่เชื่อมกับกลุ่มหลักเลยหรือไม่
4. เปรียบเทียบผลลัพธ์และเวลาที่ใช้ระหว่าง BFS กับ DFS เมื่อค้นหา "เพื่อนที่ใกล้ที่สุดที่มีความสนใจตรงกัน" (สมมติแต่ละ user มี tag ความสนใจ) — วิเคราะห์ว่าทำไม BFS เหมาะกับโจทย์นี้มากกว่า

## ข้อคิดสำคัญ (Key Takeaways)

- BFS และ DFS คือสองวิธีพื้นฐานที่สุดในการ traverse Graph (และ Tree) ทั้งคู่มี Time Complexity O(V + E) เท่ากัน แต่ให้ลำดับการเยี่ยมและใช้กรณีต่างกันโดยสิ้นเชิง
- BFS ใช้ Queue (FIFO) เดินทีละชั้น เหมาะกับการหาเส้นทางสั้นที่สุดใน Unweighted Graph
- DFS ใช้ Stack (LIFO) หรือ Recursion เดินลึกที่สุดก่อน เหมาะกับการสำรวจให้ครบทุกความเป็นไปได้ เช่น ตรวจ cycle หรือ Backtracking
- ต้องมี visited set เสมอเมื่อ Graph มี cycle ได้ (ย้ำจากบทที่ 19) มิฉะนั้นจะวนไม่รู้จบ
- BFS/DFS เป็นรากฐานของอัลกอริทึมขั้นสูงเกือบทุกตัวบน Graph และเป็นแนวคิดเดียวกับ Divide and Conquer และ Backtracking ที่จะเรียนในบทถัดไป ซึ่งล้วนใช้หลักการ "แบ่ง/สำรวจปัญหาอย่างเป็นระบบ" แบบเดียวกัน

---
*บทต่อไป (บทที่ 24): Divide and Conquer — พิมพ์ "Next" เพื่อดำเนินการต่อ*
