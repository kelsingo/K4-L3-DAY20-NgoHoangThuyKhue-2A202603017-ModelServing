# 02 - Continuous batching under load (u50)

Host `Darwin-arm64` · `--parallel 4` · 30 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 4.00 of 4 slots (100%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 6565 |

Highest sampled value was **4.00 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Nhận xét

Peak `n_busy_slots_per_decode` đạt **4.00/4 slots (100%)**, đúng với cấu hình
`--parallel 4`, nên scheduler đã thực sự gộp request trong các bước decode. Có lúc
`requests_processing=4` và `requests_deferred=46`: bốn slot đều bận và 46 request
phải chờ, vì vậy queue time góp phần làm P95 của load-50 tăng mạnh. Giá trị peak
này là trung bình theo bước decode, không phải batch tức thời tối đa; mình tin nó
cho biết mức sử dụng scheduler, còn effective concurrency trong report server cần
đối chiếu thêm với số slot bận và RPS của cùng lần chạy.
