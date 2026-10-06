# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **8 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 45.7 | 100% |
| 4 | 43.9 | 96% |
| 8 | 39.8 | 87% |
| 16 | 35.1 | 77% |

**Best**: `-t 1` at 45.7 tok/s
**Slowest tested**: `-t 16` at 35.1 tok/s (1.30x spread)
**Against the physical-core default** (`-t 8`, 39.8 tok/s): 1.15x

Use this in your run:

```bash
LAB_N_THREADS=1 make bench
```

## Giải thích

Đường cong đạt đỉnh ngay ở `-t 1` (45.7 tok/s), rồi giảm dần từ `-t 4` trở đi;
vì vậy knee thực tế nằm ở vùng 1–4 thread, chứ không nằm tại 8 physical core như
giả định thông thường. Với `ngl=99`, phần lớn phép giải mã đã được offload lên Metal
và bị giới hạn bởi băng thông bộ nhớ hoặc độ bão hòa GPU. Thêm thread không tạo thêm
công việc hữu ích mà làm tăng chi phí điều phối và tranh chấp cache/băng thông, nên
`-t 8` chỉ còn 39.8 tok/s và `-t 16` giảm tiếp xuống 35.1 tok/s. So với mặc định
physical-core `-t 8`, cấu hình tốt nhất `-t 1` nhanh hơn 1.15 lần; trên máy này nên
dùng một thread cho decode thay vì oversubscribe CPU.
