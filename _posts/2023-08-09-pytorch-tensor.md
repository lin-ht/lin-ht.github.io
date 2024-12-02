---
layout: distill
title: Pytorch coding
description: practice
giscus_comments: false
date: 2023-08-09 16:50:00

toc:
  - name: Tensor Indexing
---

## Tensor Indexing

### Indexing by tensor
<a href="https://pytorch.org/docs/stable/generated/torch.unravel_index.html#torch.unravel_index">torch.unravel_index</a>: Converts a tensor of flat indices into a tuple of coordinate tensors that index into an arbitrary tensor of the specified shape.
and 
<a href="https://pytorch.org/docs/stable/generated/torch.ravel.html#torch.ravel">torch.ravel</a>: Return a contiguous flattened tensor. A copy is made only if needed.

Example of indexing along an axis with index list:
```python
a = torch.randn([4, 5])
# tensor([[-0.1632, -0.6520,  0.0114, -0.7321, -0.1581],
#         [ 0.4762, -1.0471,  0.3505, -1.2285,  0.6787],
#         [-0.3809,  0.5646, -0.5348, -0.6608, -1.7215],
#         [-0.2506, -0.1802, -2.5219,  0.3056, -0.8323]])
y_idx_list = [1, 3]
x_idx_list = [2, 4]
y_idx = torch.IntTensor(y_idx_list)
x_idx = torch.IntTensor(x_idx_list)
a[y_idx, x_idx]
# tensor([ 0.3505, -0.8323])

# Row selection:
a[y_idx, :]
# tensor([[ 0.4762, -1.0471,  0.3505, -1.2285,  0.6787],
#         [-0.2506, -0.1802, -2.5219,  0.3056, -0.8323]])


slice_obj = slice(1, 3)
a[y_idx, slice_obj] # i.e. a[y_idx, 1:3]
# tensor([[-1.0471,  0.3505],
#         [-0.1802, -2.5219]])


# x_idx * y_idx
x_size = len(x_idx)
y_size = len(y_idx)
y_idx_repeat=y_idx.view(-1, 1).repeat(1, x_size)
x_idx_repeat=x_idx.view(1, -1).repeat(y_size, 1)
a[y_idx_repeat, x_idx_repeat]
# tensor([[ 0.3505,  0.6787],
#         [-2.5219, -0.8323]])


# Convert from flattened index to index tuple for each axis:
torch.unravel_index(torch.tensor([1234, 5678]), (10, 10, 10, 10))
# (tensor([1, 5]),
#  tensor([2, 6]),
#  tensor([3, 7]),
#  tensor([4, 8]))


# Convert index tuple to flattened index:
idx_tuple = (torch.IntTensor([1, 5]), torch.IntTensor([2, 6]), torch.IntTensor([3, 7]), torch.IntTensor([4, 8]))

strides = (1000, 100, 10, 1)
idx = torch.IntTensor([0, 0])
for stride_dim,idx_dim in zip(strides, idx_tuple):
    idx += idx_dim * stride_dim
idx
# tensor([1234, 5678], dtype=torch.int32)
```

### torch.gather vs torch.index_select
<a href="https://stackoverflow.com/questions/76455828/what-does-the-torch-gather-and-torch-index-select-do">Stackoverflow: What does the torch.gather and torch.index_select do?</a>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/gather_vs_index_select.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<div class="caption">
    Figure. Torch gather vs index_select.
</div>

### Slicing
PyTorch document: <a href="https://pytorch.org/docs/stable/torch.html#indexing-slicing-joining">SlicIndexing, Slicing, Joining, Mutating Ops</a>.

```python
data = torch.arange(10)
starts = torch.IntTensor([0, 3, 4, 1])
ends = torch.IntTensor([2, 5, 6, 3])
newtensor = torch.stack([data[slice(idx[0], idx[1])] for idx in zip(starts, ends)])
# tensor([[0, 1],
#         [3, 4],
#         [4, 5],
#         [1, 2]])

# The above will create a new tensor, which is different from the original data.
# Convert slice into index to enable writing back.
updates = newtensor * 10
idx_list = []
for s, e in zip(starts, ends):
    idx_list.append(torch.arange(s, e))
idx = torch.cat(idx_list, dim=0)
idx
# tensor([0, 1, 3, 4, 4, 5, 1, 2], dtype=torch.int32)

# Use ravel instead of flatten.
# flatten always returns a copy, while ravel returns a contiguous view of the original array whenever possible.
data[idx] = updates.ravel() 
data
# tensor([ 0, 10, 20, 30, 40, 50,  6,  7,  8,  9])
```

## References

<a href="https://pytorch.org/tutorials/advanced/cpp_extension.html">CUSTOM C++ AND CUDA EXTENSIONS</a>
