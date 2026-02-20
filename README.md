# Podman/Docker Apps

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Self-contained Docker/Podman Compose projects for quick deployment.

### Projects

| Directory | Description | Ports |
|-----------|-------------|-------|
| [ollama/](./ollama/) | Ollama LLM with NVIDIA GPU + Open WebUI | 11434, 13000 |
| [emqx/](./emqx/) | EMQX MQTT Broker | 1883, 18083 |

### Quick Start

```bash
# Start Ollama + Open WebUI
cd ollama && podman-compose up -d

# Start EMQX
cd emqx && podman-compose up -d
```

### Prerequisites

- Podman 4.x or Docker 20.x+
- For Ollama GPU: NVIDIA Container Toolkit + CDI configured

---

## 中文說明

### 概述

獨立的 Docker/Podman Compose 專案，可快速部署。

### 專案

| 目錄 | 說明 | 端口 |
|------|------|------|
| [ollama/](./ollama/) | Ollama LLM (支援 NVIDIA GPU) + Open WebUI | 11434, 13000 |
| [emqx/](./emqx/) | EMQX MQTT 訊息代理 | 1883, 18083 |

### 快速開始

```bash
# 啟動 Ollama + Open WebUI
cd ollama && podman-compose up -d

# 啟動 EMQX
cd emqx && podman-compose up -d
```

### 前置需求

- Podman 4.x 或 Docker 20.x+
- Ollama GPU: 需要 NVIDIA Container Toolkit + CDI 配置
