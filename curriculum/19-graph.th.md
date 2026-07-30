# บทที่ 19: Graph

## แนวคิดหลัก (Concept)

ในบทที่ 16 เรียนว่า Tree มีกฎเข้มงวด: แต่ละ Node มี parent ได้แค่ 1 ตัว และห้ามมีวงจร (cycle) **Graph (กราฟ)** คือโครงสร้างข้อมูลที่**ปลดล็อกข้อจำกัดทั้งหมดนั้น** กลายเป็นโครงสร้างข้อมูลที่ทั่วไปที่สุดในบรรดา Data Structures ที่เรียนมาทั้งหมด — แท้จริงแล้ว **Tree, BST, Heap, Trie ล้วนเป็น Graph ชนิดพิเศษที่มีกฎเพิ่มเติม** เท่านั้นเอง

- **Graph** — ประกอบด้วย **Vertex/Node (จุด)** และ **Edge (เส้นเชื่อม)** ที่เชื่อมระหว่างจุดสองจุด โดยไม่มีข้อจำกัดเรื่องจำนวนการเชื่อมต่อหรือทิศทาง
- **Directed Graph (กราฟมีทิศทาง)** — Edge มีทิศทาง (A → B ไม่ได้แปลว่า B → A ได้ด้วย) เช่น ความสัมพันธ์ "ติดตาม" บน Twitter
- **Undirected Graph (กราฟไม่มีทิศทาง)** — Edge เชื่อมสองทางเสมอ (A-B เท่ากับ B-A) เช่น ความสัมพันธ์ "เพื่อน" บน Facebook
- **Weighted Graph (กราฟมีน้ำหนัก)** — แต่ละ Edge มีค่าตัวเลข (น้ำหนัก/ต้นทุน) กำกับ เช่น ระยะทางระหว่างเมือง
- **Cycle (วงจร)** — เส้นทางที่เดินวนกลับมาจุดเดิมได้ (Graph อนุญาตให้มี แต่ Tree ห้ามมีเด็ดขาด)

## ทำไมต้องมีสิ่งนี้ (Why it exists)

- **โลกจริงเต็มไปด้วยความสัมพันธ์ที่ซับซ้อนกว่าลำดับชั้น**: Tree เหมาะกับข้อมูลที่เป็นลำดับชั้นชัดเจน (บริษัท, ไฟล์) แต่ความสัมพันธ์แบบเครือข่ายสังคม (ใครก็ตามเป็นเพื่อนกับใครก็ได้ ไม่มี "หัวหน้า" ตายตัว), แผนที่ถนน (ถนนเชื่อมเมืองไปมาได้หลายทิศทาง), หรือ dependency ระหว่างโมดูลซอฟต์แวร์ (โมดูล A อาจ import โมดูล B ที่ import กลับมาที่ A โดยอ้อม) ล้วนมีธรรมชาติที่ไม่ใช่ลำดับชั้น
- **Weighted Graph มีไว้จำลอง "ต้นทุน" ของความสัมพันธ์**: ไม่ใช่ทุกเส้นทางจะมีค่าเท่ากัน (ระยะทางกรุงเทพ-เชียงใหม่ ต่างจากกรุงเทพ-นครปฐม) การใส่น้ำหนักบน Edge ทำให้คำนวณ "เส้นทางที่ดีที่สุด" ได้ (จะเรียนใน Dijkstra's Algorithm บทอัลกอริทึม)
- **Directed vs Undirected มีไว้สะท้อนธรรมชาติของความสัมพันธ์จริง**: บางความสัมพันธ์เป็นทางเดียว (A ติดตาม B ไม่ได้แปลว่า B ติดตาม A) บางความสัมพันธ์เป็นสองทางเสมอ (ถนนสองเลนที่ไปมาได้ทั้งสองทิศทาง) — การเลือกชนิด graph ที่ถูกต้องสำคัญต่อความถูกต้องของโมเดล

## เปรียบเทียบกับชีวิตจริง (Real-world analogy)

- **Graph** เหมือน **แผนที่เส้นทางการบิน** — สนามบิน (Vertex) เชื่อมกันด้วยเที่ยวบิน (Edge) เมืองหนึ่งอาจมีเที่ยวบินตรงไปได้หลายเมือง ไม่มีกฎว่าต้อง "เริ่มจากจุดเดียว" หรือ "ห้ามวนกลับ" เหมือน Tree
- **Directed Graph** เหมือน **ถนนเดินรถทางเดียว** — ไปจาก A ถึง B ได้ แต่กลับจาก B ไป A ทางเดิมไม่ได้ ต้องอ้อมทางอื่น
- **Weighted Graph** เหมือน **แผนที่ Google Maps ที่บอกเวลาเดินทางแต่ละเส้นทาง** — เส้นทางบางเส้นสั้นกว่าแต่รถติดกว่า (น้ำหนักสูงกว่าในแง่เวลา) ทำให้ระบบเลือกเส้นทางที่ "ต้นทุนรวมต่ำสุด" ไม่ใช่แค่ "จำนวนเส้นทางน้อยสุด"

## แผนภาพอธิบาย (Visual explanation)

### Undirected Graph vs Directed Graph

```
Undirected (ไม่มีทิศทาง):        Directed (มีทิศทาง):

    A ─── B                        A ──► B
    │     │                        │     │
    │     │                        ▼     ▼
    C ─── D                        C ◄── D

A-B หมายความว่า A↔B ได้            A→B หมายความว่าไปทางเดียว B→A ไม่ได้
(เพื่อนกันบน Facebook)             (ติดตามบน Twitter/Instagram)
```

### Weighted Graph

```
        (5)
   A ────────── B
   │            │
  (2)          (3)
   │            │
   C ────────── D
        (1)

เส้นทางจาก A ไป D:
  A -> B -> D = 5 + 3 = 8
  A -> C -> D = 2 + 1 = 3   ← ต้นทุนต่ำกว่า! (จะหาแบบนี้ได้ด้วย Dijkstra's Algorithm)
```

### การเก็บ Graph ในโปรแกรม — Adjacency List (นิยมที่สุด)

```
Graph:   A ── B          Adjacency List (ใช้ Hash Table จากบทที่ 15!):
         │    │
         C ── D          {
                            "A": ["B", "C"],
                            "B": ["A", "D"],
                            "C": ["A", "D"],
                            "D": ["B", "C"]
                          }

แต่ละ key คือ vertex, value คือ list ของเพื่อนบ้าน (neighbor) ที่เชื่อมกันโดยตรง
```

### Adjacency Matrix (อีกวิธีหนึ่ง — ใช้ Array 2 มิติจากบทที่ 12)

```
        A   B   C   D
    A [ 0,  1,  1,  0 ]     1 = มี edge เชื่อม, 0 = ไม่มี
    B [ 1,  0,  0,  1 ]
    C [ 1,  0,  0,  1 ]
    D [ 0,  1,  1,  0 ]

ตรวจสอบว่า A-B เชื่อมกันหรือไม่: matrix[A][B] = 1 -> O(1) ทันที
แต่เสียพื้นที่ O(V²) เสมอ ไม่ว่า edge จะเยอะหรือน้อย (ต่างจาก Adjacency List)
```

## ตัวอย่างโค้ด (Code example)

```python
# ===== สร้าง Graph ด้วย Adjacency List =====
class Graph:
    def __init__(self):
        self.adjacency_list = {}          # dict: vertex -> list ของเพื่อนบ้าน

    def add_vertex(self, vertex):
        if vertex not in self.adjacency_list:
            self.adjacency_list[vertex] = []

    def add_edge(self, v1, v2, directed=False):
        self.add_vertex(v1)
        self.add_vertex(v2)
        self.adjacency_list[v1].append(v2)
        if not directed:                    # ถ้าไม่มีทิศทาง ต้องเพิ่มกลับทางด้วย
            self.adjacency_list[v2].append(v1)

    def get_neighbors(self, vertex):
        return self.adjacency_list.get(vertex, [])

graph = Graph()
graph.add_edge("A", "B")
graph.add_edge("A", "C")
graph.add_edge("B", "D")
graph.add_edge("C", "D")

print(graph.get_neighbors("A"))   # ['B', 'C']
print(graph.adjacency_list)
# {'A': ['B', 'C'], 'B': ['A', 'D'], 'C': ['A', 'D'], 'D': ['B', 'C']}

# ===== ตัวอย่าง Directed Graph: ระบบติดตามบน Social Media =====
social = Graph()
social.add_edge("Alice", "Bob", directed=True)     # Alice ติดตาม Bob
social.add_edge("Bob", "Alice", directed=True)      # Bob ติดตาม Alice กลับ (ต้องเพิ่มเอง)
print(social.get_neighbors("Alice"))                 # ['Bob'] -> Alice ติดตามใครบ้าง
```

## ทำงานทีละขั้นตอน (Step-by-step execution)

1. `graph.add_edge("A", "B")` — เรียก `add_vertex("A")` และ `add_vertex("B")` ก่อนเพื่อให้แน่ใจว่าทั้งคู่มีอยู่ใน `adjacency_list` แล้ว (สร้าง list ว่างถ้ายังไม่มี) จากนั้นเพิ่ม `"B"` เข้าไปใน list ของ `"A"` (`A → B`) และเพราะ `directed=False` (ค่าเริ่มต้น) จึงเพิ่ม `"A"` เข้าไปใน list ของ `"B"` ด้วย (`B → A`) — ทำให้เป็น Undirected Graph ที่เดินได้สองทาง
2. `social.add_edge("Alice", "Bob", directed=True)` — เพิ่มแค่ `"Bob"` เข้า list ของ `"Alice"` เท่านั้น **ไม่เพิ่มกลับทาง** เพราะระบุ `directed=True` — สะท้อนว่า "การติดตาม" เป็นความสัมพันธ์ทางเดียวตามธรรมชาติจริง (Alice ติดตาม Bob ไม่ได้แปลว่า Bob ต้องติดตาม Alice กลับ) หากต้องการให้ Bob ติดตาม Alice กลับ ต้องเรียก `add_edge("Bob", "Alice", directed=True)` แยกต่างหากอีกครั้ง (ตามตัวอย่าง)
3. `graph.get_neighbors("A")` — แค่เปิด dictionary ด้วย key `"A"` (O(1) ตามหลักการ Hash Table จากบทที่ 15) ได้ list ของเพื่อนบ้านที่เชื่อมโดยตรงกลับมาทันที ไม่ต้องไล่ตรวจทั้ง graph

## Time Complexity

| Operation | Adjacency List | Adjacency Matrix |
|---|---|---|
| ตรวจสอบว่า edge (A,B) มีอยู่หรือไม่ | O(degree of A) — ไล่ list เพื่อนบ้านของ A | O(1) — เปิด matrix[A][B] ตรงๆ |
| หาเพื่อนบ้านทั้งหมดของ vertex หนึ่ง | O(degree of vertex) | O(V) — ต้องไล่ทั้งแถว |
| เพิ่ม edge ใหม่ | O(1) | O(1) |
| พื้นที่ที่ใช้ | O(V + E) | O(V²) เสมอ |

โดย V = จำนวน vertex ทั้งหมด, E = จำนวน edge ทั้งหมด, degree = จำนวนเพื่อนบ้านของ vertex นั้น — **Adjacency List ประหยัดกว่าเมื่อ graph มี edge น้อย (sparse graph)** ซึ่งเป็นกรณีส่วนใหญ่ในโลกจริง (เช่น ผู้ใช้ Facebook คนหนึ่งไม่ได้เป็นเพื่อนกับทุกคนในโลก) ส่วน **Adjacency Matrix เหมาะเมื่อ graph มี edge หนาแน่น (dense graph)** และต้องการตรวจสอบการเชื่อมต่อบ่อยๆ

## Space Complexity

- Adjacency List: **O(V + E)** — ประหยัดพื้นที่เมื่อ graph ไม่หนาแน่น
- Adjacency Matrix: **O(V²)** เสมอ ไม่ว่า edge จะมีกี่เส้น — สิ้นเปลืองมากถ้า graph มี vertex เยอะแต่ edge น้อย

## ข้อดี (Advantages)

- จำลองความสัมพันธ์ที่ซับซ้อนได้ทุกรูปแบบ (ไม่จำกัดแค่ลำดับชั้นเหมือน Tree)
- Adjacency List ประหยัดพื้นที่สำหรับ graph ที่ไม่หนาแน่น (sparse) ซึ่งพบบ่อยในโลกจริง
- Weighted graph คำนวณ "ต้นทุน" ของเส้นทางได้ ทำให้แก้ปัญหาเชิงเพิ่มประสิทธิภาพได้ (shortest path, minimum cost)

## ข้อเสีย (Disadvantages)

- ซับซ้อนกว่า Tree ในการวิเคราะห์และ implement เพราะมี cycle ได้ (ต้องระวัง infinite loop เวลาเดิน traverse ถ้าไม่ track ว่าเคยไปที่ vertex ไหนแล้วบ้าง)
- Adjacency Matrix สิ้นเปลืองพื้นที่มากเมื่อ graph มี vertex จำนวนมากแต่ edge น้อย
- อัลกอริทึมบน Graph (BFS, DFS, Dijkstra) ซับซ้อนกว่าอัลกอริทึมบน Tree/Array ธรรมดา ต้องเรียนเพิ่มเติมในบท Algorithms

## การใช้งานจริง (Real-world usage)

- **Social Network** — ผู้ใช้เป็น vertex, ความสัมพันธ์ (เพื่อน/ติดตาม) เป็น edge
- **แผนที่และระบบนำทาง (Google Maps, GPS)** — สถานที่เป็น vertex, ถนนเป็น weighted edge (น้ำหนัก = ระยะทาง/เวลา)
- **Web pages และ hyperlink** — หน้าเว็บเป็น vertex, ลิงก์เป็น directed edge (ใช้ในอัลกอริทึม PageRank ของ Google)
- **Dependency graph** — โมดูล/package ที่ import กัน (npm, pip) เป็น directed graph ใช้ตรวจสอบ circular dependency
- **Network topology** — คอมพิวเตอร์/router ในเครือข่ายเป็น vertex, การเชื่อมต่อเป็น edge

## แนวทางปฏิบัติที่ดี (Best Practices)

- เลือก Adjacency List เป็นค่าเริ่มต้นสำหรับ graph ส่วนใหญ่ในทางปฏิบัติ (เพราะ graph ในโลกจริงมักไม่หนาแน่น) ใช้ Adjacency Matrix เฉพาะเมื่อต้องตรวจสอบการเชื่อมต่อระหว่างคู่ vertex บ่อยมากและ graph มีขนาดไม่ใหญ่เกินไป
- เมื่อ traverse graph ต้อง**เก็บ record ว่าเคยไป vertex ไหนมาแล้วบ้าง** (visited set — ใช้ Set จากบทที่ 15) เพื่อป้องกัน infinite loop จาก cycle
- ระบุให้ชัดเจนตั้งแต่ต้นว่า graph ที่ออกแบบเป็น directed หรือ undirected เพราะมีผลต่อวิธี implement และผลลัพธ์ของอัลกอริทึมทั้งหมด

## ข้อผิดพลาดที่พบบ่อย (Common Mistakes)

- ลืมเพิ่ม edge กลับทางเมื่อสร้าง Undirected Graph (เพิ่มแค่ A→B แต่ลืม B→A) ทำให้ graph ไม่สมมาตรตามที่ตั้งใจ
- Traverse graph ที่มี cycle โดยไม่เก็บ visited set ทำให้เกิด infinite loop (โปรแกรมค้าง)
- สับสนระหว่าง Tree และ Graph — คิดว่าทุก Graph ต้องไม่มี cycle เหมือน Tree (ที่จริง Graph อนุญาตให้มี cycle ได้ตามธรรมชาติ)
- ใช้ Adjacency Matrix กับ graph ขนาดใหญ่ที่มี vertex เป็นล้าน ทำให้ใช้หน่วยความจำมหาศาลเกินความจำเป็น (O(V²) กับ V=1,000,000 คือ 10^12 ช่อง!)

## คำถามสัมภาษณ์งาน (Interview Questions)

1. อธิบายความแตกต่างระหว่าง Directed Graph และ Undirected Graph พร้อมยกตัวอย่างการใช้งานจริง
2. Adjacency List ต่างจาก Adjacency Matrix อย่างไร แต่ละแบบเหมาะกับสถานการณ์ไหน?
3. ทำไม Tree ถึงถือเป็น Graph ชนิดพิเศษ? กฎอะไรที่ Tree มีเพิ่มเติมจาก Graph ทั่วไป?
4. ทำไมต้องเก็บ "visited set" เวลา traverse graph? จะเกิดอะไรขึ้นถ้าไม่เก็บ?
5. ยกตัวอย่างปัญหาจริงที่เหมาะกับการโมเดลเป็น Weighted Graph และอธิบายว่าน้ำหนักของ edge แทนอะไร

## แบบฝึกหัด (Practice Exercises)

1. เขียนฟังก์ชันแปลง Adjacency List ให้เป็น Adjacency Matrix และในทางกลับกัน
2. เขียนฟังก์ชันตรวจสอบว่า Undirected Graph มี cycle หรือไม่ (ใบ้: ใช้ visited set ระหว่าง traverse แล้วเช็คว่าเจอ vertex ที่เคยไปแล้วหรือไม่ โดยไม่นับ parent ที่เพิ่งมาจาก)
3. สร้าง Weighted Graph จำลองแผนที่เมือง 5 เมือง พร้อมระยะทางระหว่างเมือง แล้วเขียนฟังก์ชันหาผลรวมระยะทางของเส้นทางที่กำหนด (ยังไม่ต้องหาเส้นทางที่สั้นที่สุด — จะเรียนใน Dijkstra's Algorithm บทอัลกอริทึม)

## Mini Project

**"ระบบแนะนำเพื่อน (Friend Recommendation System)"**: เขียนโปรแกรมที่:
1. สร้าง Undirected Graph จำลองเครือข่ายเพื่อนของผู้ใช้หลายคน (ใช้ Adjacency List)
2. เขียนฟังก์ชันแนะนำ "เพื่อนของเพื่อน" (friend-of-friend) ที่ยังไม่ได้เป็นเพื่อนกับผู้ใช้ที่ระบุ (ไล่เพื่อนบ้านของเพื่อนบ้าน แล้วกรองคนที่เป็นเพื่อนอยู่แล้วออก)
3. เพิ่มระบบนับ "จำนวนเพื่อนร่วมกัน" (mutual friends) ระหว่างผู้ใช้สองคน เพื่อจัดอันดับคำแนะนำ (คนที่มีเพื่อนร่วมกันเยอะกว่าควรถูกแนะนำก่อน)
4. ทดสอบด้วยเครือข่ายขนาดใหญ่ (เช่น 1,000 คน) เปรียบเทียบเวลาและหน่วยความจำที่ใช้ระหว่าง Adjacency List กับ Adjacency Matrix

## ข้อคิดสำคัญ (Key Takeaways)

- Graph คือโครงสร้างข้อมูลที่ทั่วไปที่สุด ประกอบด้วย Vertex และ Edge โดยไม่มีข้อจำกัดเรื่องจำนวนการเชื่อมต่อ ทิศทาง หรือ cycle
- Tree, BST, Heap, Trie ที่เรียนมาทั้งหมดเป็น Graph ชนิดพิเศษที่มีกฎเพิ่มเติม (ไม่มี cycle, parent เดียว)
- Directed Graph มีทิศทางของความสัมพันธ์ ส่วน Undirected Graph เชื่อมสองทางเสมอ — ต้องเลือกให้ตรงกับธรรมชาติของข้อมูลจริง
- Adjacency List ประหยัดพื้นที่กว่าสำหรับ graph ที่ไม่หนาแน่น (พบบ่อยในโลกจริง) ส่วน Adjacency Matrix เหมาะกับ graph หนาแน่นที่ต้องตรวจสอบการเชื่อมต่อบ่อย
- ต้องเก็บ visited set เสมอเวลา traverse graph ที่อาจมี cycle เพื่อป้องกัน infinite loop
- Graph เป็นบทสุดท้ายของ Part 4: Data Structures — บทถัดไปจะเข้าสู่ Part 5: Algorithms โดยเริ่มจาก Big O, Big Theta, Big Omega ซึ่งเป็นเครื่องมือวิเคราะห์ประสิทธิภาพที่ใช้ตลอดทั้งบทที่ผ่านมา (และจะใช้ต่อไปในทุกอัลกอริทึมที่จะเรียน)

---
*บทต่อไป (บทที่ 20): Big O, Big Theta, Big Omega — พิมพ์ "Next" เพื่อดำเนินการต่อ*
