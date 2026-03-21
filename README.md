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

**Ручная установка:**

1. Установить podman
```bash
brew install podman podman-compose
```

1. Клонировать репозиторий с submodule:
```bash
git clone --recurse-submodules git@gitverse.ru:stray/searxng.git
```

Если уже склонировано без submodule:
```bash
git submodule update --init --recursive
```

3. Настроить git hooks:
```bash
git config core.hooksPath .githooks
```

4. Запустить сервисы:
```bash
podman-compose up -d --build
```

5. Добавить в глобальный `~/.claude.json`:
```json
{
  "mcpServers": {
    "searxng-search": {
      "type": "http",
      "url": "http://localhost:13000/mcp"
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
