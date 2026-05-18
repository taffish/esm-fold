taf-esm-fold 2.0.0-r2

TAFFISH wrapper for ESMFold, the ESM protein language model interface for
predicting protein structures from FASTA sequences.

Usage:
  taf-esm-fold [TAF-APP-OPTION]
  taf-esm-fold -- [ESM-FOLD-OPTION...]
  taf-esm-fold esm-fold [ESM-FOLD-ARGS...]
  taf-esm-fold <COMMAND> [COMMAND-ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Upstream help:
  taf-esm-fold esm-fold -h
  taf-esm-fold -- -h

Recommended examples:
  taf-esm-fold esm-fold -i proteins.fa -o pdb-out
  taf-esm-fold esm-fold -i proteins.fa -o pdb-out --chunk-size 128
  taf-esm-fold esm-fold -i proteins.fa -o pdb-out --max-tokens-per-batch 512
  taf-esm-fold -- -i proteins.fa -o pdb-out --chunk-size 128

Common upstream options:
  -i, --fasta                 Input protein FASTA file
  -o, --pdb                   Output directory for predicted PDB files
  --num-recycles              Number of recycling iterations
  --max-tokens-per-batch      Maximum tokens per GPU forward pass
  --chunk-size                Reduce axial attention memory use
  --cpu-only                  Upstream CPU-only mode
  --cpu-offload               Upstream GPU/CPU offload mode

Notes:
  - This command runs ESMFold inside the TAFFISH container image.
  - The default command is "esm-fold", so option-leading calls can use
    "taf-esm-fold -- -i proteins.fa -o pdb-out".
  - For command-mode use, name the in-container command explicitly:
    "taf-esm-fold esm-fold -i proteins.fa -o pdb-out".
  - taf-esm-fold --help and taf-esm-fold --version are handled by the TAFFISH
    wrapper. Run "taf-esm-fold esm-fold -h" to see upstream CLI help.
  - GPU acceleration is strongly recommended for real predictions.
  - TAFFISH_ESM_FOLD_GPU controls app-managed GPU runtime flags:
      auto  default; add GPU flags only on likely Linux NVIDIA hosts
      1     force GPU flags
      0     disable app-managed GPU flags
  - App-managed GPU flags are:
      Docker:    --gpus all
      Podman:    --device nvidia.com/gpu=all
      Apptainer: --nv
  - Example GPU force:
      TAFFISH_ESM_FOLD_GPU=1 taf-esm-fold esm-fold -i proteins.fa -o pdb-out
  - Example CPU/container-no-GPU run:
      TAFFISH_ESM_FOLD_GPU=0 taf-esm-fold esm-fold --cpu-only -i proteins.fa -o pdb-out
  - If auto adds GPU flags before the backend is configured for NVIDIA access,
    configure the backend or set TAFFISH_ESM_FOLD_GPU=0.
  - This image patches upstream --cpu-only so the model is converted to float32
    before CPU inference. CPU-only remains slow and memory-heavy.
  - TAFFISH_DOCKER_RUN_ARGS, TAFFISH_PODMAN_RUN_ARGS, and
    TAFFISH_APPTAINER_RUN_ARGS remain available for local site/runtime policy.
  - Apple Silicon Docker does not expose the Apple GPU to this Linux CUDA
    container. PyTorch MPS is a macOS-native backend, not a Docker Linux
    container GPU passthrough mechanism.
  - Docker and Podman runs embed --platform linux/amd64 in src/main.taf. On
    Apple Silicon macOS, use Docker emulation and disable app-managed GPU
    flags:
      TAFFISH_CONTAINER_BACKEND=docker \
      TAFFISH_ESM_FOLD_GPU=0 \
      taf-esm-fold esm-fold -h
    For CPU-only prediction, add upstream --cpu-only and expect slow execution:
      TAFFISH_CONTAINER_BACKEND=docker \
      TAFFISH_ESM_FOLD_GPU=0 \
      taf-esm-fold esm-fold --cpu-only -i proteins.fa -o pdb-out
  - CPU-only mode can be attempted with "taf-esm-fold esm-fold --cpu-only ...",
    but even tiny predictions still load/download the full ESMFold model.
  - Model weights are downloaded on first real prediction into TORCH_HOME.
    ESMFold needs the ESMFold checkpoint plus the underlying ESM2 3B weights;
    local validation observed about 8 GB of first-run Torch cache.
    The wrapper defaults TORCH_HOME to:
      $HOME/.cache/taffish/esm-fold/torch
    This path is under the TAFFISH-mounted host HOME and is reused across runs.
  - Set TORCH_HOME=/path/to/cache before running when using cluster scratch or
    another persistent cache location.
  - This image does not bundle AlphaFold, ColabFold, template databases, MSA
    databases, or visualization tools.
  - Input and output paths should be accessible from the current working
    directory or from mounted user paths.
  - Native platform support is linux/amd64 only.

Container:
  image: ghcr.io/taffish/esm-fold:2.0.0-r2
  supported backends: apptainer, podman, docker
  supported platform: linux/amd64
  runtime: NVIDIA GPU auto-detected on Linux; CPU-only is upstream-supported

Upstream:
  project: ESM / ESMFold
  homepage: https://github.com/facebookresearch/esm
  release:  https://github.com/facebookresearch/esm/releases/tag/v2.0.0
  PyPI:     https://pypi.org/project/fair-esm/2.0.0/
  license:  MIT
  citation: Lin et al. 2023, doi:10.1126/science.ade2574, PMID:36927031
