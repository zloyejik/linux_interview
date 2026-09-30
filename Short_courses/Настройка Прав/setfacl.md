Команда `setfacl` в Linux задает или изменяет списки контроля доступа (ACL) для файлов и каталогов. [1]

## Основной синтаксис

`setfacl [опции] [действие] [файл]` [2, 3]

## Популярные опции

- `-m` — изменить или добавить правила ACL.
- `-x` — удалить указанные правила ACL.
- `-b` — удалить все расширенные правила ACL.
- `-R` — применить изменения рекурсивно для всех вложенных файлов и папок.
- `-d` — установить правила по умолчанию (наследуются новыми файлами в каталоге). [2, 3, 4]

## Формат правил (acl_spec)

- `u:имя_пользователя:права` — задать права для конкретного пользователя.
- `g:имя_группы:права` — задать права для конкретной группы.
- `m::права` — установить маску (ограничивает максимальные эффективные права).
- `o::права` — права для остальных (others). [4]

_Права обозначаются стандартно: `r` (чтение), `w` (запись), `x` (выполнение) или `-` (нет прав)._

## Примеры использования

- Выдать пользователю `ivan` права на чтение и запись:  
    `setfacl -m u:ivan:rw- file.txt`
- Рекурсивно добавить права на чтение и выполнение для группы `developers` на каталог:  
    `setfacl -R -m g:developers:r-x /var/www/html`
- Удалить правило для конкретного пользователя `ivan`:  
    `setfacl -x u:ivan file.txt`
- Полностью сбросить все расширенные ACL-права у файла:  
    `setfacl -b file.txt`

Если хотите, я могу рассказать:

- Как посмотреть текущие права с помощью getfacl
- Что такое маска (mask) и как она влияет на доступ

  

[1] [https://platformv.sbertech.ru](https://platformv.sbertech.ru/docs/public/SLO/9.0-fstec/common/documents/pfstec/administration-guide/setfacl.html)

[2] [https://docs.altlinux.org](https://docs.altlinux.org/ru-RU/alt-server/11.0/html/alt-server/ch65s08.html)

[3] [https://hmarketing.ru](https://hmarketing.ru/blog/server/komanda-setfacl/)

[4] [https://www.kryukov.biz](https://www.kryukov.biz/soderzhanie/sistema-bezopasnosti/posix-acl/setfacl/)