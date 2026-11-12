# The local stack

The point of this build was to keep every layer on one machine: a laptop runs the data stores, the
API, the console and the voice pipeline; the device only streams audio and executes actions.

## What runs where

| Component | Default port | Role |
|---|---|---|
| `manager-web` (Vue) | 8001 | console: devices, models, parameters |
| `manager-api` (Java 21) | 8002 | API for the console and the voice server |
| `xiaozhi-server` (Python) | 8000 | WebSocket + OTA endpoints, ASR → LLM → TTS |
| MySQL 8 (Docker) | 3306 | devices, users, parameters |
| Redis 7 (Docker) | 6379 | cache / session state |

The device connects to `ws://<laptop-lan-ip>:8000/xiaozhi/v1/` and fetches its configuration from
`http://<laptop-lan-ip>:8002/xiaozhi/ota/`.

## Boot order

1. **Docker** — MySQL and Redis first; the API will fail fast without them.
2. **`manager-api`** — wait for the log line that says it started, then check for errors above it.
3. **`manager-web`** — open `http://127.0.0.1:8001`, register the first user (that user is the
   super-admin), and configure a model provider.
4. **`xiaozhi-server`** — after the model and the speech model are in place.
5. **Device** — flash, connect, bind.

Starting them out of order is the most common way to get a confusing error: the API reports a
database problem, the console reports an API problem, and the real cause is the container that was
never started.

## Configuration flow

```text
console (super-admin)
  ├─ parameter server.secret   ──► copy into  data/.config.yaml  (secret: ...)
  ├─ parameter server.websocket ─► ws://<lan-ip>:8000/xiaozhi/v1/
  └─ parameter server.ota       ─► http://<lan-ip>:8002/xiaozhi/ota/
firmware config (Kconfig.projbuild)
  ├─ OTA endpoint
  └─ WebSocket endpoint
device screen
  └─ 6-digit code ──► bind the device in the console
```

`server.secret` is generated fresh on every deployment. Copy it; do not hard-code it in a repo.

## The voice pipeline

1. The device sends audio frames over the WebSocket.
2. `xiaozhi-server` runs speech recognition (SenseVoiceSmall in this build).
3. The transcript goes to the configured language model.
4. The reply is synthesized to speech and streamed back.
5. If the reply contains a device command, the device executes it (for example, setting the OLED
   brightness to 20%).

The round trip was fast enough on consumer hardware to feel like a conversation rather than a query.

## Data stores

```bash
# local only — use your own password, never commit it
docker run --name xiaozhi-db \
  -e MYSQL_ROOT_PASSWORD='***' \
  -p 3306:3306 \
  -e MYSQL_DATABASE=xiaozhi_esp32_server \
  -d mysql:8

docker run --name xiaozhi-redis -d -p 6379:6379 redis:7
```

Two practical notes from the build:

- **Docker's disk location matters.** By default images and volumes live on the system drive; move
  the disk image in Docker Desktop settings before you pull large images.
- **WSL integration** has to be enabled if containers are started from a WSL shell. If the option
  is missing, install a WSL 2 distribution first.

## Model providers

The console can point at a hosted provider or a local model. In this build I exercised:

- Zhipu GLM (the console's default provider entry)
- Qwen (including a local Qwen through an Ollama-compatible endpoint)
- DeepSeek
- Doubao

Switching providers is a console change, not a firmware change — the device only sees the voice
server.

## Pitfalls that cost time

| Symptom | Real cause |
|---|---|
| Console cannot reach the API | API started before MySQL/Redis, or the wrong database password |
| Device shows “connecting” forever | WebSocket endpoint is `127.0.0.1` instead of the laptop's LAN IP |
| OTA request fails | firewall blocks port 8002, or the OTA URL is missing `/xiaozhi/ota/` |
| Voice server starts but no model reply | API key empty or provider disabled in the console |
| `npm install` stalls | registry/network; switch to a mirror and retry |
| Python environment not found | shell was opened before the environment was created; reopen it |

No keys, tokens or personal configuration are stored in this repository.
