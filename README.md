# Woow Ollama Docker Compose All

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Production-ready Docker/Podman Compose deployment for **Ollama** + **Open WebUI** with GPU acceleration. Supports both **NVIDIA CUDA** and **AMD ROCm** GPUs.

### Architecture

```
┌───────────────────────────────────────────────────────────┐
│                      Host System                           │
│                                                            │
│  ┌──────────────────────┐  ┌───────────────────────────┐  │
│  │  NVIDIA CUDA Variant │  │  AMD ROCm Variant         │  │
│  │  (ollama/)           │  │  (ollama-amd/)            │  │
│  │                      │  │                            │  │
│  │  nvidia.com/gpu=all  │  │  /dev/kfd + /dev/dri      │  │
│  │  CDI Container       │  │  ROCm / HSA               │  │
│  │  Toolkit             │  │                            │  │
│  └──────────┬───────────┘  └──────────┬─────────────────┘  │
│             │                         │                    │
│             └───────────┬─────────────┘                    │
│                         ▼                                  │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                Podman / Docker                      │  │
│  │                                                     │  │
│  │  ┌───────────────────┐  ┌───────────────────────┐  │  │
│  │  │  Ollama (GPU)     │  │  Open WebUI           │  │  │
│  │  │  :11434           │◄─│  :13000               │  │  │
│  │  │  ollama_data vol  │  │  webui_data vol       │  │  │
│  │  └───────────────────┘  └───────────────────────┘  │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────┘
```

### Project Structure

```
Woow_ollama_docker_compose_all/
├── README.md                                # This file (bilingual)
├── SKILL.md                                 # AI deployment skill reference
├── ollama/                                  # NVIDIA CUDA variant
│   ├── docker-compose.yml                   # Ollama + Open WebUI (NVIDIA GPU)
│   └── README.md                            # NVIDIA setup documentation
└── ollama-amd/                              # AMD ROCm variant
    ├── docker-compose.yml                   # Ollama + Open WebUI (AMD ROCm)
    ├── docker-compose.community.yml         # Alternative community image for gfx1151
    └── README.md                            # AMD setup documentation
```

### Choose Your Variant

| Variant | Directory | GPU | Target Hardware |
|---------|-----------|-----|-----------------|
| **NVIDIA CUDA** | `ollama/` | NVIDIA GPUs (RTX, GTX, etc.) | Any NVIDIA GPU system |
| **AMD ROCm** | `ollama-amd/` | AMD APU/GPU (Radeon 8060S, etc.) | GMKtec EVO-X2, AMD APU systems |

### Services (both variants)

| Service | Port | Description |
|---------|------|-------------|
| Ollama | 11434 | LLM API Server with GPU acceleration |
| Open WebUI | 13000 | Chat + Model Management UI |

---

### Quick Start

#### NVIDIA GPU

```bash
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama
podman-compose up -d
```

See [ollama/README.md](ollama/README.md) for detailed NVIDIA setup (driver, CUDA toolkit, CDI config).

#### AMD GPU (GMKtec EVO-X2 / Ryzen AI Max+ 395)

```bash
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama-amd
podman-compose up -d
```

See [ollama-amd/README.md](ollama-amd/README.md) for detailed AMD setup (ROCm, device permissions, community image fallback).

#### Access

- **Web UI**: http://localhost:13000
- **API**: http://localhost:11434

---

### Prerequisites Summary

#### NVIDIA Variant

| Prerequisite | Check Command |
|-------------|---------------|
| NVIDIA Driver | `nvidia-smi` |
| Container Toolkit | `nvidia-ctk --version` |
| CDI Config | `ls /etc/cdi/nvidia.yaml` |

```bash
# Install NVIDIA Container Toolkit (Ubuntu/Debian)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

#### AMD Variant

| Prerequisite | Check Command |
|-------------|---------------|
| AMD GPU detected | `lspci \| grep -i vga` |
| ROCm installed | `rocminfo \| grep gfx` |
| User in groups | `groups \| grep -E "video\|render"` |
| Device nodes | `ls /dev/kfd /dev/dri` |

```bash
# Install ROCm (Ubuntu/Debian)
sudo mkdir --parents --mode=0755 /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/latest jammy main" | \
  sudo tee /etc/apt/sources.list.d/rocm.list

sudo apt-get update
sudo apt-get install -y rocm-libs rocm-dev

# Add user to required groups
sudo usermod -a -G video,render $USER
```

---

### Model Management

#### Via Open WebUI (http://localhost:13000)

1. Open http://localhost:13000
2. Click **Settings** (gear icon) in the sidebar
3. Go to **Admin Settings** > **Models**
4. Enter model name in "Pull a model" field (e.g., `qwen2.5:7b`)
5. Click **Pull** and wait for download

#### Via CLI

```bash
# Replace "ollama" with "ollama-amd" for AMD variant
podman exec ollama ollama pull qwen2.5:7b
podman exec ollama ollama list
podman exec -it ollama ollama run qwen2.5:7b
```

---

### Recommended Models

#### NVIDIA (6GB VRAM - RTX 4050)

| Model | Size | Use Case |
|-------|------|----------|
| `qwen2.5:7b` | ~4.4GB | General purpose, Chinese/English |
| `llama3.2:3b` | ~2GB | Fast, good reasoning |
| `deepseek-coder:6.7b` | ~3.8GB | Code generation |
| `phi3:mini` | ~2.3GB | Microsoft, good for coding |
| `gemma2:2b` | ~1.6GB | Google, lightweight |

#### AMD (96-128GB Unified RAM - EVO-X2)

| Model | Size | Use Case |
|-------|------|----------|
| `llama3.1:70b` | ~40GB | Top-tier reasoning |
| `qwen2.5:72b` | ~41GB | Best Chinese/English |
| `deepseek-r1:70b` | ~42GB | Advanced reasoning |
| `deepseek-coder:33b` | ~19GB | Advanced code generation |
| `mixtral:8x7b` | ~26GB | Mixture of experts |

---

### Verified Environments

#### NVIDIA Variant

| Component | Version |
|-----------|---------|
| OS | Ubuntu 24.04 LTS |
| Kernel | 6.17.0-14-generic |
| Podman | 4.9.3 |
| NVIDIA Driver | 580.126.09 |
| CUDA | 13.0 |
| NVIDIA Container Toolkit | 1.18.2 |
| GPU | NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM) |

#### AMD Variant

| Component | Spec |
|-----------|------|
| Target Hardware | GMKtec EVO-X2 AI Mini PC |
| APU | AMD Ryzen AI Max+ 395 (gfx1151) |
| GPU | AMD Radeon 8060S (RDNA 3.5) |
| RAM | 64GB / 96GB / 128GB LPDDR5X 8000MHz |
| Docker Image | `ollama/ollama:rocm` or `ghcr.io/rjmalagon/ollama-linux-amd-apu:optm-latest` |

---

---

## 中文說明

### 概述

生產就緒的 Docker/Podman Compose 部署方案，包含支援 GPU 加速的 **Ollama** 和用於聊天與模型管理的 **Open WebUI**。同時支援 **NVIDIA CUDA** 和 **AMD ROCm** GPU。

### 專案結構

```
Woow_ollama_docker_compose_all/
├── README.md                                # 本文件（中英文）
├── SKILL.md                                 # AI 部署技能參考
├── ollama/                                  # NVIDIA CUDA 方案
│   ├── docker-compose.yml                   # Ollama + Open WebUI（NVIDIA GPU）
│   └── README.md                            # NVIDIA 設定文檔
└── ollama-amd/                              # AMD ROCm 方案
    ├── docker-compose.yml                   # Ollama + Open WebUI（AMD ROCm）
    ├── docker-compose.community.yml         # 替代社群映像（gfx1151 支援）
    └── README.md                            # AMD 設定文檔
```

### 選擇您的方案

| 方案 | 目錄 | GPU | 目標硬體 |
|------|------|-----|----------|
| **NVIDIA CUDA** | `ollama/` | NVIDIA GPU（RTX、GTX 等）| 任何 NVIDIA GPU 系統 |
| **AMD ROCm** | `ollama-amd/` | AMD APU/GPU（Radeon 8060S 等）| GMKtec EVO-X2、AMD APU 系統 |

### 服務（兩個方案相同）

| 服務 | 端口 | 說明 |
|------|------|------|
| Ollama | 11434 | LLM API 服務（GPU 加速）|
| Open WebUI | 13000 | 聊天 + 模型管理介面 |

---

### 快速開始

#### NVIDIA GPU

```bash
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama
podman-compose up -d
```

詳細 NVIDIA 設定請參考 [ollama/README.md](ollama/README.md)。

#### AMD GPU（GMKtec EVO-X2 / Ryzen AI Max+ 395）

```bash
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama-amd
podman-compose up -d
```

詳細 AMD 設定請參考 [ollama-amd/README.md](ollama-amd/README.md)。

#### 訪問

- **Web 介面**: http://localhost:13000
- **API**: http://localhost:11434

---

### 前置需求摘要

#### NVIDIA 方案

```bash
# 安裝 NVIDIA Container Toolkit（Ubuntu/Debian）
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

#### AMD 方案

```bash
# 安裝 ROCm（Ubuntu/Debian）
sudo mkdir --parents --mode=0755 /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/latest jammy main" | \
  sudo tee /etc/apt/sources.list.d/rocm.list

sudo apt-get update
sudo apt-get install -y rocm-libs rocm-dev
sudo usermod -a -G video,render $USER
```

---

### 推薦模型

#### NVIDIA（6GB VRAM - RTX 4050）

| 模型 | 大小 | 用途 |
|------|------|------|
| `qwen2.5:7b` | ~4.4GB | 通用，中英文皆優 |
| `llama3.2:3b` | ~2GB | 快速，推理能力強 |
| `deepseek-coder:6.7b` | ~3.8GB | 程式碼生成 |
| `phi3:mini` | ~2.3GB | 微軟模型，適合編程 |

#### AMD（96-128GB 統一記憶體 - EVO-X2）

| 模型 | 大小 | 用途 |
|------|------|------|
| `llama3.1:70b` | ~40GB | 頂級推理 |
| `qwen2.5:72b` | ~41GB | 最佳中英文 |
| `deepseek-r1:70b` | ~42GB | 進階推理 |
| `deepseek-coder:33b` | ~19GB | 進階程式碼生成 |
| `mixtral:8x7b` | ~26GB | 混合專家模型 |

---

### 已驗證環境

#### NVIDIA 方案

| 組件 | 版本 |
|------|------|
| 作業系統 | Ubuntu 24.04 LTS |
| Podman | 4.9.3 |
| NVIDIA 驅動 | 580.126.09 |
| CUDA | 13.0 |
| GPU | NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM) |

#### AMD 方案

| 組件 | 規格 |
|------|------|
| 目標硬體 | GMKtec EVO-X2 AI Mini PC |
| APU | AMD Ryzen AI Max+ 395 (gfx1151) |
| GPU | AMD Radeon 8060S (RDNA 3.5) |
| 記憶體 | 64GB / 96GB / 128GB LPDDR5X 8000MHz |

---

## License

MIT License
