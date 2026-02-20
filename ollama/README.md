# Ollama with NVIDIA GPU

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Deploy Ollama with NVIDIA GPU acceleration using Docker/Podman Compose.

### Prerequisites

- NVIDIA Driver installed (`nvidia-smi`)
- Podman/Docker installed
- NVIDIA Container Toolkit installed
- CDI configured (`/etc/cdi/nvidia.yaml`)

### Quick Start

```bash
# Start
podman-compose up -d

# Verify
podman ps
curl http://localhost:11434/api/tags
podman exec ollama nvidia-smi

# Pull a model
podman exec ollama ollama pull llama3.2

# Run interactively
podman exec -it ollama ollama run llama3.2
```

### Stop

```bash
podman-compose down
```

### API Endpoint

- URL: `http://localhost:11434`

---

## 中文說明

### 概述

使用 Docker/Podman Compose 部署支援 NVIDIA GPU 的 Ollama。

### 前置需求

- 已安裝 NVIDIA 驅動 (`nvidia-smi`)
- 已安裝 Podman/Docker
- 已安裝 NVIDIA Container Toolkit
- 已配置 CDI (`/etc/cdi/nvidia.yaml`)

### 快速開始

```bash
# 啟動
podman-compose up -d

# 驗證
podman ps
curl http://localhost:11434/api/tags
podman exec ollama nvidia-smi

# 拉取模型
podman exec ollama ollama pull llama3.2

# 互動式運行
podman exec -it ollama ollama run llama3.2
```

### 停止

```bash
podman-compose down
```

### API 端點

- URL: `http://localhost:11434`
