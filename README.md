# KV Cache Stress Test Benchmark

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://github.com/garys-gpu-benchmarks/127-gpu-bench-amd-vllm-kvcache-stress-ubu2404/actions/workflows/ci.yml/badge.svg)](https://github.com/garys-gpu-benchmarks/127-gpu-bench-amd-vllm-kvcache-stress-ubu2404/actions/workflows/ci.yml)

Analyzes LLM KV-cache stress under long contexts. Workload 127 of the AMD Ubuntu 24.04 bundle.

## Quick Start

On an Ubuntu 24.04 host, as root or with sudo:

```bash
git clone https://github.com/garys-gpu-benchmarks/127-gpu-bench-amd-vllm-kvcache-stress-ubu2404.git
cd 127-gpu-bench-amd-vllm-kvcache-stress-ubu2404
sudo bash setup.sh --assume-yes
bash run_benchmark.sh --smoke
```

`run_benchmark.sh` runs setup by itself if it has not been run yet. Each run prints its metrics when it finishes.

For this LLM serving workload, `--baseline` and `--extended` start the real model server, wait for it to be ready, run the client, and stop the server.

The model `mistralai/Mistral-7B-v0.3` is gated on Hugging Face. Set `HF_TOKEN` before setup (see [Troubleshooting](#troubleshooting)).

## Profiles and Parameters

| Parameter | Flag | smoke | baseline | extended |
|---|---|---|---|---|
| `model_name` | `--model-name` | mistralai/Mistral-7B-v0.3 | mistralai/Mistral-7B-v0.3 | mistralai/Mistral-7B-v0.3 |
| `dtype` | `--dtype` | bfloat16 | bfloat16 | bfloat16 |
| `kv_cache_dtype` | `--kv-cache-dtype` | auto | auto | auto |
| `gpu_memory_utilization` | `--gpu-memory-utilization` | 0.9 | 0.9 | 0.9 |
| `max_model_len` | `--max-model-len` | 128 | 8192 | 8192 |
| `input_len` | `--input-len` | 64 | 4096 | 4096 |
| `output_len` | `--output-len` | 16 | 256 | 256 |
| `num_prompts` | `--num-prompts` | 2 | 200 | 800 |
| `concurrency` | `--concurrency` | 2 | 8 | 8 |
| `seed` | `--seed` | 0 | 0 | 0 |

Pick a profile with `--smoke` (default), `--baseline` or `--extended`. Any flag in the table overrides that profile's value:

```bash
bash run_benchmark.sh --baseline --concurrency 4
bash run_benchmark.sh --help        # every option
```

Profile values live in `config/benchmark_config.yaml`.

## Metrics

| # | Metric | Key |
|---|---|---|
| 1 | Decode throughput, tokens/s | `output_tokens_per_sec` |
| 2 | Time To First Token p50, ms | `ttft_msec` |
| 3 | TPOT p50, ms | `tpot_msec` |
| 4 | Peak KV-cache usage, GiB | `kv_cache_peak_gb` |
| 5 | KV-cache usage peak, pct | `kv_cache_usage_peak_pct` |

### Notes

- The smoke profile is a quick check with reduced sizes and iteration counts. Use `--baseline` or `--extended` for representative numbers.
- Every profile starts vLLM's OpenAI API server and needs Mistral weights.
- Port is 8000.
- Exclusive GPU required.

## Output

| Path | Contents |
|---|---|
| `results/raw/YYYYMMDD_HHMMSSZ_WORKLOAD-<NNN>_SYSTEM-<hostname>/` | Raw tool output, `metrics_summary.txt`, and `hardware_info.txt`, `software_info.txt`, `errors_info.txt` |
| `results/benchmark.db` | SQLite history of every run |
| `results/summary.json` | Metrics from the most recent run |
| `/var/opt/benchmarks/runtime_ledger.csv` | One row per run, shared by every workload on the host |

Every run is checked for completeness and against any `thresholds:` or `baselines:` in `config/benchmark_config.yaml`. A run that fails the checks exits non-zero.

## Requirements

- Ubuntu 24.04, root or sudo for `setup.sh`
- AMD GPU with ROCm 10.1.0 (validated on MI300X)
- ROCm Runtime, PyTorch-ROCm, Hugging Face Transformers, vLLM, Mistral-7B-v0.3, Python 3.14 (uv-managed). `setup.sh` installs what is missing; Python packages go into a repo-local `.venv`.
- A Hugging Face token with access to `mistralai/Mistral-7B-v0.3`

## Run the Whole Bundle

This workload is also part of [`bundle-amd-ubuntu-2404`](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2404), which holds all 32 AMD Ubuntu 24.04 workloads as submodules:

```bash
git clone --recurse-submodules https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2404 /opt/benchmarks
cd /opt/benchmarks
./run_benchmark_suite.sh -w 127 --smoke     # just this workload
./run_benchmark_suite.sh --smoke            # all 32 workloads
./show_metrics.sh 127                       # this workload's runs from the ledger
```

The suite tools (`run_benchmark_suite.sh`, `get_benchmark_results.sh`, `list_runs.sh`, `show_metrics.sh`) are documented in [`gpu-bench-suite`](https://github.com/garys-gpu-benchmarks/gpu-bench-suite).

Other platforms: [NVIDIA 24.04](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404) (201–232), [AMD 26.04](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2604) (301–332), [NVIDIA 26.04](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604) (401–432).

## Troubleshooting

- **The SSH session closed during setup:** the first AMD workload on a fresh host may reboot once to load the AMD GPU driver. Setup prints a 30-second warning first and resumes by itself after boot. SSH back in and run `tail -f results/reboot_resume.log`; when it prints `SETUP COMPLETE`, run your command again. Do not start `setup.sh` again while it is resuming.
- **SSH dropped for another reason:** reconnect and run `sudo bash setup.sh --assume-yes` again. It resumes; do not delete `.venv` or `.cache`.
- **Model download refused (401 or 403):** `mistralai/Mistral-7B-v0.3` is gated. Accept its license on Hugging Face, then `export HF_TOKEN=<token>` and run setup with `sudo -E bash setup.sh --assume-yes` so the token reaches it.
- **A metric reads `na`:** the run could not measure it on this host. The reason is in the run's `metrics_summary.txt` and `errors_info.txt`; see also [Notes](#notes).

## Repository Layout

```text
.
├── setup.sh                      # installs dependencies and builds the benchmark
├── run_benchmark.sh              # runs one profile, parses, validates
├── show_metrics.sh               # prints this workload's ledger rows
├── SPEC.md                       # engineering specification
├── benchmark_specification.json  # workload definition
├── config/                       # profile values and thresholds
├── scripts/                      # build, collect, parse, validate
├── docs/                         # PRD and contracts
├── tests/
├── results/                      # run outputs
└── LICENSE
```

## Continuous Integration

- [CI](.github/workflows/ci.yml) runs lint and static checks on every pull request and push to `main`. It does not run the benchmark.
- [GPU Smoke Benchmark](.github/workflows/gpu-smoke.yml) runs a profile on a self-hosted runner labeled `gpu`, `amd`, `ubu2404`. It only starts by hand (**Actions → GPU Smoke Benchmark → Run workflow**), so pull requests and forks cannot run it.

Both workflows call shared steps in [`shared-workflows@v1`](https://github.com/garys-gpu-benchmarks/shared-workflows).
