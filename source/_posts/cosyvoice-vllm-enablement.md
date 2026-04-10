---
title: CosyVoice2 启用 VLLM 推理模型
date: 2026-03-11 11:01:25
categories: [ASR, TTS]
tags:
---

启用 VLLM 推理加速可以提高 CosyVoice 的的文字识别速度，启用前后解析时间会缩短一半左右。本教程以 CosyVoice2-0.5B 为例来介绍如何启用 vLLM 加速推理。

## 启用流程

### 安装 vLLM 依赖

```shell
pip install vllm==v0.9.0 transformers==4.51.3 numpy==1.26.4 -i https://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com
```

### 引入 CosyVoice2ForCausalLM

```python
from vllm import ModelRegistry
from cosyvoice.vllm.cosyvoice2 import CosyVoice2ForCausalLM
```

### 注册 CosyVoice2ForCausalLM

在**初始化 AutoModel 之前**执行加入以下代码注册模型：

```python
ModelRegistry.register_model("CosyVoice2ForCausalLM", CosyVoice2ForCausalLM)
```

### 增加初始化参数

```python
cosyvoice = AutoModel(model_dir='pretrained_models/CosyVoice2-0.5B',
                      fp16=True, 
                      load_vllm=True,
                      load_trt=True, 
                      load_jit=True)
```

> 注意要找一个显存充足的显卡，可通过 `export CUDA_VISIBLE_DEVICES=3` 来指定 CosyVoice 使用的显卡。

## 参考资料

- [Quick Start Examples | FunAudioLLM/CosyVoice | DeepWiki](https://deepwiki.com/FunAudioLLM/CosyVoice/2.2-quick-start-examples#vllm-accelerated-inference)
- [FunAudioLLM/CosyVoice: vLLM Usage](https://github.com/FunAudioLLM/CosyVoice?tab=readme-ov-file#vllm-usage)

