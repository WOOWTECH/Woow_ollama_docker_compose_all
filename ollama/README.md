# Ollama with NVIDIA GPU + Web UI

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Deploy Ollama with NVIDIA GPU acceleration and a web-based management UI.

### Services

| Service | Port | Description |
|---------|------|-------------|
| Ollama | 11434 | LLM API Server |
| Ollama WebUI | 3000 | Web Management Interface |

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

### First Time Setup

1. Open http://localhost:3000
2. Create an admin account
3. Go to Settings > Models
4. Download a model (e.g., `llama3.2`)

### Pull Models via CLI

```bash
# Pull a model
podman exec ollama ollama pull llama3.2

# List models
podman exec ollama ollama list

# Run interactively
podman exec -it ollama ollama run llama3.2
```

### Stop

```bash
podman-compose down
```

---

## 中文說明

### 概述

部署支援 NVIDIA GPU 的 Ollama，並附帶 Web 管理介面。

### 服務

| 服務 | 端口 | 說明 |
|------|------|------|
| Ollama | 11434 | LLM API 服務 |
| Ollama WebUI | 3000 | Web 管理介面 |

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

### 首次設定

1. 打開 http://localhost:3000
2. 建立管理員帳戶
3. 進入 Settings > Models
4. 下載模型（例如 `llama3.2`）

### 透過 CLI 拉取模型

```bash
# 拉取模型
podman exec ollama ollama pull llama3.2

# 列出模型
podman exec ollama ollama list

# 互動式運行
podman exec -it ollama ollama run llama3.2
```

### 停止

```bash
podman-compose down
```
