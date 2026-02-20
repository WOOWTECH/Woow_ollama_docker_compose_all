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
| Open WebUI | 3000 | Chat Interface + Model Management |

### Features

- **Chat Interface** - ChatGPT-style conversation with models
- **Model Management** - Download, delete, and view model information
- **NVIDIA GPU** - Hardware acceleration for fast inference

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

- **Web UI**: http://localhost:3000
- **API**: http://localhost:11434

### Model Management via Web UI

1. Open http://localhost:3000
2. Click on your profile icon (top right)
3. Go to **Admin Panel** > **Settings** > **Models**
4. Download models by entering model name (e.g., `llama3.2`)
5. View/delete existing models

### Pull Models via CLI

```bash
# Pull a model
podman exec ollama ollama pull llama3.2

# List models
podman exec ollama ollama list

# Delete a model
podman exec ollama ollama rm llama3.2

# Show model info
podman exec ollama ollama show llama3.2
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
| Open WebUI | 3000 | 聊天介面 + 模型管理 |

### 功能

- **聊天介面** - ChatGPT 風格的對話介面
- **模型管理** - 下載、刪除、查看模型資訊
- **NVIDIA GPU** - 硬體加速推理

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

- **Web 介面**: http://localhost:3000
- **API**: http://localhost:11434

### 透過 Web UI 管理模型

1. 打開 http://localhost:3000
2. 點擊右上角的個人頭像
3. 進入 **Admin Panel** > **Settings** > **Models**
4. 輸入模型名稱下載（例如 `llama3.2`）
5. 查看/刪除現有模型

### 透過 CLI 管理模型

```bash
# 拉取模型
podman exec ollama ollama pull llama3.2

# 列出模型
podman exec ollama ollama list

# 刪除模型
podman exec ollama ollama rm llama3.2

# 顯示模型資訊
podman exec ollama ollama show llama3.2
```

### 停止

```bash
podman-compose down
```
