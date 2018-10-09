![NimTorch](media/NimTorchBanner.png)

[![Join the chat at https://gitter.im/nimtorch/Lobby](https://badges.gitter.im/nimtorch/Lobby.svg)](https://gitter.im/nimtorch/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Pytorch - Py + Nim

A Nim frontend for pytorch, aiming to be mostly auto-generated and internally using ATen.

**Because Nim compiles to C++, this is not a wrapper or binding library. It generates 1-to-1 native ATen code.**

The only requirement from pytorch is ATen's core tensor library. Because of this, nimtorch is extremely versatile and can compile on any kind of device.

## Current status

**Early stage**

- [x] Automatically generated, from `Declarations.yaml`, the full ATen API
- [x] Cuda support ( add -d:cuda when compiling with nim )
- [x] WASM support ( add -d:wasm when compiling with nim )
- [x] Automatically generated, from `derivatives.yaml`, gradient procs
- [x] Autograd
- [ ] Add missing derivatives
- [ ] More high level pytorch API (Module, Models etc)
- [ ] ...

The final aim is to be as compatible as possible with the pytorch API.

## Why

Ease of use of the python language while keeping fully bare metal native C++ performance

### Python code

```python
# GRUCell
gi = x.matmul(w_input.t()) + b_input
gh = hidden.matmul(w_recur.t()) + b_recur
i_r, i_i, i_n = gi.chunk(3, 1)
h_r, h_i, h_n = gh.chunk(3, 1)
resetgate = (i_r + h_r).sigmoid()
inputgate = torch.sigmoid(i_i + h_i)
newgate = (i_n + resetgate * h_n).tanh()
hy = newgate + inputgate * (hidden - newgate)
```

### Nim code

```nimrod
# GRUCell
let
  gi = x.matmul(w_input.t()) + b_input
  gh = hidden.matmul(w_recur.t()) + b_recur
  (i_r, i_i, i_nn) = gi.chunk(3, 1)
  (h_r, h_i, h_n)  = gh.chunk(3, 1)
  resetgate = (i_r + h_r).sigmoid()
  inputgate = torch.sigmoid(i_i + h_i)
  newgate = (i_nn + resetgate * h_n).tanh()
  hy = newgate + inputgate * (hidden - newgate)
```

Also python is a complete mess...

![python mess](https://camo.githubusercontent.com/953249a42e8fe655a8c1fdfe80744a42b4d25723/68747470733a2f2f696d67732e786b63642e636f6d2f636f6d6963732f707974686f6e5f656e7669726f6e6d656e742e706e67)

## Getting started

### Super easy, using conda

`conda create -n nimtorch -c fragcolor nimtorch`

`source activate nimtorch`

This will install: nim and ATen binaries, fragments and nimtorch all in one command, nothing else needed.

Make sure you use a recent version of conda and have gcc installed in your system.

Also make sure your system is recent (ubuntu 18.04 reference) and you have cuda 9.2 at least installed (if you need cuda).

Test with with something like:

`nim cpp $ATEN/dist/nimtorch/tests/test_xor.nim`

### Semi manual way

Check what version of ATen/PyTorch we need in
`conda/nimtorch/meta.yaml` - should be something like `aten ==2018.10.09`

Note the version as you will need it in the next step

**Linux or OSX**

`conda create -n aten -c fragcolor aten==<version>`

or

**WASM on a Linux system**

`conda create -n aten -c fragcolor aten_wasm==<version>`

activate aten environment

`source activate aten`

1. Make sure you have a recent Nim version in your path
    * [Easy option: install nim with choosenim](https://github.com/dom96/choosenim)
      * `choosenim devel`
2. clone this repo
3. `cd nimtorch`
4. `nimble develop`

*finally*

(optional) run self test `nim cpp -r torch.nim`

### Windows support coming soon (already builds, just needs to be automated to publish with conda)

## Know issues

* The commit hash we are using for ATen has some bugs with openmp, if you run in high CPU load issues even if your task is quite easy, try reduce openmp CPU count with this environment variable:
`export OMP_NUM_THREADS=1`