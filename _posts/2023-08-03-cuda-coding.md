---
layout: post
title:  Cuda coding
date:   2023-08-03 19:30:00
description: 
tags: notes code
categories: note-posts
---
#### Cuda multithreading

<a href="https://developer.nvidia.com/blog/cuda-pro-tip-always-set-current-device-avoid-multithreading-bugs/">CUDA Pro Tip: Always Set the Current Device to Avoid Multithreading Bugs</a>

How to select which GPU to execute CUDA calls on? The CUDA runtime API is <b>state-based</b>, and threads should execute `cudaSetDevice()` to set the current GPU. 

The CUDA runtime API is thread-safe, which means it maintains <b>per-thread state</b> about the current device. This is very important as it allows threads to concurrently submit work to different devices. If current device is not set for the thread, either it will use the default device (device 0) or if it is a reused thread it will reuse its last device setting.

Pro Tip: always call `cudaSetDevice()` first when you spawn a new host thread.

```c++
cudaSetDevice(1);
cudaMalloc(&a,bytes);

#pragma omp parallel
{
  // multiple threads calling the kernel
  cudaSetDevice(1);
  kernel<<<blocks,threads>>>(a);
}
```

#### Concurrency with streams

<a href="https://developer.nvidia.com/blog/gpu-pro-tip-cuda-7-streams-simplify-concurrency/">GPU Pro Tip: CUDA 7 Streams Simplify Concurrency</a>

CUDA Applications manage concurrency by executing asynchronous commands in streams, sequences of commands that execute in order. 
[See the post <a href="https://developer.nvidia.com/blog/parallelforall/how-overlap-data-transfers-cuda-cc/">How to Overlap Data Transfers in CUDA C/C++</a> for an example.]

When you execute asynchronous CUDA commands without specifying a stream, the runtime uses the default stream. 

```c++
  kernel<<< blocks, threads, bytes >>>();    // stream 0 (default)
  kernel<<< blocks, threads, bytes, 0 >>>(); // stream 0
```

Before CUDA 7, the default stream is a single <b>per-device</b> default stream which implicitly synchronizes with all other streams on the device.

CUDA 7 introduces an independent <b>per-thread</b> default stream for every host thread, which avoids the serialization of the legacy default stream. These default streams are regular streams. The commands issued to the default streams by different host threads can run concurrently.

To enable per-thread default streams in CUDA 7 and later, you can either compile with the `nvcc` command-line option `--default-stream per-thread`, or `#define CUDA_API_PER_THREAD_DEFAULT_STREAM` preprocessor macro before including CUDA headers (`cuda.h` or `cuda_runtime.h`).

##### Asynchronous Commands in CUDA
As described by the CUDA C Programming Guide, asynchronous commands return control to the calling host thread before the device has finished the requested task (they are non-blocking). These commands are:

<ul>
    <li>Kernel launches;</li>
    <li>Memory copies between two addresses to the same device memory;</li>
    <li>Memory copies from host to device of a memory block of 64 KB or less;</li>
    <li>Memory copies performed by functions with the Async suffix;</li>
    <li>Memory set function calls.</li>
</ul>


##### A Multi-threading Example

```c++
#include <pthread.h>
#include <stdio.h>

const int N = 1 << 20;

__global__ void kernel(float *x, int n)
{
    int tid = threadIdx.x + blockIdx.x * blockDim.x;
    for (int i = tid; i < n; i += blockDim.x * gridDim.x) {
        x[i] = sqrt(pow(3.14159,i));
    }
}

void *launch_kernel(void *dummy)
{
    // Bound thread to device for each thread.
    cudaSetDevice(0);

    float *data;
    cudaMalloc(&data, N * sizeof(float));

    // Use default stream: per-device or per thread default?
    kernel<<<1, 64>>>(data, N);

    // Sync stream 0 within each thread after kernel run.
    cudaStreamSynchronize(0);

    return NULL;
}

int main()
{
    const int num_threads = 8;

    pthread_t threads[num_threads];

    for (int i = 0; i < num_threads; i++) {
        if (pthread_create(&threads[i], NULL, launch_kernel, 0)) {
            fprintf(stderr, "Error creating threadn");
            return 1;
        }
    }

    for (int i = 0; i < num_threads; i++) {
        if(pthread_join(threads[i], NULL)) {
            fprintf(stderr, "Error joining threadn");
            return 2;
        }
    }

    cudaDeviceReset();

    return 0;
}
```

(a) Compile with the legacy default stream will behavior:
```bash
nvcc ./pthread_test.cu -o pthreads_legacy
```
When we run this in `nvvp`, we see a single stream, the default stream, with all kernel launches serialized.

(b) Compile it with the new per-thread default stream option:
```bash
nvcc --default-stream per-thread ./pthread_test.cu -o pthreads_per_thread
```
Each thread creates a new stream automatically and they do not synchronize, so the kernels from all eight threads run concurrently.


