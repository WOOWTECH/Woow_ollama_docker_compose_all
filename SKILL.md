# Skill: Deploy Ollama + Open WebUI (NVIDIA / AMD)

## Overview

This skill enables AI assistants to quickly deploy Ollama (with GPU acceleration) + Open WebUI using Docker/Podman Compose. Supports both **NVIDIA CUDA** and **AMD ROCm** GPUs.

## Repository

Clone from: `https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git`

## Variants

| Variant | Directory | Docker Image | Device |
|---------|-----------|-------------|--------|
| NVIDIA CUDA | `ollama/` | `ollama/ollama:latest` | `nvidia.com/gpu=all` |
| AMD ROCm | `ollama-amd/` | `ollama/ollama:rocm` | `/dev/kfd` + `/dev/dri` |
| AMD ROCm (community) | `ollama-amd/` | `ghcr.io/rjmalagon/ollama-linux-amd-apu:optm-latest` | `/dev/kfd` + `/dev/dri` |

---

## Quick Deploy: NVIDIA CUDA

### Step 1: Verify Prerequisites

```bash
nvidia-smi
nvidia-ctk --version
```

### Step 2: Install NVIDIA Container Toolkit (if missing)

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

### Step 3: Deploy

```bash
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama
podman-compose up -d
```

### Step 4: Verify

```bash
podman ps
podman exec ollama nvidia-smi
curl http://localhost:11434/api/tags
```

---

## Quick Deploy: AMD ROCm (GMKtec EVO-X2)

### Step 1: Verify Prerequisites

```bash
lspci | grep -i vga
rocminfo | grep gfx
ls /dev/kfd /dev/dri
groups | grep -E "video|render"
```

### Step 2: Install ROCm (if missing)

```bash
sudo mkdir --parents --mode=0755 /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/latest jammy main" | \
  sudo tee /etc/apt/sources.list.d/rocm.list

sudo apt-get update
sudo apt-get install -y rocm-libs rocm-dev
sudo usermod -a -G video,render $USER
```

### Step 3: Deploy

```bash
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama-amd

# Option A: Official ROCm image (try first)
podman-compose up -d

# Option B: Community image (if GPU not detected)
podman-compose -f docker-compose.community.yml up -d
```

### Step 4: Verify

```bash
podman ps
podman logs ollama-amd 2>&1 | grep -i "gpu\|rocm\|gfx"
watch -n 1 cat /sys/class/drm/card0/device/gpu_busy_percent
```

---

## Access (both variants)

- **Open WebUI**: http://localhost:13000
- **Ollama API**: http://localhost:11434

## Download a Model

```bash
# NVIDIA variant
podman exec ollama ollama pull qwen2.5:7b

# AMD variant
podman exec ollama-amd ollama pull qwen2.5:7b
```

---

## NVIDIA Configuration Reference

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    devices: [nvidia.com/gpu=all]
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility
```

## AMD ROCm Configuration Reference

```yaml
services:
  ollama:
    image: ollama/ollama:rocm
    devices:
      - /dev/kfd:/dev/kfd
      - /dev/dri:/dev/dri
    environment:
      - HSA_OVERRIDE_GFX_VERSION=11.5.1
      - HIP_VISIBLE_DEVICES=0
      - OLLAMA_HOST=0.0.0.0
    group_add: [video, render]
    security_opt: [seccomp=unconfined]
```

## AMD Community Image Configuration Reference

```yaml
services:
  ollama:
    image: ghcr.io/rjmalagon/ollama-linux-amd-apu:optm-latest
    devices:
      - /dev/kfd:/dev/kfd
      - /dev/dri:/dev/dri
    environment:
      - HSA_OVERRIDE_GFX_VERSION=11.5.1
      - OLLAMA_FLASH_ATTENTION=true
      - OLLAMA_KV_CACHE_TYPE=q8_0
    command: serve
```

---

## Recommended Models

### NVIDIA (6GB VRAM)

| Model | Command |
|-------|---------|
| qwen2.5:7b | `podman exec ollama ollama pull qwen2.5:7b` |
| llama3.2:3b | `podman exec ollama ollama pull llama3.2:3b` |
| deepseek-coder:6.7b | `podman exec ollama ollama pull deepseek-coder:6.7b` |

### AMD (96-128GB Unified RAM)

| Model | Command |
|-------|---------|
| llama3.1:70b | `podman exec ollama-amd ollama pull llama3.1:70b` |
| qwen2.5:72b | `podman exec ollama-amd ollama pull qwen2.5:72b` |
| deepseek-r1:70b | `podman exec ollama-amd ollama pull deepseek-r1:70b` |
| deepseek-coder:33b | `podman exec ollama-amd ollama pull deepseek-coder:33b` |

---

## Troubleshooting

| Problem | Variant | Solution |
|---------|---------|----------|
| GPU not detected | NVIDIA | `sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml` |
| GPU not detected | AMD | Try community image: `podman-compose -f docker-compose.community.yml up -d` |
| Falls back to CPU | AMD | Check `watch -n 1 cat /sys/class/drm/card0/device/gpu_busy_percent` |
| Permission denied | AMD | `sudo usermod -a -G video,render $USER` then re-login |
| SELinux blocks GPU | AMD | `sudo setsebool container_use_devices=1` |
| Container won't start | Both | `podman logs ollama` or `podman logs ollama-amd` |
| Open WebUI can't connect | Both | Verify `curl http://localhost:11434/api/tags` works |

---

## Target Hardware

### NVIDIA (Verified)

- Ubuntu 24.04 LTS, Podman 4.9.3
- NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM)
- NVIDIA Driver 580.126.09, CUDA 13.0

### AMD (Target)

- GMKtec EVO-X2 AI Mini PC
- AMD Ryzen AI Max+ 395 (gfx1151, Zen 5, 16C/32T)
- AMD Radeon 8060S (RDNA 3.5)
- 64GB / 96GB / 128GB LPDDR5X 8000MHz unified memory
