# PRD: Ollama Docker/Podman GPU Deployment

## Product Requirements Document

**Version:** 1.0
**Date:** 2026-02-20
**Status:** Implemented

---

## 1. Overview / 概述

### English
This project provides a containerized deployment solution for Ollama with NVIDIA GPU acceleration, designed for local development environments. The solution uses Docker Compose / Podman Compose for easy deployment and management.

### 中文
本專案提供 Ollama 的容器化部署方案，支援 NVIDIA GPU 加速，專為本地開發環境設計。解決方案使用 Docker Compose / Podman Compose 實現簡易部署和管理。

---

## 2. Goals / 目標

### Primary Goals / 主要目標
1. **GPU Acceleration** - Enable NVIDIA GPU usage in containerized Ollama
2. **Reproducible Deployment** - One-command deployment with `docker-compose up -d`
3. **Data Persistence** - Model data persists across container restarts
4. **Documentation** - Bilingual documentation for future reference

### 主要目標（中文）
1. **GPU 加速** - 在容器化的 Ollama 中啟用 NVIDIA GPU
2. **可重現部署** - 使用 `docker-compose up -d` 一鍵部署
3. **資料持久化** - 模型資料在容器重啟後保留
4. **文檔** - 雙語文檔供未來參考

---

## 3. Technical Requirements / 技術需求

### Environment / 環境
| Component | Requirement |
|-----------|-------------|
| OS | Ubuntu/Debian (tested on Ubuntu 24.04 LTS) |
| Container Runtime | Podman 4.x or Docker 20.x+ |
| GPU | NVIDIA GPU with compute capability |
| Driver | NVIDIA Driver 470+ |
| Toolkit | NVIDIA Container Toolkit 1.x |

### Dependencies / 依賴
- `nvidia-container-toolkit` - GPU passthrough to containers
- `podman-compose` or `docker-compose` - Container orchestration
- CDI (Container Device Interface) configuration for Podman

---

## 4. Architecture / 架構

```
┌─────────────────────────────────────────────────────┐
│                    Host System                       │
│  ┌───────────────────────────────────────────────┐  │
│  │              NVIDIA Driver                     │  │
│  │         (580.126.09 / CUDA 13.0)              │  │
│  └───────────────────────────────────────────────┘  │
│                        │                             │
│  ┌───────────────────────────────────────────────┐  │
│  │         NVIDIA Container Toolkit               │  │
│  │              (CDI: nvidia.yaml)                │  │
│  └───────────────────────────────────────────────┘  │
│                        │                             │
│  ┌───────────────────────────────────────────────┐  │
│  │              Podman / Docker                   │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │           Ollama Container              │  │  │
│  │  │  ┌─────────────────────────────────┐    │  │  │
│  │  │  │    ollama/ollama:latest         │    │  │  │
│  │  │  │    - API Port: 11434            │    │  │  │
│  │  │  │    - GPU: nvidia.com/gpu=all    │    │  │  │
│  │  │  │    - Volume: ollama_data        │    │  │  │
│  │  │  └─────────────────────────────────┘    │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 5. File Structure / 檔案結構

```
podman_docker_app/
├── docker-compose.yml      # Container orchestration
├── .env.example            # Environment template
├── README.md               # Bilingual documentation
└── docs/
    └── plans/
        └── prd.md          # This document
```

---

## 6. Deployment Checklist / 部署檢查清單

### Prerequisites / 前置需求
- [ ] NVIDIA driver installed (`nvidia-smi` works)
- [ ] Podman or Docker installed
- [ ] NVIDIA Container Toolkit installed
- [ ] CDI configured (`/etc/cdi/nvidia.yaml` exists)

### Deployment / 部署
- [ ] Clone/copy project files
- [ ] Run `podman-compose up -d` or `docker-compose up -d`
- [ ] Verify container running: `podman ps`
- [ ] Test API: `curl http://localhost:11434/api/tags`
- [ ] Verify GPU: `podman exec ollama nvidia-smi`

### Post-Deployment / 部署後
- [ ] Pull desired model: `podman exec ollama ollama pull <model>`
- [ ] Test model inference

---

## 7. Verified Configuration / 已驗證配置

The following configuration was successfully tested on 2026-02-20:

**Host Environment:**
- OS: Ubuntu 24.04 LTS (Linux 6.17.0-14-generic)
- Podman: 4.9.3
- NVIDIA Driver: 580.126.09
- CUDA: 13.0
- NVIDIA Container Toolkit: 1.18.2

**GPU:**
- Model: NVIDIA GeForce RTX 4050 Laptop GPU
- VRAM: 6141 MiB

**Container:**
- Image: ollama/ollama:latest
- GPU Access: Verified via `nvidia-smi` inside container
- API: Responding on port 11434

---

## 8. AI-Ready Deployment Guide / AI 快速部署指南

For AI assistants to quickly deploy this setup:

```bash
# 1. Ensure NVIDIA Container Toolkit is installed
nvidia-ctk --version || {
  curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
    sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
  curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
  sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
  sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
}

# 2. Deploy Ollama
cd /path/to/podman_docker_app
podman-compose up -d  # or docker-compose up -d

# 3. Verify
podman exec ollama nvidia-smi
curl http://localhost:11434/api/tags
```

---

## 9. Version History / 版本歷史

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-20 | Initial release with GPU support |
