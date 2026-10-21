# Changelog

## 0.1.1 (2026-10-21), evaluation release

- Evaluation mode and licence keys: without a key, up to 2 GPUs and 4 models per GPU per host;
  signed pilot and production keys lift the limits. `perch-agent --check-licence` and `/status`
  show the active licence.
- The installer asks you to accept the evaluation licence before installing anything.
- `perch-vllm` starts a model for Perch.

## 0.1.0 (2026-10-21)

First release: `perch-agent`, the compiled vLLM plugin, `perch-gateway` and `perch-sim`.
Measured on 2x H200 NVL with vLLM 0.31 and Qwen2.5-7B: 6 models on 2 GPUs with 99.9-100% of
tokens on time and no answers lost; 20 models on 2 GPUs with 98.1% of tokens on time.
