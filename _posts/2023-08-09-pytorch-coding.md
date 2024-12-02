---
layout: distill
title: Pytorch coding
description: practice
giscus_comments: false
date: 2023-08-09 16:50:00

toc:
  - name: Installation
  - name: Checkpoints saving and loading
  - name: TorchScript tracing tutorial
  - name: Pytorch Extensions
---

## Installation
### Example

#### Install conda
```bash
# https://docs.anaconda.com/miniconda/
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh

~/miniconda3/bin/conda init bash
#~/miniconda3/bin/conda init zsh

conda create -n dev python=3.10.14
conda activate dev
```

#### Install CUDA toolkit
```bash
# add to ~/.bashrc
CUDA_HOME=/usr/local/cuda
LD_LIBRARY_PATH=$CUDA_HOME/lib:/usr/lib/x86_64-linux-gnu${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
PATH=$CUDA_HOME/bin${PATH:+:${PATH}}
```

```bash
nvcc --version
# release 12.4, V12.4.131

# check cudnn installation
ldconfig -p | grep -i cudnn

# Install cuSparseLt
# url: https://docs.nvidia.com/cuda/cusparselt/getting_started.html
wget https://developer.download.nvidia.com/compute/cusparselt/0.6.2/local_installers/cusparselt-local-repo-ubuntu2004-0.6.2_1.0-1_amd64.deb
sudo dpkg -i cusparselt-local-repo-ubuntu2004-0.6.2_1.0-1_amd64.deb
sudo cp /var/cusparselt-local-repo-ubuntu2004-0.6.2/cusparselt-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install libcusparselt0 libcusparselt-dev

ldconfig -p | grep -i cusparselt
```

```bash
# Test cuSparseLt

```

#### Compile Pytorch
##### Install dependencies
```bash
git clone --recurse-submodules -j8 https://github.com/pytorch/pytorch.git
# if you are updating an existing checkout
git submodule sync
git submodule update --init --recursive

conda install cmake ninja
# Run this command from the PyTorch directory after cloning the source code using the “Get the PyTorch Source“ section below
pip install -r requirements.txt
```


```bash
pip install mkl-static mkl-include
# CUDA only: Add LAPACK support for the GPU if needed
conda install -c pytorch magma-cuda124
make triton
```

##### Install Pytorch
```bash
export CMAKE_PREFIX_PATH=${CONDA_PREFIX:-"$(dirname $(which conda))/../"}
export USE_CUSPARSELT=1
export CUSPARSELT_ROOT=/usr/lib/x86_64-linux-gnu

pip install pyyaml
pip install typing-extensions
python setup.py develop
# python setup.py install
```

##### Install torchao
```bash
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
torch 2.5.0a0+git3d45717 requires filelock, which is not installed.
torch 2.5.0a0+git3d45717 requires fsspec, which is not installed.
torch 2.5.0a0+git3d45717 requires jinja2, which is not installed.
torch 2.5.0a0+git3d45717 requires networkx, which is not installed.
torch 2.5.0a0+git3d45717 requires sympy==1.13.1, but you have sympy 1.13.2 which is incompatible.
Successfully installed mpmath-1.3.0 sympy-1.13.2

pip install sympy-1.13.1
pip install pandas
```


### Using prebuilt package
Check your ubuntu system:
```bash
cat /etc/*ease

# Example output:
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.6 LTS"
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.6 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

Check CUDA compiler and others:
```bash
echo $CUDA_HOME
CUDA_HOME=/usr/local/cuda
LD_LIBRARY_PATH=$CUDA_HOME/lib${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
PATH=$CUDA_HOME/bin${PATH:+:${PATH}}

nvcc --version
# Cuda compilation tools, release 12.4, V12.4.131

gcc --version
# gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0

# check cuSparseLt
cd $CUDA_HOME
find -name libcusparse* 
# ./targets/x86_64-linux/lib/libcusparse.so
# ./targets/x86_64-linux/lib/libcusparse_static.a
# ./targets/x86_64-linux/lib/stubs/libcusparse.so
# ./targets/x86_64-linux/lib/libcusparse.so.12
# ./targets/x86_64-linux/lib/libcusparse.so.12.3.1.170


# Install cuSparseLt (different from cuSparse)
# url: https://developer.nvidia.com/cusparse
wget https://developer.download.nvidia.com/compute/cusparselt/0.6.2/local_installers/cusparselt-local-repo-ubuntu2004-0.6.2_1.0-1_amd64.deb
sudo dpkg -i cusparselt-local-repo-ubuntu2004-0.6.2_1.0-1_amd64.deb
sudo cp /var/cusparselt-local-repo-ubuntu2004-0.6.2/cusparselt-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install libcusparselt0 libcusparselt-dev

# Check where the package is installed.
ldconfig -p | grep -i cusparselt
```

Check here to understand cuda architectures: <a href="https://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/">Matching CUDA arch and CUDA gencode for various NVIDIA architectures.</a>


Better to use conda manager to install pytorch:
```bash
conda create -n sparse python=3.10.14

# pick the right installation according to nvcc version
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch-nightly -c nvidia
```


Check the build configuration:
```python
import torch
print(torch.__config__.show())

# Example output:
PyTorch built with:
  - GCC 9.3
  - C++ Version: 201703
  - Intel(R) oneAPI Math Kernel Library Version 2024.2-Product Build 20240605 for Intel(R) 64 architecture applications
  - Intel(R) MKL-DNN v3.4.2 (Git Hash 1137e04ec0b5251ca2b4400a4fd3c667ce843d67)
  - OpenMP 201511 (a.k.a. OpenMP 4.5)
  - LAPACK is enabled (usually provided by MKL)
  - NNPACK is enabled
  - CPU capability usage: AVX2
  - CUDA Runtime 12.4
  - NVCC architecture flags: -gencode;arch=compute_50,code=sm_50;-gencode;arch=compute_60,code=sm_60;-gencode;arch=compute_70,code=sm_70;-gencode;arch=compute_75,code=sm_75;-gencode;arch=compute_80,code=sm_80;-gencode;arch=compute_86,code=sm_86;-gencode;arch=compute_90,code=sm_90;-gencode;arch=compute_90,code=compute_90
  - CuDNN 90.1
  - Magma 2.6.1
  - Build settings: BLAS_INFO=mkl, BUILD_TYPE=Release, CUDA_VERSION=12.4, CUDNN_VERSION=9.1.0, CXX_COMPILER=/opt/rh/devtoolset-9/root/usr/bin/c++, CXX_FLAGS= -D_GLIBCXX_USE_CXX11_ABI=0 -fabi-version=11 -fvisibility-inlines-hidden -DUSE_PTHREADPOOL -DNDEBUG -DUSE_KINETO -DLIBKINETO_NOROCTRACER -DLIBKINETO_NOXPUPTI=ON -DUSE_FBGEMM -DUSE_PYTORCH_QNNPACK -DUSE_XNNPACK -DSYMBOLICATE_MOBILE_DEBUG_HANDLE -O2 -fPIC -Wall -Wextra -Werror=return-type -Werror=non-virtual-dtor -Werror=bool-operation -Wnarrowing -Wno-missing-field-initializers -Wno-type-limits -Wno-array-bounds -Wno-unknown-pragmas -Wno-unused-parameter -Wno-strict-overflow -Wno-strict-aliasing -Wno-stringop-overflow -Wsuggest-override -Wno-psabi -Wno-error=old-style-cast -Wno-missing-braces -fdiagnostics-color=always -faligned-new -Wno-unused-but-set-variable -Wno-maybe-uninitialized -fno-math-errno -fno-trapping-math -Werror=format -Wno-stringop-overflow, LAPACK_INFO=mkl, PERF_WITH_AVX=1, PERF_WITH_AVX2=1, PERF_WITH_AVX512=1, TORCH_VERSION=2.5.0, USE_CUDA=ON, USE_CUDNN=ON, USE_CUSPARSELT=1, USE_EXCEPTION_PTR=1, USE_GFLAGS=OFF, USE_GLOG=OFF, USE_GLOO=ON, USE_MKL=ON, USE_MKLDNN=ON, USE_MPI=OFF, USE_NCCL=1, USE_NNPACK=ON, USE_OPENMP=ON, USE_ROCM=OFF, USE_ROCM_KERNEL_ASSERT=OFF,


# Another example:
PyTorch built with:
  - GCC 9.3
  - C++ Version: 201703
  - Intel(R) oneAPI Math Kernel Library Version 2022.1-Product Build 20220311 for Intel(R) 64 architecture applications
  - Intel(R) MKL-DNN v3.4.2 (Git Hash 1137e04ec0b5251ca2b4400a4fd3c667ce843d67)
  - OpenMP 201511 (a.k.a. OpenMP 4.5)
  - LAPACK is enabled (usually provided by MKL)
  - NNPACK is enabled
  - CPU capability usage: AVX2
  - CUDA Runtime 12.1
  - NVCC architecture flags: -gencode;arch=compute_50,code=sm_50;-gencode;arch=compute_60,code=sm_60;-gencode;arch=compute_61,code=sm_61;-gencode;arch=compute_70,code=sm_70;-gencode;arch=compute_75,code=sm_75;-gencode;arch=compute_80,code=sm_80;-gencode;arch=compute_86,code=sm_86;-gencode;arch=compute_90,code=sm_90
  - CuDNN 90.1  (built against CUDA 12.4)
  - Magma 2.6.1
  - Build settings: BLAS_INFO=mkl, BUILD_TYPE=Release, CUDA_VERSION=12.1, CUDNN_VERSION=9.1.0, CXX_COMPILER=/opt/rh/devtoolset-9/root/usr/bin/c++, CXX_FLAGS= -D_GLIBCXX_USE_CXX11_ABI=0 -fabi-version=11 -fvisibility-inlines-hidden -DUSE_PTHREADPOOL -DNDEBUG -DUSE_KINETO -DLIBKINETO_NOROCTRACER -DLIBKINETO_NOXPUPTI=ON -DUSE_FBGEMM -DUSE_PYTORCH_QNNPACK -DUSE_XNNPACK -DSYMBOLICATE_MOBILE_DEBUG_HANDLE -O2 -fPIC -Wall -Wextra -Werror=return-type -Werror=non-virtual-dtor -Werror=bool-operation -Wnarrowing -Wno-missing-field-initializers -Wno-type-limits -Wno-array-bounds -Wno-unknown-pragmas -Wno-unused-parameter -Wno-strict-overflow -Wno-strict-aliasing -Wno-stringop-overflow -Wsuggest-override -Wno-psabi -Wno-error=old-style-cast -Wno-missing-braces -fdiagnostics-color=always -faligned-new -Wno-unused-but-set-variable -Wno-maybe-uninitialized -fno-math-errno -fno-trapping-math -Werror=format -Wno-stringop-overflow, LAPACK_INFO=mkl, PERF_WITH_AVX=1, PERF_WITH_AVX2=1, PERF_WITH_AVX512=1, TORCH_VERSION=2.5.0, USE_CUDA=ON, USE_CUDNN=ON, USE_CUSPARSELT=1, USE_EXCEPTION_PTR=1, USE_GFLAGS=OFF, USE_GLOG=OFF, USE_GLOO=ON, USE_MKL=ON, USE_MKLDNN=ON, USE_MPI=OFF, USE_NCCL=ON, USE_NNPACK=ON, USE_OPENMP=ON, USE_ROCM=OFF, USE_ROCM_KERNEL_ASSERT=OFF, 

```

Check the dynamic libs that pytorch linked to:
```bash
cd /opt/venv/lib/python3.10/site-packages/torch
ls
ldd _C.cpython-310-x86_64-linux-gnu.so
    # linux-vdso.so.1 (0x00007ffddb04d000)
    # libtorch_python.so => /opt/venv/lib/python3.10/site-packages/torch/./lib/libtorch_python.so (0x00007f15261b9000)
    # libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f1526189000)
    # libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f1525f97000)
    # libtorch.so => /opt/venv/lib/python3.10/site-packages/torch/./lib/libtorch.so (0x00007f1525f6f000)
    # libshm.so => /opt/venv/lib/python3.10/site-packages/torch/./lib/libshm.so (0x00007f1525f63000)
    # libnvToolsExt.so.1 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/nvtx/lib/libnvToolsExt.so.1 (0x00007f1525d59000)
    # libtorch_cpu.so => /opt/venv/lib/python3.10/site-packages/torch/./lib/libtorch_cpu.so (0x00007f1511672000)
    # libtorch_cuda.so => /opt/venv/lib/python3.10/site-packages/torch/./lib/libtorch_cuda.so (0x00007f14c99a7000)
    # libc10_cuda.so => /opt/venv/lib/python3.10/site-packages/torch/./lib/libc10_cuda.so (0x00007f14c98f4000)
    # libcudart.so.12 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cuda_runtime/lib/libcudart.so.12 (0x00007f14c9645000)
    # libc10.so => /opt/venv/lib/python3.10/site-packages/torch/./lib/libc10.so (0x00007f14c952d000)
    # libcudnn.so.9 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cudnn/lib/libcudnn.so.9 (0x00007f14c9313000)
    # libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f14c912f000)
    # libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f14c9114000)
    # /lib64/ld-linux-x86-64.so.2 (0x00007f152761c000)
    # librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f14c910a000)
    # libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f14c8fbb000)
    # libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f14c8fb5000)
    # libgomp-a34b3233.so.1 => /opt/venv/lib/python3.10/site-packages/torch/./lib/libgomp-a34b3233.so.1 (0x00007f14c8d89000)
    # libcupti.so.12 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cuda_cupti/lib/libcupti.so.12 (0x00007f14c83ed000)
    # libcusparse.so.12 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cusparse/lib/libcusparse.so.12 (0x00007f14b759f000)
    # libcufft.so.11 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cufft/lib/libcufft.so.11 (0x00007f14a59a2000)
    # libcusparseLt-f80c68d1.so.0 => /opt/venv/lib/python3.10/site-packages/torch/./lib/libcusparseLt-f80c68d1.so.0 (0x00007f14a2e04000)
    # libcurand.so.10 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/curand/lib/libcurand.so.10 (0x00007f149c9be000)
    # libcublas.so.12 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cublas/lib/libcublas.so.12 (0x00007f1495f06000)
    # libcublasLt.so.12 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cublas/lib/libcublasLt.so.12 (0x00007f147806f000)
    # libnccl.so.2 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/nccl/lib/libnccl.so.2 (0x00007f1469884000)
    # libutil.so.1 => /lib/x86_64-linux-gnu/libutil.so.1 (0x00007f146987d000)
    # libnvJitLink.so.12 => /opt/venv/lib/python3.10/site-packages/torch/./lib/../../nvidia/cusparse/lib/../../nvjitlink/lib/libnvJitLink.so.12 (0x00007f14662ec000)

# readelf will display all definitions inside the executable.
readelf -a -W libcusparseLt-f80c68d1.so.0 | grep cusparse
```

Use vscode editor:
```bash
code -r <filename>
```

Check CuDNN:
```bash
# Check CuDNN
# The installation of CuDNN is just copying some files.
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
cat /usr/local/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2

# Install CuDNN
# url: https://developer.nvidia.com/cudnn
# Example: cuDNN 9.3.0 under ubuntu 20.04
wget https://developer.download.nvidia.com/compute/cudnn/9.3.0/local_installers/cudnn-local-repo-ubuntu2004-9.3.0_1.0-1_amd64.deb
sudo dpkg -i cudnn-local-repo-ubuntu2004-9.3.0_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-ubuntu2004-9.3.0/cudnn-*-keyring.gpg /usr/share/keyrings/
# Ensure no confilicting/duplicated items in Signed-in lists (/etc/apt/sources.list.d)
sudo apt-get update
sudo apt-get -y install cudnn
# or precisely:
sudo apt-get -y install cudnn-cuda-12 # updated automatically to cudnn9-cuda-12

# Check the apt
apt show cudnn9-cuda-12
# or
dpkg -s cudnn9-cuda-12

# Show all files brought in by the package
dpkg -L cudnn9-cuda-12
# Locate the lib:
ldconfig -p | grep libcudnn
# Check version 
ldconfig -v | grep -i cudnn
```


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


### Loading Your Script Module in C++

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

## Pytorch Distributed Training
<a href="https://pytorch.org/docs/stable/distributed.html">Torch.distributed</a>

Some <a href="https://pytorch.org/docs/stable/elastic/run.html">definitions</a>:

1. Node - A physical instance or a container; maps to the unit that the job manager works with.
2. Worker - A worker in the context of distributed training.
3. WorkerGroup - The set of workers that execute the same function (e.g. trainers).
4. LocalWorkerGroup - A subset of the workers in the worker group running on the same node.
5. RANK - The rank of the worker within a worker group.
6. WORLD_SIZE - The total number of workers in a worker group.
7. LOCAL_RANK - The rank of the worker within a local worker group.
8. LOCAL_WORLD_SIZE - The size of the local worker group.
10. rdzv_id - A user-defined id that uniquely identifies the worker group for a job. This id is used by each node to join as a member of a particular worker group.
11. rdzv_backend - The backend of the rendezvous (e.g. c10d). This is typically a strongly consistent key-value store.
12. rdzv_endpoint - The rendezvous backend endpoint; usually in form <host>:<port>.

A Node runs LOCAL_WORLD_SIZE workers which comprise a LocalWorkerGroup. The union of all LocalWorkerGroups in the nodes in the job comprise the WorkerGroup.

## Pytorch DDP Debugging in VSCode
<a href="https://medium.com/@franoisponchon/pytorch-ddp-debugging-in-vscode-4fb162eba07e">Pytorch DDP Debugging in VSCode</a>
Distributed Data Parallel (DDP) Debugging

### Non-distributed version
Example launch.json:
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            // ${file} will be replaced by the current opened file
            // It can be a problem when you want to run code from a file
            // and debug a dependancy.
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": false
        },

        {
            "name": "Example: Classification Training",
            "type": "python",
            "request": "launch",
            // In this example, we run everytime the same file
            "program": "examples/train_classification.py",
            "console": "integratedTerminal",
            "justMyCode": false,

        }
   ]
}
```

Under the hood, the command is:
```bash
cd {workspace_dir} ; /usr/bin/env {env_path}/bin/python \
{vscode_dir}/debugpy/launcher 52843 -- examples/train_classification.py 
```
Vscode asks python to launch a debugpy server listening on port 52843 to listen the process that we want to debug.

### Distributed version
When you use distributed code is that you no longer run it with traditionnal python command, for example:

```bash
# Not distributed
python example/train_classification.py

# Distributed (no longer use python).
torchrun --nproc_per_node=2 example/train_classification_multicpu.py
# Equivalent to : 
python -m torch.distributed.launch --use_env --nproc_per_node=2 example/train_classification_multicpu.py
```
<a href="https://pytorch.org/docs/stable/elastic/run.html">Torchrun</a> is a python console script to the main module torch.distributed.run declared in the entry_points configuration in setup.py. It is equivalent to invoking python -m torch.distributed.run.

#### Assist with Accelerate from Huggingface.
First, <a href="https://huggingface.co/docs/accelerate/index?source=post_page-----4fb162eba07e--------------------------------">setup the code with accelerate</a>.
Just note that accelerate script can be run with traditionnal DDP commands.
```bash
# Accelerate command
accelerate launch --num_processes 2 example/train_classification.py
# Almost Equivalent
torchrun --nproc_per_node=2 example/train_classification_multicpu.py
```

#### Gloo Backend / CPU Distributed Training
The first one, at the beggining of the code, we’ll force pytorch to use <a href="https://github.com/facebookincubator/gloo">gloo</a> backend. 
gloo allows us to avoid nccl. Another advantage is that gloo can be run on windows when nccl (to my knowledge) is only available on linux.
```python
import accelerate
import torch.distributed as dist

# Because torch is initialized before accelerate, 
# accelerate will take in account this configuration
# source : 
# https://github.com/huggingface/accelerate/issues/141
dist.init_process_group(backend='gloo')
ws = dist.get_world_size()
```

Finally, because we don’t have multiple gpu on our computer, we’ll distribute our job over multiple cpus.
```python
# Use accelerator on cpu
accelerator = Accelerator(cpu=True)
```
Because we use accelerate, it’s the two only operations you have to do.


#### Code example with bugs
See the original post.

#### Debug in VSCode
Modifications for the launch.json:
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
         ...,
        {
            "name": "Example: Classification Training",
            "type": "python",
            "request": "launch",
            // In this example, we run everytime the same file
            "program": "examples/train_classification.py",
            "console": "integratedTerminal",
            "justMyCode": false,

        },
        {
            "name": "Example: Classification - MultiCPU",
            "type": "python",
            "request": "launch",
            // we launch a module...
            "module":"torch.distributed.launch",
            // with args...
            "args":["--use_env","--nproc_per_node=2","example/train_classification_multicpu.py"],
            "console": "integratedTerminal",
            "justMyCode": false
        },
   ]
}
```
When running this way, we see that everything appears in double in our terminal.
If we put a breakpoint, we’ll see that vscode will stop us 2 times, one in each process!

#### Dynamic configuration
If we need dynamic configurations, e.g. different args for each run, we can write a script to automatically update the launch.json as needed.

## Pytorch Extensions

<a href="https://pytorch.org/tutorials/advanced/cpp_extension.html">CUSTOM C++ AND CUDA EXTENSIONS</a>
