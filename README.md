# SearXNG MCP Server

Локальная установка SearXNG с MCP сервером для web поиска.

## Структура

- `docker-compose.yml` — определение сервисов SearXNG и MCP сервера
- `searxng/settings.yml` — настройки SearXNG (движки, таймауты)
- `Dockerfile.mcp-searxng` — Dockerfile для сборки MCP сервера
- `vendor/mcp-searxng/` — git submodule с исходным кодом MCP сервера (read-only)
- `.githooks/` — git hooks для защиты submodule от изменений
- `ADR.md` — Architecture Decision Records

## Установка

1. Клонировать репозиторий с submodule:
```bash
git clone --recurse-submodules git@gitverse.ru:stray/searxng.git
```

Если уже склонировано без submodule:
```bash
git submodule update --init --recursive
# если submodule не подтянулся (пустая папка), клонируйте напрямую:
rm -rf vendor/mcp-searxng && git clone --depth 1 https://github.com/ihor-sokoliuk/mcp-searxng.git vendor/mcp-searxng
```

2. Настроить git hooks (защита от изменений в submodule):
```bash
git config core.hooksPath .githooks
```

3. (Опционально) Установить секретный ключ для SearXNG:
```bash
export SEARXNG_SECRET_KEY="$(openssl rand -hex 32)"
```

4. Запустить сервисы:
```bash
podman-compose up -d --build
```

5. Настроить Claude (`.claude.json`):

**Для HTTP transport:**
```json
{
  "mcpServers": {
    "searxng-search": {
      "transport": {
        "type": "http",
        "url": "http://localhost:13000/mcp"
      }
    }
  }
}
```

## Использование

После запуска:
- SearXNG web UI: http://localhost:18088
- MCP HTTP endpoint: http://localhost:13000/mcp
- Health check: http://localhost:13000/health

## Безопасность

- MCP сервер запускается с **read-only** root filesystem
- Submodule `vendor/mcp-searxng` защищён от изменений git hooks
- SearXNG secret_key передаётся через environment variable
- Healthcheck проверяет готовность SearXNG перед запуском MCP

## Обновление MCP сервера

```bash
git submodule update --remote vendor/mcp-searxng
podman-compose build mcp-searxng
podman-compose up -d mcp-searxng
```
