1. Создаем VM для работы с докером, подключаем репозиторий, импортируем ключи, устанавливаем докер.
   ```
   sudo apt update
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
   sudo apt update
   sudo apt install docker-ce
   ```
2. Проверяем, что докер запущен.
   ```
   sudo systemctl status docker
    ● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-03-25 05:35:45 UTC; 18min ago
    TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 4073 (dockerd)
      Tasks: 9
     Memory: 24.4M (peak: 28.0M)
        CPU: 595ms
     CGroup: /system.slice/docker.service
             └─4073 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
   ```
3. Создаем директорию.
   ```
   sudo mkdir -p /var/lib/postgres
   ```
4. Загружаем образ докера.
   ```
   sudo docker pull postgres
   ```
5. Так как нам понадобится обращаться из одного контейнера к другому - создадим локальную сеть.
    ```
    sudo docker network create local-net
    6e0e936410dc1258c4f94258082f336dd9862f20b49b983ed46ad33d2c167a96
    ```
6. Запускаем контейнер с сервером.
   Сразу настраиваем порт, прокидываем Volume на нашу директорию.
   ```
   sudo docker run --name postgres --network local-net -e POSTGRES_PASSWORD=adminPass -d -p 5432:5432 -v /var/lib/postgres/data:/var/lib/postgresql/data postgres
   ```
7. Проверяем, что контейнер запущен.
   ```
    sudo docker container ls
    CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
    77ca02744614   postgres   "docker-entrypoint.s…"   12 seconds ago   Up 11 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres
   ```
8. Запустим в докере клиент, подключившись к ранее поднятной бд.
    ```
    sudo docker run -it --rm --network local-net jbergknoff/postgresql-client postgresql://postgres:adminPass@postgres:5432
    ```
9. Создадим таблицу и несколько записей в нашей бд:
    ```
    create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

    CREATE TABLE
    INSERT 0 1
    INSERT 0 1
    WARNING:  there is no transaction in progress
    COMMIT
    postgres=# select* from persons;
    id | first_name | second_name
    ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
    (2 rows)
    ```
10. Убедимся, что записи созданы:
    ```
    postgres=# select* from persons;
    id | first_name | second_name
    ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
    (2 rows)
    ```
11. Прокидываем порт 5432 из виртуальной машины и подключаемся к нему при помощи psql.
    ```
    psql postgresql://postgres:adminPass@localhost:5432
    psql (17.4)
    WARNING: Console code page (437) differs from Windows code page (1252)
            8-bit characters might not work correctly. See psql reference
            page "Notes for Windows users" for details.
    Type "help" for help.

    postgres=#
    ```
    Так как подключение производится с виндового хоста - мы видим предупреждение о разности кодировок.
12. Убеждаемся, что записи на месте:
    ```
    postgres=# select* from persons;
    id | first_name | second_name
    ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
    (2 rows)
    ```
13. Отключаемся от БД, останавливаем, затем удаляем контейнер.
    ```
    alexandr@ubuntu-vm-docker:/var/lib$ sudo docker stop postgres
    postgres
    alexandr@ubuntu-vm-docker:/var/lib$ sudo docker remove postgres
    postgres
    ```
14. Повторно запускаем контейнер, монтируя ранее используемый каталог.
    ```sudo docker run --name postgres --network local-net -e POSTGRES_PASSWORD=adminPass -d -p 5432:5432 -v /var/lib/postgres/data:/var/lib/postgresql/data postgres```
15.  Проверяем что контейнер запущен:
    ```
    sudo docker container ls
    CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
    54e477051d98   postgres   "docker-entrypoint.s…"   31 seconds ago   Up 30 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres
    ```
    Как видмс id контейнера изменился.
16. Подключаемся к контейнеру и делаем запрос:
    ```
    postgres=# select * from persons;
    id | first_name | second_name
    ----+------------+-------------
    1 | ivan       | ivanov
    2 | petr       | petrov
    (2 rows)
    ```
    Все данные остались на месте.