# 🐳 Шпаргалка по решению типичных проблем Docker

> ✅ Актуально для Docker Engine v24+ / Docker Compose v2. Команды проверены на Linux, macOS и Windows (WSL2).

---

## 🔍 Быстрая диагностика
| Задача | Команда |
|--------|---------|
| Все контейнеры (включая упавшие) | `docker ps -a` |
| Логи в реальном времени | `docker logs <container> --tail 200 -f` |
| Подробные метаданные | `docker inspect <container\|image\|network>` |
| Потребление CPU/RAM | `docker stats` |
| Использование диска Docker | `docker system df` |
| Проверка YAML Compose | `docker compose config` |

---

## 🚪 Контейнер падает сразу после запуска
**Симптомы:** `Exited (1)` или `Exited (0)` через секунду.  
**Чек-лист:**
1. `docker logs <id>` → ищи traceback, `missing command`, `permission denied`.
2. В образе нет долгоживущего процесса → добавь `tty: true` и `stdin_open: true` (compose) или `docker run -it ...`.
3. Для отладкиEntryPoint: `docker run -it --entrypoint /bin/sh <image>` или `--entrypoint /bin/bash`.
4. В `docker-compose.yml` временно замени команду: `command: ["sleep", "infinity"]`.

---

## 🌐 Сетевые проблемы
| Проблема | Решение |
|----------|---------|
| Не могу достучаться с хоста | Проверь `-p 8080:80`. Сервис внутри должен слушать `0.0.0.0`, а не `127.0.0.1`. |
| Контейнеры не видят друг друга | Используй пользовательскую сеть: `docker network create appnet`. В Compose сеть создаётся автоматически, но имена сервисов = DNS-имена. |
| DNS не резолвится внутри | `docker run --dns 8.8.8.8 ...` или в `/etc/docker/daemon.json`: `"dns": ["8.8.8.8", "1.1.1.1"]` → `sudo systemctl restart docker`. |
| Порт занят на хосте | `sudo lsof -i :8080` или `ss -tulpn | grep 8080`. Смени `-p` или останови конфликтующий сервис. |

---

## 💾 Тома, файлы и права
| Проблема                                     | Решение                                                                                                                                                  |
| -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Файлы не сохраняются после `docker rm`       | Используй `named volumes` или корректный bind mount: `-v /host/path:/container/path`.                                                                    |
| `Permission denied` в примонтированной папке | UID/GID хоста ≠ пользователя в контейнере. Фикс: `docker run -u $(id -u):$(id -g) ...` или `chown -R 1000:1000 /host/path` (зависит от `USER` в образе). |
| Медленный I/O на macOS/Windows               | Bind mount сильно тормозит на файловых прокси. Замени на `named volume` для баз данных, логов, кэшей. На Win используй WSL2 backend.                     |
| Контейнер не видит новые файлы на хосте      | Docker Desktop не следит за изменениями в некоторых каталогах. Перезапусти контейнер или проверь настройки File Sharing.                                 |

---

## 🐳 Сборка и образы
| Проблема | Решение |
|----------|---------|
| `docker build` падает с сетевой ошибкой | Прокси/DNS. В Dockerfile: `ENV http_proxy=...` или настрой Docker Desktop → Settings → Resources → Proxies. |
| Образ слишком большой (>1GB) | Multi-stage build, `alpine`/`slim` базовые образы, `.dockerignore` (`node_modules`, `.git`, `*.log`, `Dockerfile`). |
| `pull` не работает / `unauthorized` | `docker login`, проверь тег (`:latest` может отсутствовать в registry), очисть кэш: `docker image prune`. |
| Кэш билда мешает | `docker build --no-cache -t myapp:latest .` |

---

## 📦 Docker Compose
| Проблема | Решение |
|----------|---------|
| `ERROR: yaml: line X: mapping values are not allowed` | Отступы только пробелами (не табы). Проверь через `docker compose config`. |
| Сервисы стартуют не в том порядке | `depends_on` без healthcheck ждёт только запуск процесса. Используй: ```yaml depends_on: db: condition: service_healthy ``` |
| Переменные окружения не подставляются | Файл должен называться `.env` и лежать рядом с `docker-compose.yml`. В compose: `${DB_PASS:-secret}`. После правок `.env` → `docker compose up -d`. |
| Переопределение настроек для dev/prod | Создай `docker-compose.override.yml` (подхватится автоматически) или запускай с `-f docker-compose.yml -f docker-compose.prod.yml`. |

---

## 💥 Проблемы с демоном / системой
| Проблема | Решение |
|----------|---------|
| Docker не стартует | `sudo systemctl status docker`, `journalctl -u docker -f`. Часто: нет места (`df -h`), битый `/var/lib/docker`, конфликт с `containerd`. |
| Высокое CPU/RAM | `docker stats`. Ограничь: `--memory="512m" --cpus="1.0"` или в Compose: `mem_limit: 512m`, `cpus: 1.0`. |
| `No space left on device` при пулле/билде | `docker system prune -af --volumes` (⚠️ удалит всё неиспользуемое). Увеличь виртуальный диск в Docker Desktop. |
| Контейнер висит в `Created` или `Dead` | `docker rm -f <id>`, затем `docker system prune`. Если не помогает: `sudo systemctl restart docker`. |

---

## 🧹 Очистка и обслуживание
```bash
# Удалить stopped контейнеры
docker container prune -f

# Удалить dangling образы
docker image prune -f

# Удалить неиспользуемые сети
docker network prune -f

# ⚠️ Полная очистка (образы, контейнеры, volumes, сети)
docker system prune -a --volumes -f

# Очистить лог конкретного контейнера (без удаления)
truncate -s 0 $(docker inspect --format='{{.LogPath}}' <container>)
```

---

## 💡 Pro-советы для стабильности
1. **Всегда фиксируй теги:** `nginx:1.25-alpine`, а не `nginx:latest`.
2. **Healthcheck:** добавь в Dockerfile/Compose, чтобы оркестратор знал, когда сервис готов.
3. **Не от root:** укажи `USER appuser` в Dockerfile, избегай `sudo` внутри контейнера.
4. **Read-only FS:** `docker run --read-only --tmpfs /tmp --tmpfs /var/run ...` повышает безопасность.
5. **Логирование:** настрой `dockerd` json-file с ротацией, иначе логи съедят диск:
   ```json
   { "log-driver": "json-file", "log-opts": { "max-size": "10m", "max-file": "3" } }
   ```

---
📌 **Нужно решение под конкретную ошибку?** Приложи вывод `docker logs`, `docker inspect` или фрагмент `docker-compose.yml` → разберём детально.