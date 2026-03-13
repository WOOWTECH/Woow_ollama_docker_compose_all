# Skill: Deploy Ollama + Open WebUI + EMQX

## Overview

This skill enables AI assistants to quickly deploy Ollama (with NVIDIA GPU) + Open WebUI and EMQX MQTT Broker using Docker/Podman Compose.

## Repository

Clone from: `https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git`

## Quick Deploy Checklist

### Step 1: Verify Prerequisites

```bash
# Check container runtime
podman --version || docker --version

# Check NVIDIA driver
nvidia-smi

# Check NVIDIA Container Toolkit
nvidia-ctk --version
```

### Step 2: Install NVIDIA Container Toolkit (if missing)

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

### Step 3: Clone Repository

```bash
git clone https://github.com/WOOWTECH/Woow_ollama_docker_compose_all.git
cd Woow_ollama_docker_compose_all
```

### Step 4: Deploy Ollama + Open WebUI

```bash
cd ollama
podman-compose up -d  # or docker-compose up -d
```

### Step 5: Verify Deployment

```bash
# Check containers
podman ps

# Check GPU access
podman exec ollama nvidia-smi

# Check API
curl http://localhost:11434/api/tags
```

### Step 6: Access Services

- **Open WebUI**: http://localhost:13000
- **Ollama API**: http://localhost:11434

### Step 7: Download a Model

```bash
podman exec ollama ollama pull qwen2.5:7b
```

### Step 8: Deploy EMQX (optional)

```bash
cd ../emqx
podman-compose up -d
```

- **EMQX Dashboard**: http://localhost:18083 (admin/public)

---

## Service Configuration Reference

### Ollama (ollama/docker-compose.yml)

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports: ["11434:11434"]
    volumes: [ollama_data:/root/.ollama]
    devices: [nvidia.com/gpu=all]
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports: ["13000:8080"]
    volumes: [open_webui_data:/app/backend/data]
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - WEBUI_AUTH=false
```

### EMQX (emqx/docker-compose.yml)

```yaml
services:
  emqx:
    image: emqx/emqx:latest
    container_name: emqx
    ports:
      - "1883:1883"    # MQTT TCP
      - "8883:8883"    # MQTT SSL/TLS
      - "8083:8083"    # MQTT WebSocket
      - "8084:8084"    # MQTT WebSocket SSL
      - "18083:18083"  # Dashboard
    volumes:
      - emqx_data:/opt/emqx/data
      - emqx_log:/opt/emqx/log
```

---

## Recommended Models for 6GB VRAM

| Model | Command | Use Case |
|-------|---------|----------|
| qwen2.5:7b | `podman exec ollama ollama pull qwen2.5:7b` | General, Chinese/English |
| llama3.2:3b | `podman exec ollama ollama pull llama3.2:3b` | Fast reasoning |
| deepseek-coder:6.7b | `podman exec ollama ollama pull deepseek-coder:6.7b` | Code generation |
| phi3:mini | `podman exec ollama ollama pull phi3:mini` | Coding |
| gemma2:2b | `podman exec ollama ollama pull gemma2:2b` | Lightweight |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| GPU not detected | `sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml` |
| Container won't start | `podman logs ollama` to check errors |
| Open WebUI can't connect | Verify `curl http://localhost:11434/api/tags` works |
| Port already in use | Change port mapping in docker-compose.yml |

---

## Verified Environment

- OS: Ubuntu 24.04 LTS
- Podman: 4.9.3
- NVIDIA Driver: 580.126.09
- CUDA: 13.0
- NVIDIA Container Toolkit: 1.18.2
- GPU: NVIDIA GeForce RTX 4050 Laptop GPU (6GB VRAM)
