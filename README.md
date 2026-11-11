# websearch

Self-hosted поисковый стек для LLM/MCP-агентов: [SearXNG](https://github.com/searxng/searxng)
(метапоиск с JSON API) + [Crawl4AI](https://github.com/unclecode/crawl4ai)
(извлечение содержимого страниц в markdown). Поднимается одной командой.

Оба сервиса слушают **только 127.0.0.1** — наружу не торчат, доступ через reverse proxy
или SSH-туннель.

## Состав

| Сервис | Адрес | Описание |
|---|---|---|
| SearXNG | http://localhost:8081 | агрегация Google, Bing, DuckDuckGo, Startpage, Qwant + GitHub, StackOverflow |
| Crawl4AI | http://localhost:11235 | рендеринг страниц в браузере и конвертация в markdown для LLM |

Контейнеры в общей сети `search-net`, лимит памяти crawl4ai — 3 ГБ, `shm_size` 1 ГБ.

## Требования
- Docker + Docker Compose v2
- ~3–4 ГБ свободной RAM (crawl4ai тянет headless-браузер)

## Быстрый старт
```bash
git clone git@github.com:selfcortex/websearch.git
cd websearch
cp .env.example .env
```
Заполни `.env`:
```
SEARXNG_SECRET=<openssl rand -hex 32>
CRAWL4AI_API_TOKEN=<любая длинная случайная строка>
```
Запуск:
```bash
docker compose up -d
```
SearXNG: http://localhost:8081 · Crawl4AI: http://localhost:11235

## Использование

Поиск в JSON (формат уже включён в `searxng-settings.yml`):
```bash
curl "http://localhost:8081/search?q=searxng+api&format=json"
```

Извлечение страницы через Crawl4AI:
```bash
curl -X POST http://localhost:11235/crawl \
  -H "Authorization: Bearer $CRAWL4AI_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"urls":["https://example.com"],"crawler_config":{"type":"CrawlerRunConfig","params":{"stream":false}}}'
```

Типовой пайплайн агента: `SearXNG /search` → список URL → `Crawl4AI /crawl` → markdown в контекст LLM.

## Конфигурация
| Файл | Назначение |
|---|---|
| `docker-compose.yml` | сервисы, порты, сеть, лимиты |
| `searxng-settings.yml` | движки, таймауты, JSON-формат, UI |
| `.env` | секреты (не в git) |

Секрет SearXNG задаётся переменной `SEARXNG_SECRET` и перекрывает `secret_key` из YAML —
в файле лежит намеренная заглушка.

## Полезные команды
```bash
docker compose logs -f searxng
docker compose restart searxng    # после правки settings.yml
docker compose pull && docker compose up -d
docker compose down
```

## Заметки
- Образы закреплены на `:latest` — для воспроизводимости лучше прибить конкретные теги.
- Redis в `settings.yml` указан (`redis://redis:6379/0`), но сервиса redis в compose нет:
  limiter и кэш автокомплита работать не будут. Либо добавь контейнер redis, либо убери секцию.
