# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 4151 | 147 / 292 | 24.2 / 28.1 | 1664 / 1840 / 1840 | 41.3 |
| UD-Q2_K_XL | 2.24 | 3088 | 145 / 389 | 21.3 / 24.5 | 1474 / 1920 / 1920 | 47.0 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.14x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation

On this Mac, `UD-Q2_K_XL` is the better latency and memory tradeoff for interactive
use: it is 0.73 GB smaller, loads about 1.06 seconds faster, and decodes 1.14x
faster than `UD-Q4_K_XL`. The 4-bit answer I tested was coherent and technically
detailed, but it used the full 128-token limit and was cut off at `finish_reason:
length`. I could not complete the matching 2-bit quality check because the first
server was still holding port 8080, so I am not claiming a quality win for Q2 from
that incomplete comparison. Based on the measured speed and size, I would choose
Q2 for local serving when throughput and memory matter; I would keep Q4 when a
quality-sensitive task justifies the extra 0.73 GB.
