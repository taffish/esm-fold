taf-esm-fold 2.0.0-r3

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

Notes:
  The default command is esm-fold, so option-leading calls can use
  taf-esm-fold -- -i proteins.fa -o pdb-out.
  This image patches upstream --cpu-only so the model is converted to float32
  before CPU inference. CPU-only remains slow and memory-heavy.
  Docker and Podman embed --platform linux/amd64 in src/main.taf. Apple Silicon
  Docker does not expose the Apple GPU to this Linux CUDA container; disable
  app-managed GPU flags for local CPU/emulation checks.
  Model weights download on first real prediction into TORCH_HOME, defaulting
  to $HOME/.cache/taffish/esm-fold/torch.
  AlphaFold, ColabFold, template databases, MSA databases, and visualization
  tools are not bundled.

Container:
  image: ghcr.io/taffish/esm-fold:2.0.0-r3
  native platform: linux/amd64
  runtime: NVIDIA GPU auto-detected on Linux; CPU-only is upstream-supported

Upstream:
  source:  https://github.com/facebookresearch/esm
  release: v2.0.0
  PyPI:    fair-esm 2.0.0
  license: MIT
  citation: Lin et al. 2023, doi:10.1126/science.ade2574
