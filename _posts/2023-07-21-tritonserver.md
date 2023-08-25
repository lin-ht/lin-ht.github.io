---
layout: post
title:  Triton server tutorial
date:   2023-07-21 17:49:00
description: Cheatsheet
tags: notes code
categories: note-posts
---

<a href="https://www.run.ai/guides/machine-learning-engineering/triton-inference-server">Triton Inference Server: The Basics and a Quick Tutorial</a>

Github of <a href="https://github.com/triton-inference-server"> Triton inference server</a>.

#### Introduction

Specify triton model by providing model repository path:
```bash
tritonserver --model-repository=<repository-path>
```

There can be multiple versions of each model, with each version stored in a numerically-named subdirectory. The subdirectory’s name must be the model’s version number and it should not be 0.

For example, an ONNX model directory structure looks like this:
```
<repository-path>/
-<model-name>/
--config.pbtxt
--1/
---model.onnx
```

How Triton Client communicate with Triton?
Through GRPC or HTTP requests, to send inputs to Triton and receive outputs.
Examples could be found <a href="https://github.com/triton-inference-server/server/tree/main/docs/examples/model_repository">here</a>.

#### Install and Run Triton

##### Install Triton Docker Image

```bash
docker pull nvcr.io/nvidia/tritonserver:<xx.yy>-py3 
#<xx.yy> represents the version of Triton
```

##### Create Your Model Repository

##### Run Triton

```bash
docker run --gpus=3 --rm -p8000:8000 -p8001:8001 -p8002:8002 -v/full/path/to/docs/examples/model_repository:/models nvcr.io/nvidia/tritonserver:<xx.yy>-py3 tritonserver --model-repository=/models
```
