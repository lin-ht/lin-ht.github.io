---
layout: post
title:  Deep Learning Model Deployment
date:   2023-06-01 16:30:00
description: frameworks and tools
tags: notes
categories: note-posts
---
#### Frameworks and tools
Note: the order doesn't indicate its popularity.
##### 1. Ray
<a href="https://docs.ray.io/en/latest/ray-overview/index.html">Ray</a> is an open-source unified framework for scaling AI and Python applications like machine learning. It provides the compute layer for parallel processing so that you don’t need to be a distributed systems expert.

##### 2. Tensor Comprehensions
<a href="https://facebookresearch.github.io/TensorComprehensions/">
 Tensor Comprehensions </a>(<a href="https://github.com/facebookresearch/TensorComprehensions/">TC</a>)
is a notation based on generalized Einstein notation for computing on multi-dimensional arrays. TC greatly simplifies ML framework implementations by providing a concise and powerful syntax which can be efficiently translated to high-performance computation kernels, automatically.

##### 3. Apache TVM
<a href="https://tvm.apache.org/">Apache TVM</a> is an End to End Machine Learning Compiler Framework for CPUs, GPUs and accelerators. It aims to enable machine learning engineers to optimize and run computations efficiently on any hardware backend.


##### 4. OpenAI Triton
<a href="https://openai.com/research/triton">Triton</a> is an open-source Python-like programming language which enables researchers with no CUDA experience to write highly efficient GPU code—most of the time on par with what an expert would be able to produce.


##### 5. AITemplate
<a href="https://facebookincubator.github.io/AITemplate/index.html">AITemplate</a>(AIT) is a Python framework that transforms deep neural networks into CUDA (NVIDIA GPU) / HIP (AMD GPU) C++ code for lightning-fast inference serving.


#### Profiling tools:
##### 1. NVIDIA NSight Systems
<a href="https://developer.nvidia.com/nsight-systems">NVIDIA Nsight™ Systems</a> is a system-wide performance analysis tool designed to visualize an application’s algorithms, help you identify the largest opportunities to optimize, and tune to scale efficiently across any quantity or size of CPUs and GPUs, from large servers to our smallest system on a chip (SoC).

