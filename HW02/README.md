# ДЗ2. Работа с уровнями изоляции транзакции в PostgreSQL

> создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом

Создаем новую виртуальную машину в VirtualBox, Xubuntu 22.04 64-bit

> поставить на нем Docker Engine
    ```
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```
``` sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin```
``` sudo docker run hello-world ```

> сделать каталог /var/lib/postgres

```sudo mkdir /var/lib/postgresql/```

 1. Создаем docker-сеть: 
sudo docker network create pg-net


-- 2. подключаем созданную сеть к контейнеру сервера Postgres:
```sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -v /var/lib/postgres:/var/lib/postgresql/data postgres:15```

E:\Помойка\OTUS\домашки\HW2\1.png

> развернуть контейнер с клиентом postgres

 Запускаем отдельный контейнер с клиентом в общей сети с БД: 
```sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres``

> подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

```CREATE TABLE learning (id SERIAL PRIMARY KEY, title INTEGER);```
```INSERT INTO learning (id, title) VALUES (1, '1');```
    
```INSERT INTO learning (id, title) VALUES (2, '2');```
E:\Помойка\OTUS\домашки\HW2\2.png

> удалить контейнер с сервером
E:\Помойка\OTUS\домашки\HW2\3.png

При попытке создать новый контейнер выскакивает ошибка
 E:\Помойка\OTUS\домашки\HW2\4.png

Убиваем действующий процесс
  E:\Помойка\OTUS\домашки\HW2\5.png

> создать его заново
 sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
  E:\Помойка\OTUS\домашки\HW2\6.png
  
> Проверяем что все осталось на месте
 E:\Помойка\OTUS\домашки\HW2\7.png