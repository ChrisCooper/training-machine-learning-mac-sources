# training-machine-learning-mac-sources
This is a repository to collect links/knowledge on the state of training on modern Macs, since the information is currently spread widely around the web, and it's difficult to get code running. The focus is currently on getting higher-level stable diffusion training setups working, but a lot of the information is general enough to be useful outside that.

# Orientation

### MPS
Most ML projects out there at the moment rely on PyTorch, which originally only supported Nvidia GPUs via CUDA, but in late 2022 an additional backend was introduced called MPS which supports running on M-series Mac GPUs. (https://pytorch.org/docs/stable/notes/mps.html). This backend supports a lot of operations, but some remain unsupported. [MPS](https://developer.apple.com/documentation/metalperformanceshaders) is essentially CUDA for Mac GPUs. When this document refers to "MPS", it generally means the `mps` backend for PyTorch.

### MLX
[MLX](https://github.com/ml-explore/mlx) is a separate framework from PyTorch that leverages the GPU on Macs as well. Most projects use PyTorch with MPS currently rather than MPX, but you can find some MLX examples [here](https://github.com/ml-explore/mlx-examples).

### Architecture
M-series Macs use the `arm64` architecture.

# Useful system libraries

You should probably install these for supporting various utilities related to training. This guide assumes you have installed [Homebrew](https://brew.sh/) for managing packages. If you don't have it or don't want to use it, you can install these dependencies manually. Consult licenses to undestand how you can use these.

### LLVM
- Contains a buch of components that suport compilers
- You might need this if packages are failing to build with `clang: error: unsupported option`
- Run `brew install llvm`
- Then look at the path it installs to and add the following (assuming it installed to `/opt/homebrew/opt/llvm`) to `~/.zshenv`
```
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
export CC="/opt/homebrew/opt/llvm/bin/clang"
export CXX="/opt/homebrew/opt/llvm/bin/clang++"
export LDFLAGS="-L/opt/homebrew/opt/llvm/lib"
export CPPFLAGS="-I/opt/homebrew/opt/llvm/include"
```

If these conflict with your existing projects, you can instead add these lines to a script you can run with `source llvm_env.sh` to enable for particular projects when needed.

### OpenMP
- Supports running shared-memory multi-core operations
- Run `brew install libomp`
- Add the following (see LLVM above for notes) to `~/.zshenv`:
```
export LDFLAGS="-L/opt/homebrew/opt/libomp/lib"
export CPPFLAGS="-I/opt/homebrew/opt/libomp/include"
```

### Image libraries
- These are used by `torchvision` and other image utilities like `Pillow`
- Run `brew install jpeg libpng`

### FFmpeg
- Supports operations on video and audio files and streams
- Run `brew install ffmpeg`

# General tips

### pyenv
This is subjective and not Mac-specific, but [pyenv](https://github.com/pyenv/pyenv) can help a lot with managing/experimenting with different versions of python. E.g. if you are working in a directory with a project that requires Python 3.12, simply run `pyenv local 3.12` and you can immediately use that version for that project only.

### Increasing available VRAM
You can increase the limit of memory that can be used for training/inference with the following:
```
sudo sysctl iogpu.wired_limit_mb=XXXX
```
XXXX should be a value in MB, generally your total RAM minus a safe buffer for running the OS etc. E.g. if you have a 64GB Mac, `64GB-8GB*1025MB/GB = 57344MB`, so:
```
sudo sysctl iogpu.wired_limit_mb=57344
```
Be careful setting this value too high as it could cause system instability if you push it.  More info [here](https://www.reddit.com/r/LocalLLaMA/comments/186phti/m1m2m3_increase_vram_allocation_with_sudo_sysctl/).

### Disable `xformers`?
- You can definitely install the `xformers` package, but I've seen conflicting information on whether Mac is supported/working or not. Might be worth disabling it where possible (e.g. `use_memory_efficient_attention` to false in `AUTOMATIC1111`, cross_attention to `none` in `kohya_ss`, etc.)

### Floating Point support
- Often to save on memory, it's useful to use smaller sized floating-point numbers in training/inference. Currently, the `mps` backend has limited support for other sizes of floating point, and might either not work or convert to `fp32`, negating savings.
- `fp64`/`float64` -  Not currently support in MPS ([relevant issue on another repo](https://github.com/huggingface/transformers/issues/28334))
- `fp32`/`float32` - Supported, and the default in MPS
- `fp16`/`float16` - Looks like not supported? ([issue](https://github.com/pytorch/pytorch/issues/96113))
- `fp8`/`float8` - Not supported ([GitHub issue](https://github.com/pytorch/pytorch/issues/132624))



# Information sources/threads
- [Metal FlashAttention](https://github.com/philipturner/metal-flash-attention)
- [Installing OpenMP on OSX](https://gist.github.com/ijleesw/4f863543a50294e3ba54acf588a4a421)
- [Getting xformers working on M-series macs](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/8188)