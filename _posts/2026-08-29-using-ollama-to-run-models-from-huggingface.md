---
layout: post
title: "Using Ollama to run models from huggingface"
date: 2026-08-29
---

Been experimenting with ollama and vllm.  Vllm in many cases is preferable for ollama but ollama is simple to use.

I've found that the unsloth page on huggingface has a number of models which will run on a dgx spark (near capacity) but that ollama can easily use

```bash
hf download unsloth/Qwen3.8-Flash-Next-GGUF --local-dir Qwen3.8-Flash-Next-GGUF --include "*UD-Q4_K_XL*"
```

```bash
hf download unsloth/DeepSeek-V4-Flash-0731-GGUF --local-dir DeepSeek-V4-Flash-0731-GGUF --include "*UD_IQ3_XXS*"
```

```bash
Then run ollama create in the UD_IQ3_XXS subdirectory
ollama create <some-deep-seek-v4-name>
```

```bash
Then you can see the model and run it through ollama
ollama list
NAME                                        ID              SIZE      MODIFIED    
DeepSeek-V4-Flash-0731-UD-IQ3_XXS:latest  
```

Problematically ollama doesn't support qwen3.8-flash-next's architecture so you can't actually use ollama to create it, in fact vllm doesn't support it either but they provide a docker image which you can run the qwen3.8-flash-next models with

```bash
docker run --gpus all   -v ~/.cache/huggingface:/root/.cache/huggingface   --env "HF_TOKEN=$HF_TOKEN"   -p 8000:8000   --ipc=host   vllm/vllm-openai:qwen38-flash-next   --model Qwen/Qwen3.8-Flash-Next-FP8   --tensor-parallel-size 1   --gpu-memory-utilization 0.99   --max-num-seqs 256   --enable-prefix-caching   --no-enable-flashinfer-autotune   --enable-auto-tool-choice   --tool-call-parser qwen3_xml   --reasoning-parser qwen3
```

Then it's just a matter of using vllm, you can use openai compatible json format like in open code for example with opencode

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "vllm": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "vLLM (local)",
      "options": {
        "baseURL": "http://<server-ip>:8000/v1"
      },
      "models": {
        "Qwen/Qwen3.8-27B": {
          "name": "Qwen3.8-27B",
          "attachment": true,
          "modalities": {
            "input": ["text", "image"],
            "output": ["text"]
          },
          "settings": {
              "body": {
                "extra_body": {
                  "chat_template_kwargs": { "enable_thinking": true },
                  "reasoning_effort": "low"
                }
              }
            }
        }
      }
    }
  },
  "model": "vllm/Qwen/Qwen3.8-27B",
  "mcp": {
    "computer-use-linux": {
      "type": "local",
      "command": ["/home/taylor/.local/bin/computer-use-linux", "mcp"],
      "enabled": true
    }
  }
}
```

For a ollama example it's similar

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "ollama",
      "options": {
        "baseURL": "http://192.168.50.87:11434/v1"
      },
      "models": {
        "DeepSeek-V4-Flash-0731-UD-IQ3_XXS:latest": {
          "name": "DeepSeek V4",
        }
      }
    }
  },
  "model": "ollama/DeepSeek-V4-Flash-0731-UD-IQ3_XXS:latest",
  "mcp": {
    "computer-use-linux": {
      "type": "local",
      "command": ["/home/taylor/.local/bin/computer-use-linux", "mcp"],
      "enabled": true
    }
  }
}
```

I'm starting to use openhands because it is the one of if the only agents that supports open source models and image inputs in the prompting

To install and run openhands

```bash
npm install -g @openhands/agent-canvas
agent-canvas
```

Then you just need to provide the endpoint which is <server-ip>:11434 for ollama or :8000 for vllm along with the model name as the server knows it


