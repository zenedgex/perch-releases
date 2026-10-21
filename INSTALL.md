# Installing and running Perch 0.1.1 (evaluation release)

Perch runs beneath unmodified vLLM. On each GPU node you install Perch, start your models,
and run one agent; a gateway can route across nodes.

## Requirements

| | |
|---|---|
| OS / CPU | Linux, x86_64 or aarch64 (Grace) |
| GPU | NVIDIA, Ampere or newer |
| Engine | vLLM 0.31, Python 3.10, 3.11 or 3.12 |
| Host memory | About 16 GB of RAM per model in the catalog |

## Install (per GPU node)

```sh
tar xzf perch-0.1.1-linux-x86_64.tar.gz     # or ...-linux-aarch64
cd perch-0.1.1
sudo PYTHON=$(which python3) ./install.sh   # PYTHON = the interpreter that runs vLLM
# unattended: PERCH_ACCEPT_LICENSE=yes PYTHON=... ./install.sh
```

The installer asks you to accept the evaluation licence, then installs the compiled vLLM plugin
into vLLM's environment (no fork of vLLM) and the `perch-agent`, `perch-gateway`, `perch-sim`
and `perch-vllm` commands.

## Licence keys

Without a key Perch runs in evaluation mode: up to **2 GPUs and 4 models per GPU** per host.
Pilot and production keys come from ZenEdgeX (hello@zenedgex.in). Install a key for the user
that runs vLLM and the agent:

```sh
mkdir -p ~/.perch && cp acme.license ~/.perch/license    # or PERCH_LICENSE=<key>
perch-agent --check-licence
```

## Run

1. **Models.** Start each model on its GPU, one at a time per GPU:

   ```sh
   perch-vllm --model Qwen/Qwen2.5-7B-Instruct --name qwen-7b --gpu 0 --port 9001
   perch-vllm --model Qwen/Qwen2.5-3B-Instruct --name qwen-3b --gpu 0 --port 9002
   ```

2. **Agent.** Copy `configs/agent.example.toml`, list your GPUs and models, and start it:

   ```sh
   perch-agent --config agent.toml --park-all
   ```

   Clients use the agent's OpenAI-compatible `/v1/completions` and `/v1/chat/completions`.
   `GET /status` and `GET /metrics` (Prometheus) show its state.

3. **Several nodes (optional).** `perch-gateway --agents a=http://node-a:8080,b=http://node-b:8080`,
   or `--k8s-selector app=perch-agent` inside Kubernetes; clients then talk to the gateway.

## Configuration

| Key | Meaning |
|---|---|
| `[policy] ttft_s` | First-token deadline in seconds (default 10) |
| `[policy] tbt_s` | Per-token deadline in seconds after the first (default 0.1) |
| `[[gpu]] id`, `capacity_bytes` | Each GPU, and the memory Perch may plan with on it: start with the card's memory minus 4 GB per model on that GPU |
| `[[engine]]` | One per model: `model` (the name clients use), `gpu`, `url` (where `perch-vllm` listens), `weight_bytes` (model size) |

Leave other settings at their defaults; ZenEdgeX tunes them with you during a pilot.

## Sizing before buying GPUs

`perch-sim` replays a request log through the same scheduler and reports how many of your
deadlines hold:

```sh
perch-sim --config agent.toml --trace trace.json --ttft 4
```

A trace is a JSON list of `{rid, model, arrival, in_len, out_len}`. Its estimates run about
2 to 3 points optimistic; plan with that margin.
