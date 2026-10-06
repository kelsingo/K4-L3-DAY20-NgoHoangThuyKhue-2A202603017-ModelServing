# 03 - Integrate: RAG pipeline run

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 1786.8 | 1786.8 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 830.6 | 830.6 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 869.9 | 870.0 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **1162.4** · total **1162.5**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Các thành phần N16–N19

- **N16 Cloud/IaC:** stub — pipeline chạy localhost, chưa triển khai cluster hoặc
  stack cloud.
- **N17 Data pipeline:** stub — dữ liệu là danh sách trong bộ nhớ.
- **N18 Lakehouse:** stub — chưa dùng Delta/Iceberg; corpus toy nằm trong `TOY_DOCS`.
- **N19 Vector + features:** stub — retrieval dùng keyword overlap, không dùng vector
  index hay embedding thật.
- **N20 Serving:** real — gọi `llama-server` qua API tương thích OpenAI.

LLM là stage chiếm ưu thế tuyệt đối (**1162.4 ms, 100% tổng thời gian**), đúng với
dự đoán vì embedding và retrieval hiện chỉ là toy path. Nếu cần giảm latency 2 lần,
mình sẽ tối ưu LLM trước: giảm output/context hoặc dùng quantization Q2; thay retrieval
không tạo khác biệt đáng kể khi nó hiện chỉ tốn khoảng 0.1 ms.
