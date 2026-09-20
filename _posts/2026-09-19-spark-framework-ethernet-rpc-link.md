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

I had to set some flags on the boot for the amdgpu to use more than the default 64GB
```
bash
amdgpu.no_system_mem_limit=1 amdgpu.gttsize=90112 ttm.pages_limit=23068672 ttm.page_pool_size=23068672 amdttm.pages_limit=23068672 amdttm.page_pool_size=23068672
```
You can translate/look these up on chatgpt or another llm  probably some of these flags are superflous but probably need ttm poolsize and page limit


It looks like it's working
```
bash
llama-bench   -m Qwen3.8-Flash-Next-UD-Q5_K_XL-00001-of-00006.gguf  --rpc 10.50.0.2:50052   --device CUDA0/RPC0   -n 256   -r 5
ggml_cuda_init: found 1 CUDA devices (Total VRAM: 124610 MiB):
  Device 0: NVIDIA GB10, compute capability 12.1, VMM: yes, VRAM: 124610 MiB
| model                          |       size |     params | backend    | ngl | dev          |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------------ | --------------: | -------------------: |
| qwen4exp A3B Q5_K - Medium     | 147.41 GiB |   176.94 B | CUDA,RPC   |  -1 | CUDA0/RPC0   |           pp512 |       460.41 ± 27.33 |
| qwen4exp A3B Q5_K - Medium     | 147.41 GiB |   176.94 B | CUDA,RPC   |  -1 | CUDA0/RPC0   |           tg256 |         18.01 ± 1.10 |

I found that when testing on Q8_0 which I should have enough pooled memory for there was significant host RAM usage which is unified with the GPU and llama-server is not splitting it safely
```
bash
llama-fit-params -m Qwen3.8-Flash-Next-Q8_0-00001-of-00006.gguf --ctx-size 262144 --rpc 10.50.0.2:50052 --device CUDA0,RPC0 -ngl all --split-mode layer --tensor-split 40,60 -lv 5
...
0.00.676.447 I common_memory_breakdown_print: | memory breakdown [MiB]     |  total     free     self   model   context   compute    unaccounted |
0.00.676.453 I common_memory_breakdown_print: |   - CUDA0 (GB10)           | 124610 = 117434 + (58557 = 52652 +    2927 +    2977) +      -51381 |
0.00.676.453 I common_memory_breakdown_print: |   - RPC0 (10.50.0.2:50052) |  90112 =  89899 + (81512 = 74317 +    4097 +    3097) +      -81300 |
0.00.676.454 I common_memory_breakdown_print: |   - Host                   |                    54087 = 52524 +       0 +    1563  CUDA0, backend CUDA0
...
You can observe with 117434 free on GB10, 58557 + 54087 is 112... which is too thin.
...
llama-bench -m Qwen3.8-Flash-Next-Q8_0-00001-of-00006.gguf --rpc 10.50.0.2:50052 --device CUDA0,RPC0 --split-mode layer --tensor-split 33,67 -n 256 -r 5
...
but the bench is failing to load, let's see if I can load the model with the split suggested by llama-fit-params.
...
0.22.978.822 I load_tensors:        CUDA0 model buffer size = 52652.80 MiB
0.22.978.823 I load_tensors:    CUDA_Host model buffer size = 52524.27 MiB
0.22.978.823 I load_tensors: RPC0[10.50.0.2:50052] model buffer size = 74317.87 MiB
...
llama-server -m Qwen3.8-Flash-Next-Q8_0-00001-of-00006.gguf --ctx-size 262144 --rpc 10.50.0.2:50052 --device CUDA0,RPC0 -ngl all --host 0.0.0.0 --split-mode layer --tensor-split 33,67
...
success
```
