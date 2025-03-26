
Подключаем репозиторий Postgresql

```
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

Производим установку 15 версии.

```
sudo apt update
sudo apt install postgresql-15
```

Проверяем, что кластер запущен.

```
sudo -u postgres pg_lsclusters

Ver Cluster Port Status Owner    Data directory              Log file
15  main    5433 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```

Заходим в psql

```
sudo -u postgres psql
```

Проверяем работу базы, создав в ней таблицу и заполнив ее данными.

```
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=# \q
```

Останавливаем кластер
```
sudo systemctl stop postgresql@15-main
```

Проверяем, что он остановился

```
sudo -u postgres pg_lsclusters

Ver Cluster Port Status Owner    Data directory              Log file
15  main    5433 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```

Подключаем диск 10GB, проверяем его доступность.

```
lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  73.9M  1 loop /snap/core22/1748
loop1                       7:1    0 144.5M  1 loop /snap/docker/3064
loop2                       7:2    0  44.4M  1 loop /snap/snapd/23771
sda                         8:0    0    25G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0    23G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0  11.5G  0 lvm  /
sdb                         8:16   0    10G  0 disk
sr0                        11:0    1  1024M  0 rom
```

Судя по объему - подключенный диск получил имя sdb.

Устанавливаем parted.
```sudo apt install parted```

Устанавливем тип разметки диска как GPT.
```sudo parted /dev/sdb mklabel gpt```

Создаем раздел на диске.
```sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%```

Форматируем его, установив метку - datalabel.
```sudo mkfs.ext4 -L datalabel /dev/sdb1```

Создаем папку для монтирования.
```sudo mkdir -p /mnt/data```

Добавляем новый диск в fstab.
```
sudo nano /etc/fstab

LABEL=datalabel /mnt/data ext4 defaults 0 2
```

Применяем установленные настройки
```
sudo mount -a
sudo systemctl daemon-reload
```

Комадой lsblk - убеждаемся, что диск смонтирован.
Перезагружаем через reboot - диск по прежнему смонтирован.

```
alexandr@ubuntu-vm:~$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0 144.5M  1 loop /snap/docker/3064
loop1                       7:1    0  73.9M  1 loop /snap/core22/1748
loop2                       7:2    0  44.4M  1 loop /snap/snapd/23771
sda                         8:0    0    25G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0    23G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0  11.5G  0 lvm  /
sdb                         8:16   0    10G  0 disk
└─sdb1                      8:17   0    10G  0 part /mnt/data
sr0                        11:0    1  1024M  0 rom
```

Останавливаем postgres.
```
sudo service postgresql stop
```

Даем права пользователю postgres, затем переносим данные в новую папку.
```
sudo chown -R postgres:postgres /mnt/data/
```

Кластер по понятным причинам не запустился.
```
sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner     Data directory              Log file
15  main    5433 down   <unknown> /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```


Поменял в конфигурации строки с расположением:
```
data_directory = '/mnt/data/15/main'            # use data in another directory
```

Запустил кластер:
```
sudo service postgresql start
sudo -u postgres pg_ctlcluster 15 main start
```

Проверяем данные в БД.

```
sudo -u postgres psql
psql (17.4 (Ubuntu 17.4-1.pgdg24.04+2), server 15.12 (Ubuntu 15.12-1.pgdg24.04+1))
Type "help" for help.

postgres=# select * from test;
 c1
----
 1
(1 row)
 ```

## Со *

Создаем новый инстанс VM, устанавливаем на него Postgress, монтируем диск.
Далее - нам нужно остановить postgres, создать директорию для монтирования.

```
sudo service postgresql stop
sudo chown -R postgres:postgres /mnt/dataNew/
```

Затем поменять конфигурацию.

```
sudo nano /etc/postgresql/15/main/postgresql.conf

data_directory = '/mnt/dataNew/15/main'
```

Осталось только запустить сервер.
```
sudo service postgresql start
sudo -u postgres pg_ctlcluster 15 main start
```

И проверить.
```
sudo -u postgres psql
psql (17.4 (Ubuntu 17.4-1.pgdg24.04+2), server 15.12 (Ubuntu 15.12-1.pgdg24.04+1))
Type "help" for help.

postgres=# select * from test;
 c1
----
 1
(1 row)
```