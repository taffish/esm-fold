FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONNOUSERSITE=1
ENV PIP_NO_CACHE_DIR=1

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG ESM_VERSION=2.0.0
ARG OPENFOLD_COMMIT=4b41059694619831a7db195b7e0988fc4ff3a307

WORKDIR /root

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      git \
      build-essential \
      ninja-build \
      python3 \
      python3-dev \
      python3-pip \
      python-is-python3 \
 && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --upgrade \
      "pip<24" \
      "setuptools==59.5.0" \
      wheel

RUN python -m pip install \
      --extra-index-url https://download.pytorch.org/whl/cu113 \
      "torch==1.12.1+cu113"

RUN python -m pip install \
      "numpy==1.21.2" \
      "biopython==1.79" \
      "deepspeed==0.5.9" \
      "dm-tree==0.1.6" \
      "ml-collections==0.1.0" \
      "PyYAML==5.4.1" \
      "requests==2.26.0" \
      "scipy==1.7.1" \
      "tqdm==4.62.2" \
      "typing-extensions==3.10.0.2" \
      "pytorch-lightning==1.5.10" \
      "torchmetrics==0.6.0" \
      "einops==0.4.1" \
      "omegaconf==2.1.2" \
      "hydra-core==1.1.2" \
      "packaging" \
      "psutil"

RUN python -m pip install "fair-esm[esmfold]==${ESM_VERSION}" \
 && python -m pip install "dllogger @ git+https://github.com/NVIDIA/dllogger.git" \
 && MAX_JOBS=4 python -m pip install "openfold @ git+https://github.com/aqlaboratory/openfold.git@${OPENFOLD_COMMIT}"

RUN mkdir -p /opt/esm/scripts \
 && curl -fsSL "https://raw.githubusercontent.com/facebookresearch/esm/v${ESM_VERSION}/scripts/esmfold_inference.py" \
      -o /opt/esm/scripts/esmfold_inference.py \
 && python -c 'from pathlib import Path; p=Path("/opt/esm/scripts/esmfold_inference.py"); s=p.read_text(); old="    if args.cpu_only:\n        model.cpu()\n"; new="    if args.cpu_only:\n        model = model.float()\n        model.cpu()\n"; assert old in s; p.write_text(s.replace(old, new))' \
 && printf '%s\n' \
      '#!/usr/bin/env bash' \
      'python /opt/esm/scripts/esmfold_inference.py "$@"' \
      > /usr/local/bin/esm-fold \
 && chmod +x /usr/local/bin/esm-fold \
 && esm-fold -h >/dev/null \
 && grep -F 'model = model.float()' /opt/esm/scripts/esmfold_inference.py >/dev/null \
 && python -c 'import importlib.metadata as m; assert m.version("fair-esm") == "2.0.0"; import torch, esm, openfold, deepspeed, dllogger, tree; assert callable(esm.pretrained.esmfold_v1)' \
 && python -c 'from pathlib import Path; from esm.data import read_fasta; p=Path("/tmp/taf-esm-fold-smoke.fa"); p.write_text(">toy\nMKT\n"); assert list(read_fasta(p)) == [("toy", "MKT")]' \
 && nvcc --version >/dev/null

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=esm-fold
