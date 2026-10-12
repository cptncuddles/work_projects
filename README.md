# Local LLM Stack Deployment & Architecture Guide

## Overview

This document outlines the architecture, step-by-step configuration, directory initialization, model acquisition, performance benchmarking, and LAN exposure procedures used to deploy a local Large Language Model (LLM) stack (`llama.cpp` + `Open WebUI`) with Nginx Proxy Manager.

---

## 1. System Architecture & Component Stack

* **Backend Engine (`llama.cpp`):** Built with Vulkan backend support (`ghcr.io/ggml-org/llama.cpp:server-vulkan`) for hardware acceleration on AMD GPUs (`/dev/dri:/dev/dri`).
* **Frontend UI (`Open WebUI`):** Built using `ghcr.io/open-webui/open-webui:main`, configured to interface directly with `llama.cpp` as an OpenAI-compatible API provider.
* **Reverse Proxy (`Nginx Proxy Manager`):** Routes traffic from port 80/443 to port 3000, supporting WebSockets for real-time token streaming.

---

## 2. Step-by-Step Implementation

### Step 1: Directory Setup & Project Initialization
Create a dedicated project root directory (e.g., `llm-stack`) on your host system along with the necessary subdirectories for model weights and container caching.

```bash
mkdir -p ~/llm-stack/models
mkdir -p ~/llm-stack/cache
cd ~/llm-stack
```

### Step 2: Model Acquisition and Placement
Download your desired `.gguf` model files (e.g., from Hugging Face) and place them directly into the `./models` directory.

* **Single-File Models:** Place the `.gguf` file directly in `./models/`.
* **Multi-Part Sharded Models:** If a model is split into multiple parts (e.g., `model-00001-of-00003.gguf`), ensure **all split shards** are placed together in `./models/`.

Verify the files are correctly positioned on the host filesystem:
```bash
ls -la ./models
```

### Step 3: Configuring Docker Compose for the LLM Stack
Create the `docker-compose.yml` file inside your project root (`~/llm-stack/docker-compose.yml`).

```yaml
services:
  llama:
    image: ghcr.io/ggml-org/llama.cpp:server-vulkan
    container_name: llama
    restart: unless-stopped
    devices:
      - /dev/dri:/dev/dri
    ports:
      - "8080:8080"
    volumes:
      - ./models:/models:Z
      - ./cache:/root/.cache:Z
    command:
      - --model
      - /models/qwen3.5-35b-a3b-q3_K_M.gguf
      - --cache-reuse
      - "256"
      - --flash-attn
      - "on"
      - -b
      - "2048"
      - -ub
      - "1024"
      - --host
      - 0.0.0.0
      - --port
      - "8080"
      - --n-gpu-layers
      - "99"
      - --ctx-size
      - "65536"

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    volumes:
      - open-webui:/app/backend/data
    environment:
      - OPENAI_API_BASE_URL=http://llama:8080/v1
      - OPENAI_API_KEY=llama-cpp
      - ENABLE_OLLAMA_API=False
    depends_on:
      - llama

volumes:
  open-webui:
```

### Step 4: Bringing Up the Stack
Launch the containers in detached mode:
```bash
docker compose up -d
```

### Step 5: Performance Benchmarking and Verification
Performance benchmarks across tested models evaluated context ingestion and generation speeds:
* **Context Processing (Prompt Processing):** Achieved up to $441.99 \text{ tokens/sec}$ depending on quantization level and parameter scale.
* **Inference Generation:** Averaged between $8.29 \text{ tokens/sec}$ and $21.07 \text{ tokens/sec}$ on quantized weights.

---

## 3. LAN Access via FQDN & Reverse Proxy Setup

To make Open WebUI accessible across your local network via a clean FQDN (e.g., `chat.homelab.local`):

1. **Network Configuration:** Set up Nginx Proxy Manager using a custom bridge network (`home_lab`).
2. **Reverse Proxy Rules:**
   * **Forward Host/IP:** Host IP or container target on port `3000`.
   * **WebSockets:** Enabled WebSocket support (`Upgrade` and `Connection` headers) to prevent token streaming timeouts.
   * **Buffering:** Disabled proxy buffering to allow real-time Server-Sent Events (SSE) streaming from `llama.cpp`.

---

## 4. Troubleshooting & Maintenance Reference

* **Viewing Container Logs:**
  ```bash
  docker logs -f llama
  ```
* **Verifying Model Endpoint Directly:**
  ```bash
  curl http://localhost:8080/v1/models
  ```
* **Refreshing Open WebUI Connection:** If models do not appear after boot, go to **Admin Panel > Settings > Connections** and click the **Refresh** icon next to the OpenAI API base URL.
