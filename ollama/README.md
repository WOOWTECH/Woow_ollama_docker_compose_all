# Ollama with NVIDIA GPU + Open WebUI

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Deploy Ollama with NVIDIA GPU acceleration and Open WebUI for chat and model management.

### Services

| Service | Port | Description |
|---------|------|-------------|
| Ollama | 11434 | LLM API Server |
| Open WebUI | 13000 | Chat Interface + Model Management |

### Features

- **Chat Interface** - ChatGPT-style conversation with models
- **Model Management** - Download, delete, and view model information
- **NVIDIA GPU** - Hardware acceleration for fast inference
- **No Authentication** - Direct access without login

### Prerequisites

- NVIDIA Driver installed (`nvidia-smi`)
- Podman/Docker installed
- NVIDIA Container Toolkit installed
- CDI configured (`/etc/cdi/nvidia.yaml`)

### Quick Start

```bash
# Start all services
podman-compose up -d

# Verify
podman ps

# Check GPU access
podman exec ollama nvidia-smi
```

### Access

- **Web UI**: http://localhost:13000
- **API**: http://localhost:11434

### Download Models via Web UI

1. Open http://localhost:13000
2. Click **Settings** (gear icon) in the sidebar
3. Go to **Admin Settings** > **Models**
4. Enter model name in "Pull a model" field (e.g., `qwen2.5:7b`)
5. Click **Pull** and wait for download

### Recommended Models (6GB VRAM)

| Model | Size | Use Case |
|-------|------|----------|
| `qwen2.5:7b` | ~4.4GB | General purpose, Chinese/English |
| `llama3.2:3b` | ~2GB | Fast, good reasoning |
| `deepseek-coder:6.7b` | ~3.8GB | Code generation |
| `phi3:mini` | ~2.3GB | Microsoft, good for coding |

### CLI Commands

```bash
# Pull a model
podman exec ollama ollama pull qwen2.5:7b

# List models
podman exec ollama ollama list

# Delete a model
podman exec ollama ollama rm <model_name>

# Run interactively
podman exec -it ollama ollama run qwen2.5:7b
```

### Stop

```bash
podman-compose down
```

---

## 中文說明

### 概述

部署支援 NVIDIA GPU 的 Ollama，搭配 Open WebUI 進行聊天和模型管理。

### 服務

| 服務 | 端口 | 說明 |
|------|------|------|
| Ollama | 11434 | LLM API 服務 |
| Open WebUI | 13000 | 聊天介面 + 模型管理 |

### 功能

- **聊天介面** - ChatGPT 風格的對話介面
- **模型管理** - 下載、刪除、查看模型資訊
- **NVIDIA GPU** - 硬體加速推理
- **無需認證** - 直接訪問，無需登入

### 前置需求

- 已安裝 NVIDIA 驅動 (`nvidia-smi`)
- 已安裝 Podman/Docker
- 已安裝 NVIDIA Container Toolkit
- 已配置 CDI (`/etc/cdi/nvidia.yaml`)

### 快速開始

```bash
# 啟動所有服務
podman-compose up -d

# 驗證
podman ps

# 檢查 GPU 訪問
podman exec ollama nvidia-smi
```

### 訪問

- **Web 介面**: http://localhost:13000
- **API**: http://localhost:11434

### 透過 Web UI 下載模型

1. 打開 http://localhost:13000
2. 點擊側邊欄的 **Settings**（齒輪圖標）
3. 進入 **Admin Settings** > **Models**
4. 在 "Pull a model" 輸入框中輸入模型名稱（例如 `qwen2.5:7b`）
5. 點擊 **Pull** 等待下載完成

### 推薦模型（6GB VRAM）

| 模型 | 大小 | 用途 |
|------|------|------|
| `qwen2.5:7b` | ~4.4GB | 通用，中英文皆優 |
| `llama3.2:3b` | ~2GB | 快速，推理能力強 |
| `deepseek-coder:6.7b` | ~3.8GB | 程式碼生成 |
| `phi3:mini` | ~2.3GB | 微軟模型，適合編程 |

### CLI 命令

```bash
# 拉取模型
podman exec ollama ollama pull qwen2.5:7b

# 列出模型
podman exec ollama ollama list

# 刪除模型
podman exec ollama ollama rm <model_name>

# 互動式運行
podman exec -it ollama ollama run qwen2.5:7b
```

### 停止

```bash
podman-compose down
```
