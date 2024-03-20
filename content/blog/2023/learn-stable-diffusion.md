---
title: stable diffusion
description: 从程序员角度学习 Stable Diffusion
tags:
  - stable-diffusion
  - sd
date: 2022-10-07
draft: false
---

# 开始

https://github.com/AUTOMATIC1111/stable-diffusion-webui
https://github.com/comfyanonymous/ComfyUI

# 入门学习（开发者角度）
- huggingface 的公开课 https://github.com/huggingface/diffusion-models-class
- 相对系统的总结了一些 sd-webui 的使用和基本概念 https://stable-diffusion-art.com/

# 基础模型

> https://huggingface.co/runwayml/stable-diffusion-v1-5

- [v1-5-pruned-emaonly.ckpt](https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.ckpt) - 4.27GB, ema-only weight. uses less VRAM - suitable for inference
  - 适合推理
- [v1-5-pruned.ckpt](https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned.ckpt) - 7.7GB, ema+non-ema weights. uses more VRAM - suitable for fine-tuning
  - 适合调优

> 更推荐下载 `safetensors` 格式

## 更多模型

- C站 https://civitai.com/
- 哩布哩布 https://www.liblib.ai/

