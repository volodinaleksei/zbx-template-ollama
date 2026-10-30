# Zabbix шаблон: Ollama API

Мониторинг [Ollama](https://ollama.com) через Zabbix 7.x: два HTTP raw-ITEM'а + dependent-считалки.

## Эндпоинты

| Master | Эндпоинт | Что даёт |
|---|---|---|
| `ollama.models.available` | `/v1/models` (5m) | список **установленных** моделей |
| `ollama.ps.raw` | `/api/ps` (1m) | **загруженные** модели: name, size, size_vram, expires_at |

## ITEM'ы

- `ollama.models.count` / `ollama.models.list` — установлено (число / список через запятую)
- `ollama.models.loaded.count` / `ollama.models.loaded.list` — в памяти
- `ollama.loaded.vram` — сумма `size_vram` загруженных моделей, байт
- `net.tcp.service[http,,{$OLLAMA_PORT}]` — simple check порта

## LLD

Правило `ollama.models.discovery` (по `/api/ps`) создаёт на каждую модель:
`ollama.model.vram[{#MODELID}]`, `ollama.model.ctx[{#MODELID}]`, флаг `inmem`.
Триггер-прототип: «Ollama: модель {#MODELID} выгружена из памяти» (была в памяти <3h, сейчас нет).

## Триггеры

| Событие | Sev | Условие |
|---|---|---|
| Ollama: порт {$OLLAMA_PORT} недоступен | High | TCP-порт закрыт |
| Ollama: API отвечает, но список моделей пуст | Warning | порт живой, `/v1/models` пуст |

## Макросы

| Макрос | По умолчанию |
|---|---|
| `{$OLLAMA_PORT}` | `11434` |

## Установка

1. Import `template_ollama.yaml`.
2. Хост с Agent-interface (нужен только как `{HOST.CONN}`), IP хоста Ollama.
3. Привязать шаблон.
