# Zabbix template: Ollama API

Monitoring for [Ollama](https://ollama.com) on Zabbix 7.x: two HTTP raw items + dependent counters.

## Endpoints

| Master item | Endpoint | Provides |
|---|---|---|
| `ollama.models.available` | `/v1/models` (5m) | list of **installed** models |
| `ollama.ps.raw` | `/api/ps` (1m) | **loaded** models: name, size, size_vram, expires_at |

## Items

- `ollama.models.count` / `ollama.models.list` — installed (count / comma-separated list)
- `ollama.models.loaded.count` / `ollama.models.loaded.list` — loaded into memory
- `ollama.loaded.vram` — sum of `size_vram` across loaded models, bytes
- `net.tcp.service[http,,{$OLLAMA_PORT}]` — simple port check

## LLD

`ollama.models.discovery` (from `/api/ps`) creates per model:
`ollama.model.vram[{#MODELID}]`, `ollama.model.ctx[{#MODELID}]`, and an in-memory flag.
Trigger prototype: "Ollama: model {#MODELID} unloaded from memory" (was loaded <3h ago, gone now).

## Triggers

| Event | Severity | Condition |
|---|---|---|
| Ollama: port {$OLLAMA_PORT} unreachable | High | TCP port closed |
| Ollama: API responds but model list is empty | Warning | port alive, `/v1/models` empty |

## Macros

| Macro | Default |
|---|---|
| `{$OLLAMA_PORT}` | `11434` |

## Setup

1. Import `template_ollama.yaml`.
2. Host with an Agent interface (used only as `{HOST.CONN}`) pointing at the Ollama host.
3. Link the template.
