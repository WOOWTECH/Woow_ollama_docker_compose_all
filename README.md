# Ollama Docker/Podman Deployment with NVIDIA GPU

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Deploy Ollama with NVIDIA GPU acceleration using Docker Compose / Podman Compose.

**Environment Verified:**
- OS: Ubuntu 24.04 LTS
- Container Runtime: Podman 4.9.3
- GPU: NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM)
- NVIDIA Driver: 580.126.09
- CUDA Version: 13.0
- NVIDIA Container Toolkit: 1.18.2

### Prerequisites

1. **NVIDIA Driver** - Ensure NVIDIA driver is installed
   ```bash
   nvidia-smi
   ```

2. **Podman or Docker** - Container runtime
   ```bash
   podman --version
   # or
   docker --version
   ```

3. **NVIDIA Container Toolkit** - Required for GPU passthrough
   ```bash
   nvidia-ctk --version
   ```

### Installing NVIDIA Container Toolkit (Ubuntu/Debian)

```bash
# 1. Add NVIDIA GPG key
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# 2. Add repository
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# 3. Install
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# 4. Configure CDI for Podman
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# 5. Verify installation
nvidia-ctk --version
```

### Quick Start

```bash
# Clone or navigate to this directory
cd podman_docker_app

# Start Ollama
podman-compose up -d
# or
docker-compose up -d

# Verify container is running
podman ps
# or
docker ps

# Test API
curl http://localhost:11434/api/tags

# Verify GPU access inside container
podman exec ollama nvidia-smi
# or
docker exec ollama nvidia-smi
```

### Pull and Run a Model

```bash
# Pull a model (e.g., llama3.2)
podman exec ollama ollama pull llama3.2

# Run interactively
podman exec -it ollama ollama run llama3.2

# API usage
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Hello, how are you?"
}'
```

### File Structure

```
podman_docker_app/
├── docker-compose.yml    # Main compose file
├── .env.example          # Environment variables template
├── README.md             # This documentation
└── docs/
    └── plans/
        └── prd.md        # Product Requirements Document
```

### Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `NVIDIA_VISIBLE_DEVICES` | `all` | GPU devices to expose |
| `NVIDIA_DRIVER_CAPABILITIES` | `compute,utility` | Driver capabilities |
| `OLLAMA_PORT` | `11434` | API port |

### Data Persistence

Model data is stored in the `ollama_data` Docker volume.

```bash
# View volume location
podman volume inspect ollama_data

# Backup models
podman run --rm -v ollama_data:/data -v $(pwd):/backup alpine tar cvf /backup/ollama_backup.tar /data
```

### Troubleshooting

**GPU not detected:**
```bash
# Check CDI configuration
cat /etc/cdi/nvidia.yaml

# Regenerate CDI
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

**Container fails to start:**
```bash
# Check logs
podman logs ollama

# Verify GPU access
podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi
```

---

## 中文說明

### 概述

使用 Docker Compose / Podman Compose 部署支援 NVIDIA GPU 加速的 Ollama。

**已驗證環境：**
- 作業系統：Ubuntu 24.04 LTS
- 容器運行時：Podman 4.9.3
- GPU：NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM)
- NVIDIA 驅動：580.126.09
- CUDA 版本：13.0
- NVIDIA Container Toolkit：1.18.2

### 前置需求

1. **NVIDIA 驅動** - 確保已安裝 NVIDIA 驅動
   ```bash
   nvidia-smi
   ```

2. **Podman 或 Docker** - 容器運行時
   ```bash
   podman --version
   # 或
   docker --version
   ```

3. **NVIDIA Container Toolkit** - GPU 透傳所需
   ```bash
   nvidia-ctk --version
   ```

### 安裝 NVIDIA Container Toolkit (Ubuntu/Debian)

```bash
# 1. 添加 NVIDIA GPG 金鑰
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# 2. 添加軟體源
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# 3. 安裝
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# 4. 配置 CDI 給 Podman
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# 5. 驗證安裝
nvidia-ctk --version
```

### 快速開始

```bash
# 進入此目錄
cd podman_docker_app

# 啟動 Ollama
podman-compose up -d
# 或
docker-compose up -d

# 確認容器運行中
podman ps
# 或
docker ps

# 測試 API
curl http://localhost:11434/api/tags

# 驗證容器內 GPU 訪問
podman exec ollama nvidia-smi
# 或
docker exec ollama nvidia-smi
```

### 拉取並運行模型

```bash
# 拉取模型（例如 llama3.2）
podman exec ollama ollama pull llama3.2

# 互動式運行
podman exec -it ollama ollama run llama3.2

# API 使用
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "你好，請問今天天氣如何？"
}'
```

### 檔案結構

```
podman_docker_app/
├── docker-compose.yml    # 主要 compose 檔案
├── .env.example          # 環境變數範本
├── README.md             # 本文檔
└── docs/
    └── plans/
        └── prd.md        # 產品需求文檔
```

### 配置說明

| 變數 | 預設值 | 說明 |
|------|--------|------|
| `NVIDIA_VISIBLE_DEVICES` | `all` | 暴露的 GPU 設備 |
| `NVIDIA_DRIVER_CAPABILITIES` | `compute,utility` | 驅動能力 |
| `OLLAMA_PORT` | `11434` | API 端口 |

### 資料持久化

模型資料存儲在 `ollama_data` Docker 卷中。

```bash
# 查看卷位置
podman volume inspect ollama_data

# 備份模型
podman run --rm -v ollama_data:/data -v $(pwd):/backup alpine tar cvf /backup/ollama_backup.tar /data
```

### 故障排除

**GPU 未檢測到：**
```bash
# 檢查 CDI 配置
cat /etc/cdi/nvidia.yaml

# 重新生成 CDI
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

**容器啟動失敗：**
```bash
# 查看日誌
podman logs ollama

# 驗證 GPU 訪問
podman run --rm --device nvidia.com/gpu=all ubuntu nvidia-smi
```

---

## License

MIT License
