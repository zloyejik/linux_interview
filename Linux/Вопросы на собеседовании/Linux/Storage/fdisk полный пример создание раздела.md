Вот пошаговый пример полного цикла работы: **создание раздела → форматирование → получение UUID → монтирование → автозагрузка через `/etc/fstab`**.

> ⚠️ **Важно:** В примере используется диск `/dev/sdb`. Убедитесь, что на нём **нет важных данных**. Для системных дисков (`/dev/sda`, `/dev/nvme0n1`) меняйте имена аккуратно или используйте Live-USB.

---
### 📋 Шаг 1. Проверка целевого диска
```bash
lsblk
```
Убедитесь, что `/dev/sdb` не смонтирован и не содержит нужных данных:
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   50G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
└─sda2   8:2    0   49G  0 part /
sdb      8:16   0   20G  0 disk          ← наш диск
```

---
### 🔧 Шаг 2. Создание раздела через `fdisk`
```bash
sudo fdisk /dev/sdb
```
В интерактивном режиме вводим последовательно:
```
Command (m for help): n      # создать новый раздел
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p        # основной раздел
Partition number (1-4, default 1): 1
First sector (2048-41943039, default 2048): [Enter]
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943039, default 41943039): +5G

Created a new partition 1 of type 'Linux filesystem' and of size 5 GiB.

Command (m for help): w      # записать изменения и выйти
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

> 💡 Если ядро не подхватило изменения автоматически, выполните:
> ```bash
> sudo partprobe /dev/sdb
> ```

Проверим:
```bash
lsblk /dev/sdb
```
```
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sdb      8:16   0  20G  0 disk 
└─sdb1   8:17   0   5G  0 part 
```

---
### 📦 Шаг 3. Форматирование раздела
Создаём файловую систему `ext4`:
```bash
sudo mkfs.ext4 /dev/sdb1
```
Вывод покажет параметры ФС (размер блока, inode, суперблоки и т.д.).

---
### 🔍 Шаг 4. Получение UUID через `blkid`
```bash
sudo blkid -s UUID -o value /dev/sdb1
```
Пример вывода:
```
a1b2c3d4-e5f6-7890-abcd-ef1234567890
```
Скопируйте этот UUID – он понадобится для `/etc/fstab`.

---
### 📂 Шаг 5. Монтирование
Создаём точку монтирования и монтируем временно:
```bash
sudo mkdir -p /mnt/mydata
sudo mount /dev/sdb1 /mnt/mydata
```
Проверка:
```bash
df -h /mnt/mydata
lsblk -f | grep sdb1
```

---
### 📜 Шаг 6. Настройка автозагрузки (`/etc/fstab`)
Откройте файл:
```bash
sudo nano /etc/fstab
```
Добавьте строку в конец файла (подставьте ваш UUID):
```
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /mnt/mydata  ext4  defaults,noatime  0  2
```
**Разбор полей:**
| Поле | Значение |
|------|----------|
| `UUID=...` | Уникальный идентификатор раздела |
| `/mnt/mydata` | Точка монтирования |
| `ext4` | Тип файловой системы |
| `defaults,noatime` | Параметры монтирования |
| `0` | Не делать резервную копию `dump` |
| `2` | Порядок проверки `fsck` (1 – для `/`, 2 – для остальных) |

> ✅ **Обязательно проверьте корректность перед перезагрузкой:**
> ```bash
> sudo umount /mnt/mydata      # отмонтируем временное монтирование
> sudo mount -a                # смонтировать всё из fstab
> df -h /mnt/mydata            # убедиться, что раздел доступен
> ```

---
### 🎯 Итоговая проверка
```bash
lsblk -f
```
Вывод должен показать структуру, тип ФС и UUID:
```
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sdb                                                         
└─sdb1 ext4   1.0         a1b2c3d4-e5f6-7890-abcd-ef1234567890    4.5G     1% /mnt/mydata
```

Теперь раздел будет автоматически монтироваться при каждой загрузке системы. Если нужно настроить права доступа, квоты, RAID или LVM поверх этого раздела – напишите, дополню пример.