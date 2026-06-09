FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONNOUSERSITE=1
ENV PIP_NO_CACHE_DIR=1
ENV PIP_DISABLE_PIP_VERSION_CHECK=1

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG ESM_VERSION=2.0.0
ARG OPENFOLD_COMMIT=4b41059694619831a7db195b7e0988fc4ff3a307
ARG TORCH_WHEEL_URL=https://download.pytorch.org/whl/cu113/torch-1.12.1%2Bcu113-cp38-cp38-linux_x86_64.whl
ARG PYPI_INDEX_URL=https://pypi.org/simple
ARG PYPI_TRUSTED_HOST=pypi.org

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

RUN curl -fL --retry 8 --retry-delay 5 --connect-timeout 30 \
      "${TORCH_WHEEL_URL}" \
      -o /tmp/torch-1.12.1+cu113-cp38-cp38-linux_x86_64.whl \
 && python -m pip install --no-deps /tmp/torch-1.12.1+cu113-cp38-cp38-linux_x86_64.whl \
 && rm -f /tmp/torch-1.12.1+cu113-cp38-cp38-linux_x86_64.whl

RUN python -m pip install \
      --retries 20 \
      --timeout 120 \
      --index-url "${PYPI_INDEX_URL}" \
      --trusted-host pypi.org \
      --trusted-host "${PYPI_TRUSTED_HOST}" \
      --trusted-host files.pythonhosted.org \
      "typing-extensions==3.10.0.2" \
 && python -c 'import torch; assert torch.__version__.startswith("1.12.1")'

RUN python -m pip install \
      --retries 20 \
      --timeout 120 \
      --index-url "${PYPI_INDEX_URL}" \
      --trusted-host pypi.org \
      --trusted-host "${PYPI_TRUSTED_HOST}" \
      --trusted-host files.pythonhosted.org \
      --prefer-binary \
      "numpy==1.21.2" \
      "biopython==1.79" \
      "deepspeed==0.5.9" \
      "dm-tree==0.1.6" \
      "ml-collections==0.1.0" \
      "PyYAML==5.4.1" \
      "requests==2.26.0" \
      "scipy==1.7.1" \
      "tqdm==4.62.2" \
      "pytorch-lightning==1.5.10" \
      "torchmetrics==0.6.0" \
      "einops==0.4.1" \
      "omegaconf==2.1.2" \
      "hydra-core==1.1.2" \
      "packaging" \
      "psutil"

RUN python -m pip install \
      --retries 20 \
      --timeout 120 \
      --index-url "${PYPI_INDEX_URL}" \
      --trusted-host pypi.org \
      --trusted-host "${PYPI_TRUSTED_HOST}" \
      --trusted-host files.pythonhosted.org \
      --prefer-binary \
      "fair-esm[esmfold]==${ESM_VERSION}" \
 && python -m pip install \
      --retries 20 \
      --timeout 120 \
      --index-url "${PYPI_INDEX_URL}" \
      --trusted-host pypi.org \
      --trusted-host "${PYPI_TRUSTED_HOST}" \
      --trusted-host files.pythonhosted.org \
      --prefer-binary \
      "dllogger @ git+https://github.com/NVIDIA/dllogger.git" \
 && MAX_JOBS=4 python -m pip install \
      --retries 20 \
      --timeout 120 \
      --index-url "${PYPI_INDEX_URL}" \
      --trusted-host pypi.org \
      --trusted-host "${PYPI_TRUSTED_HOST}" \
      --trusted-host files.pythonhosted.org \
      --prefer-binary \
      "openfold @ git+https://github.com/aqlaboratory/openfold.git@${OPENFOLD_COMMIT}"

COPY docker/esm-fold-download-models docker/esm-fold-check-models /usr/local/bin/

RUN mkdir -p /opt/esm/scripts \
 && curl -fsSL "https://raw.githubusercontent.com/facebookresearch/esm/v${ESM_VERSION}/scripts/esmfold_inference.py" \
      -o /opt/esm/scripts/esmfold_inference.py \
 && python -c 'from pathlib import Path; p=Path("/opt/esm/scripts/esmfold_inference.py"); s=p.read_text(); old="    if args.cpu_only:\n        model.cpu()\n"; new="    if args.cpu_only:\n        model = model.float()\n        model.cpu()\n"; assert old in s; p.write_text(s.replace(old, new))' \
 && printf '%s\n' \
      '#!/usr/bin/env bash' \
      'python /opt/esm/scripts/esmfold_inference.py "$@"' \
      > /usr/local/bin/esm-fold \
 && chmod +x /usr/local/bin/esm-fold /usr/local/bin/esm-fold-download-models /usr/local/bin/esm-fold-check-models \
 && esm-fold -h >/dev/null \
 && esm-fold-download-models --help >/dev/null \
 && esm-fold-download-models --dry-run | grep -F 'esmfold_3B_v1.pt' >/dev/null \
 && esm-fold-check-models --help >/dev/null \
 && grep -F 'model = model.float()' /opt/esm/scripts/esmfold_inference.py >/dev/null \
 && python -c 'import importlib.metadata as m; assert m.version("fair-esm") == "2.0.0"; import torch, esm, openfold, deepspeed, dllogger, tree; assert callable(esm.pretrained.esmfold_v1)' \
 && python -c 'from pathlib import Path; from esm.data import read_fasta; p=Path("/tmp/taf-esm-fold-smoke.fa"); p.write_text(">toy\nMKT\n"); assert list(read_fasta(p)) == [("toy", "MKT")]' \
 && nvcc --version >/dev/null

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=esm-fold
