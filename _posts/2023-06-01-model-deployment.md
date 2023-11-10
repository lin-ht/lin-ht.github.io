---
layout: post
title:  Deep Learning Model Deployment
date:   2023-06-01 16:30:00
description: frameworks and tools
tags: notes
categories: note-posts
---
#### Deployment server
Note: the order doesn't indicate its popularity.
##### 1. Ray
<a href="https://docs.ray.io/en/latest/ray-overview/index.html">Ray</a> is an open-source unified framework for scaling AI and Python applications like machine learning. It provides the compute layer for parallel processing so that you don’t need to be a distributed systems expert.

##### 2. Nvidia Triton
<a href="https://developer.nvidia.com/triton-inference-server">Nvidia Triton</a> an open-source inference serving software, standardizes AI model deployment and execution and delivers fast and scalable AI in production. 

##### 3. Truss
<a href="https://github.com/basetenlabs/truss">Truss</a>: the simplest way to serve AI/ML models in production.

#### Model conversion or coding languages

##### 1. TensorRT
<a href="https://developer.nvidia.com/tensorrt">
 NVIDIA TensorRT </a> is an SDK for high-performance deep learning inference, includes a deep learning inference optimizer and runtime that delivers low latency and high throughput for inference applications.

##### 2. AITemplate
<a href="https://facebookincubator.github.io/AITemplate/index.html">AITemplate</a>(AIT) is a Python framework that transforms deep neural networks into CUDA (NVIDIA GPU) / HIP (AMD GPU) C++ code for lightning-fast inference serving.

##### 3. TorchScript
<a href="https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html">TorchScript</a> is an intermediate representation of a PyTorch model (subclass of nn.Module) that can then be run in a high-performance environment such as C++.

Here is an <a href="https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html">introduction tutorial</a> on TorchScript and the <a href="https://pytorch.org/tutorials/advanced/cpp_export.html">documentation</a> about it.

##### 4. Tensor Comprehensions
<a href="https://facebookresearch.github.io/TensorComprehensions/">
 Tensor Comprehensions </a>(<a href="https://github.com/facebookresearch/TensorComprehensions/">TC</a>)
is a notation based on generalized Einstein notation for computing on multi-dimensional arrays. TC greatly simplifies ML framework implementations by providing a concise and powerful syntax which can be efficiently translated to high-performance computation kernels, automatically.

##### 5. Apache TVM
<a href="https://tvm.apache.org/">Apache TVM</a> is an End to End Machine Learning Compiler Framework for CPUs, GPUs and accelerators. It aims to enable machine learning engineers to optimize and run computations efficiently on any hardware backend.

##### 6. OpenAI Triton
<a href="https://openai.com/research/triton">Triton</a> is an open-source Python-like programming language which enables researchers with no CUDA experience to write highly efficient GPU code—most of the time on par with what an expert would be able to produce.

##### 6. ONNX
<a href="https://onnx.ai/">ONNX</a> is an open format built to represent machine learning models. ONNX defines a common set of operators - the building blocks of machine learning and deep learning models - and a common file format to enable AI developers to use models with a variety of frameworks, tools, runtimes, and compilers.

##### 7. Pytorch Lightning
<a href="https://lightning.ai/docs/app/stable/">Lightning</a> is a hyper-minimalistic framework used to build machine learning components that can plug into existing ML workflows.

##### 8. Torch.compile
<a href="https://pytorch.org/tutorials/intermediate/torch_compile_tutorial.html">Torch.compile</a> makes PyTorch code run faster by JIT-compiling PyTorch code into optimized kernels, all while requiring minimal code changes.

#### Profiling tools:
##### 1. NVIDIA NSight Systems
<a href="https://developer.nvidia.com/nsight-systems">NVIDIA Nsight™ Systems</a> is a system-wide performance analysis tool designed to visualize an application’s algorithms, help you identify the largest opportunities to optimize, and tune to scale efficiently across any quantity or size of CPUs and GPUs, from large servers to our smallest system on a chip (SoC).

