---
layout: distill
title: Pytorch coding
description: practice
giscus_comments: false
date: 2023-08-09 16:50:00

toc:
  - name: Checkpoints saving and loading
  - name: TorchScript tracing tutorial
  - name: Pytorch Extensions
---

## Checkpoints saving and loading

<a href="https://pytorch.org/tutorials/recipes/recipes/saving_and_loading_a_general_checkpoint.html">Saving and loading a general checkpoint in pytorch</a>

### Define, initialize and train a model:

```python
import torch
import torch.nn as nn
import torch.optim as optim

class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.pool = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

net = Net()
print(net)


# optimizer
optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)

# Train
# ...

```

### Save checkpoint

```python
# Additional information
EPOCH = 5
PATH = "model.pt"
LOSS = 0.4

torch.save({
            'epoch': EPOCH,
            'model_state_dict': net.state_dict(),
            'optimizer_state_dict': optimizer.state_dict(),
            'loss': LOSS,
            }, PATH)
```

### Load checkpoint

```python
# load the checkpoint
model = Net()
optimizer = optim.SGD(model.parameters(), lr=0.001, momentum=0.9)

checkpoint = torch.load(PATH)
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']

# continue to evaluation or training
model.eval()
# - or -
model.train()
```

## TorchScript tracing tutorial
Here is an <a href="https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html">introduction tutorial</a> on TorchScript and the documentation (<a href="https://pytorch.org/tutorials/advanced/cpp_export.html">LOADING A TORCHSCRIPT MODEL IN C++</a>) about it.

### Converting to Torch Script via Tracing
```python
import torch
import torchvision

# An instance of your model.
model = torchvision.models.resnet18()

# An example input you would normally provide to your model's forward() method.
example = torch.rand(1, 3, 224, 224)

# Use torch.jit.trace to generate a torch.jit.ScriptModule via tracing.
traced_script_module = torch.jit.trace(model, example)
```

### Converting to Torch Script via Annotation
In case your model employs particular forms of control flow (data dependent if-else).

```python
class MyModule(torch.nn.Module):
    def __init__(self, N, M):
        super(MyModule, self).__init__()
        self.weight = torch.nn.Parameter(torch.rand(N, M))

    def forward(self, input):
        if input.sum() > 0:
          output = self.weight.mv(input)
        else:
          output = self.weight + input
        return output

my_module = MyModule(10,20)
sm = torch.jit.script(my_module)
```

### Serializing Your Script Module to a File

```python
traced_script_module.save("traced_resnet_model.pt")
```


### Loading Loading Your Script Module in C++

```cpp
#include <torch/script.h> // One-stop header.

#include <iostream>
#include <memory>

int main(int argc, const char* argv[]) {
  if (argc != 2) {
    std::cerr << "usage: example-app <path-to-exported-script-module>\n";
    return -1;
  }


  torch::jit::script::Module module;
  try {
    // Deserialize the ScriptModule from a file using torch::jit::load().
    module = torch::jit::load(argv[1]);
  }
  catch (const c10::Error& e) {
    std::cerr << "error loading the model\n";
    return -1;
  }

  std::cout << "ok\n";
}
```

Depending on LibTorch and Building the Application:
```bash
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(custom_ops)

find_package(Torch REQUIRED)

add_executable(example-app example-app.cpp)
target_link_libraries(example-app "${TORCH_LIBRARIES}")
set_property(TARGET example-app PROPERTY CXX_STANDARD 14)
```

## Pytorch Extensions

<a href="https://pytorch.org/tutorials/advanced/cpp_extension.html">CUSTOM C++ AND CUDA EXTENSIONS</a>
