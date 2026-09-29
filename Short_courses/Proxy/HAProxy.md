# 🚀 Краткая подготовка к собеседованию по HAProxy

---

## 1. Что такое HAProxy
- **High Availability Proxy** — высокопроизводительный TCP/HTTP-балансировщик нагрузки и reverse-proxy.
- Написан на C, однопоточный (event-driven, epoll), крайне низкое потребление ресурсов.
- Работает на **L4 (TCP)** и **L7 (HTTP)**.

---

## 2. Архитектура конфигурации
```
global        → глобальные настройки (процесс, логи, лимиты)
defaults      → параметры по умолчанию для всех секций
frontend      → принимает клиентские соединения
backend       → пул реальных серверов
listen        → frontend + backend в одном блоке (удобно для TCP)
```

---

## 3. Режимы работы
| Режим | Описание |
|-------|----------|
| `mode tcp` | L4, проброс «как есть» (БД, SMTP, SSH) |
| `mode http` | L7, парсит HTTP-заголовки, cookies, URI |

---

## 4. Алгоритмы балансировки
| Алгоритм | Суть |
|----------|------|
| `roundrobin` | По кругу (по умолчанию) |
| `leastconn` | На сервер с наименьшим числом соединений |
| `source` | По хэшу IP клиента (sticky без cookies) |
| `uri` | По хэшу URI (кэширование) |
| `hdr(name)` | По HTTP-заголовку |
| `first` | Заполняет первый сервер до лимита |

---

## 5. Health Checks
```haproxy
backend app
    option httpchk GET /health
    http-check expect status 200
    server s1 10.0.0.1:80 check inter 3s fall 3 rise 2
```
- **inter** — интервал, **fall** — сколько фейлов = down, **rise** — сколько успехов = up.
- TCP-check: `option tcp-check` (для БД).

---

## 6. Sticky Sessions (Affinity)
- **Cookie-based** (L7):
  ```haproxy
  cookie SERVERID insert indirect nocache
  server s1 10.0.0.1:80 cookie s1
  ```
- **Source IP** (L4): `balance source`
- **Stick tables**: `stick-table type ip size 1m expire 30m`

---

## 7. SSL/TLS Termination
```haproxy
frontend https_front
    bind *:443 ssl crt /etc/haproxy/certs/site.pem
    mode http
    default_backend app
```
- HAProxy **терминирует** TLS → бэкенды получают чистый HTTP.
- Поддержка SNI, OCSP stapling, HSTS.

---

## 8. ACL и маршрутизация
```haproxy
acl is_api path_beg /api
acl is_mobile hdr_sub(User-Agent) Mobile
use_backend api_servers if is_api
use_backend mobile_servers if is_mobile
```

---

## 9. Логирование
```haproxy
global
    log /dev/log local0
defaults
    log global
    option httplog       # детальный HTTP-лог
    option dontlognull
```
Формат включает: `client_ip`, `frontend`, `backend/server`, `Tq/Tw/Tc/Tr/Tt` (тайминги), `status`, `bytes`.

---

## 10. Высокая доступность самого HAProxy
- **Keepalived + VRRP**: VIP плавает между master/backup HAProxy.
- **Active-Active**: несколько VIP или DNS round-robin.
- **Runtime API / Socket**: `stats socket /run/haproxy.sock` — hot-reload, управление без рестарта.

---

## 11. Частые вопросы на собеседовании

| Вопрос | Краткий ответ |
|--------|---------------|
| L4 vs L7? | L4 — TCP, не видит HTTP; L7 — парсит заголовки, можно рулить по URI/cookie |
| Graceful reload? | `haproxy -sf $(cat /var/run/haproxy.pid)` — старые коннекты доживают |
| Чем отличается от Nginx? | HAProxy — чистый прокси/балансировщик, лучше в L4 и сложных health checks; Nginx — ещё и веб-сервер |
| Как ограничить rate? | `stick-table` + `http-request track-sc0` + `http-request deny if { sc_http_req_rate(0) gt 100 }` |
| Что такое `option forwardfor`? | Добавляет `X-Forwarded-For` с IP клиента |
| Что такое `option http-server-close`? | Закрывает соединение к бэкенду после ответа, но держит keep-alive с клиентом |

---

## 12. Полезные команды
```bash
haproxy -c -f /etc/haproxy/haproxy.cfg   # проверка конфига
echo "show stat" | socat /run/haproxy.sock stdio  # статистика
echo "show info" | socat ...             # инфо о процессе
```

---

> **Совет:** На собеседовании акцентируйте понимание **разницы L4/L7**, **health checks**, **sticky sessions** и **graceful reload** — это спрашивают чаще всего. Удачи! 🍀