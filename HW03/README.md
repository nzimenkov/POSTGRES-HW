# ДЗ №3. Установка и настройка PostgreSQL

Цель:
создавать дополнительный диск для уже существующей виртуальной машины, размечать его и делать на нем файловую систему
переносить содержимое базы данных PostgreSQL на дополнительный диск
переносить содержимое БД PostgreSQL между виртуальными машинами


> создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
> поставьте на нее PostgreSQL 15 через sudo apt

```sudo apt install postgresql-client-15 postgresql-15``` 

> проверьте что кластер запущен через sudo -u postgres pg_lsclusters

КАРТИНКА E:\Помойка\OTUS\домашки\HW3\1.png

> зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым

КАРТИНКА E:\Помойка\OTUS\домашки\HW3\2.png


> остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
```sudo -u postgres pg_ctlcluster 15 main stop```
```sudo systemctl stop postgresql@15-main```


> создайте новый диск к ВМ размером 10GB
> добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk

КАРТИНКА E:\Помойка\OTUS\домашки\HW3\.3png

> добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
> проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
> перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)


Монтируем новый диск в /mnt/data
```sudo mkdir /mnt/data```
```sudo mkfs.ext4 /dev/sdb```
```sudo mount /dev/sdb /mnt/data```

> сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

```sudo chown -R postgres:postgres /mnt/data```


> перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
от пользователя postgres перенесим все данные
```mv /var/lib/postgresql/15/* /mnt/data```

> попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
> напишите получилось или нет и почему
Выдается ошибка:
Error: /var/lib/postgresql/15/main is not accessible or does not exist

При попытке запуска параметры ссылаются на путь, /var/lib/postgresql/15/main, в котором уже нет данных

> задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
> напишите что и почему поменяли

в /etc/postgresql/15/main/postgresql.conf ставим  data_directory = '/mnt/data/15/main/'

Проверяем 
E:\Помойка\OTUS\домашки\HW3\.4png

