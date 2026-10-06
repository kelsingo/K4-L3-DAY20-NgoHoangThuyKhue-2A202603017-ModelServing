# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Ngô Hoàng Thụy Khuê
**MSSV:** 2A202603017
**Cohort:** K4-L3
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** macOS (Darwin arm64)
- **CPU:** Apple M3
- **Cores:** 8 physical / 8 logical
- **CPU extensions:** NEON
- **RAM:** 16 GB
- **Accelerator:** Apple Metal
- **llama.cpp asset đã tải:** `llama-b10488-bin-macos-arm64.tar.gz`
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** `UD-Q4_K_XL` + `UD-Q2_K_XL` (từ `models/active.json`)

**Chạy ở đâu:** laptop cá nhân (Apple M3, macOS)
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Chạy local trên MacBook M3, không dùng Colab/Kaggle. Runtime llama.cpp b10488
được tải dạng binary Metal; model Gemma 4 E2B và hai quantization đã có sẵn.
Lần tải runtime đầu gặp lỗi chứng chỉ SSL của Python, nên downloader được cấu hình
dùng CA bundle `certifi`; sau đó setup và các bước đo/serve chạy được.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 4151 | 147 / 292 | 24.2 / 28.1 | 1664 / 1840 / 1840 | 41.3 |
| UD-Q2_K_XL | 2.24 | 3088 | 145 / 389 | 21.3 / 24.5 | 1474 / 1920 / 1920 | 47.0 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhỏ hơn 0.73 GB và giải mã nhanh hơn 1.14×. Bản 4-bit mạch lạc và có chi tiết kỹ thuật;
so sánh chất lượng Q2 cùng prompt chưa hoàn tất, nên mình chọn Q2 khi ưu tiên tốc độ/bộ nhớ
và giữ Q4 cho tác vụ nhạy cảm chất lượng.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.00 | 8400 | 12000 | 13000 | 8.4 | 0.0% |
| 50 | 1.00 | 33000 | 49000 | 50000 | 30.4 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** **1.00×** (20% tuyến tính)
- **P95 tăng:** **4.08×**
- **Effective concurrency ở 50 users:** **30.4** so với `--parallel` = **4** slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): **4.00 / 4 slots**

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hòa ở mức 50 user: tải tăng 5× nhưng RPS chỉ 1.00×, còn P95 tăng 4.08× lên
49 giây. Effective concurrency là 30.4 so với 4 slot và có 46 request defer, nên phần
latency tăng thêm chủ yếu là queue time sau khi compute đã kín. Mình sẽ giảm chi phí mỗi
request (Q2 hoặc output/context ngắn hơn) trước khi tăng slot.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost stack | stub |
| N17 Data pipeline | danh sách trong bộ nhớ | stub |
| N18 Lakehouse | `TOY_DOCS`, chưa có Delta/Iceberg | stub |
| N19 Vector + features | keyword overlap, chưa có vector index | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: **0.0 ms**
- retrieve: **0.0 ms**
- llm: **1162.4 ms**
- **stage chiếm nhiều nhất:** **llm (100% của total)**

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

Pipeline hiện dùng các stub toy cho N16–N19, còn N20 là `llama-server` thật. LLM chiếm
1162.4 ms (100%), đúng với kỳ vọng vì embedding và keyword retrieval gần như miễn phí.
Muốn giảm latency 2×, mình sẽ giảm output/context hoặc dùng quantization Q2 trước; tối ưu
retrieval không đáng kể ở baseline này.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** giảm số thread decode từ `-t 8` xuống `-t 1`

```
before:  39.8 tok/s (`-t 8`)
after:   45.7 tok/s (`-t 1`)
speedup: 1.15×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

`-t 1` đạt 45.7 tok/s, cao hơn 39.8 tok/s của mặc định `-t 8`; `-t 16` còn chậm hơn
với 35.1 tok/s. Vì `ngl=99`, decode đã bão hòa Metal/băng thông bộ nhớ; thread CPU
thừa chủ yếu thêm chi phí điều phối và tranh chấp cache. Giảm thread vì vậy tăng throughput
1.15× trên máy này.

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [x] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [x] Repo GitHub ở chế độ **public**
- [x] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [x] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

_(Công cụ nào, dùng vào việc gì. Ghi "Không dùng" nếu không dùng.)_

Codex: giải thích yêu cầu khó hiểu, điền báo cáo MD dựa theo hướng dẫn cụ thể sau khi xem kết quả output. 
