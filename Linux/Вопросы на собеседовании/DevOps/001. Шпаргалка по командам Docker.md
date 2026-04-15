# 🐳 Шпаргалка по командам Docker (CLI)

> 💡 Современный стандарт: вместо `docker-compose` используйте встроенный `docker compose`. Команды `docker <subcommand>` и `docker <object> <action>` полностью взаимозаменяемы.

---

## 🔍 Общие и диагностика
```bash
docker version          # Версия клиента и демона
docker info             # Системная информация, лимиты, драйверы
docker --help           # Справка по подкомандам
```

---

## 📦 Образы (Images)
```bash
docker pull <image>                     # Скачать образ
docker pull <image>:<tag>               # Скачать конкретную версию
docker images                           # Список локальных образов
docker image ls                         # То же самое (современный синтаксис)
docker build -t <name>:<tag> .          # Собрать образ из Dockerfile в текущей папке
docker tag <image> <repo>:<tag>         # Добавить тег для публикации
docker push <repo>:<tag>                # Отправить в registry
docker rmi <image>                      # Удалить образ
docker rmi -f <image>                   # Принудительное удаление
docker history <image>                  # История слоёв образа
docker inspect <image>                  # Подробные метаданные образа
docker image prune                      # Удалить "висячие" (dangling) образы
```

---

## 📦 Контейнеры (Containers)
```bash
docker ps                               # Запущенные контейнеры
docker ps -a                            # Все контейнеры (включая остановленные)
docker run <image>                      # Запустить контейнер
docker run -d --name <name> <image>     # Фоновый запуск с именем
docker run -it --rm <image> bash        # Интерактивный терминал, автоудаление после выхода
docker run -p 8080:80 <image>           # Проброс порта: хост:контейнер
docker run -e VAR=value <image>         # Переменные окружения
docker start/stop/restart <container>   # Управление состоянием
docker pause/unpause <container>        # Заморозка/разморозка процессов
docker rm <container>                   # Удалить остановленный контейнер
docker rm -f <container>                # Принудительное удаление (даже если запущен)
docker logs <container>                 # Вывести логи
docker logs -f --tail 100 <container>   # Следить за логами в реальном времени
docker exec -it <container> bash        # Зайти в запущенный контейнер
docker exec -it <container> sh          # Если bash нет (Alpine и др.)
docker cp <src> <container>:/path       # Копировать файл/папку в контейнер
docker cp <container>:/path <dest>      # Копировать из контейнера
docker stats                            # Мониторинг CPU, RAM, сети всех контейнеров
docker inspect <container>              # JSON с полной конфигурацией и IP
docker rename <old> <new>               # Переименовать контейнер
docker top <container>                  # Процессы внутри контейнера
docker diff <container>                 # Изменения в файловой системе контейнера
```

---

## 🌐 Сети (Networks)
```bash
docker network ls                       # Список сетей
docker network create <name>            # Создать пользовательскую сеть
docker network connect <net> <cont>     # Подключить контейнер к сети
docker network disconnect <net> <cont>  # Отключить контейнер от сети
docker network rm <name>                # Удалить сеть
docker network inspect <name>           # Подробности сети и подключённые контейнеры
```

---

## 💾 Тома (Volumes)
```bash
docker volume ls                        # Список томов
docker volume create <name>             # Создать именованный том
docker volume rm <name>                 # Удалить том
docker volume inspect <name>            # Путь на хосте и метаданные
docker volume prune                     # Удалить неиспользуемые тома

# Примеры монтирования:
docker run -v myvol:/data <image>       # Именованный том (рекомендуется)
docker run -v /host/path:/container/path <image>  # Bind mount
docker run --mount type=volume,src=myvol,target=/data <image>  # Современный синтаксис
```

---

## 🧹 Очистка системы
```bash
docker system df                        # Занимаемое место (образы, контейнеры, тома, кэш)
docker container prune                  # Удалить все остановленные контейнеры
docker image prune                      # Удалить dangling-образы
docker image prune -a                   # Удалить ВСЕ неиспользуемые образы
docker volume prune                     # Удалить неиспользуемые тома
docker network prune                    # Удалить неиспользуемые сети
docker system prune                     # Очистить контейнеры, сети, dangling-образы, кэш сборки
docker system prune -a --volumes        # Жёсткая очистка: всё, включая неиспользуемые образы и тома
```

---

## 🐙 Docker Compose (v2, встроен в CLI)
```bash
docker compose up -d                    # Запустить сервисы фоном
docker compose up --build -d            # Пересобрать и запустить
docker compose down                     # Остановить и удалить контейнеры/сети
docker compose down --volumes           # + удалить тома
docker compose down --rmi all           # + удалить образы
docker compose ps                       # Статус сервисов
docker compose logs -f <service>        # Логи конкретного сервиса
docker compose exec <service> bash      # Зайти в сервис
docker compose build                    # Собрать образы без запуска
docker compose pull                     # Скачать последние версии образов
docker compose restart <service>        # Перезапустить сервис
docker compose config                   # Проверить и вывести итоговый YAML
```

---

## 🔑 Полезные флаги `docker run`
| Флаг | Описание |
|------|----------|
| `-d` | Фоновый режим (detached) |
| `-it` | Интерактивный терминал + псевдо-TTY |
| `--rm` | Автоматически удалить контейнер после выхода |
| `--name <n>` | Задать имя контейнера |
| `-p <host>:<cont>` | Проброс порта |
| `-e <KEY>=<VAL>` | Переменная окружения |
| `-v <src>:<dst>` | Монтирование тома/папки |
| `--network <net>` | Подключить к сети |
| `--restart unless-stopped` | Автозапуск после ребута |
| `--memory=512m` / `--cpus=1.5` | Лимиты ресурсов |
| `--health-cmd` / `--health-interval` | Проверка работоспособности |

---

## 💡 Советы
1. **Всегда используйте именованные тома** (`-v name:/path`) вместо bind mount для данных БД и персистентных данных.
2. **Не храните секреты в образах**. Используйте `--env-file`, Docker Secrets или внешние vault.
3. **Ограничивайте ресурсы**: `-m 256m --cpus=0.5` предотвратят "захват" хоста одним контейнером.
4. **Healthcheck** в Dockerfile или `--health-cmd` помогает оркестраторам и Compose понимать, готов ли сервис.
5. **Compose v2** работает как `docker compose`, а не `docker-compose`. Флаг `-f docker-compose.yml` можно опустить, если файл называется `compose.yaml` или `docker-compose.yml` в текущей папке.

---
📌 Сохраните файл как `docker-cheatsheet.md` и используйте `Ctrl+F` для быстрого поиска.  
Нужна шпаргалка в формате PDF, Markdown для Obsidian/Notion или с примерами `Dockerfile`/`docker-compose.yml`? Напишите, подготовлю.