# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 59 | 1.00 | 8400 | 12000 | 13000 | 8.4 | 0.0% |
| 50 | 59 | 1.00 | 33000 | 49000 | 50000 | 30.4 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.00x** (20% of linear) |
| P95 latency | **4.08x** |
| Effective concurrency at 50 users | 30.4 vs `--parallel 4` slots (occupancy/slot ratio 7.59) |

**Saturated.** Throughput delivered only 1.00x for 5x the offered load, and effective concurrency (30.4) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.00x while P95 moved 4.08x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Nhận xét

Server đã bão hòa ở mức tải 50 user (và chắc chắn từ một mức nào đó dưới 50): tải
tăng 5 lần nhưng throughput chỉ giữ nguyên **1.00x**, trong khi P95 tăng **4.08x**
(12 giây lên 49 giây). Effective concurrency tăng lên **30.4**, cao hơn nhiều so
với 4 slot, còn batching cho thấy cả 4 slot đều bận và có 46 request bị defer. Đây
là bằng chứng latency tăng chủ yếu do queue time sau khi compute đạt công suất. Nếu
chọn SLO P95 = 12 giây, load-10 vẫn giữ khoảng 1.00 RPS trong SLO còn load-50 gần
như không giữ được request nào; để nâng goodput trước tiên mình sẽ giảm chi phí mỗi
request (dùng Q2 hoặc giảm output/context), rồi mới tăng slot.
