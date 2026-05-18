# taf-esm-fold

TAFFISH wrapper for [ESMFold](https://github.com/facebookresearch/esm), the
protein language model structure-prediction interface from the ESM project.

This repository packages `fair-esm` 2.0.0 with the upstream ESMFold command-line
script as a TAFFISH tool app. The published command is
`taf-esm-fold`; the in-container upstream command is `esm-fold`.

## Installation

Install from the public TAFFISH Hub index:

```sh
taf update
taf install esm-fold
```

Install the exact release:

```sh
taf install esm-fold 2.0.0-r2
```

For local testing before the app is published to the public index:

```sh
taf install --from .
```

## Usage

Show TAFFISH app help:

```sh
taf-esm-fold --help
```

Show upstream ESMFold help:

```sh
taf-esm-fold esm-fold -h
taf-esm-fold -- -h
```

Predict structures from a protein FASTA file:

```sh
taf-esm-fold esm-fold -i proteins.fa -o pdb-out
```

Reduce memory use for long proteins:

```sh
taf-esm-fold esm-fold -i proteins.fa -o pdb-out --chunk-size 128
```

Reduce batch size when short sequences still run out of GPU memory:

```sh
taf-esm-fold esm-fold -i proteins.fa -o pdb-out --max-tokens-per-batch 512
```

The default command is `esm-fold`, so option-leading calls can also use the
TAFFISH `--` separator:

```sh
taf-esm-fold -- -i proteins.fa -o pdb-out --chunk-size 128
```

Because this is a command-mode TAFFISH tool, the first non-option argument is
treated as an executable inside the container. For normal ESMFold use, name the
upstream command explicitly:

```sh
taf-esm-fold esm-fold -i proteins.fa -o pdb-out
taf-esm-fold python -c 'import esm; print(esm.pretrained.esmfold_v1)'
```

## GPU And CPU Runtime

ESMFold is much more practical with GPU acceleration, but the upstream CLI also
supports `--cpu-only`. This app therefore uses a conservative automatic GPU
policy rather than always adding GPU flags.

By default, `TAFFISH_ESM_FOLD_GPU=auto`. In auto mode, the wrapper adds the
backend GPU flag only when the host appears to be a Linux NVIDIA environment:

```text
Docker:    --gpus all
Podman:    --device nvidia.com/gpu=all
Apptainer: --nv
```

Force GPU runtime flags when auto-detection is too conservative:

```sh
TAFFISH_ESM_FOLD_GPU=1 taf-esm-fold esm-fold -i proteins.fa -o pdb-out
```

Force CPU/container-no-GPU mode:

```sh
TAFFISH_ESM_FOLD_GPU=0 taf-esm-fold esm-fold --cpu-only -i proteins.fa -o pdb-out
```

Valid values are `auto`, `1`/`true`/`yes`/`on`, and
`0`/`false`/`no`/`off`.

If auto-detection adds GPU flags on a host whose container backend is not yet
configured for NVIDIA access, either configure that backend or set
`TAFFISH_ESM_FOLD_GPU=0` for CPU/container-no-GPU runs.

The image patches the upstream ESMFold CLI so `--cpu-only` converts the model to
float32 before inference. Without this, PyTorch CPU LayerNorm can fail on
half-precision tensors. CPU-only mode is still very slow and memory-heavy; local
validation used a 36 aa test sequence, about 8 GB of cached weights, and a Docker
VM with about 16 GB memory.

`TAFFISH_DOCKER_RUN_ARGS`, `TAFFISH_PODMAN_RUN_ARGS`, and
`TAFFISH_APPTAINER_RUN_ARGS` remain available for local site policy, such as
extra mounts or cluster-specific runtime options. They are appended after the
app-declared runtime arguments.

Even tiny CPU-only predictions still load/download the full ESMFold model, so
they are not used as publish smoke tests.

## Scope

This app exposes the upstream ESMFold FASTA-to-PDB CLI:

```text
esm-fold -i FASTA -o PDB_DIR
```

Common upstream options include:

```text
--num-recycles
--max-tokens-per-batch
--chunk-size
--cpu-only
--cpu-offload
```

The upstream script supports `--cpu-only`. In practice, ESMFold model inference
is large and slow without GPU acceleration, so GPU-backed Docker/Podman/Apptainer
is the recommended route for real predictions.

Model weights are not baked into the image. On first real prediction, PyTorch
downloads the ESMFold checkpoint plus the underlying ESM2 model weights into the
user's Torch cache. In local validation, the first-run cache was about 8 GB
(`esmfold_3B_v1.pt`, `esm2_t36_3B_UR50D.pt`, and a small contact-regression
file). The wrapper sets `TORCH_HOME` to
`${TORCH_HOME:-$HOME/.cache/taffish/esm-fold/torch}` at container runtime, so
the default cache is under the host home directory that TAFFISH mounts into the
container. This keeps the image smaller and lets the same host cache be reused
across runs.

For shared scratch or cluster storage, choose an explicit cache directory:

```sh
TORCH_HOME=/path/to/torch-cache taf-esm-fold esm-fold -i proteins.fa -o pdb-out
```

This app does not bundle AlphaFold, ColabFold, template databases, MSA
databases, or protein visualization tools. ESMFold predicts directly from
single protein sequences and does not require an MSA database for the standard
`esm-fold` path.

## Platform

This release declares native `linux/amd64` only. The container is based on an
NVIDIA CUDA devel image and installs CUDA-enabled PyTorch plus OpenFold.

Apple Silicon and other arm64 hosts are not native targets for this release. For
Docker and Podman, `src/main.taf` declares `--platform linux/amd64`, so amd64
emulation may be useful for inspecting the image and running lightweight
commands. It does not expose the Apple GPU to this Linux CUDA container. PyTorch
MPS is a macOS-native backend, not a Docker Linux container GPU passthrough
mechanism.

On Apple Silicon macOS, use Docker emulation and disable app-managed GPU flags:

```sh
TAFFISH_CONTAINER_BACKEND=docker \
TAFFISH_ESM_FOLD_GPU=0 \
taf-esm-fold esm-fold -h
```

CPU-only prediction can also be attempted through amd64 emulation:

```sh
TAFFISH_CONTAINER_BACKEND=docker \
TAFFISH_ESM_FOLD_GPU=0 \
taf-esm-fold esm-fold --cpu-only -i proteins.fa -o pdb-out
```

This is a compatibility path, not native Apple GPU support. It needs the full
model cache, enough Docker Desktop memory, and patience; local validation used
about 16 GB of Docker VM memory.

## Package

```text
name: esm-fold
command: taf-esm-fold
version: 2.0.0-r2
kind: tool
image: ghcr.io/taffish/esm-fold:2.0.0-r2
```

## Container

The container image is built from `docker/Dockerfile`. It starts from
`nvidia/cuda:11.3.1-devel-ubuntu20.04`, installs Python 3, CUDA-enabled PyTorch
1.12.1, `fair-esm[esmfold]` 2.0.0, NVIDIA `dllogger`, and OpenFold at the
upstream commit recommended by the ESM project.

The Dockerfile also installs the upstream `scripts/esmfold_inference.py` file as
`/usr/local/bin/esm-fold`, because the `fair-esm` wheel provides the Python
package but not a console entry point. The installed script is patched for
CPU-only float32 inference inside this container.

The TAFFISH metadata declares a Docker smoke check:

```text
exist: esm-fold, python, python3, pip, nvcc
test:  upstream CLI help is available
test:  fair-esm is pinned to 2.0.0
test:  PyTorch imports with the pinned 1.12.1 runtime
test:  esm, openfold, deepspeed, dllogger, and dm-tree import
test:  the CPU-only float32 patch is present
test:  esm.data.read_fasta parses a tiny FASTA file
```

Smoke does not run a full prediction, because that would download large model
weights and require a real GPU during index validation.

Manual local validation also completed a CPU-only prediction for a 36 aa FASTA
sequence and produced a 300-line PDB file after the model cache was prepared.

## Upstream

- Project: ESM / ESMFold
- Homepage: <https://github.com/facebookresearch/esm>
- Release: <https://github.com/facebookresearch/esm/releases/tag/v2.0.0>
- PyPI: <https://pypi.org/project/fair-esm/2.0.0/>
- License: MIT
- Citation: Lin et al. 2023, doi:10.1126/science.ade2574, PMID:36927031
