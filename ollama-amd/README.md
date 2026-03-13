# Ollama with AMD ROCm GPU + Open WebUI

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Deploy Ollama with **AMD ROCm GPU acceleration** and Open WebUI for chat and model management. Designed for AMD APU systems like the **GMKtec EVO-X2** (AMD Ryzen AI Max+ 395 / Radeon 8060S).

### Target Hardware

| Component | Spec |
|-----------|------|
| Mini PC | GMKtec EVO-X2 AI Mini PC |
| CPU/APU | AMD Ryzen AI Max+ 395 (16C/32T, Zen 5, up to 5.1GHz) |
| Integrated GPU | AMD Radeon 8060S (gfx1151, RDNA 3.5) |
| RAM | 64GB / 96GB / 128GB LPDDR5X 8000MHz (unified memory) |
| NPU | AMD XDNA 2 (50 TOPS) |

> **Key advantage**: The Ryzen AI Max+ 395 uses **unified memory** — the GPU shares system RAM, so a 128GB config gives up to ~96GB usable VRAM. This enables running **70B+ parameter models** locally.

### Services

| Service | Port | Description |
|---------|------|-------------|
| Ollama | 11434 | LLM API Server (AMD ROCm GPU) |
| Open WebUI | 13000 | Chat Interface + Model Management |

### Features

- **Chat Interface** — ChatGPT-style conversation with models
- **Model Management** — Download, delete, and view model information
- **AMD ROCm GPU** — Hardware acceleration via ROCm for AMD integrated GPU
- **No Authentication** — Direct access without login
- **Unified Memory** — Leverage all system RAM as VRAM for larger models

---

### Prerequisites

#### 1. Container Runtime

```bash
# Check Podman
podman --version

# Or Docker
docker --version
```

#### 2. AMD GPU Driver

```bash
# Verify AMD GPU is detected
lspci | grep -i vga

# Check ROCm info (if rocminfo is installed)
rocminfo | grep gfx
# Should show: gfx1151
```

#### 3. Install ROCm (Ubuntu/Debian)

```bash
# Add AMD ROCm repository
sudo mkdir --parents --mode=0755 /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null

# Add repo (adjust version as needed)
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/latest jammy main" | \
  sudo tee /etc/apt/sources.list.d/rocm.list

sudo apt-get update
sudo apt-get install -y rocm-libs rocm-dev

# Verify
rocminfo
```

#### 4. User Permissions

```bash
# Add user to video and render groups (required for GPU access)
sudo usermod -a -G video,render $USER

# Apply (logout/login or run)
newgrp video && newgrp render
```

#### 5. Verify GPU Access

```bash
# Check device nodes exist
ls -la /dev/kfd /dev/dri

# Test with container
podman run --rm --device /dev/kfd --device /dev/dri \
  ollama/ollama:rocm rocminfo | grep gfx
```

---

### Deploy

#### Option A: Official Ollama ROCm Image (try first)

```bash
cd ollama-amd

# Start services
podman-compose up -d
# or: docker-compose up -d

# Verify
podman ps
```

#### Option B: Community Image (if GPU not detected)

If the official `ollama/ollama:rocm` image does not properly detect your Radeon 8060S GPU (silently falls back to CPU), use the community-built image with native gfx1151 support:

```bash
cd ollama-amd

# Use community compose file
podman-compose -f docker-compose.community.yml up -d
# or: docker-compose -f docker-compose.community.yml up -d

# Verify
podman ps
```

> The community image is from [rjmalagon/ollama-linux-amd-apu](https://github.com/rjmalagon/ollama-linux-amd-apu) and includes bleeding-edge ROCm support for Strix Halo (gfx1151).

### Verify GPU is Being Used

```bash
# Check Ollama logs for GPU detection
podman logs ollama-amd 2>&1 | grep -i "gpu\|rocm\|hip\|gfx"

# Check if model layers are offloaded to GPU
podman exec ollama-amd ollama run qwen2.5:7b "hello" 2>&1 | head -5

# Monitor GPU utilization
# (inside the host, not the container)
watch -n 1 cat /sys/class/drm/card0/device/gpu_busy_percent
```

### Access

- **Web UI**: http://localhost:13000
- **API**: http://localhost:11434

### Stop

```bash
podman-compose down
# or for community image:
podman-compose -f docker-compose.community.yml down
```

---

### Compose Files

| File | Description |
|------|-------------|
| `docker-compose.yml` | Official `ollama/ollama:rocm` image |
| `docker-compose.community.yml` | Community image with native gfx1151 ROCm support |

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
podman exec ollama-amd ollama pull qwen2.5:7b

# List models
podman exec ollama-amd ollama list

# Delete a model
podman exec ollama-amd ollama rm <model_name>

# Run interactively
podman exec -it ollama-amd ollama run qwen2.5:7b

# Show model info
podman exec ollama-amd ollama show qwen2.5:7b
```

#### Recommended Models by RAM Configuration

##### 64GB RAM (~48GB usable VRAM)

| Model | Size | Use Case |
|-------|------|----------|
| `qwen2.5:32b` | ~20GB | High quality Chinese/English |
| `llama3.1:8b` | ~4.7GB | General purpose |
| `deepseek-coder:33b` | ~19GB | Advanced code generation |
| `mixtral:8x7b` | ~26GB | Mixture of experts |
| `phi3:mini` | ~2.3GB | Lightweight coding |

##### 96GB RAM (~72GB usable VRAM)

| Model | Size | Use Case |
|-------|------|----------|
| `llama3.1:70b` | ~40GB | Top-tier reasoning |
| `qwen2.5:72b` | ~41GB | Best Chinese/English |
| `deepseek-coder:33b` | ~19GB | Advanced code generation |
| `command-r:35b` | ~20GB | Retrieval-augmented generation |

##### 128GB RAM (~96GB usable VRAM)

| Model | Size | Use Case |
|-------|------|----------|
| `llama3.1:70b` | ~40GB | Top-tier reasoning |
| `qwen2.5:72b` | ~41GB | Best Chinese/English |
| `deepseek-r1:70b` | ~42GB | Advanced reasoning |
| `mixtral:8x22b` | ~80GB | Largest mixture of experts |

---

### Key Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `HSA_OVERRIDE_GFX_VERSION` | `11.5.1` | Force ROCm to recognize gfx1151 GPU |
| `HIP_VISIBLE_DEVICES` | `0` | Select GPU device index |
| `OLLAMA_HOST` | `0.0.0.0` | Listen on all interfaces |
| `OLLAMA_FLASH_ATTENTION` | `true` | Enable flash attention (community image) |
| `OLLAMA_KV_CACHE_TYPE` | `q8_0` | Quantized KV cache for memory savings |

---

### GTT Memory Tuning (Optional)

By default, the Graphics Translation Table (GTT) memory is half of system RAM. To maximize VRAM available to the GPU:

```bash
# Create modprobe config
sudo tee /etc/modprobe.d/ttm.conf << 'EOF'
# Increase GTT memory for AMD integrated GPU
# Value = number of 4KB pages (e.g., 25165824 = 96GB)
options ttm pages_limit=25165824
options ttm page_pool_size=25165824
EOF

# Apply (requires reboot)
sudo reboot
```

| RAM Config | Recommended pages_limit | Effective GTT |
|------------|------------------------|---------------|
| 64GB | 12582912 | ~48GB |
| 96GB | 18874368 | ~72GB |
| 128GB | 25165824 | ~96GB |

---

### Troubleshooting

#### GPU not detected / falls back to CPU

```bash
# 1. Verify device nodes
ls -la /dev/kfd /dev/dri

# 2. Check user groups
groups | grep -E "video|render"

# 3. Check ROCm detection
podman run --rm --device /dev/kfd --device /dev/dri \
  ollama/ollama:rocm rocminfo | grep gfx

# 4. If gfx1151 not recognized, try community image
podman-compose -f docker-compose.community.yml up -d
```

#### SELinux blocking GPU access (Fedora/RHEL)

```bash
sudo setsebool container_use_devices=1
```

#### Container won't start

```bash
# Check logs
podman logs ollama-amd
podman logs open-webui-amd

# Verify device access
ls -la /dev/kfd /dev/dri/render*
```

#### Open WebUI can't connect to Ollama

```bash
# Verify Ollama is running
curl http://localhost:11434/api/tags

# Check network
podman network inspect ollama-amd_default
```

#### Performance is slow (CPU fallback)

```bash
# Check if GPU is actually being used
watch -n 1 cat /sys/class/drm/card0/device/gpu_busy_percent

# If 0%, GPU is not being used - try community image
podman-compose down
podman-compose -f docker-compose.community.yml up -d
```

---

### References

- [Ollama Docker Documentation](https://docs.ollama.com/docker)
- [Ollama GPU Hardware Support](https://docs.ollama.com/gpu)
- [AMD ROCm Installation Guide](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/)
- [rjmalagon/ollama-linux-amd-apu (Community Image)](https://github.com/rjmalagon/ollama-linux-amd-apu)
- [gdkrmr/ollama-rocm-docker-gfx1151](https://github.com/gdkrmr/ollama-rocm-docker-gfx1151)
- [GMKtec EVO-X2 Product Page](https://www.gmktec.com/products/amd-ryzen-ai-max-395-evo-x2-ai-mini-pc)

---

---

## 中文說明

### 概述

使用 **AMD ROCm GPU 加速** 部署 Ollama，搭配 Open WebUI 進行聊天和模型管理。專為 AMD APU 系統設計，如 **GMKtec EVO-X2**（AMD Ryzen AI Max+ 395 / Radeon 8060S）。

### 目標硬體

| 組件 | 規格 |
|------|------|
| 迷你電腦 | GMKtec EVO-X2 AI Mini PC |
| CPU/APU | AMD Ryzen AI Max+ 395（16核32線程，Zen 5，最高5.1GHz）|
| 內建 GPU | AMD Radeon 8060S（gfx1151，RDNA 3.5）|
| 記憶體 | 64GB / 96GB / 128GB LPDDR5X 8000MHz（統一記憶體）|
| NPU | AMD XDNA 2（50 TOPS）|

> **關鍵優勢**：Ryzen AI Max+ 395 使用**統一記憶體** — GPU 共享系統 RAM，128GB 配置可提供最多 ~96GB 可用 VRAM，可在本地運行 **70B+ 參數模型**。

### 服務

| 服務 | 端口 | 說明 |
|------|------|------|
| Ollama | 11434 | LLM API 服務（AMD ROCm GPU）|
| Open WebUI | 13000 | 聊天介面 + 模型管理 |

---

### 前置需求

#### 1. 容器運行時

```bash
podman --version  # 或 docker --version
```

#### 2. AMD GPU 驅動

```bash
# 確認 AMD GPU 被偵測
lspci | grep -i vga

# 檢查 ROCm（如已安裝 rocminfo）
rocminfo | grep gfx
# 應顯示：gfx1151
```

#### 3. 安裝 ROCm（Ubuntu/Debian）

```bash
sudo mkdir --parents --mode=0755 /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/latest jammy main" | \
  sudo tee /etc/apt/sources.list.d/rocm.list

sudo apt-get update
sudo apt-get install -y rocm-libs rocm-dev
```

#### 4. 使用者權限

```bash
# 將使用者加入 video 和 render 群組
sudo usermod -a -G video,render $USER
# 需要重新登入
```

#### 5. 驗證 GPU 存取

```bash
ls -la /dev/kfd /dev/dri
```

---

### 部署

#### 方案 A：官方 Ollama ROCm 映像（先嘗試）

```bash
cd ollama-amd
podman-compose up -d
```

#### 方案 B：社群映像（如 GPU 未偵測到）

如果官方 `ollama/ollama:rocm` 映像未正確偵測 Radeon 8060S（靜默回退到 CPU），使用社群映像：

```bash
cd ollama-amd
podman-compose -f docker-compose.community.yml up -d
```

> 社群映像來自 [rjmalagon/ollama-linux-amd-apu](https://github.com/rjmalagon/ollama-linux-amd-apu)，包含最新 ROCm 支援。

### 驗證 GPU 使用

```bash
# 檢查 Ollama 日誌中的 GPU 偵測
podman logs ollama-amd 2>&1 | grep -i "gpu\|rocm\|hip\|gfx"

# 監控 GPU 使用率
watch -n 1 cat /sys/class/drm/card0/device/gpu_busy_percent
```

### 訪問

- **Web 介面**: http://localhost:13000
- **API**: http://localhost:11434

### 停止

```bash
podman-compose down
```

---

### 模型管理

#### 透過 Open WebUI (http://localhost:13000)

1. 打開 http://localhost:13000
2. 點擊側邊欄的 **Settings**（齒輪圖標）
3. 進入 **Admin Settings** > **Models**
4. 輸入模型名稱（例如 `qwen2.5:7b`）
5. 點擊 **Pull** 等待下載完成

#### 透過 CLI

```bash
podman exec ollama-amd ollama pull qwen2.5:7b
podman exec ollama-amd ollama list
podman exec -it ollama-amd ollama run qwen2.5:7b
```

#### 推薦模型（依記憶體配置）

##### 128GB 記憶體（~96GB 可用 VRAM）

| 模型 | 大小 | 用途 |
|------|------|------|
| `llama3.1:70b` | ~40GB | 頂級推理 |
| `qwen2.5:72b` | ~41GB | 最佳中英文 |
| `deepseek-r1:70b` | ~42GB | 進階推理 |
| `mixtral:8x22b` | ~80GB | 最大混合專家模型 |

##### 96GB 記憶體（~72GB 可用 VRAM）

| 模型 | 大小 | 用途 |
|------|------|------|
| `llama3.1:70b` | ~40GB | 頂級推理 |
| `qwen2.5:72b` | ~41GB | 最佳中英文 |
| `command-r:35b` | ~20GB | 檢索增強生成 |

##### 64GB 記憶體（~48GB 可用 VRAM）

| 模型 | 大小 | 用途 |
|------|------|------|
| `qwen2.5:32b` | ~20GB | 高品質中英文 |
| `deepseek-coder:33b` | ~19GB | 進階程式碼生成 |
| `mixtral:8x7b` | ~26GB | 混合專家模型 |

---

### 故障排除

#### GPU 未偵測到 / 回退到 CPU

```bash
# 驗證裝置節點
ls -la /dev/kfd /dev/dri

# 檢查使用者群組
groups | grep -E "video|render"

# 如 gfx1151 未被識別，使用社群映像
podman-compose -f docker-compose.community.yml up -d
```

#### 效能緩慢（CPU 回退）

```bash
# 檢查 GPU 是否實際使用中
watch -n 1 cat /sys/class/drm/card0/device/gpu_busy_percent

# 如果為 0%，GPU 未使用 — 嘗試社群映像
```

---

### 參考連結

- [Ollama Docker 文檔](https://docs.ollama.com/docker)
- [Ollama GPU 硬體支援](https://docs.ollama.com/gpu)
- [AMD ROCm 安裝指南](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/)
- [rjmalagon/ollama-linux-amd-apu（社群映像）](https://github.com/rjmalagon/ollama-linux-amd-apu)
- [GMKtec EVO-X2 產品頁面](https://www.gmktec.com/products/amd-ryzen-ai-max-395-evo-x2-ai-mini-pc)
