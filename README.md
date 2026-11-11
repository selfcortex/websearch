# websearch

Self-hosted метапоиск на [SearXNG](https://github.com/searxng/searxng), разворачивается
одним `docker compose up`. Отдаёт результаты в JSON — удобно использовать как
поисковый бэкенд для LLM-приложений (Open WebUI, LibreChat, n8n, Dify и т.п.).

## Возможности
- агрегация результатов по нескольким поисковикам без трекеров;
- JSON API (`/search?q=...&format=json`);
- вся конфигурация в файлах, ничего не настраивается руками в UI.

## Требования
- Docker + Docker Compose v2

## Быстрый старт
```bash
git clone git@github.com:selfcortex/websearch.git
cd websearch
cp .env.example .env      # заполнить своими значениями
docker compose up -d
```
Интерфейс: http://localhost:8080

## Конфигурация
| Файл | Назначение |
|---|---|
| `docker-compose.yml` | сервисы, порты, тома |
| `searxng-settings.yml` | движки, лимиты, форматы ответа |
| `.env` | секреты и локальные переменные (не в git) |

JSON-формат включается в `searxng-settings.yml`:
```yaml
search:
  formats:
    - html
    - json
```

## Пример запроса к API
```bash
curl "http://localhost:8080/search?q=test&format=json"
```

## Полезные команды
```bash
docker compose logs -f      # логи
docker compose restart      # перезапуск после правки настроек
docker compose pull && docker compose up -d   # обновление
docker compose down         # остановить
```
