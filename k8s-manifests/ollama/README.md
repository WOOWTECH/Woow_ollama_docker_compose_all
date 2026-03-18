# Ollama K3s/Kubernetes 部署指南

[English](#english) | [中文](#中文)

---

## English

### Overview

Local LLM (Large Language Model) inference server that makes it easy to run open-source models like Llama 3.1, DeepSeek R1, Qwen 2.5, Mistral, Gemma, and more. Ollama provides an OpenAI-compatible API and handles model downloading, quantization, and GPU acceleration. This deployment supports optional NVIDIA GPU passthrough and is configured for use by other cluster services such as Open WebUI and AnythingLLM.

> **GitHub Repo (Podman/Docker):** [Woow_ollama_docker_compose_all](https://github.com/WOOWTECH/Woow_ollama_docker_compose_all)

### Architecture

```
                     ┌─────────────────────────────────────────────┐
                     │              External Access                │
                     │         http://<node-ip>:31434              │
                     └────────────────┬────────────────────────────┘
                                      │
                                      ▼
                     ┌─────────────────────────────────────────────┐
                     │          NodePort Service                   │
                     │     ollama (31434 → 11434)                  │
                     └────────────────┬────────────────────────────┘
                                      │
                                      ▼
                     ┌─────────────────────────────────────────────┐
                     │          ClusterIP Service                  │
                     │  ollama.ollama.svc.cluster.local:11434      │
                     └────────────────┬────────────────────────────┘
                                      │
                                      ▼
                     ┌─────────────────────────────────────────────┐
                     │        StatefulSet: ollama                  │
                     │     ┌───────────────────────────┐           │
                     │     │    ollama/ollama:latest    │           │
                     │     │    Port: 11434             │           │
                     │     │    GPU: optional (NVIDIA)  │           │
                     │     └─────────┬─────────────────┘           │
                     │               │                             │
                     │     ┌─────────▼─────────────────┐           │
                     │     │  PVC: ollama-data (50Gi)   │           │
                     │     │  /root/.ollama             │           │
                     │     │  (models, config, cache)   │           │
                     │     └───────────────────────────┘           │
                     └─────────────────────────────────────────────┘
                                      │
                       ┌──────────────┴──────────────┐
                       ▼                             ▼
              ┌─────────────────┐          ┌─────────────────┐
              │   Open WebUI    │          │  AnythingLLM    │
              │ (cluster client)│          │ (cluster client)│
              └─────────────────┘          └─────────────────┘
```

### Features

- OpenAI-compatible REST API (`/v1/chat/completions`)
- Native Ollama API (`/api/generate`, `/api/tags`)
- Optional NVIDIA GPU acceleration via GPU Operator
- Persistent model storage across pod restarts
- Configurable parallel inference slots and loaded model limits
- Cluster-internal DNS for seamless integration with other services

### Quick Start

```bash
# 1. Deploy Ollama
kubectl apply -k k8s-manifests/ollama/

# 2. Verify pods are running
kubectl -n ollama get pods

# 3. Pull your first model
kubectl -n ollama exec -it sts/ollama -- ollama pull llama3.1:8b

# 4. Test the model
kubectl -n ollama exec -it sts/ollama -- ollama run llama3.1:8b "Hello, world!"
```

### Configuration

#### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `OLLAMA_HOST` | Bind address | `0.0.0.0` | Yes |
| `OLLAMA_MODELS` | Model storage path | `/root/.ollama/models` | No |
| `OLLAMA_NUM_PARALLEL` | Number of parallel model inference slots | `2` | No |
| `OLLAMA_MAX_LOADED_MODELS` | Maximum models loaded simultaneously in memory | `1` | No |

#### GPU Support (Optional)

To enable NVIDIA GPU acceleration:

1. Install the [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html) on your k3s cluster
2. Uncomment the following lines in `ollama-statefulset.yaml`:

```yaml
# runtimeClassName: nvidia

resources:
  limits:
    # nvidia.com/gpu: "1"
```

3. Re-apply:

```bash
kubectl apply -k k8s-manifests/ollama/
```

### Accessing the Service

| Endpoint | URL | Protocol |
|----------|-----|----------|
| Ollama API (external) | `http://<node-ip>:31434` | HTTP (NodePort) |
| Ollama API (cluster) | `http://ollama.ollama.svc.cluster.local:11434` | HTTP (ClusterIP) |

#### API Examples

```bash
# List models
curl http://<node-ip>:31434/api/tags

# Generate a response
curl http://<node-ip>:31434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "Explain Kubernetes in one paragraph"
}'

# OpenAI-compatible chat endpoint
curl http://<node-ip>:31434/v1/chat/completions -d '{
  "model": "llama3.1:8b",
  "messages": [{"role": "user", "content": "Hello!"}]
}'
```

#### Pulling Models

```bash
# Popular models
kubectl -n ollama exec sts/ollama -- ollama pull llama3.1:8b      # 4.7GB
kubectl -n ollama exec sts/ollama -- ollama pull llama3.1:70b     # 40GB
kubectl -n ollama exec sts/ollama -- ollama pull deepseek-r1:8b   # 4.9GB
kubectl -n ollama exec sts/ollama -- ollama pull qwen2.5:7b       # 4.7GB
kubectl -n ollama exec sts/ollama -- ollama pull mistral:7b       # 4.1GB
kubectl -n ollama exec sts/ollama -- ollama pull gemma2:9b        # 5.5GB
```

### Data Persistence

| PVC Name | Mount Path | Size | Purpose |
|----------|------------|------|---------|
| `ollama-data` | `/root/.ollama` | 50Gi | Downloaded models, configuration, model cache |

The PVC uses the `local-path` storage class (k3s default). Models can be large (7B models are approximately 4GB, 70B models approximately 40GB), so ensure adequate disk space.

### Backup & Restore

#### Backup

```bash
# List downloaded models
kubectl -n ollama exec sts/ollama -- ollama list

# Backup model data
kubectl -n ollama exec sts/ollama -- tar czf /tmp/ollama-backup.tar.gz /root/.ollama
kubectl -n ollama cp ollama/ollama-0:/tmp/ollama-backup.tar.gz ./ollama-backup.tar.gz
```

#### Restore

```bash
# Restore model data
kubectl -n ollama cp ./ollama-backup.tar.gz ollama/ollama-0:/tmp/ollama-backup.tar.gz
kubectl -n ollama exec sts/ollama -- tar xzf /tmp/ollama-backup.tar.gz -C /

# Restart Ollama
kubectl -n ollama rollout restart sts/ollama
```

**Alternative:** Since models are downloadable, you can skip backup and simply re-pull models after a fresh deployment.

### Useful Commands

```bash
# Check pod status
kubectl -n ollama get pods

# View real-time logs
kubectl -n ollama logs sts/ollama -f

# List downloaded models
kubectl -n ollama exec sts/ollama -- ollama list

# Show running models and GPU usage
kubectl -n ollama exec sts/ollama -- ollama ps

# Interactive chat with a model
kubectl -n ollama exec -it sts/ollama -- ollama run llama3.1:8b

# Delete a model
kubectl -n ollama exec sts/ollama -- ollama rm <model-name>

# Check disk usage
kubectl -n ollama exec sts/ollama -- df -h /root/.ollama

# Restart the service
kubectl -n ollama rollout restart sts/ollama
```

### Troubleshooting

#### Ollama pod OOMKilled

LLMs require significant RAM. The default limit is 8Gi. For larger models (13B+), increase the limit in `ollama-statefulset.yaml`:

```yaml
resources:
  limits:
    memory: "16Gi"
```

#### Model download fails or is slow

```bash
# Check available disk space
kubectl -n ollama exec sts/ollama -- df -h /root/.ollama

# Check download progress in logs
kubectl -n ollama logs sts/ollama -f
```

#### GPU not detected

```bash
# Verify GPU operator is installed
kubectl get pods -n gpu-operator

# Check if Ollama detects the GPU
kubectl -n ollama exec sts/ollama -- ollama ps
kubectl -n ollama logs sts/ollama | grep -i gpu
```

#### Other services cannot reach Ollama

Verify the ClusterIP service is working:

```bash
kubectl -n ollama get svc ollama
# Should show: ollama ClusterIP ... 11434/TCP

# Test from another namespace
kubectl -n open-webui exec deploy/open-webui -- wget -qO- http://ollama.ollama.svc.cluster.local:11434
```

#### Slow inference on CPU

CPU inference is expected to be slower than GPU. Tips to improve:
- Use smaller quantized models (e.g., `llama3.1:8b` instead of `llama3.1:70b`)
- Increase `OLLAMA_NUM_PARALLEL` if you have sufficient RAM
- Set `OLLAMA_MAX_LOADED_MODELS: "1"` to avoid memory contention

### File Structure

```
ollama/
├── kustomization.yaml          # Kustomize orchestration
├── namespace.yaml              # Namespace: ollama
├── ollama-statefulset.yaml     # StatefulSet with GPU support
├── ollama-service.yaml         # ClusterIP + NodePort (31434)
├── pvc.yaml                    # PVC: ollama-data (50Gi)
└── README.md                   # This file
```

---

## 中文

### 概述

本地 LLM（大型語言模型）推理伺服器，可輕鬆運行 Llama 3.1、DeepSeek R1、Qwen 2.5、Mistral、Gemma 等開源模型。Ollama 提供 OpenAI 相容 API，並處理模型下載、量化及 GPU 加速。本部署支援可選的 NVIDIA GPU 直通，並已設定供叢集內其他服務（如 Open WebUI 和 AnythingLLM）使用。

> **GitHub 儲存庫 (Podman/Docker)：** [Woow_ollama_docker_compose_all](https://github.com/WOOWTECH/Woow_ollama_docker_compose_all)

### 架構

```
                     ┌─────────────────────────────────────────────┐
                     │              外部存取                       │
                     │         http://<node-ip>:31434              │
                     └────────────────┬────────────────────────────┘
                                      │
                                      ▼
                     ┌─────────────────────────────────────────────┐
                     │          NodePort 服務                      │
                     │     ollama (31434 → 11434)                  │
                     └────────────────┬────────────────────────────┘
                                      │
                                      ▼
                     ┌─────────────────────────────────────────────┐
                     │          ClusterIP 服務                     │
                     │  ollama.ollama.svc.cluster.local:11434      │
                     └────────────────┬────────────────────────────┘
                                      │
                                      ▼
                     ┌─────────────────────────────────────────────┐
                     │        StatefulSet: ollama                  │
                     │     ┌───────────────────────────┐           │
                     │     │    ollama/ollama:latest    │           │
                     │     │    連接埠: 11434           │           │
                     │     │    GPU: 可選 (NVIDIA)      │           │
                     │     └─────────┬─────────────────┘           │
                     │               │                             │
                     │     ┌─────────▼─────────────────┐           │
                     │     │  PVC: ollama-data (50Gi)   │           │
                     │     │  /root/.ollama             │           │
                     │     │  (模型、設定、快取)         │           │
                     │     └───────────────────────────┘           │
                     └─────────────────────────────────────────────┘
                                      │
                       ┌──────────────┴──────────────┐
                       ▼                             ▼
              ┌─────────────────┐          ┌─────────────────┐
              │   Open WebUI    │          │  AnythingLLM    │
              │   (叢集用戶端)   │          │  (叢集用戶端)   │
              └─────────────────┘          └─────────────────┘
```

### 功能特色

- OpenAI 相容 REST API（`/v1/chat/completions`）
- 原生 Ollama API（`/api/generate`、`/api/tags`）
- 透過 GPU Operator 可選啟用 NVIDIA GPU 加速
- 跨 Pod 重啟的持久化模型儲存
- 可設定平行推理插槽及已載入模型數量上限
- 叢集內部 DNS，與其他服務無縫整合

### 快速開始

```bash
# 1. 部署 Ollama
kubectl apply -k k8s-manifests/ollama/

# 2. 確認 Pod 正在運行
kubectl -n ollama get pods

# 3. 拉取第一個模型
kubectl -n ollama exec -it sts/ollama -- ollama pull llama3.1:8b

# 4. 測試模型
kubectl -n ollama exec -it sts/ollama -- ollama run llama3.1:8b "Hello, world!"
```

### 設定

#### 環境變數

| 變數 | 說明 | 預設值 | 必要 |
|------|------|--------|------|
| `OLLAMA_HOST` | 綁定位址 | `0.0.0.0` | 是 |
| `OLLAMA_MODELS` | 模型儲存路徑 | `/root/.ollama/models` | 否 |
| `OLLAMA_NUM_PARALLEL` | 平行模型推理插槽數 | `2` | 否 |
| `OLLAMA_MAX_LOADED_MODELS` | 同時載入記憶體的最大模型數 | `1` | 否 |

#### GPU 支援（可選）

啟用 NVIDIA GPU 加速：

1. 在 k3s 叢集上安裝 [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html)
2. 取消 `ollama-statefulset.yaml` 中以下行的註解：

```yaml
# runtimeClassName: nvidia

resources:
  limits:
    # nvidia.com/gpu: "1"
```

3. 重新套用：

```bash
kubectl apply -k k8s-manifests/ollama/
```

### 存取服務

| 端點 | URL | 協定 |
|------|-----|------|
| Ollama API（外部） | `http://<node-ip>:31434` | HTTP (NodePort) |
| Ollama API（叢集內部） | `http://ollama.ollama.svc.cluster.local:11434` | HTTP (ClusterIP) |

#### API 範例

```bash
# 列出模型
curl http://<node-ip>:31434/api/tags

# 產生回應
curl http://<node-ip>:31434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "用一段話解釋 Kubernetes"
}'

# OpenAI 相容聊天端點
curl http://<node-ip>:31434/v1/chat/completions -d '{
  "model": "llama3.1:8b",
  "messages": [{"role": "user", "content": "你好！"}]
}'
```

#### 拉取模型

```bash
# 熱門模型
kubectl -n ollama exec sts/ollama -- ollama pull llama3.1:8b      # 4.7GB
kubectl -n ollama exec sts/ollama -- ollama pull llama3.1:70b     # 40GB
kubectl -n ollama exec sts/ollama -- ollama pull deepseek-r1:8b   # 4.9GB
kubectl -n ollama exec sts/ollama -- ollama pull qwen2.5:7b       # 4.7GB
kubectl -n ollama exec sts/ollama -- ollama pull mistral:7b       # 4.1GB
kubectl -n ollama exec sts/ollama -- ollama pull gemma2:9b        # 5.5GB
```

### 資料持久化

| PVC 名稱 | 掛載路徑 | 大小 | 用途 |
|-----------|----------|------|------|
| `ollama-data` | `/root/.ollama` | 50Gi | 已下載模型、設定檔、模型快取 |

PVC 使用 `local-path` 儲存類別（k3s 預設）。模型檔案可能很大（7B 模型約 4GB，70B 模型約 40GB），請確保有足夠的磁碟空間。

### 備份與還原

#### 備份

```bash
# 列出已下載的模型
kubectl -n ollama exec sts/ollama -- ollama list

# 備份模型資料
kubectl -n ollama exec sts/ollama -- tar czf /tmp/ollama-backup.tar.gz /root/.ollama
kubectl -n ollama cp ollama/ollama-0:/tmp/ollama-backup.tar.gz ./ollama-backup.tar.gz
```

#### 還原

```bash
# 還原模型資料
kubectl -n ollama cp ./ollama-backup.tar.gz ollama/ollama-0:/tmp/ollama-backup.tar.gz
kubectl -n ollama exec sts/ollama -- tar xzf /tmp/ollama-backup.tar.gz -C /

# 重新啟動 Ollama
kubectl -n ollama rollout restart sts/ollama
```

**替代方案：** 由於模型可從網路下載，您可以跳過備份，在全新部署後重新拉取模型即可。

### 實用指令

```bash
# 檢查 Pod 狀態
kubectl -n ollama get pods

# 即時檢視日誌
kubectl -n ollama logs sts/ollama -f

# 列出已下載模型
kubectl -n ollama exec sts/ollama -- ollama list

# 顯示運行中的模型及 GPU 使用情況
kubectl -n ollama exec sts/ollama -- ollama ps

# 與模型互動聊天
kubectl -n ollama exec -it sts/ollama -- ollama run llama3.1:8b

# 刪除模型
kubectl -n ollama exec sts/ollama -- ollama rm <model-name>

# 檢查磁碟使用量
kubectl -n ollama exec sts/ollama -- df -h /root/.ollama

# 重新啟動服務
kubectl -n ollama rollout restart sts/ollama
```

### 疑難排解

#### Ollama Pod 被 OOMKilled

LLM 需要大量記憶體。預設限制為 8Gi。對於較大的模型（13B+），請在 `ollama-statefulset.yaml` 中增加限制：

```yaml
resources:
  limits:
    memory: "16Gi"
```

#### 模型下載失敗或速度緩慢

```bash
# 檢查可用磁碟空間
kubectl -n ollama exec sts/ollama -- df -h /root/.ollama

# 在日誌中查看下載進度
kubectl -n ollama logs sts/ollama -f
```

#### 未偵測到 GPU

```bash
# 確認 GPU Operator 已安裝
kubectl get pods -n gpu-operator

# 檢查 Ollama 是否偵測到 GPU
kubectl -n ollama exec sts/ollama -- ollama ps
kubectl -n ollama logs sts/ollama | grep -i gpu
```

#### 其他服務無法連線至 Ollama

確認 ClusterIP 服務正常運作：

```bash
kubectl -n ollama get svc ollama
# 應顯示: ollama ClusterIP ... 11434/TCP

# 從其他命名空間測試
kubectl -n open-webui exec deploy/open-webui -- wget -qO- http://ollama.ollama.svc.cluster.local:11434
```

#### CPU 推理速度緩慢

CPU 推理預期會比 GPU 慢。改善建議：
- 使用較小的量化模型（例如 `llama3.1:8b` 而非 `llama3.1:70b`）
- 若有足夠記憶體，可增加 `OLLAMA_NUM_PARALLEL`
- 設定 `OLLAMA_MAX_LOADED_MODELS: "1"` 以避免記憶體競爭

### 檔案結構

```
ollama/
├── kustomization.yaml          # Kustomize 編排檔
├── namespace.yaml              # 命名空間: ollama
├── ollama-statefulset.yaml     # StatefulSet（含 GPU 支援）
├── ollama-service.yaml         # ClusterIP + NodePort (31434)
├── pvc.yaml                    # PVC: ollama-data (50Gi)
└── README.md                   # 本檔案
```
