---
layout: distill
title: Training and Inference performance
description: practice
giscus_comments: false
date: 2024-10-21

toc:
  - name: Train Large Models
  - name: Reference
---

## Train Large Models
If a model cannot fit into a single GPU, you can use several strategies to distribute the model and its computations across multiple GPUs or even multiple machines. Here are some common approaches:

### 1. Model Parallelism

Model parallelism involves splitting the model itself across multiple GPUs. Different layers or parts of the model are placed on different GPUs.

Example:

```python
import torch
import torch.nn as nn

class ModelParallelResNet50(nn.Module):
    def __init__(self):
        super(ModelParallelResNet50, self).__init__()
        self.seq1 = nn.Sequential(
            nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3, bias=False),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2, padding=1),
        ).to('cuda:0')
        self.seq2 = nn.Sequential(
            nn.Conv2d(64, 128, kernel_size=3, stride=2, padding=1, bias=False),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
        ).to('cuda:1')

    def forward(self, x):
        x = self.seq1(x)
        x = x.to('cuda:1')
        x = self.seq2(x)
        return x

model = ModelParallelResNet50()
input = torch.randn(16, 3, 224, 224).to('cuda:0')
output = model(input)
```

### 2. Data Parallelism

Data parallelism involves splitting the data across multiple GPUs. Each GPU processes a different subset of the data, and the results are combined.

Example:

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, Dataset
from torch.nn.parallel import DataParallel

class ExampleDataset(Dataset):
    def __init__(self, data, labels):
        self.data = data
        self.labels = labels

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx], self.labels[idx]

class ExampleModel(nn.Module):
    def __init__(self):
        super(ExampleModel, self).__init__()
        self.fc = nn.Linear(10, 2)

    def forward(self, x):
        return self.fc(x)

# Create dataset and dataloader
data = torch.randn(1000, 10)
labels = torch.randint(0, 2, (1000,))
dataset = ExampleDataset(data, labels)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True, num_workers=4, pin_memory=True)

# Initialize model, loss function, and optimizer
model = ExampleModel()
model = DataParallel(model)  # Wrap the model with DataParallel
model = model.to('cuda')
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Training loop
for epoch in range(10):
    model.train()
    for inputs, targets in dataloader:
        inputs, targets = inputs.to('cuda'), targets.to('cuda')

        optimizer.zero_grad()
        outputs = model(inputs)
        loss = criterion(outputs, targets)
        loss.backward()
        optimizer.step()

    print(f"Epoch {epoch+1}, Loss: {loss.item()}")

# Save the model
torch.save(model.state_dict(), "example_model.pth")
```

### 3. Distributed Data Parallelism

#### Example of using DDP:
Distributed Data Parallelism (DDP) is a more advanced form of data parallelism that scales across multiple GPUs and multiple nodes.


```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, Dataset
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP

def setup(rank, world_size):
    dist.init_process_group("nccl", rank=rank, world_size=world_size)

def cleanup():
    dist.destroy_process_group()

class ExampleDataset(Dataset):
    def __init__(self, data, labels):
        self.data = data
        self.labels = labels

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx], self.labels[idx]

class ExampleModel(nn.Module):
    def __init__(self):
        super(ExampleModel, self).__init__()
        self.fc = nn.Linear(10, 2)

    def forward(self, x):
        return self.fc(x)

def main(rank, world_size):
    setup(rank, world_size)

    # Create dataset and dataloader
    data = torch.randn(1000, 10)
    labels = torch.randint(0, 2, (1000,))
    dataset = ExampleDataset(data, labels)
    sampler = torch.utils.data.distributed.DistributedSampler(dataset, num_replicas=world_size, rank=rank)
    dataloader = DataLoader(dataset, batch_size=32, shuffle=False, num_workers=4, pin_memory=True, sampler=sampler)

    # Initialize model, loss function, and optimizer
    model = ExampleModel().to(rank)
    model = DDP(model, device_ids=[rank])
    criterion = nn.CrossEntropyLoss().to(rank)
    optimizer = optim.Adam(model.parameters(), lr=0.001)

    # Training loop
    for epoch in range(10):
        model.train()
        for inputs, targets in dataloader:
            inputs, targets = inputs.to(rank), targets.to(rank)

            optimizer.zero_grad()
            outputs = model(inputs)
            loss = criterion(outputs, targets)
            loss.backward()
            optimizer.step()

        print(f"Rank {rank}, Epoch {epoch+1}, Loss: {loss.item()}")

    cleanup()

if __name__ == "__main__":
    world_size = 2
    torch.multiprocessing.spawn(main, args=(world_size,), nprocs=world_size, join=True)
```

#### Example of ZeRO (Zero Redundancy Optimizer) with Microsoft DeepSpeed.
ZeRO is a technique developed by Microsoft DeepSpeed that reduces memory redundancy across data-parallel processes. It partitions model states, gradients, and optimizer states across multiple GPUs.

```python
import deepspeed
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, Dataset

class ExampleDataset(Dataset):
    def __init__(self, data, labels):
        self.data = data
        self.labels = labels

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx], self.labels[idx]

class ExampleModel(nn.Module):
    def __init__(self):
        super(ExampleModel, self).__init__()
        self.fc = nn.Linear(10, 2)

    def forward(self, x):
        return self.fc(x)

# Create dataset and dataloader
data = torch.randn(1000, 10)
labels = torch.randint(0, 2, (1000,))
dataset = ExampleDataset(data, labels)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True, num_workers=4, pin_memory=True)

# Initialize model, loss function, and optimizer
model = ExampleModel()
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# DeepSpeed configuration
ds_config = {
    "train_batch_size": 32,
    "zero_optimization": {
        "stage": 2,
        "allgather_partitions": True,
        "reduce_scatter": True,
        "allgather_bucket_size": 5e8,
        "reduce_bucket_size": 5e8,
    },
    "fp16": {
        "enabled": True,
    },
}

# Initialize DeepSpeed
model, optimizer, _, _ = deepspeed.initialize(
    model=model,
    optimizer=optimizer,
    model_parameters=model.parameters(),
    config=ds_config
)

# Training loop
for epoch in range(10):
    model.train()
    for inputs, targets in dataloader:
        inputs, targets = inputs.to(model.local_rank), targets.to(model.local_rank)

        optimizer.zero_grad()
        outputs = model(inputs)
        loss = criterion(outputs, targets)
        model.backward(loss)
        model.step()

    print(f"Epoch {epoch+1}, Loss: {loss.item()}")

# Save the model
model.save_checkpoint("example_model_checkpoint")
```

#### Example of Megatron-LM:
Megatron-LM is a framework developed by NVIDIA for training large transformer models. It uses model parallelism and data parallelism to scale training across multiple GPUs and nodes.

```python
# Megatron-LM requires a specific setup and configuration.
# Please refer to the official Megatron-LM repository for detailed instructions:
# https://github.com/NVIDIA/Megatron-LM
```

### 4. Hybrid Sharding Data Parallel
PyTorch Hybrid Sharding Data Parallel (HSDP) is a distributed training strategy that combines the benefits of Fully Sharded Data Parallel (FSDP) and traditional Data Parallelism (DP).

#### How it works:
##### FSDP within a node:
HSDP first shards the model parameters, gradients, and optimizer states across GPUs within a single node using FSDP. This minimizes memory usage on each GPU, allowing for larger models or batch sizes.
##### DP across nodes:
HSDP then replicates the sharded model across multiple nodes using DP. This allows for faster training by distributing the workload across multiple machines.
#### Benefits:
##### Reduced memory usage:
HSDP significantly reduces memory usage compared to DP, enabling training of larger models on limited GPU memory.
##### Improved scalability:
HSDP scales well across multiple nodes, allowing for faster training on large datasets.
##### Flexibility:
HSDP provides a flexible trade-off between memory usage and communication overhead, allowing you to optimize for your specific hardware and model.

#### Example
```python
import torch
import torch.distributed as dist
from torch.distributed.device_mesh import init_device_mesh
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP, ShardingStrategy
from torch.distributed.algorithms.ddp_comm_hooks import default_hooks as default

# Initialize process group
dist.init_process_group(backend='nccl')

# Create a model
class ToyModel(nn.Module):
    def __init__(self):
        super(ToyModel, self).__init__()
        self.net1 = nn.Linear(10, 10)
        self.relu = nn.ReLU()
        self.net2 = nn.Linear(10, 5)

    def forward(self, x):
        return self.net2(self.relu(self.net1(x)))

model = ToyModel()

# HSDP: MeshShape(2, 4)
mesh_2d = init_device_mesh("cuda", (2, 4))

# Wrap the model with FSDP within each node
model = FSDP(model, device_mesh=mesh_2d, sharding_strategy=ShardingStrategy.HYBRID_SHARD)

# Wrap the model with DP across nodes
model = torch.nn.parallel.DistributedDataParallel(model)

# Train the model
...
```
#### Important Considerations:
##### DeviceMesh:
HSDP utilizes the DeviceMesh abstraction introduced in PyTorch 1.13 to manage the mapping of devices.
##### Communication overhead:
While HSDP reduces memory usage, it introduces additional communication overhead compared to DP. This overhead can be mitigated by careful tuning of the sharding strategy and communication optimizations.
##### Compatibility:
Not all models are suitable for HSDP. Models with complex dependencies between layers might require manual modifications.
#### For more information:
<ul>
    <li><a href="https://static.sched.com/hosted_files/pytorch2023/d1/%5BPTC%2023%5D%20Composable%20PyTorch%20Distributed%20with%20PT2.pdf">Talk: Composable PyTorch Distributed with PT2</a></li>
    <li>PyTorch documentation: <a href="https://pytorch.org/tutorials/recipes/distributed_device_mesh.html">distributed_device_mesh.html</a></li>
    <li><a href="https://github.com/pytorch/examples/blob/main/distributed/tensor_parallelism/fsdp_tp_example.py">2D parallel combining Tensor/Sequance Parallel with FSDP</a></li>
    <li>Hugging Face Accelerate: <a href=https://huggingface.co/docs/accelerate/usage_guides/fsdp>fsdp usage_guides</a></li>
</ul>


### 5. Gradient Checkpointing

Gradient checkpointing trades compute for memory by saving only a subset of activations during the forward pass and recomputing them during the backward pass.

Example:

```python
import torch
import torch.nn as nn
from torch.utils.checkpoint import checkpoint

class CheckpointedModel(nn.Module):
    def __init__(self):
        super(CheckpointedModel, self).__init__()
        self.seq = nn.Sequential(
            nn.Linear(10, 50),
            nn.ReLU(),
            nn.Linear(50, 2)
        )

    def forward(self, x):
        def custom_forward(*inputs):
            return self.seq(*inputs)
        return checkpoint(custom_forward, x)

model = CheckpointedModel().to('cuda')
input = torch.randn(16, 10).to('cuda')
output = model(input)
```

### Summary

If a model cannot fit into a single GPU, you can use strategies like model parallelism, data parallelism, distributed data parallelism, and gradient checkpointing to distribute the model and its computations across multiple GPUs or even multiple machines. Each approach has its own trade-offs and can be chosen based on the specific requirements and constraints of your application.


## Reference
TODO

