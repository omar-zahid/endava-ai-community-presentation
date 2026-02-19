## Docker Deployment (Traefik + Open WebUI + Ollama GPU)

This setup keeps Ollama containerized and internal-only, so Open WebUI talks to Ollama over Docker networking (`http://ollama:11434`) without `host.docker.internal` or UFW routing hacks.

### Prerequisites

1. NVIDIA driver is installed on the VM and works:

```bash
nvidia-smi
```

2. Docker Engine + Compose plugin are installed.

3. NVIDIA Container Toolkit is installed and configured (via Ansible role `nvidia_container_toolkit`).

4. Verify GPU access from containers:

```bash
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

### One-time setup

From repo root:

```bash
touch docker/traefik/data/acme.json
chmod 600 docker/traefik/data/acme.json
docker network create proxy
```

If the `proxy` network already exists, the `docker network create proxy` command can be ignored.

### Bring the stack up

1. Start Traefik first:

```bash
docker compose -f docker/traefik/docker-compose.yml up -d
```

2. Start Open WebUI + Ollama:

```bash
docker compose -f docker/open-webui/docker-compose.yml up -d
```

### Verify

```bash
docker compose -f docker/open-webui/docker-compose.yml ps
docker logs ollama --tail 100
docker exec ollama ollama list
```

Then open your Open WebUI host (the Traefik host configured in labels) and run a prompt.

### Pull a model (first time)

```bash
docker exec -it ollama ollama pull qwen3-coder:30b
docker exec -it ollama ollama run qwen3-coder:30b
```

### Restart and upgrade flow

```bash
docker compose -f docker/open-webui/docker-compose.yml pull
docker compose -f docker/open-webui/docker-compose.yml up -d
docker compose -f docker/traefik/docker-compose.yml pull
docker compose -f docker/traefik/docker-compose.yml up -d
```

### Troubleshooting

- If container GPU access fails, re-check toolkit config:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

- If Open WebUI cannot see Ollama, inspect internal DNS and networking:

```bash
docker exec open-webui getent hosts ollama
docker logs open-webui --tail 100
```

- Keep port `11434` private. Only expose `80/443` via Traefik.
