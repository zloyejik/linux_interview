**`systemd-networkd`** и **`systemd-resolved`** — это нативные, легковесные компоненты `systemd` для управления сетевыми интерфейсами и DNS. Они идеально подходят для серверов, виртуальных машин и контейнеров, но **не управляют Wi-Fi** (для этого нужен `iwd` или `wpa_supplicant`).

Ниже представлена краткая шпаргалка по их настройке.

---

### 📌 Подготовка и запуск
1. **Включите службы:**
   ```bash
   sudo systemctl enable --now systemd-networkd systemd-resolved
   ```
2. **Отключите конфликты:** Если у вас установлен NetworkManager или netplan, их нужно отключить, чтобы они не перехватывали управление сетью:
   ```bash
   sudo systemctl disable --now NetworkManager
   ```

---

### 1. systemd-networkd (Управление сетью)
Конфигурационные файлы хранятся в `/etc/systemd/network/`. Они имеют расширение `.network` (для настроек интерфейса) и `.netdev` (для создания виртуальных устройств, например, мостов или VLAN).

*Важно: Файлы читаются в алфавитном порядке. Используйте префиксы (например, `10-eth0.network`), чтобы управлять приоритетом.*

#### Пример 1: Получение IP по DHCP
Создайте файл `/etc/systemd/network/20-dhcp.network`:
```ini
[Match]
Name=eth0  # Имя интерфейса (или MAC=, или Driver=)

[Network]
DHCP=yes
```

#### Пример 2: Статический IP-адрес
Создайте файл `/etc/systemd/network/20-static.network`:
```ini
[Match]
Name=eth0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=1.1.1.1

# Опционально: IPv6
# Address=2001:db8::100/64
# Gateway=2001:db8::1
```

#### Применение изменений
После создания или изменения файлов, перезагрузите конфигурацию:
```bash
sudo networkctl reload
```
*(Или перезапустите службу: `sudo systemctl restart systemd-networkd`)*

---

### 2. systemd-resolved (Управление DNS)
Отвечает за кэширование DNS и генерацию файла `/etc/resolv.conf`.

#### Глобальная настройка DNS
Отредактируйте файл `/etc/systemd/resolved.conf`:
```ini
[Resolve]
DNS=8.8.8.8 1.1.1.1
FallbackDNS=9.9.9.9 8.8.4.4
#DNSOverTLS=opportunistic  # Включить DNS-over-TLS (опционально)
```
Примените изменения: `sudo systemctl restart systemd-resolved`.

#### Настройка resolv.conf
Чтобы система использовала `systemd-resolved` (локальный кэширующий DNS на `127.0.0.53`), нужно создать символическую ссылку:
```bash
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```
*(Если файл `/etc/resolv.conf` уже существует как обычный файл, удалите его перед созданием ссылки).*

#### Полезные команды (утилита `resolvectl`)
*   **Проверить статус и текущие DNS-серверы:**
    `resolvectl status`
*   **Очистить кэш DNS:**
    `resolvectl flush-caches`
*   **Проверить резолвинг конкретного домена:**
    `resolvectl query example.com`
*   **Посмотреть статистику кэша:**
    `resolvectl statistics`

---

### 3. Управление и диагностика (утилита `networkctl`)
`networkctl` — это аналог `nmcli` для `systemd-networkd`.

*   **Список всех интерфейсов и их статус:**
    `networkctl`
*   **Подробная информация по интерфейсу** (IP, маршруты, DNS, DHCP-аренда):
    `networkctl status eth0`
*   **Перезагрузить конфигурацию** (без разрыва соединений, если возможно):
    `sudo networkctl reload`
*   **Принудительно переподключить интерфейс** (аналог `ifdown/ifup`):
    `sudo networkctl reconfigure eth0`

---

### 💡 Полезные советы
1. **Синтаксис:** Конфиги `systemd-networkd` похожи на `.ini` файлы. Секции `[Match]` определяют, к какому интерфейсу применять настройки, а `[Network]` — сами настройки.
2. **Wi-Fi:** Как упоминалось, `systemd-networkd` не умеет работать с Wi-Fi. Для беспроводных сетей в экосистеме systemd используйте **`iwd`** (iNet Wireless Daemon), который отлично интегрируется с `systemd-networkd` и `systemd-resolved`.
3. **Логи:** Если сеть не поднимается, смотрите логи службы:
   `journalctl -u systemd-networkd -e`