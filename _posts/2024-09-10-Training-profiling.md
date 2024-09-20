---
layout: distill
title: Training and Inference performance
description: practice
giscus_comments: false
date: 2024-09-10

toc:
  - name: Git repos
  - name: Profiling tools 
  - name: Reference
---

## Git repos
<a href="https://github.com/pytorch/torchtitan/tree/main">TorchTitan: A native PyTorch Library for large model training.</a>

<a href="https://github.com/pytorch/ao">Torchao: PyTorch native quantization and sparsity for training and inference.</a>

<a href="https://github.com/pytorch/ao">Diffusers-torchao: End-to-end recipes for optimizing diffusion models with torchao and diffusers (inference and FP8 training).</a>

<a href="https://github.com/facebookincubator/gloo">Gloo: Collective communications library with various primitives for multi-machine training.</a>

<a href="https://github.com/NVIDIA/TensorRT">TensorRT open source: NVIDIA® TensorRT™ is an SDK for high-performance deep learning inference on NVIDIA GPUs.</a>

## Pytorch cuda best practice
<a href="https://pytorch.org/docs/main/notes/cuda.html#best-practices">Best practices</a>.

## Profiling tools
### Pytorch profiler
<a href="https://pytorch.org/docs/stable/torch.compiler_profiling_torch_compile.html">Profiling to understand torch.compile performance</a>
Example:
```python
import torch
from torchvision.models import resnet18

model = resnet18().cuda()
inputs = [torch.randn((5, 3, 224, 224), device='cuda') for _ in range(10)]

model_c = torch.compile(model)

def fwd_bwd(inp):
    out = model_c(inp)
    out.sum().backward()

def warmup_compile():
    def fn(x):
        return x.sin().relu()

    x = torch.rand((2, 2), device='cuda', requires_grad=True)
    fn_c = torch.compile(fn)
    out = fn_c(x)
    out.sum().backward()

with torch.profiler.profile() as prof:
    with torch.profiler.record_function("warmup compile"):
        warmup_compile()

    with torch.profiler.record_function("resnet18 compile"):
        fwd_bwd(inputs[0])

prof.export_chrome_trace("trace_compile.json")
```
<a href="https://pytorch.org/docs/stable/torch.compiler_faq.html#torch-compiler-graph-breaks">Why am I not seeing speedups? Graph Breaks</a>
Identify the cause of graph breaks:
```python
import torch
import torch._dynamo as dynamo
def toy_example(a, b):
    x = a / (torch.abs(a) + 1)
    print("woo")
    if b.sum() < 0:
        b = b * -1
    return x * b
explanation = dynamo.explain(toy_example)(torch.randn(10), torch.randn(10))
print(explanation)
"""
Graph Count: 3
Graph Break Count: 2
Op Count: 5
Break Reasons:
  Break Reason 1:
    Reason: builtin: print [<class 'torch._dynamo.variables.constant.ConstantVariable'>] False
    User Stack:
      <FrameSummary file foo.py, line 5 in toy_example>
  Break Reason 2:
    Reason: generic_jump TensorVariable()
    User Stack:
      <FrameSummary file foo.py, line 6 in torch_dynamo_resume_in_toy_example_at_5>
Ops per Graph:
  ...
Out Guards:
  ...
"""
```

### Google trace viewer
<a href="https://ui.perfetto.dev/">perfetto</a>, originally "chrome://tracing", which is <a href="https://chromium.googlesource.com/catapult/+/refs/heads/main/tracing/docs/perfetto.md">deprecated.</a>

<a href="https://perfetto.dev/docs/quickstart/trace-analysis"> Quickstart: SQL-based analysis and trace-based metrics</a>.

### TensorBoard
<a href="https://pytorch.org/tutorials/intermediate/tensorboard_profiler_tutorial.html">PyTorch Profiler With TensorBoard</a>

## Reference
<a href="https://github.com/NVIDIA/TensorRT-LLM">TensorRT-LLM github.</a>

