# Local AI Server Setup (Ubuntu / Linux Mint)

## Overview

``` text
Ubuntu / Linux Mint Server
│
├── Ollama
│     ├── Runs the LLM
│     └── REST API (:11434)
│
├── Open WebUI
│     └── Browser interface (:3000)
│
└── Your scripts
      ├── Python
      ├── Bash
      └── REST API
```

## 1. Update the server

``` bash
sudo apt update
sudo apt upgrade -y
sudo apt install git curl wget htop -y
```

## 2. Install Docker

``` bash
sudo apt install docker.io docker-compose-v2 -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

Log out and back in, then verify:

``` bash
docker --version
docker compose version
```

## 3. Install Ollama

``` bash
curl -fsSL https://ollama.com/install.sh | sh
```

Check the service:

``` bash
systemctl status ollama
```

## 4. Download a model

Small model:

``` bash
ollama pull qwen2.5:7b
```

Run it:

``` bash
ollama run qwen2.5:7b
```

## 5. Test the API

``` bash
curl http://localhost:11434/api/generate \
-d '{
  "model":"qwen2.5:7b",
  "prompt":"Say hello."
}'
```

## 6. Install Open WebUI

Generate a secret:

``` bash
openssl rand -hex 32
```

Run:

``` bash
docker run -d \
  --name open-webui \
  --restart unless-stopped \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  -e WEBUI_SECRET_KEY=YOUR_SECRET_HERE \
  ghcr.io/open-webui/open-webui:main
```

## 7. Access the UI

Open:

    http://YOUR_SERVER_IP:3000

Create your admin account.

## 8. Python API

``` bash
pip install openai
```

``` python
from openai import OpenAI

client = OpenAI(
    api_key="ollama",
    base_url="http://localhost:11434/v1"
)

response = client.chat.completions.create(
    model="qwen2.5:7b",
    messages=[{"role":"user","content":"Hello"}]
)

print(response.choices[0].message.content)
```

## 9. Optional: Allow LAN Access

Configure the Ollama service with:

    OLLAMA_HOST=0.0.0.0:11434

Restart the service after making the change.

## Recommended Stack

-   Ubuntu Server / Linux Mint
-   Docker
-   Ollama
-   Open WebUI
-   Qwen2.5 7B
-   Python + Pandas/Polars
