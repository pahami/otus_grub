# Домашняя работа 9

## Тема: Загрузка системы_Работа с загрузчиком
---
## Домашнее задание:
  1. Включить отображение меню Grub.
  2. Попасть в систему без пароля несколькими способами.
  3. Установить систему с LVM, после чего переименовать VG.
---

## Выполнение задания:

  Требуется предварительно установленный и работоспособный Oracle VirtualBox. Все дальнейшие действия были проверены при использовании Ubuntu 24.04 в качестве хостовой ОС VirtualBox v7.0.18  и гостевой системе Ubuntu 22.04. Серьёзные отступления от этой конфигурации могут потребовать адаптации с вашей стороны.

  - В файле README.md описание действий для выполнения задания. Выполнение задания зафиксировано в файлах PDF.


### 1. **[Включить отображение меню Grub]**
```
Для отображения меню нужно отредактировать конфигурационный файл:

```
nano /etc/default/grub
```

<details>
<summary>
  Комментируем строку, скрывающую меню и ставим задержку для выбора пункта меню в 10 секунд.
</summary>
```
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
```
</details>

Обновляем конфигурацию загрузчика и перезагружаемся для проверки.

```
update-grub
reboot
```
При загрузке в окне виртуальной машины мы должны увидеть меню загрузчика.
```
### 2. **[Попасть в систему без пароля несколькими способами]**
### 3. **[Попасть в систему без пароля несколькими способами]**











Обновляем конфигурацию загрузчика и перезагружаемся для проверки.
Смотрим список всех дисков, которые есть в виртуальной машине: `lsblk`

<details>
<summary>
   результат выполнения команд
</summary>
   
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  512M  0 disk 
sdb      8:16   0  512M  0 disk 
sdc      8:32   0  512M  0 disk 
sdd      8:48   0  512M  0 disk 
sde      8:64   0  512M  0 disk 
sdf      8:80   0  512M  0 disk 
sdg      8:96   0  512M  0 disk 
sdh      8:112  0  512M  0 disk 
sdi      8:128  0   40G  0 disk 
└─sdi1   8:129  0   40G  0 part /
```
</details>

Создадим 4 пула из двух дисков в режиме RAID 1:

```
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sda
```

- Команда `zpool status` показывает информацию о каждом диске, состоянии сканирования и об ошибках чтения, записи и совпадения хэш-сумм. Команда `zpool list` показывает информацию о размере пула, количеству занятого и свободного места, дедупликации и т.д. 

Проверяем с помощью `zpool list` и `zpool tatus`:

<details>
<summary>
   результат выполнения команд
</summary>

`zpool list`
```
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M   100K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
```
`zpool status`
```
pool: otus1
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus1       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus2       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdd     ONLINE       0     0     0
            sde     ONLINE       0     0     0

errors: No known data errors

  pool: otus3
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus3       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdf     ONLINE       0     0     0
            sdg     ONLINE       0     0     0

errors: No known data errors

  pool: otus4
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus4       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sda     ONLINE       0     0     0
            sdh     ONLINE       0     0     0

errors: No known data errors
```
</details>

Добавим разные алгоритмы сжатия в каждую файловую систему (lzjb, lz4, gzip, zle):

```
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zle otus4
```

Проверим, что все файловые системы имеют разные методы сжатия:

`zfs get all | grep compression`

<details>
<summary> результат выполнения команды: </summary>

```
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local
```
</details>

Скачаем один и тот же текстовый файл во все пулы: 

```
for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```

Проверим, что файл был скачан во все пулы:

`ls -l /otus*`

<details>
<summary> результат выполнения команды: </summary>

```
/otus1:
total 22092
-rw-r--r--. 1 root root 41107603 Dec  2 08:56 pg2600.converter.log

/otus2:
total 18004
-rw-r--r--. 1 root root 41107603 Dec  2 08:56 pg2600.converter.log

/otus3:
total 10965
-rw-r--r--. 1 root root 41107603 Dec  2 08:56 pg2600.converter.log

/otus4:
total 40173
-rw-r--r--. 1 root root 41107603 Dec  2 08:56 pg2600.converter.log
```
</details>

Проверим, сколько места занимает один и тот же файл в разных пулах и проверим степень сжатия файлов:

`zfs list`

<details>
<summary> результат выполнения команды: </summary>

```
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.7M   330M     21.6M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.3M   313M     39.3M  /otus4
```
Алгоритм gzip-9 самый эффективный по сжатию. 
</details>

---
### 3. Определение настроек пула

Скачиваем архив в домашний каталог:

```
wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
```
Разархивируем его:

```
tar -xzvf archive.tar.gz
```
Проверим, возможно ли импортировать данный каталог в пул:

`zpool import -d zpoolexport/`

<details>
<summary> результат выполнения команды: </summary>

```
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE
```
</details>

Данный вывод показывает нам имя пула, тип raid и его состав. 
Сделаем импорт данного пула к нам в ОС:

```
zpool import -d zpoolexport/ otus
zpool status
```

<details>
<summary> результат выполнения команды: </summary>

```
 pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

  pool: otus1
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus1       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus2       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdd     ONLINE       0     0     0
            sde     ONLINE       0     0     0

errors: No known data errors

  pool: otus3
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus3       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdf     ONLINE       0     0     0
            sdg     ONLINE       0     0     0

errors: No known data errors

  pool: otus4
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus4       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sda     ONLINE       0     0     0
            sdh     ONLINE       0     0     0

errors: No known data errors
```
</details>
- Команда zpool status выдаст нам информацию о составе импортированного пула.
- Если у Вас уже есть пул с именем otus, то можно поменять его имя во время импорта: `zpool import -d zpoolexport/ otus newotus`

Далее нам нужно определить настройки: `zpool get all otus`
Запрос сразу всех параметром файловой системы: `zfs get all otus`

C помощью команды grep можно уточнить конкретный параметр, например:
`zfs get available otus`
`zfs get readonly otus`
`zfs get recordsize otus`
`zfs get compression otus`
`zfs get checksum otus`

<details>
<summary> результат выполнения команды: </summary>

`zpool get all otus`

```
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupditto                     0                              default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.09M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      5861468300515118215            -
otus  autotrim                       off                            default
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local
otus  feature@bookmark_v2            enabled                        local
```
`zfs get available otus`
```
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
```
`zfs get readonly otus`
```
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
```
`zfs get recordsize otus`
```
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
```
`zfs get compression otus`
```
NAME  PROPERTY     VALUE     SOURCE
otus  compression  zle       local
```
`zfs get checksum otus`
```
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```
</details>

---
### 4 Работа со снапшотом, поиск сообщения от преподавателя

Скачаем файл, указанный в задании:

`wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download`

<details>
<summary> результат выполнения команды: </summary>

```
--2025-01-06 16:13:04--  https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 64.233.162.132, 2a00:1450:4010:c05::84
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|64.233.162.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5432736 (5.2M) [application/octet-stream]
Saving to: ‘otus_task2.file’

100%[==========================================================================>] 5,432,736   14.2MB/s   in 0.4s   

2025-01-06 16:13:13 (14.2 MB/s) - ‘otus_task2.file’ saved [5432736/5432736]
[1]+  Done                    wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI
```
</details>

Восстановим файловую систему из снапшота:

`zfs receive otus/test@today < otus_task2.file`

<details>
<summary> результат выполнения команды: </summary>

```
[1]+  Done                    wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI
```
</details>


Далее, ищем в каталоге /otus/test файл с именем “secret_message”

`find /otus/test -name "secret_message"`

<details>
<summary> результат выполнения команды: </summary>

```
/otus/test/task1/file_mess/secret_message
```
</details>

Смотрим содержимое найденного файла:

`cat /otus/test/task1/file_mess/secret_message`

<details>
<summary> результат выполнения команды </summary>
```
https://otus.ru/lessons/linux-hl/
```
</details>

Тут мы видим ссылку на курс OTUS, задание выполнено.