# Local AI Server Setup (Ubuntu / Linux Mint)

## Overview

```text
Ubuntu / Linux Mint Server
│
├── Ollama (Native systemd service)
│     ├── Runs the LLM
│     ├── REST API (127.0.0.1:11434)
│     └── Private to the host
│
├── Docker
│     └── Open WebUI
│           ├── Host networking
│           ├── Accessible on 0.0.0.0:8080
│           └── Connects to http://127.0.0.1:11434
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

In another terminal, run this:

``` bash
curl http://localhost:11434/api/generate \
-d '{
  "model":"qwen2.5:7b",
  "prompt":"Say hello."
}'
```

## 6. Install Open WebUI

Run:

``` bash
docker run -d \
  --name open-webui \
  --network host \
  --restart unless-stopped \
  -e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main
```

Once it finishes downloading, and starts, give it about 30-40 seconds and the Webui will become available.

## 7. Access the UI

Open:

    http://YOUR_SERVER_IP:8080

Create your admin account.


## 8. Configure Open WebUI

After creating your administrator account:

1. Open **Admin Panel → Settings → Connections**.
2. Click **Add Connection**.
3. Choose **Ollama**.
4. Configure:

```
Name: Local Ollama
URL:  http://127.0.0.1:11434
```

5. Save the connection.
6. Verify that your model (for example `qwen2.5:7b`) appears in the model selector.

## Troubleshooting

If Open WebUI reports **No model selected**:

```bash
systemctl status ollama
ollama list
docker exec open-webui curl http://127.0.0.1:11434/api/tags
```

If the last command returns your models, the networking is working. Ensure the Ollama connection above exists in Open WebUI.


## 9. Python API (OPTIONAL)

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

## Security

Keep Ollama bound to `127.0.0.1:11434`.

Expose only Open WebUI on `0.0.0.0:8080` via host networking.

## 10. Optional: Allow LAN Access (Not Recommended)

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

