# syntax=docker/dockerfile:1
FROM nvidia/cuda:12.6.3-runtime-ubuntu24.04

# Prevent Python from writing .pyc files and enable buffered output
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    UV_LINK_MODE=copy \
    NVIDIA_VISIBLE_DEVICES=all \
    NVIDIA_DRIVER_CAPABILITIES=compute,utility \
    LD_LIBRARY_PATH=/usr/local/cuda-12.6/compat:/usr/local/nvidia/lib64:${LD_LIBRARY_PATH}

# Install Python 3.12 and system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3.12 \
    python3.12-venv \
    python3.12-dev \
    build-essential \
    git \
    curl \
    ca-certificates \
    libgl1 \
    libglib2.0-0 \
    && ln -sf /usr/bin/python3.12 /usr/bin/python3 \
    && ln -sf /usr/bin/python3.12 /usr/bin/python \
    && rm -rf /var/lib/apt/lists/*

# Create symlinks for NVIDIA libs (for WSL2 compatibility)
# WSL2 mounts drivers in /usr/lib/wsl/drivers/nv_dispi.inf_*/
RUN mkdir -p /usr/local/nvidia/lib64 && \
    ln -sf /usr/local/cuda-12.6/compat/libcuda.so.1 /usr/local/nvidia/lib64/libcuda.so.1 && \
    ln -sf /usr/local/cuda-12.6/compat/libcuda.so /usr/local/nvidia/lib64/libcuda.so && \
    mkdir -p /usr/lib/wsl/lib && \
    echo '#!/bin/bash\nWSL_DRIVER_DIR=$(find /usr/lib/wsl/drivers -maxdepth 1 -type d -name "nv_dispi*" 2>/dev/null | head -1)\nif [ -n "$WSL_DRIVER_DIR" ] && [ ! -L /usr/lib/wsl/lib/libcuda.so.1 ]; then\n  ln -sf "$WSL_DRIVER_DIR/libcuda.so.1.1" /usr/lib/wsl/lib/libcuda.so.1 2>/dev/null || true\n  ln -sf "$WSL_DRIVER_DIR/libcuda.so.1.1" /usr/lib/wsl/lib/libcuda.so 2>/dev/null || true\n  ln -sf "$WSL_DRIVER_DIR/libnvidia-ml.so.1" /usr/lib/wsl/lib/libnvidia-ml.so.1 2>/dev/null || true\n  ln -sf "$WSL_DRIVER_DIR/libnvidia-ptxjitcompiler.so.1" /usr/lib/wsl/lib/libnvidia-ptxjitcompiler.so.1 2>/dev/null || true\nfi' > /usr/local/bin/setup-wsl-nvidia.sh && \
    chmod +x /usr/local/bin/setup-wsl-nvidia.sh

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app

# Clone repo to get pyproject.toml and uv.lock for dependency installation
RUN git clone --depth 1 https://github.com/MatheusLevy/PFP-IA-Engineering.git /app

# Pre-install all dependencies (heavy packages like torch, cuda libs, etc)
# This layer will be cached and reused across job runs
RUN uv sync --frozen

RUN mkdir -p /root/.aws

# Clean up the cloned repo - will be re-cloned fresh in the job
RUN rm -rf /app/.git /app/src /app/data /app/dvc.yaml /app/dvc.lock /app/*.dvc

# Default shell
CMD ["/bin/bash"]
