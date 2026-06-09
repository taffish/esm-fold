taf-esm-fold 2.0.0-r4

TAFFISH wrapper for ESMFold, the ESM protein language model interface for
predicting protein structures from FASTA sequences.

Usage:
  taf-esm-fold [TAF-APP-OPTION]
  taf-esm-fold -- [ESM-FOLD-OPTION...]
  taf-esm-fold esm-fold [ESM-FOLD-ARGS...]
  taf-esm-fold COMMAND [COMMAND-ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Help and examples:
  taf-esm-fold esm-fold -h
  taf-esm-fold -- -h
  taf-esm-fold esm-fold-download-models --dry-run
  taf-esm-fold esm-fold-download-models
  taf-esm-fold esm-fold-check-models
  taf-esm-fold esm-fold -i proteins.fa -o pdb-out
  taf-esm-fold esm-fold -i proteins.fa -o pdb-out --chunk-size 128
  taf-esm-fold esm-fold -i proteins.fa -o pdb-out --max-tokens-per-batch 512
  taf-esm-fold -- -i proteins.fa -o pdb-out --chunk-size 128

Common upstream options:
  -i, --fasta                 Input protein FASTA file
  -o, --pdb                   Output directory for predicted PDB files
  --num-recycles              Recycling iterations
  --max-tokens-per-batch      Maximum tokens per batch
  --chunk-size                Reduce axial attention memory use
  --cpu-only                  Upstream CPU-only mode
  --cpu-offload               Upstream GPU/CPU offload mode

GPU and runtime flags:
  GPU acceleration is strongly recommended for real predictions.
  TAFFISH_ESM_FOLD_GPU controls app-managed GPU runtime flags:
    auto  default; add GPU flags only on likely Linux NVIDIA hosts
    1     force GPU flags
    0     disable app-managed GPU flags

  Docker uses --gpus all, Podman uses --device nvidia.com/gpu=all, and
  Apptainer uses --nv when GPU flags are enabled.

  Force GPU flags:
    TAFFISH_ESM_FOLD_GPU=1 taf-esm-fold esm-fold -i proteins.fa -o pdb-out

  Disable GPU flags:
    TAFFISH_ESM_FOLD_GPU=0 taf-esm-fold esm-fold --cpu-only -i proteins.fa -o pdb-out

Model cache:
  Model weights are not baked into the image. Real predictions require the
  ESMFold and ESM2 checkpoint files in TORCH_HOME/hub/checkpoints.

  Prepare the default cache:
    taf-esm-fold esm-fold-download-models
    taf-esm-fold esm-fold-check-models

  Show planned files and URLs without downloading:
    taf-esm-fold esm-fold-download-models --dry-run

  Heavier manual cache validation:
    taf-esm-fold esm-fold-check-models --load-model

  The helper uses fixed upstream ESM checkpoint URLs and size sanity checks.
  The practical end-to-end cache validation is --load-model after download.

  Default TORCH_HOME:
    $HOME/.cache/taffish/esm-fold/torch

  Standard cache paths discovered by the wrapper:
    $HOME/.cache/taffish/esm-fold/torch
    $HOME/.local/share/taffish/models/esm-fold/torch
    /usr/local/share/taffish/models/esm-fold/torch
    /opt/taffish/models/esm-fold/torch

  Explicit cache path:
    mkdir -p /path/to/torch-cache
    TAFFISH_ESM_FOLD_TORCH_HOME=/path/to/torch-cache \
      taf-esm-fold esm-fold-download-models

  Existing explicit cache directories are mounted automatically for Docker,
  Podman, and Apptainer. Set TAFFISH_ESM_FOLD_AUTO_MOUNT=0 if you want to
  provide backend mount arguments manually.

Notes:
  The default command is esm-fold, so option-leading calls can use
  taf-esm-fold -- -i proteins.fa -o pdb-out.
  This image patches upstream --cpu-only so the model is converted to float32
  before CPU inference. CPU-only remains slow and memory-heavy.
  Docker and Podman embed --platform linux/amd64 in src/main.taf. Apple Silicon
  Docker does not expose the Apple GPU to this Linux CUDA container; disable
  app-managed GPU flags for local CPU/emulation checks.
  Model weights can be prepared ahead of time with esm-fold-download-models.
  Offline systems should copy a prepared TORCH_HOME from a networked host.
  AlphaFold, ColabFold, template databases, MSA databases, and visualization
  tools are not bundled.

Container:
  image: ghcr.io/taffish/esm-fold:2.0.0-r4
  native platform: linux/amd64
  runtime: NVIDIA GPU auto-detected on Linux; CPU-only is upstream-supported
  helpers: esm-fold-download-models, esm-fold-check-models

Upstream:
  source:  https://github.com/facebookresearch/esm
  release: v2.0.0
  PyPI:    fair-esm 2.0.0
  license: MIT
  citation: Lin et al. 2023, doi:10.1126/science.ade2574
