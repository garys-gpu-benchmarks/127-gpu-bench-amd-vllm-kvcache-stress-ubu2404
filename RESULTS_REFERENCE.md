# Results: 127 KV Cache Stress Test

**Main goal:** Analyze LLM KV-cache stress under long contexts  
**Tools:** Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_141 · recorded 2026-11-14). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× AMD Instinct MI300X VF (196288 MB) · INTEL(R) XEON(R) PLATINUM 8568Y+ · Ubuntu 24.04.5 LTS · kernel 6.8.0-147-generic · driver 7.1.9.31600000
- **Run:** 2026-11-14T11:35:55Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 04:20 · benchmark 00:07 · whole run 05:50
- **Source:** run 127_20261114_113555
- **Command:** `bash run_benchmark.sh --smoke --model-name mistralai/Mistral-7B-v0.3 --dtype bfloat16 --kv-cache-dtype auto --gpu-memory-utilization 0.9 --max-model-len 128 --input-len 64 --output-len 16 --num-prompts 2 --concurrency 2 --seed 0`

| # | Metric | Result |
|---|---|---|
| 1 | Decode throughput, tokens/s (output_tokens_per_sec) | 421.792286533793 |
| 2 | Time To First Token p50, ms (ttft_msec) | 18.32534100049088 |
| 3 | TPOT p50, ms (tpot_msec) | 4.736044899982517 |
| 4 | Peak KV-cache usage, GiB (kv_cache_peak_gb) | 0.019532248989820367 |
| 5 | KV-cache usage peak, pct (kv_cache_usage_peak_pct) | 0.01278706971510335 |
<!-- /profile:smoke -->
