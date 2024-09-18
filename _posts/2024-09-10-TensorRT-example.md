---
layout: distill
title: TensorRT example
description: practice
giscus_comments: false
date: 2024-09-10

toc:
  - name: TensorRT examples
  - name: Reference
---

## TensorRT examples
Examples of exporting torch model to TensorRT.

### Dynamic shapes
Prepare running env:
```bash
pip install cuda-python
pip install tensorrt==10.3.0
```

Example of TensorRT conversion with inputs and outputs of dynamic shapes.

```python
import os
import torch

from pathlib import Path
from copy import deepcopy

from functools import reduce
import onnx
import onnxruntime
import tensorrt as trt
from cuda import cudart


def compute_rms_norm(x, dim, eps):
    dtype = reduce(torch.promote_types, (x.dtype, torch.float32))
    mean_sq = x.to(dtype).square().mean(dim=dim, keepdim=True)
    return (mean_sq + eps).rsqrt().to(x.dtype)


class TopK(torch.nn.Module):
    def __init__(self, k: int, dim: int, eps: float = 1e-5) -> None:
        super().__init__()
        self.k = k
        self.dim = dim
        self.eps = eps

    def forward(self, x):
        v = compute_rms_norm(x, -1, self.eps)
        values, indices = torch.topk(v, self.k, dim=self.dim)
        return v, indices


class SlicedLinearModel(torch.nn.Module):
    def __init__(self, input_size, output_size, topk, dim):
        super(SlicedLinearModel, self).__init__()
        self.linear = torch.nn.Linear(input_size, output_size)
        self.topk = TopK(topk, dim)
        self.dim = dim

    def forward(self, x):
        x = self.linear(x)
        v, k = self.topk(x)
        x = torch.index_select(x, self.dim, k.flatten())
        return x, v, k


def load_model(input_size=5, output_size=5, topk=4):
    model = SlicedLinearModel(input_size, output_size, topk=topk, dim=1)
    torch.nn.init.xavier_uniform_(model.linear.weight)
    torch.nn.init.zeros_(model.linear.bias)
    return model


def compile_onnx_model(model, input_x, onnx_path="slicing_test_dynamic.onnx", force_export=False):
    example_input = (input_x)
    input_names = ["input_x"]
    output_names = ["pred_x", "pred_v", "pred_k"]

    dynamic_axes = {
        "input_x" : {1: "sequence_length"},
        "pred_x" : {1: "sequence_length"},
        "pred_v" : {1: "sequence_length"},
        "pred_k" : {1: "sequence_length"}
    }

    if force_export or not os.path.exists(onnx_path):
        with torch.no_grad():
            torch.onnx.export(model, example_input, onnx_path, input_names=input_names, output_names=output_names, dynamic_axes=dynamic_axes, export_params=True, do_constant_folding=True, opset_version=18)

            with torch.no_grad():
                pred_x, pred_v, pred_k = model(input_x)

                print(f"Calibration input_x[{input_x.shape}]:\n{input_x}")
                print(f"Calibration pred_x[{pred_x.shape}]:\n{pred_x}")
                print(f"Calibration pred_v[{pred_v.shape}]:\n{pred_v}")
                print(f"Calibration pred_k[{pred_k.shape}]:\n{pred_k}")
    else:
        print(f"ONNX model already exists at {onnx_path}")


TRT_LOGGER = trt.Logger(trt.Logger.WARNING)
TRT_DTYPE_TO_TORCH = {
    trt.float32: torch.float32,
    trt.float16: torch.float16,
    trt.int32: torch.int32,
    trt.int64: torch.int64,
    trt.int8: torch.int8,
    trt.bool: torch.bool,
    trt.bfloat16: torch.bfloat16,
}


def compile_trt_model(input_shape, output_shapes, onnx_path, trt_path=None):
    # Convert onnx model to tensorrt
    batch_size, seq_length, input_size = input_shape
    _, topk, output_size = output_shapes[0]

    # Load the ONNX model
    onnx_model = onnx.load(onnx_path)

    # Create a TensorRT builder and network
    builder = trt.Builder(TRT_LOGGER)
    config = builder.create_builder_config()
    # Set cache
    cache = config.create_timing_cache(b"")
    config.set_timing_cache(cache, ignore_mismatch=False)
    config.set_flag(trt.BuilderFlag.FP16) # BF16
    # config.set_flag(trt.BuilderFlag.STRICT_TYPES)
    config.profiling_verbosity = trt.ProfilingVerbosity.DETAILED
    config.runtime_platform = trt.RuntimePlatform.SAME_AS_BUILD
    # config.set_flag(trt.BuilderFlag.ExportLayerInfo)

    # https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#build_engine_python
    # max_workspace = (1 << 30)
    # config.set_memory_pool_limit(trt.MemoryPoolType.WORKSPACE, max_workspace)

    # Set profile
    # https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#work_dynamic_shapes
    opt_profile = builder.create_optimization_profile()
    opt_profile.set_shape("input_x", (batch_size, 1, input_size), (batch_size, topk, input_size), (batch_size, 50, input_size))
    # opt_profile.set_shape("pred_x", (batch_size, topk, output_size), (batch_size, topk, output_size), (batch_size, topk, output_size))
    opt_profile.set_shape("pred_v", (batch_size, 1, 1), (batch_size, topk, 1), (batch_size, 50, 1))
    # opt_profile.set_shape("pred_k", (batch_size, topk, 1), (batch_size, topk, 1), (batch_size, topk, 1))
    # preprocessorConfig = builder.create_network_config()
    # preprocessorConfig.add_optimization_profile(profile)
    config.add_optimization_profile(opt_profile)

    # https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#version-compat
    # https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#explicit-implicit-batch
    flag = 1 << int(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)
    network = builder.create_network(flag)

    # Create an ONNX parser and parse the ONNX model into the TensorRT network
    parser = trt.OnnxParser(network, TRT_LOGGER)
    # Parse the ONNX model from file
    # with open(onnx_path, "rb") as f:
    #     if not parser.parse(f.read()):
    #         print(f"ERROR: Failed to parse the ONNX file {onnx_path}")
    #         for error in range(parser.num_errors):
    #             print(parser.get_error(error))
    onnx_model_str = onnx_model.SerializeToString()
    if not parser.parse(onnx_model_str):
        print(f"ERROR: Failed to parse the ONNX file {onnx_model_str}")
        for error in range(parser.num_errors):
            print(parser.get_error(error))

    # exort_layer_info = "slicing_test_layer_info.json"
    # https://github.com/NVIDIA/TensorRT/issues/2247
    # https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/infer/Core/EngineInspector.html
    # https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/infer/Core/Engine.html#tensorrt.ICudaEngine.create_engine_inspector
    # https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/infer/Core/BuilderConfig.html#tensorrt.ProfilingVerbosity
    # https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/infer/Core/ExecutionContext.html#tensorrt.IExecutionContext.report_to_profiler

    # Build the TensorRT engine
    engine_bytes = builder.build_serialized_network(network, config)
    if engine_bytes:
        # Save the TensorRT engine to a file
        trt_path = onnx_path[:-4]+ "trt" if trt_path is None else trt_path
        with open(trt_path, "wb") as f:
            f.write(engine_bytes)

        print("TensorRT engine saved successfully!")
    else:
        print("ERROR: Failed to build the TensorRT engine!")


def main():
    input_size = 5
    output_size = 5
    topk = 4
    model = load_model(input_size, output_size, topk)

    # Avoid accuracy issues with float16 betweeen CPU and GPU, use float32 to test onnx
    dtype = torch.bfloat16 # torch.bfloat16 # torch.float32 # torch.float16
    device = "cuda"
    model = model.to(dtype).to(device).eval()

    batch_size = 1
    seq_length = 20
    input_x = torch.rand(batch_size, seq_length, input_size).to(dtype).to(device)
    output_shapes = [torch.Size([batch_size, topk, input_size]), torch.Size([batch_size, seq_length, 1]), torch.Size([batch_size, topk, 1])]

    onnx_path="slicing_test_dynamic.onnx"
    compile_onnx_model(model, input_x, onnx_path=onnx_path, force_export=True)
    trt_path = onnx_path[:-4]+ "trt"
    compile_trt_model(input_shape=(batch_size, seq_length, input_size), output_shapes=output_shapes, onnx_path=onnx_path, trt_path=trt_path)

    # Onnx doesn't work with torch.bfloat16
    if dtype == torch.float32:
        verify_onnx_model(model, (batch_size, seq_length, input_size), onnx_path, dtype, device)

    verify_trt_model(model, (batch_size, seq_length, input_size), topk, trt_path, dtype, device, dynamic_shape=True)

    print("All checks pass!")


def verify_onnx_model(model, input_shape, onnx_path, dtype, device="cuda"):
    # Verify the ONNX model and TensorRT engine
    # Load the ONNX model
    onnx_model = onnx.load(onnx_path)
    # Create an inference session with the ONNX model
    session = onnxruntime.InferenceSession(onnx_model.SerializeToString())

    batch_size, seq_length, input_size = input_shape
    for i in range(2):
        seq_length = torch.randint(5, 51, (1,)).item()
        input_x = torch.rand(batch_size, seq_length, input_size).to(dtype).to(device)
        with torch.no_grad():
            pred_x, pred_v, pred_k = model(input_x)

        print(f"input_x[{input_x.shape}]:\n{input_x}")
        print(f"pred_x[{pred_x.shape}]:\n{pred_x}")
        print(f"pred_v[{pred_v.shape}]:\n{pred_v}")
        print(f"pred_k[{pred_k.shape}]:\n{pred_k}")

        # Prepare the input data
        input_data = {"input_x": input_x.cpu().numpy()}

        # Run the model
        output = session.run(None, input_data)

        # Get the output tensor
        onnx_pred_x = torch.from_numpy(output[0]).to(device)
        onnx_pred_v = torch.from_numpy(output[1]).to(device)
        onnx_pred_k = torch.from_numpy(output[2]).to(device)

        print(f"onnx run pred_x[{onnx_pred_x.shape}]:\n{onnx_pred_x}")
        print(f"onnx run pred_v[{onnx_pred_v.shape}]:\n{onnx_pred_v}")
        print(f"onnx run pred_k[{onnx_pred_k.shape}]:\n{onnx_pred_k}")

        # Verify if pred_x is equal to onnx_pred_x
        assert torch.allclose(pred_x, onnx_pred_x.to(device), atol=1e-3), "pred_x and onnx_pred_x are not equal"
        assert torch.allclose(pred_v, onnx_pred_v.to(device), atol=1e-3), "pred_v and onnx_pred_v are not equal"
        assert torch.allclose(pred_k, onnx_pred_k.to(device), atol=1e-3), "pred_k and onnx_pred_k are not equal"

    print("Onnx model checks pass!")


def verify_trt_model(model, input_shape, topk, trt_path, dtype, device="cuda", dynamic_shape=False):
    # Verify TensorRT engine
    # Load the TRT model

    profile_idx = 0 if dynamic_shape else None
    runtime = trt.Runtime(TRT_LOGGER)

    with open(trt_path, "rb") as f:
        engine = runtime.deserialize_cuda_engine(f.read())
        if engine is None:
            raise RuntimeError(f"Failed to reload TRT cuda engine from {trt_path}.")
        else:
            print(f"Finished loading TRT engine from {trt_path}")

    trt_stream = cudart.cudaStreamCreate()[1]
    trt_context = engine.create_execution_context()
    trt_inputs: dict[str, torch.Tensor] = {}
    trt_outputs: dict[str, torch.Tensor] = {}

    # Allocate buffers for inputs and outputs
    default_max_dim_size = 50
    with torch.cuda.device(device="cuda:0"):
        # current_device = torch.cuda.current_device()
        for i in range(engine.num_io_tensors):
            tensor_name = engine.get_tensor_name(i)

            trt_shape = engine.get_tensor_shape(tensor_name)
            if torch.any(torch.tensor(trt_shape) == -1):
                # Must define all dynamic shapes in the profile
                trt_shape_profile = engine.get_tensor_profile_shape(tensor_name, profile_idx)[-1]
                replace_negative_dims = False
                try:
                    if len(trt_shape_profile) == len(trt_shape):
                        trt_shape = trt_shape_profile
                    else:
                        replace_negative_dims = True
                except Exception as e:
                    replace_negative_dims = True

                if replace_negative_dims:
                    # breakpoint()
                    print(f"Profile shape {trt_shape_profile} does not match the engine shape {trt_shape}")
                    trt_shape = tuple([default_max_dim_size if dim == -1 else dim for dim in trt_shape])
                    print(f"Warnning: Using default max shape {trt_shape}")

            trt_tensor = torch.empty(tuple(trt_shape),
                                    dtype=TRT_DTYPE_TO_TORCH[engine.get_tensor_dtype(tensor_name)],
                                    device="cuda:0")

            if engine.get_tensor_mode(tensor_name) == trt.TensorIOMode.INPUT:
                trt_inputs[tensor_name] = trt_tensor
            else:
                trt_outputs[tensor_name] = trt_tensor
            trt_context.set_tensor_address(tensor_name, trt_tensor.data_ptr())

    batch_size, seq_length, input_size = input_shape
    try:
        for i in range(2):
            seq_length = torch.randint(5, 51, (1,)).item()
            input_x = torch.rand(batch_size, seq_length, input_size).to(dtype).to(device)
            output_shapes = [torch.Size([batch_size, topk, input_size]), torch.Size([batch_size, seq_length, 1]), torch.Size([batch_size, topk, 1])]

            with torch.no_grad():
                pred_x, pred_v, pred_k = model(input_x)

            print(f"input_x[{input_x.shape}]:\n{input_x}")
            print(f"pred_x[{pred_x.shape}]:\n{pred_x}")
            print(f"pred_v[{pred_v.shape}]:\n{pred_v}")
            print(f"pred_k[{pred_k.shape}]:\n{pred_k}")

            # Run TRT model
            input_data = {"input_x": input_x}
            with torch.cuda.device(device="cuda:0"):
                for key, value in input_data.items():
                    trt_inputs[key][:, :int(value.shape[1]), :].copy_(value)
                    trt_context.set_input_shape(key, tuple(value.shape))
                trt_context.execute_async_v3(trt_stream)

            trt_pred_x_, trt_pred_v_, trt_pred_k_ = trt_outputs["pred_x"], trt_outputs["pred_v"], trt_outputs["pred_k"]

            # Flatten the tensors
            trt_pred_x = trt_pred_x_.flatten()[:output_shapes[0].numel()].view(output_shapes[0])
            trt_pred_v = trt_pred_v_.flatten()[:output_shapes[1].numel()].view(output_shapes[1])
            trt_pred_k = trt_pred_k_.flatten()[:output_shapes[2].numel()].view(output_shapes[2])

            print(f"trt run pred_x[{trt_pred_x.shape}]:\n{trt_pred_x}")
            print(f"trt run pred_v[{trt_pred_v.shape}]:\n{trt_pred_v}")
            print(f"trt run pred_k[{trt_pred_k.shape}]:\n{trt_pred_k}")

            # Verify if pred_x is equal to onnx_pred_x
            # Use default atol=1e-5 and rtol=1.6e-2 for bfloat16
            # Due to accuracy issue in pred_v, the topk order may not be the same,
            # therefore, the following assertion sometimes does not pass.
            torch.testing.assert_close(pred_x, trt_pred_x.to(device)), "pred_x and onnx_pred_x are not equal"
            torch.testing.assert_close(pred_v, trt_pred_v.to(device)), "pred_v and onnx_pred_v are not equal"
            torch.testing.assert_close(pred_k, trt_pred_k.to(device)), "pred_k and onnx_pred_k are not equal"
    finally:
        # Deallocate buffers
        for i in range(engine.num_io_tensors):
            binding = engine[i]
            if binding in trt_inputs.keys():
                del trt_inputs[binding]
            else:
                del trt_outputs[binding]
            print(f"Deleted {binding}")


if __name__ == "__main__":
    main()
```

### Multiple inputs
```python
import os
import torch

import onnx
import onnxruntime
import tensorrt as trt
from cuda import cudart

# Reference:
# https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#conditional-examples

@torch.jit.script
def sum_k_mods(items: torch.IntTensor, k: torch.Tensor) -> torch.Tensor:
    s = torch.zeros(1, dtype=torch.float, device=items.device)
    # Must use tensor operations when involves input tensor.
    m = torch.remainder(items, k) == 0
    for i, c in enumerate(items):
        if m[i]:
            s += c
    return s

# Error with the version below:
# Param k is considered as a constant and removed from the input list.
# @torch.jit.script
# def sum_k_mods(items: torch.IntTensor, k: torch.Tensor) -> torch.Tensor:
#     k = k.item()
#     s = torch.zeros(1, dtype=torch.float, device=items.device)
#     for c in items:
#         if c % k == 0:
#             s += c
#     return s

class ExampleModel(torch.nn.Module):
    def __init__(self):
        super().__init__()

    # Only use tensor inputs
    def forward(self, items, k):
        return sum_k_mods(items, k)


def load_model():
    model = ExampleModel()
    return model


def generate_input(batch_size):
    input_x = torch.randint(1, 100, (batch_size,))
    k = torch.randint(2, 5, (1,))
    return input_x, k


def compile_onnx_model(model, inputs, onnx_path, force_export=True):
    example_input = tuple(inputs)
    input_names = ["input_x", "k"]
    output_names = ["pred_x"]

    if force_export or not os.path.exists(onnx_path):
        with torch.no_grad():
            torch.onnx.export(
                model,
                example_input,
                onnx_path,
                input_names=input_names,
                output_names=output_names,
                export_params=True,
                do_constant_folding=True,
                opset_version=18)

            # Check the onnx model
            pred_x = model(*example_input)
            for i, name in enumerate(input_names):
                print(f"Calibration {name}[{inputs[i].shape}]:\n{inputs[i]}")

            # Convert to tuple
            outputs = tuple(pred_x)
            for i, name in enumerate(output_names):
                print(f"Calibration {name}[{outputs[i].shape}]:\n{outputs[i]}")
        print(f"ONNX model is compiled and saved to {onnx_path}")
    else:
        print(f"ONNX model already exists at {onnx_path}")


TRT_LOGGER = trt.Logger(trt.Logger.WARNING)
EXPLICIT_BATCH = 1 << (int)(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)
TRT_DTYPE_TO_TORCH = {
    trt.float32: torch.float32,
    trt.float16: torch.float16,
    trt.int32: torch.int32,
    trt.int64: torch.int64,
    trt.int8: torch.int8,
    trt.bool: torch.bool,
    trt.bfloat16: torch.bfloat16,
}


def compile_trt_model(onnx_path, trt_path=None):
    # Convert onnx model to tensorrt

    # Load the ONNX model
    onnx_model = onnx.load(onnx_path)

    # Create a TensorRT builder and network
    builder = trt.Builder(TRT_LOGGER)
    config = builder.create_builder_config()
    # Set cache
    cache = config.create_timing_cache(b"")
    config.set_timing_cache(cache, ignore_mismatch=False)
    config.set_flag(trt.BuilderFlag.FP16) # BF16
    # config.set_flag(trt.BuilderFlag.STRICT_TYPES)
    config.profiling_verbosity = trt.ProfilingVerbosity.DETAILED
    config.runtime_platform = trt.RuntimePlatform.SAME_AS_BUILD
    # config.set_flag(trt.BuilderFlag.ExportLayerInfo)

    # https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#build_engine_python
    # max_workspace = (1 << 30)
    # config.set_memory_pool_limit(trt.MemoryPoolType.WORKSPACE, max_workspace)

    network = builder.create_network(EXPLICIT_BATCH)

    # Create an ONNX parser and parse the ONNX model into the TensorRT network
    parser = trt.OnnxParser(network, TRT_LOGGER)
    # Parse the ONNX model from file
    # with open(onnx_path, "rb") as f:
    #     if not parser.parse(f.read()):
    #         print(f"ERROR: Failed to parse the ONNX file {onnx_path}")
    #         for error in range(parser.num_errors):
    #             print(parser.get_error(error))
    onnx_model_str = onnx_model.SerializeToString()
    if not parser.parse(onnx_model_str):
        print(f"ERROR: Failed to parse the ONNX file {onnx_model_str}")
        for error in range(parser.num_errors):
            print(parser.get_error(error))

    # Build the TensorRT engine
    engine_bytes = builder.build_serialized_network(network, config)
    if engine_bytes:
        # Save the TensorRT engine to a file
        trt_path = onnx_path[:-4]+ "trt" if trt_path is None else trt_path
        with open(trt_path, "wb") as f:
            f.write(engine_bytes)

        print("TensorRT engine saved successfully!")
    else:
        print("ERROR: Failed to build the TensorRT engine!")


def verify_onnx_model(model, batch_size, onnx_path, dtype, device="cuda"):
    # Verify the ONNX model and TensorRT engine
    # Load the ONNX model
    onnx_model = onnx.load(onnx_path)
    # Create an inference session with the ONNX model
    session = onnxruntime.InferenceSession(onnx_model.SerializeToString())

    for i in range(2):
        input_x, k = generate_input(batch_size)
        inputs = (input_x.to(dtype).to(device), k.to(dtype).to(device))
        with torch.no_grad():
            pred_x = model(*inputs)

        print(f"input_x[{inputs[0].shape}]:\n{inputs}")
        print(f"pred_x[{pred_x.shape}]:\n{pred_x}")

        # Prepare the input data
        input_data = {"input_x": inputs[0].cpu().numpy(), "k": inputs[1].cpu().numpy()}
        # Run the model
        output = session.run(None, input_data)
        # Get the output tensor
        onnx_pred_x = torch.from_numpy(output[0]).to(pred_x.device)
        print(f"onnx run pred_x[{onnx_pred_x.shape}]:\n{onnx_pred_x}")

        # Verify if pred_x is equal to onnx_pred_x
        assert torch.allclose(pred_x, onnx_pred_x, atol=1e-3), "pred_x and onnx_pred_x are not equal"
    print("Onnx model checks pass!")


def verify_trt_model(model, batch_size, trt_path, dtype, device="cuda", dynamic_shape=False):
    # Verify TensorRT engine
    # Load the TRT model

    profile_idx = 0 if dynamic_shape else None
    runtime = trt.Runtime(TRT_LOGGER)

    with open(trt_path, "rb") as f:
        engine = runtime.deserialize_cuda_engine(f.read())
        if engine is None:
            raise RuntimeError(f"Failed to reload TRT cuda engine from {trt_path}.")
        else:
            print(f"Finished loading TRT engine from {trt_path}")

    trt_stream = cudart.cudaStreamCreate()[1]
    trt_context = engine.create_execution_context()
    trt_inputs: dict[str, torch.Tensor] = {}
    trt_outputs: dict[str, torch.Tensor] = {}

    # Allocate buffers for inputs and outputs
    default_max_dim_size = 50
    with torch.cuda.device(device="cuda:0"):
        # current_device = torch.cuda.current_device()
        for i in range(engine.num_io_tensors):
            tensor_name = engine.get_tensor_name(i)

            trt_shape = engine.get_tensor_shape(tensor_name)
            if torch.any(torch.tensor(trt_shape) == -1):
                # Must define all dynamic shapes in the profile
                trt_shape_profile = engine.get_tensor_profile_shape(tensor_name, profile_idx)[-1]
                replace_negative_dims = False
                try:
                    if len(trt_shape_profile) == len(trt_shape):
                        trt_shape = trt_shape_profile
                    else:
                        replace_negative_dims = True
                except Exception as e:
                    replace_negative_dims = True

                if replace_negative_dims:
                    # breakpoint()
                    print(f"Profile shape {trt_shape_profile} does not match the engine shape {trt_shape}")
                    trt_shape = tuple([default_max_dim_size if dim == -1 else dim for dim in trt_shape])
                    print(f"Warnning: Using default max shape {trt_shape}")

            trt_tensor = torch.empty(tuple(trt_shape),
                                    dtype=TRT_DTYPE_TO_TORCH[engine.get_tensor_dtype(tensor_name)],
                                    device="cuda:0")

            if engine.get_tensor_mode(tensor_name) == trt.TensorIOMode.INPUT:
                trt_inputs[tensor_name] = trt_tensor
            else:
                trt_outputs[tensor_name] = trt_tensor
            trt_context.set_tensor_address(tensor_name, trt_tensor.data_ptr())

    try:
        for i in range(2):
            input_x, k = generate_input(batch_size)
            inputs = (input_x.to(dtype).to(device), k.to(dtype).to(device))
            output_shapes = [torch.Size([1])]

            with torch.no_grad():
                pred_x = model(*inputs)

            print(f"input_x[{inputs[0].shape}]:\n{inputs}")
            print(f"pred_x[{pred_x.shape}]:\n{pred_x}")

            # Run TRT model
            input_data = {"input_x": inputs[0], "k": inputs[1]}
            with torch.cuda.device(device="cuda:0"):
                for key, value in input_data.items():
                    # Copy value to the input buffer
                    trt_inputs[key].copy_(value)
                    trt_context.set_input_shape(key, tuple(value.shape))
                trt_context.execute_async_v3(trt_stream)

            trt_pred_x_ = trt_outputs["pred_x"]
            # Flatten the tensors
            trt_pred_x = trt_pred_x_.flatten()[:output_shapes[0].numel()].view(output_shapes[0])
            print(f"trt run pred_x[{trt_pred_x.shape}]:\n{trt_pred_x}")

            # Verify if pred_x is equal to onnx_pred_x
            # Use default atol=1e-5 and rtol=1.6e-2 for bfloat16
            # Due to accuracy issue in pred_v, the topk order may not be the same,
            # therefore, the following assertion sometimes does not pass.
            torch.testing.assert_close(pred_x, trt_pred_x.to(pred_x.device)), "pred_x and onnx_pred_x are not equal"
    finally:
        # Deallocate buffers
        for i in range(engine.num_io_tensors):
            binding = engine[i]
            if binding in trt_inputs.keys():
                del trt_inputs[binding]
            else:
                del trt_outputs[binding]
            print(f"Deleted {binding}")


def main():
    model = load_model()

    # Avoid accuracy issues with float16 betweeen CPU and GPU, use float32 to test onnx
    dtype = torch.float32 # torch.bfloat16 # torch.float32 # torch.float16
    device = "cuda"
    model = model.to(dtype).to(device).eval()

    batch_size = 4
    input_x, k = generate_input(batch_size)
    inputs = (input_x.to(dtype).to(device), k.to(dtype).to(device))

    onnx_path="condition_k_mods.onnx"
    compile_onnx_model(model, inputs, onnx_path=onnx_path)
    trt_path = onnx_path[:-4]+ "trt"
    compile_trt_model(onnx_path=onnx_path, trt_path=trt_path)

    # Onnx doesn't work with torch.bfloat16
    if dtype == torch.float32:
        verify_onnx_model(model, batch_size, onnx_path, dtype, device)

    verify_trt_model(model, batch_size, trt_path, dtype, device, dynamic_shape=True)

    print("All checks pass!")


if __name__ == "__main__":
    main()
```
## Reference

<a href="https://github.com/NVIDIA/TensorRT/blob/84dd6ed18c2ec7a2d8a199ec1a437d09284d86bd/demo/Diffusion/utilities.py#L303">TensorRT diffusion demo code.</a>

<a href="https://github.com/microsoft/onnxruntime/blob/cc0de0d5262cd1a76532b79ddde4a7d799f840b4/onnxruntime/core/providers/tensorrt/tensorrt_execution_provider.h#L121">Flexible output allocation of dynamic shapes through subclassing the allocator.</a>

<a href="https://github.com/NVIDIA/TensorRT-LLM">TensorRT-LLM github.</a>

