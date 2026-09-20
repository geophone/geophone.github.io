---
layout: post
title: "spark bridged llama-server with framework pc"
date: 2026-09-19
---

#spark
```bash
sudo ip addr flush dev enP7s7
sudo ip addr add 10.50.0.1/24 dev enP7s7
sudo ip link set enP7s7 up
```

#framework
```bash
sudo ip addr flush dev enp191s0
sudo ip addr add 10.50.0.2/24 dev enp191s0
sudo ip link set enp191s0 up
```

#you can use iperf3 to test the link I found about 4.6GB/sec

Compile llama-server with rpc support
```bash
-DGGML_RPC=ON \ flag at minimum with the build flags
```

#framework
start the rpc server
```bash
ggml-rpc-server \
    -H 10.50.0.2 \
    -p 50052
```

#spark
start the model > 128GB
llama-server -m Qwen3.8-Flash-Next-UD-Q4_K_XL-00001-of-00004.gguf --mmproj ../mmproj... --ctx-size 262144  --host 0.0.0.0 --device CUDA0/RPC0   --rpc 10.50.0.2:50052


benchmarks look linear but larger vram available
```bash
llama-bench   -m Qwen3.8-Flash-Next-UD-Q4_K_XL-00001-of-00004.gguf   --rpc 10.50.0.2:50052   --device CUDA0/RPC0   -n 256   -r 5
ggml_cuda_init: found 1 CUDA devices (Total VRAM: 124610 MiB):
  Device 0: NVIDIA GB10, compute capability 12.1, VMM: yes, VRAM: 124610 MiB
| model                          |       size |     params | backend    | ngl | dev          |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------------ | --------------: | -------------------: |
| qwen4exp A3B Q4_K - Medium     | 103.68 GiB |   176.94 B | CUDA,RPC   |  -1 | CUDA0/RPC0   |           pp512 |       604.35 ± 37.23 |
| qwen4exp A3B Q4_K - Medium     | 103.68 GiB |   176.94 B | CUDA,RPC   |  -1 | CUDA0/RPC0   |           tg256 |         22.81 ± 0.10 |

```

```bash
llama-bench   -m Qwen3.8-Flash-Next-UD-Q4_K_XL-00001-of-00004.gguf  --device CUDA0   -n 256   -r 5
ggml_cuda_init: found 1 CUDA devices (Total VRAM: 124610 MiB):
  Device 0: NVIDIA GB10, compute capability 12.1, VMM: yes, VRAM: 124610 MiB
| model                          |       size |     params | backend    | ngl | dev          |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------------ | --------------: | -------------------: |
| qwen4exp A3B Q4_K - Medium     | 103.68 GiB |   176.94 B | CUDA       |  -1 | CUDA0        |           pp512 |       545.52 ± 78.87 |
| qwen4exp A3B Q4_K - Medium     | 103.68 GiB |   176.94 B | CUDA       |  -1 | CUDA0        |           tg256 |         26.44 ± 0.44 |
```
