# Проверки

Уровни проверены по порядку. Полные команды и вывод — по ссылкам ниже. HTTP-запросы выполнялись через Python urllib; в логах указаны эквивалентные команды curl.

**4 — один контейнер.** Образ собран через `docker build -t dz333-counter:local .`, контейнер запущен на порту 8081 с `GREETING='Hello from Artem!'`. GET вернул приветствие и count = 0, POST `/increment` — 1, повторный GET — 1. Логи просмотрены через `docker logs dz333-level4-check`. [Команды и вывод](logs/level4.txt).

**6 — SQLite.** `compose.sqlite.yaml`, порт 8082, приветствие `SQLite check`. Три POST увеличили счётчик с 0 до 3. После `down` и повторного `up -d` GET вернул 3: база сохранилась в именованном volume, контейнер пересоздан. [Команды и вывод](logs/level6.txt).

**8 — Redis.** `compose.yaml`, порт 8083, приветствие `Redis check`. Оба healthcheck прошли, наружу опубликован только порт приложения. `CONFIG GET appendonly` вернул `yes`.

- Три POST `/increment`: счётчик 0 → 3, N = 3.
- После `down` и `up -d --wait --wait-timeout 90`: GET `/` вернул 200 и count = 3.
- После `stop redis`: GET `/healthz` — 200; GET `/readyz` и POST `/increment` — 503. GET `/` тоже вернул 503.
- После `start redis` и ожидания готовности: GET `/` — 200 и count = 3, следующий POST — 200 и count = 4.
- ID контейнера приложения до и после остановки Redis совпал: приложение не перезапускалось.

Все операции Redis выполнялись с префиксом `docker compose -p dz333-redis-check -f compose.yaml`, переменными `APP_PORT=8083` и `GREETING='Redis check'`. [Полные команды и ответы](logs/level8.txt).

После проверок проекты остановлены через `down` без `-v`. Volumes сохранены, поэтому повторный запуск тех же проектов продолжит счётчик.
