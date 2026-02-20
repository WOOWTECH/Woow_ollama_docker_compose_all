# EMQX MQTT Broker

[中文版](#中文說明) | [English](#english)

---

## English

### Overview

Deploy EMQX - a scalable, distributed MQTT message broker for IoT.

### Quick Start

```bash
# Start
podman-compose up -d

# Verify
podman ps
podman exec emqx emqx ping
```

### Ports

| Port | Protocol | Description |
|------|----------|-------------|
| 1883 | TCP | MQTT |
| 8883 | TCP | MQTT SSL/TLS |
| 8083 | TCP | MQTT WebSocket |
| 8084 | TCP | MQTT WebSocket SSL |
| 18083 | HTTP | Dashboard |

### Dashboard

- URL: `http://localhost:18083`
- Default username: `admin`
- Default password: `public`

### MQTT Test

```bash
# Subscribe (terminal 1)
mosquitto_sub -h localhost -p 1883 -t "test/topic"

# Publish (terminal 2)
mosquitto_pub -h localhost -p 1883 -t "test/topic" -m "Hello MQTT"
```

### Stop

```bash
podman-compose down
```

---

## 中文說明

### 概述

部署 EMQX - 可擴展的分布式 MQTT 訊息代理，適用於 IoT。

### 快速開始

```bash
# 啟動
podman-compose up -d

# 驗證
podman ps
podman exec emqx emqx ping
```

### 端口

| 端口 | 協議 | 說明 |
|------|------|------|
| 1883 | TCP | MQTT |
| 8883 | TCP | MQTT SSL/TLS |
| 8083 | TCP | MQTT WebSocket |
| 8084 | TCP | MQTT WebSocket SSL |
| 18083 | HTTP | 管理面板 |

### 管理面板

- URL: `http://localhost:18083`
- 預設用戶名: `admin`
- 預設密碼: `public`

### MQTT 測試

```bash
# 訂閱 (終端 1)
mosquitto_sub -h localhost -p 1883 -t "test/topic"

# 發布 (終端 2)
mosquitto_pub -h localhost -p 1883 -t "test/topic" -m "Hello MQTT"
```

### 停止

```bash
podman-compose down
```
