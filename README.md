# Woow Ollama Docker Compose All

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Production-ready Docker/Podman Compose deployment for **Ollama** with NVIDIA GPU acceleration and **Open WebUI** for chat and model management.

### Architecture

```
┌──────────────────────────────────────────────────┐
│                   Host System                     │
│                                                   │
│  ┌─────────────────────────────────────────────┐  │
│  │  NVIDIA Driver + Container Toolkit (CDI)    │  │
│  └─────────────────────────────────────────────┘  │
│                        │                          │
│  ┌─────────────────────────────────────────────┐  │
│  │            Podman / Docker                  │  │
│  │                                             │  │
│  │  ┌───────────────────┐  ┌───────────────┐  │  │
│  │  │  Ollama (GPU)     │  │  Open WebUI   │  │  │
│  │  │  :11434           │◄─│  :13000       │  │  │
│  │  │  ollama_data vol  │  │  webui_data   │  │  │
│  │  └───────────────────┘  └───────────────┘  │  │
│  │                                             │  │
│  └─────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

### Project Structure

```
Woow_ollama_docker_compose_all/
├── README.md                          # This file (bilingual)
├── SKILL.md                           # AI deployment skill reference
└── ollama/
    ├── docker-compose.yml             # Ollama + Open WebUI
    └── README.md                      # Ollama documentation
```

### Services

| Service | Port | Description |
|---------|------|-------------|
| Ollama | 11434 | LLM API Server with NVIDIA GPU |
| Open WebUI | 13000 | Chat + Model Management UI |

---

### Prerequisites

#### 1. Container Runtime

```bash
# Check Podman
podman --version

# Or Docker
docker --version
```

#### 2. NVIDIA Driver (for GPU acceleration)

```bash
nvidia-smi
```

#### 3. NVIDIA Container Toolkit

```bash
# Check if installed
nvidia-ctk --version

# If not installed (Ubuntu/Debian):
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# Configure CDI for Podman
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# Verify
nvidia-ctk --version
```

#### 4. Verify GPU Access in Container

```bash
podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi
```

---

### Deploy

```bash
# Clone repository
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama

# Start services
podman-compose up -d
# or: docker-compose up -d

# Verify
podman ps

# Check GPU
podman exec ollama nvidia-smi
```

### Access

- **Web UI**: http://localhost:13000
- **API**: http://localhost:11434

### Stop

```bash
cd ollama && podman-compose down
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
# Pull a model
podman exec ollama ollama pull qwen2.5:7b

# List models
podman exec ollama ollama list

# Delete a model
podman exec ollama ollama rm <model_name>

# Run interactively
podman exec -it ollama ollama run qwen2.5:7b

# Show model info
podman exec ollama ollama show qwen2.5:7b
```

#### Recommended Models (6GB VRAM - RTX 4050)

| Model | Size | Use Case |
|-------|------|----------|
| `qwen2.5:7b` | ~4.4GB | General purpose, Chinese/English |
| `llama3.2:3b` | ~2GB | Fast, good reasoning |
| `deepseek-coder:6.7b` | ~3.8GB | Code generation |
| `phi3:mini` | ~2.3GB | Microsoft, good for coding |
| `gemma2:2b` | ~1.6GB | Google, lightweight |

---

### Troubleshooting

#### GPU not detected in Ollama container

```bash
# Regenerate CDI
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# Restart Ollama
cd ollama && podman-compose down && podman-compose up -d
```

#### Container won't start

```bash
# Check logs
podman logs ollama
podman logs open-webui

# Verify GPU access
podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi
```

#### Open WebUI can't connect to Ollama

```bash
# Verify Ollama is running
curl http://localhost:11434/api/tags

# Check network
podman network inspect ollama_default
```

---

### Verified Environment

| Component | Version |
|-----------|---------|
| OS | Ubuntu 24.04 LTS |
| Kernel | 6.17.0-14-generic |
| Podman | 4.9.3 |
| NVIDIA Driver | 580.126.09 |
| CUDA | 13.0 |
| NVIDIA Container Toolkit | 1.18.2 |
| GPU | NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM) |

---

---

## 中文說明

### 概述

生產就緒的 Docker/Podman Compose 部署方案，包含支援 NVIDIA GPU 加速的 **Ollama** 和用於聊天與模型管理的 **Open WebUI**。

### 架構

```
┌──────────────────────────────────────────────────┐
│                   主機系統                         │
│                                                   │
│  ┌─────────────────────────────────────────────┐  │
│  │  NVIDIA 驅動 + Container Toolkit (CDI)      │  │
│  └─────────────────────────────────────────────┘  │
│                        │                          │
│  ┌─────────────────────────────────────────────┐  │
│  │            Podman / Docker                  │  │
│  │                                             │  │
│  │  ┌───────────────────┐  ┌───────────────┐  │  │
│  │  │  Ollama (GPU)     │  │  Open WebUI   │  │  │
│  │  │  :11434           │◄─│  :13000       │  │  │
│  │  │  ollama_data 卷   │  │  webui_data   │  │  │
│  │  └───────────────────┘  └───────────────┘  │  │
│  │                                             │  │
│  └─────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

### 專案結構

```
Woow_ollama_docker_compose_all/
├── README.md                          # 本文件（中英文）
├── SKILL.md                           # AI 部署技能參考
└── ollama/
    ├── docker-compose.yml             # Ollama + Open WebUI
    └── README.md                      # Ollama 文檔
```

### 服務列表

| 服務 | 端口 | 說明 |
|------|------|------|
| Ollama | 11434 | LLM API 服務（NVIDIA GPU 加速）|
| Open WebUI | 13000 | 聊天 + 模型管理介面 |

---

### 前置需求

#### 1. 容器運行時

```bash
# 檢查 Podman
podman --version

# 或 Docker
docker --version
```

#### 2. NVIDIA 驅動（GPU 加速所需）

```bash
nvidia-smi
```

#### 3. NVIDIA Container Toolkit

```bash
# 檢查是否已安裝
nvidia-ctk --version

# 若未安裝（Ubuntu/Debian）：
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# 配置 CDI 給 Podman
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# 驗證
nvidia-ctk --version
```

#### 4. 驗證容器中的 GPU 訪問

```bash
podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi
```

---

### 部署

```bash
# 克隆倉庫
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all/ollama

# 啟動服務
podman-compose up -d
# 或：docker-compose up -d

# 驗證
podman ps

# 檢查 GPU
podman exec ollama nvidia-smi
```

### 訪問

- **Web 介面**: http://localhost:13000
- **API**: http://localhost:11434

### 停止

```bash
cd ollama && podman-compose down
```

---

### 模型管理

#### 透過 Open WebUI (http://localhost:13000)

1. 打開 http://localhost:13000
2. 點擊側邊欄的 **Settings**（齒輪圖標）
3. 進入 **Admin Settings** > **Models**
4. 在 "Pull a model" 輸入框中輸入模型名稱（例如 `qwen2.5:7b`）
5. 點擊 **Pull** 等待下載完成

#### 透過 CLI

```bash
# 拉取模型
podman exec ollama ollama pull qwen2.5:7b

# 列出模型
podman exec ollama ollama list

# 刪除模型
podman exec ollama ollama rm <model_name>

# 互動式運行
podman exec -it ollama ollama run qwen2.5:7b

# 顯示模型資訊
podman exec ollama ollama show qwen2.5:7b
```

#### 推薦模型（6GB VRAM - RTX 4050）

| 模型 | 大小 | 用途 |
|------|------|------|
| `qwen2.5:7b` | ~4.4GB | 通用，中英文皆優 |
| `llama3.2:3b` | ~2GB | 快速，推理能力強 |
| `deepseek-coder:6.7b` | ~3.8GB | 程式碼生成 |
| `phi3:mini` | ~2.3GB | 微軟模型，適合編程 |
| `gemma2:2b` | ~1.6GB | Google 模型，輕量高效 |

---

### 故障排除

#### GPU 未檢測到

```bash
# 重新生成 CDI
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# 重啟 Ollama
cd ollama && podman-compose down && podman-compose up -d
```

#### 容器無法啟動

```bash
# 查看日誌
podman logs ollama
podman logs open-webui

# 驗證 GPU 訪問
podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi
```

#### Open WebUI 無法連接 Ollama

```bash
# 確認 Ollama 運行中
curl http://localhost:11434/api/tags

# 檢查網路
podman network inspect ollama_default
```

---

### 已驗證環境

| 組件 | 版本 |
|------|------|
| 作業系統 | Ubuntu 24.04 LTS |
| 核心 | 6.17.0-14-generic |
| Podman | 4.9.3 |
| NVIDIA 驅動 | 580.126.09 |
| CUDA | 13.0 |
| NVIDIA Container Toolkit | 1.18.2 |
| GPU | NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM) |

---

## License

MIT License
