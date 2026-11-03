# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:** Teerachot

**Student ID / รหัสนิสิต:** 6732017921

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:** B

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Availability / durability | Accept every review on the kiosk while it is offline. Sync it when the connection returns. |
| 2 | Turnaround time | Label every accepted review by the end of that day. |
| 3 | Accuracy and cost | Keep accuracy at or above 90%. Keep large-model calls at or below 40% for a typical workload. |

## 2. Design / การออกแบบ

Topology: T4 asynchronous/batch. An asynchronous design accepts reviews before labeling. An outbox is a local store for unsent reviews. The kiosk keeps each review in its outbox while it is offline, then sends it to a central queue when the network returns. A queue holds work until a worker can process it. A worker is a service that processes queued reviews. It labels reviews in batches before the end of the day. A cascade asks a small model first and uses the large model for uncertain reviews. A cache stores past predictions for reuse. The cache must include the active model version.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

```mermaid
C4Container
    title CampusEats: offline capture and batch labeling
    Person(customer, "Customer", "Writes restaurant reviews")
    Person(owner, "Restaurant owner", "Reads labeled reviews")
    System_Boundary(campuseats, "CampusEats") {
        Container_Boundary(kiosk_boundary, "Canteen kiosk") {
            Container(kiosk, "Review kiosk", "Tablet app", "Accepts submissions")
            Container(outbox, "Local outbox", "Encrypted local store", "Durably holds reviews offline and syncs on reconnect")
        }
        Container(api, "Review API", "Python, FastAPI", "Stores incoming reviews as pending")
        ContainerDb(review_db, "Review database", "SQL", "Stores reviews, labels, and processing status")
        Container(queue, "Pending review queue", "Message broker", "Buffers work for asynchronous scoring")
        Container(worker, "Batch labeling worker", "Python worker", "Processes queued reviews and retries failures")
        Container(inference, "Inference service", "Python, FastAPI", "Cascade and versioned prediction cache")
    }
    Rel(customer, kiosk, "Submits a review", "App")
    Rel(kiosk, outbox, "Persists review before acknowledgement", "Local write")
    Rel(outbox, api, "Syncs queued reviews when online", "HTTPS/JSON")
    Rel(api, review_db, "Stores pending review")
    Rel(api, queue, "Enqueues review ID")
    Rel(worker, queue, "Consumes reviews in batches")
    Rel(worker, inference, "Requests sentiment score", "HTTP/JSON")
    Rel(inference, worker, "Returns label")
    Rel(worker, review_db, "Loads review text and saves label or retry status")
    Rel(owner, api, "Reads labeled reviews", "HTTPS/JSON")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| t0-small | The small model reached 72.5% accuracy. This is below the 90% target. |
| t2-0.8 | Accuracy was 93.5%. The large model handled 64% of reviews before caching. |
| t3-cache | Accuracy stayed at 93.5%. Large-model calls fell to 40%, with 34% cache hits. |
| t1-after | With 10% simulated hangs, fallback had 0 errors. It saved 26 pending reviews out of 200. |

## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** Save offline reviews and label them later

**Status:** Proposed

**Context:** Scenario B kiosks lose Wi-Fi several times each day. The system must accept every review and label it by the end of the day. A synchronous model request cannot work while a kiosk is offline. Large-model calls also cost money and have a 20-call-per-second limit.

**Decision:** Use T4 asynchronous/batch. Save each review in the kiosk outbox while it is offline. Send it to the server when the network returns. Store it as pending and add its ID to the queue. The worker reads queued reviews and scores them in batches. It uses cascade at threshold 0.8 and a cache for the active model version. It saves each label and retries failed requests.

**Consequences:**
- Positive: Reviews stay safe during Wi-Fi outages. The worker can spread scoring across the day. The measured cascade/cache run kept 93.5% accuracy and used the large model for 40% of reviews.
- Negative: Labels arrive later. The design adds a local store, queue, worker, retries, and cache updates. The 200-review test did not test offline sync or prove the end-of-day target. Test both before release.

**Alternatives considered:** T1 synchronous model-as-a-service fails when a kiosk is offline. The small model reached 72.5% accuracy in `t0-small`, below the 90% target. The selected design saves each review first and labels it later with cascade and cache.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*
