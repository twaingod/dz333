# ДЗ 3 — Docker

Счётчик с Redis. Исходники в `app/` не изменены. Нужны Docker с Compose и curl. Команды выполнять из корня репозитория при запущенном Docker.

```bash
cp .env.example .env
docker compose up -d --build --wait --wait-timeout 90
curl --max-time 5 -fsS http://127.0.0.1:8080/
curl --max-time 5 -fsS -X POST http://127.0.0.1:8080/increment
docker compose ps
docker compose logs --no-color
```

В `.env`: `APP_PORT` — внешний порт (по умолчанию 8080), `GREETING` — приветствие. После изменения выполнить `docker compose up -d`; при смене порта заменить его и в curl.

В `compose.yaml` два сервиса: `app` и `redis`. У приложения `STORAGE=redis`, `REDIS_URL=redis://redis:6379/0`, внутренний порт 8000. Наружу открыт только порт приложения. Redis сохраняет данные в volume `redis_data`, подключённом к `/data`, запись AOF включена через `--appendonly yes`.

Healthcheck: Redis — `redis-cli ping`, приложение — `/readyz`. Приложение запускается после готовности Redis.

Промежуточный вариант с SQLite для уровня 6:

```bash
APP_PORT=8082 GREETING='SQLite check' docker compose -p dz333-sqlite -f compose.sqlite.yaml up -d --build
```

SQLite хранит базу в volume `sqlite_data` в `/data`. После запуска дождаться успешного `/readyz` на порту 8082.

Остановка основного проекта с сохранением данных:

```bash
docker compose down
```

Не добавлять `-v`, если данные нужны. Проверки уровней 4, 6 и 8, команды и логи — в [REPORT.md](REPORT.md).
