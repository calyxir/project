# Setting Up Allo/CIRCT/MLIR Flow from Scratch

These are in-progress *self-contained* instructions for setting up the whole Allo-to-CIRCT-to-Calyx flow on any machine.

## set up Allo

We'll be following [the Allo installation instructions][allo-inst], with some tweaks.

[allo-inst]: https://cornell-zhang.github.io/allo/setup/index.html#install-from-source

### get Allo

```
git clone --recursive https://github.com/cornell-zhang/allo.git
cd allo
```

### get a suitable Python

Deviating a bit from the official instructions, we're going to use [uv][] to install a recent Python and work in a virtualenv. (The official docs use Conda instead, which I would like to avoid.) Allo currently requires Python 3.12.

```
uv venv --python 3.12
. .venv/bin/activate
```

We activate the virtualenv here because we'll need to use the right Python in the next step.

[uv]: https://docs.astral.sh/uv/

### build Allo-patched LLVM

We are going to need pybind11 for this build. Let's make sure we have one that matches our virtualenv's Python version:

```
uv pip install pybind11
```

Now, follow a slight variation on the official instructions to build LLVM:

```
cd allo/externals/llvm-project
git apply ../llvm_patch
mkdir build
cmake -G Ninja ../llvm \
    -DLLVM_ENABLE_PROJECTS="clang;mlir;openmp" \
    -DLLVM_BUILD_EXAMPLES=ON \
    -DLLVM_TARGETS_TO_BUILD="host" \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DLLVM_INSTALL_UTILS=ON \
    -DMLIR_ENABLE_BINDINGS_PYTHON=ON
ninja
```

This seemed to pick up the right libpython/headers/etc. because the virtualenv was activated.

### build Allo

Let's go back to the root of the Allo repository:

```
cd ../../..
```

We will need to fix one thing in Allo to make it build: we'll delete the dependency on `past`. Edit `requirements.txt` and delete the `past @ http://...`. This seems to be an optional dependency necessary only for the verifier, and it is provided only as a platform-specific wheel that can't be installed everywhere.

Finally, let's build the project itself:

```
LLVM_BUILD_DIR=`pwd`/externals/llvm-project/build uv pip install -e .
```

With all that in place, we can finally:

```
python
>>> import allo
```

## use Allo to generate MLIR code

### apply a load-bearing patch to Allo's PyTorch frontend

To make this work, there is one change we will need to the PyTorch frontend.
Put this in a file called `torch.patch`:

```
diff --git a/allo/frontend/pytorch.py b/allo/frontend/pytorch.py
index e5e2e55..6c1cbde 100644
--- a/allo/frontend/pytorch.py
+++ b/allo/frontend/pytorch.py
@@ -67,6 +67,8 @@ def from_pytorch(
     s = customize(
         code, verbose=verbose, global_vars=global_vars, enable_tensor=enable_ten
sor
     )
+    if target == "calyx":
+        return s.module
     mod = s.build(target=target, mode=mode, project=project)
     if verbose:
         print(s.module)
```

And run `git patch torch.patch` to apply. Or just made the edit manually yourself.

TODO: Can we upstream this change, or something like it?

### get Jiahan's FFNN generator script

Put this in a file called `lower_ffnn.py`:

```
import torch
import torch.nn as nn
import torch.nn.functional as F
import json

import allo

torch.set_printoptions(precision=4)

input_size = 64
hidden_size = 48
output_size = 4

mean = 3
std = 5

lower_bound = 2
upper_bound = 6

torch.manual_seed(42)

random_tensor = torch.randn(hidden_size, input_size) * std + mean

clipped_tensor = torch.clamp(random_tensor, min=lower_bound, max=upper_bound)


class Model(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super(Model, self).__init__()
        self.l1 = nn.Linear(input_size, hidden_size)
        self.activation = F.relu
        self.l2 = nn.Linear(hidden_size, output_size)

    def forward(self, x):
        output = self.l1(x)
        output = self.activation(output)
        output = self.l2(output)
        return output

model = Model(input_size, hidden_size, output_size)
model.eval()
example_inputs = [clipped_tensor]
mlir_mod = allo.frontend.from_pytorch(
    model, example_inputs=example_inputs, verbose=False, target="calyx",
)
print(mlir_mod)

# print(example_inputs[0])
# print(model.l1.weight.data)
# print(model.l1.bias.data)
# print(model.l2.weight.data)
# print(model.l2.bias.data)
# expected_output = model(example_inputs[0])
# print("Expected Output:", expected_output)
```

TODO: Let's put this into version control somewhere... a new repository? A directory in the Calyx monorepo? Not sure.

### finally, generate some MLIR

Now would be a good time to make sure your virtualenv is still activated.
Then:

```
uv pip install torch
python lower_ffnn.py > ffnn.mlir
```

## compile to Calyx?

### set up CIRCT

The rest of the flow (going from the generated MLIR code to Calyx) requires [building the CIRCT tools][circt-build].
Yes, this does in fact mean that we're going to end up with a separate LLVM source tree;
this appears to be necessary because we (unlike Allo) need the special CIRCT version of MLIR.

TODO: Let's check if, after building this CIRCT and MLIR, we can actually reuse that for Allo? Seems dicey...

Let's follow [the CIRCT setup instructions][circt-build] to check this out next to the Allo repository:

```
cd ..
git clone git@github.com:circt/circt.git
cd circt
git submodule init
git submodule update
```

Next, we have to build LLVM:

```
cd llvm
mkdir build
cd build
cmake -G Ninja ../llvm \
    -DLLVM_ENABLE_PROJECTS="mlir" \
    -DLLVM_TARGETS_TO_BUILD="host" \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DCMAKE_BUILD_TYPE=DEBUG \
    -DLLVM_USE_SPLIT_DWARF=ON
ninja
```

Finally, we can build CIRCT (not sure why this doesn't all happen in one CMake call, TBH):

```
cd ../..
mkdir build
cd build
cmake -G Ninja .. \
    -DMLIR_DIR=$PWD/../llvm/build/lib/cmake/mlir \
    -DLLVM_DIR=$PWD/../llvm/build/lib/cmake/llvm \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DCMAKE_BUILD_TYPE=DEBUG \
    -DLLVM_USE_SPLIT_DWARF=ON
ninja
```

[circt-build]: https://circt.llvm.org/docs/GettingStarted/

### run compilation pipeline

The next step is to use `mlir-opt`, `circt-opt`, and `circt-translate` to run the compilation pipeline to produce Calyx code.
We'll go back to where we have our MLIR code sitting:

```
cd ../../allo
```

Now, the goal is to run this huge pipeline:

```
../circt/llvm/build/bin/mlir-opt ffnn.mlir \
    --empty-tensor-to-alloc-tensor \
    --one-shot-bufferize="allow-return-allocs-from-loops bufferize-function-boundaries" \
    -drop-equivalent-buffer-results \
    --buffer-results-to-out-params="hoist-static-allocs" \
    --convert-linalg-to-loops \
    -test-math-polynomial-approximation \
    -test-expand-math \
    --arith-expand \
    --lower-affine \
    --canonicalize | \
../circt/build/bin/circt-opt \
    --flatten-memref \
    --canonicalize | \
../circt/build/bin/circt-opt \
    --lower-scf-to-calyx="top-level-function=forward write-json=data" \
    --canonicalize | \
../circt/build/bin/circt-translate --export-calyx
```

Sadly, while the first `mlir-opt` stage works fine, the next one (the first `circt-opt` invocation) crashes with a segfault for me.
It seems like we may need a different version of CIRCT?
The quest continues...
