# PRD.md:  "The Why"; Product requirements, benchmark metadata table, high-level requirements, etc.

Product Requirements Document

"The Why"; Product requirements, benchmark metadata table, high-level requirements, etc. Defines the benchmark goal, validation objective, test name, benchmark number, category, and high-level success criteria.

## Benchmark Matrix Document Metadata (via benchmark_specification.json)

This PRD.md section is populated from benchmark_specification.json, which is the structured source of benchmark-specific product requirements.

## Workload Number
127

## Workload Name
KV Cache Stress Test

## Execution Summary (Run and Measure)
Start python -m vllm.entrypoints.openai.api_server --model mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, wait for /v1/models, run scripts/benchmark_serving.py once with yaml input_len, output_len, num_prompts, and concurrency, then stop the server, to measure KV-cache stress

## Main Goal
Analyze LLM KV-cache stress under long contexts

## Validation Objective
Validates decode throughput, TTFT, TPOT, and KV-cache occupancy from the real vLLM OpenAI server. Every profile starts python -m vllm.entrypoints.openai.api_server

## Workload Category
LLM Inference & Serving

## Validation Requirement

The benchmark must include an automated SQLite-integrated validation layer that verifies persisted results from `results/benchmark.db`. Validation must confirm:

1. The benchmark run completed successfully with no tool errors.
2. Required samples and aggregate metrics were persisted for every swept shape.
3. Metrics are finite and physically sensible (positive, within plausible bounds).
4. Measured values satisfy configured thresholds when the workload defines pass/fail gates.
5. The benchmark fails validation when required data is missing, invalid, or outside bounds.

## Non-Functional Requirements

| Requirement | Target |
|---|---|
| Automation | Runs to completion without manual intervention after `bash run_benchmark.sh` |
| Idempotency | Re-running `run_benchmark.sh` appends a new run; never corrupts existing rows |
| Persistence | All metrics survive script exit; `results/benchmark.db` is the durable record |
