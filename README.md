# Perch: more models per GPU, no lost answers (evaluation release)

Perch runs beneath **unmodified vLLM** and lets each GPU serve many more models while
first-token and per-token deadlines hold. Your clients keep the same OpenAI-compatible API.

Measured on 2x NVIDIA H200 NVL with vLLM 0.31 and Qwen2.5-7B models, a 4 s first-token and
100 ms per-token deadline, bursty long-tail traffic:

| Setup | Tokens on time | Answers lost | GPUs vs one model per GPU |
|---|---|---|---|
| Perch, 6 models on 2 GPUs | 99.9-100% | 0 | **3x fewer** |
| Perch, 20 models on 2 GPUs (10 per GPU) | 98.1% | 0 cut off, 4 of 752 timed out | **10x fewer** |
| Stock vLLM sharing the GPUs, 6 models on 2 GPUs | 84.1% | 183 of 840 cut off | - |

Same traffic for every row. Outputs with Perch are identical to each model served alone. With
the memory of an 80 GB-class card, plain colocation fits 3 to 4 such models per GPU; Perch served 10.

## Licensing

Perch is proprietary software, free to evaluate. Without a licence key it runs in evaluation
mode, up to **2 GPUs and 4 models per GPU** per host, on hardware you control. Signed keys for
pilots and production: hello@zenedgex.in. See [EVALUATION-LICENSE.md](EVALUATION-LICENSE.md).
Your models, prompts and traffic stay yours; Perch sends nothing to ZenEdgeX.

## Try it

Requirements: Linux (x86_64 or aarch64), NVIDIA GPUs (Ampere or newer), vLLM 0.31 on
Python 3.10, 3.11 or 3.12, and about 16 GB of host RAM per model in your catalog.

Download the bundle for your CPU from the [latest release](https://github.com/zenedgex/perch-releases/releases/latest)
(checksums in SHA256SUMS), then on each GPU node:

```sh
tar xzf perch-0.1.1-linux-x86_64.tar.gz && cd perch-0.1.1
sudo PYTHON=$(which python3) ./install.sh      # the Python that runs vLLM; asks you to accept the licence
```

Start your models, write an agent config, and send requests:

```sh
perch-vllm --model Qwen/Qwen2.5-7B-Instruct --name qwen-7b --gpu 0 --port 9001
perch-agent --config agent.toml --park-all
curl localhost:8080/v1/completions -H 'content-type: application/json' \
  -d '{"model": "qwen-7b", "prompt": "Hello", "max_tokens": 32}'
```

Full guide: [INSTALL.md](INSTALL.md).

## What is in a release

| Asset | Contents |
|---|---|
| `perch-0.1.1-linux-x86_64.tar.gz`, `...-aarch64.tar.gz` | `perch-agent` (OpenAI-compatible endpoint per node), `perch-gateway` (routing across nodes), `perch-sim` (capacity estimates from your request log), `perch-vllm` (starts a model), compiled vLLM plugin, installer, docs |
| `perch_engine-0.1.1-cp3XX-...whl` | The compiled vLLM plugin alone |
| `SHA256SUMS` | Checksums |

## Design partners

We work with providers serving many fine-tuned or long-tail models. Send a two-week request log
(model id, timestamp, prompt and output lengths; no content) and we return a capacity report for
your catalog and GPUs. hello@zenedgex.in
