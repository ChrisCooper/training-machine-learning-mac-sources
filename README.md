# training-ML-mac-sources
This is a repository to collect links/knowledge on the state of training on modern Macs, since the information is currently spread widely around the web, and it's difficult to get code running.

# Orientation

Most ML projects out there at the moment rely on PyTorch, which originally only supported NVidia GPUs via CUDA, but in late 2022 an additional backend was introduced called MPS which supports running on M-series Mac GPUs. (https://pytorch.org/docs/stable/notes/mps.html). This backend supports a lot of operations, but some remain unsupported.

# Useful system libraries

You should probably install these for supporting various utilities related to training. This guide assumes you have installed [Homebrew](https://brew.sh/) for managing packages. If you don't have it or don't want to use it, you can install these dependencies manually.

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

### Image libraries
- These are used by `torchvision` and other image utilities like `Pillow`
- Run `brew install jpeg libpng`

### OpenMP
- Supports running shared-memory multi-core operations
- Run `brew install libomp`
- Add the following (see LLVM above for notes) to `~/.zshenv`:
```
export LDFLAGS="-L/opt/homebrew/opt/libomp/lib"
export CPPFLAGS="-I/opt/homebrew/opt/libomp/include"
```

# General tips

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


# Information sources/threads
- [Metal FlashAttention](https://github.com/philipturner/metal-flash-attention)
- [Installing OpenMP on OSX](https://gist.github.com/ijleesw/4f863543a50294e3ba54acf588a4a421)
- [Getting xformers working on M-series macs](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/8188), 