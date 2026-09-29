**`nmcli`** — это мощный инструмент командной строки для управления сетью через NetworkManager. 

Ниже представлена краткая шпаргалка по основным командам. 
*Примечание: для изменения настроек обычно требуются права суперпользователя (`sudo`).*

---

### 📌 Базовый синтаксис
`nmcli [объект] [команда] [параметры]`
**Главные объекты:**
*   `general` (или `g`) — общий статус NetworkManager.
*   `device` (или `d`) — физические и виртуальные сетевые интерфейсы.
*   `connection` (или `c`) — профили (конфигурации) подключений.
*   `radio` — управление радиомодулями (Wi-Fi, WWAN).

---

### 1. Просмотр статуса и устройств
*   **Общий статус NetworkManager:**
    `nmcli general status`
*   **Список всех сетевых интерфейсов и их статус:**
    `nmcli device status`
*   **Подробная информация по конкретному интерфейсу** (например, `eth0` или `wlan0`):
    `nmcli device show eth0`

---

### 2. Управление подключениями (профилями)
*   **Список всех сохраненных профилей подключений:**
    `nmcli connection show`
*   **Активировать (подключить) профиль:**
    `nmcli connection up "Имя_профиля"`
*   **Деактивировать (отключить) профиль:**
    `nmcli connection down "Имя_профиля"`
*   *Альтернатива:* Включить/выключить интерфейс напрямую:
    `nmcli device connect eth0` / `nmcli device disconnect eth0`

---

### 3. Работа с Wi-Fi
*   **Включить / Выключить Wi-Fi адаптер:**
    `nmcli radio wifi on` / `nmcli radio wifi off`
*   **Поиск доступных Wi-Fi сетей:**
    `nmcli device wifi list`
*   **Подключиться к Wi-Fi сети:**
    `nmcli device wifi connect "Имя_Сети" password "Ваш_Пароль"`
    *(Если сеть открытая, параметр `password` можно опустить).*

---

### 4. Создание новых подключений
*   **Ethernet (получение IP по DHCP):**
    `sudo nmcli connection add type ethernet con-name "Мой_ETH" ifname eth0`
*   **Ethernet (статический IP):**
    `sudo nmcli connection add type ethernet con-name "Мой_Static" ifname eth0 ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns "8.8.8.8 8.8.4.4" ipv4.method manual`
*   **Wi-Fi (клиент):**
    `sudo nmcli connection add type wifi con-name "Мой_WiFi" ssid "Имя_Сети" ifname wlan0 wifi-sec.key-mgmt wpa-psk wifi-sec.psk "Ваш_Пароль"`

---

### 5. Редактирование подключений
Вы можете изменять параметры существующих профилей без входа в интерактивный режим.
*   **Изменить IP-адрес на статический:**
    `sudo nmcli connection modify "Имя_профиля" ipv4.addresses 192.168.1.50/24 ipv4.method manual`
*   **Сделать подключение автоматическим (DHCP):**
    `sudo nmcli connection modify "Имя_профиля" ipv4.method auto`
*   **Установить автоподключение при загрузке системы:**
    `sudo nmcli connection modify "Имя_профиля" connection.autoconnect yes`
*   **Интерактивный редактор** (удобно для сложных настроек):
    `nmcli connection edit "Имя_профиля"`

---

### 💡 Полезные советы
1. **Сокращения:** Вы можете использовать сокращения объектов. Например, `nmcli c` вместо `nmcli connection`, `nmcli d` вместо `nmcli device`.
2. **Автодополнение:** Нажмите `Tab` в терминале, чтобы автодополнить имена профилей или интерфейсов.
3. **Справка:** Если забыли команду, используйте `nmcli help` или `nmcli connection help`.
4. **Применение изменений:** После изменения профиля через `modify`, не забудьте применить его: `nmcli connection up "Имя_профиля"`.