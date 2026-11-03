# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** Teerachot

**Student ID / รหัสนิสิต:** 6732017921

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 510| 596| 93.5%| 100%|
| t0-small | 31| 73| 72.5%| 0%|

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> Large model improves accuracy from 72.5% to 93.5%, but it increases response time and costs more because every review goes to it.

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before |24.8| 255| 415| 14| 0| 94.1%|
| t1-after | 9.5 | 389 | 1009 | 0 | 26 | 92.5% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> Requests that take long time are result as errors which don't count in response time. So, the response time includes less long request, and thus seems to have better p50 or p90.

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> Both a slow review and an inference hang keep the same request and network connections open in the review service and gateway. In this lab, a normal large-model call takes about 200 ms, but a simulated hang lasts 20 seconds, so it holds those resources longer and can delay other reviews. If a slow review waits 20 seconds too, it causes the same resource pressure. The gateway timeout is 60 seconds for its call to the review service; before Task 1, the review service has no timeout for its call to inference.

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> They should be labelled when resource usage is low. The topology is asynchronous/batch, using a work queue and background worker to score the pending reviews later.

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> 0.1 s timeout is too short for normal large model calls, which take about 200 ms, so those reviews would become pending. A 10 s timeout is too long, it holds the request resources much longer, and the load-test client also gives up after 10 s.

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 10 | 685 | 91.5% | 38% |
| t2-0.7 | 0.7 | 365 | 574 | 93.0% | 53% |
| t2-0.8 | 0.8 | 442 | 587 | 93.5% | 64% |
| t2-0.9 | 0.9 | 473 | 589 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> I would deploy 0.6, because it meets the 90% accuracy requirement at 91.5% and uses the large model on only 38% of reviews. I trade some accuracy to reduce inference cost.

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> More than half of reviews are answered by the fast small model, so p50 falls to 10 ms. The slowest reviews still escalate to the large model and wait for its capacity, so p95 remains 685 ms.

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> A short negated review such as “not cold” can make the bag-of-words small model focus on “cold” and give a confident but wrong label. The cascade then skips the large model, so the restaurant owner may act on an incorrect sentiment.

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 11 | 558 | 93.5% | 40% | 34% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> Reviews benefit when their normalized text has appeared before. New or differently worded reviews miss, so unique reviews never benefit from this cache.

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> Clear or version the cache when deploying a new model so old predictions are not served as if they came from the new model. Include the model version in the cache namespace or key.

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> The cache cut p50 from 442 ms (`t2-0.8`) to 11 ms while p95 moved only from 587 ms to 558 ms. Repeated reviews make typical responses fast, but the uncached hard cases still determine much of the tail latency.
