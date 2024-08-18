# Настройка бэкапов с помощью WAL-G

Из всех возможных копий, выбран бэкап физически на тот же носитель
Работая непосредственно на сервере базы данных как пользователь postgres, wal-g может читать файлы базы данных из файловой системы. Эта опция обеспечивает высокую производительность и дополнительные возможности, такие как частичное восстановление или резервное копирование Delta.


После установки из репозитория

```
# Install latest Go compiler
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go

# Install lib dependencies
sudo apt install libbrotli-dev liblzo2-dev libsodium-dev curl cmake

# Fetch project and build
# Go 1.15 and below
go get github.com/wal-g/wal-g
# Go 1.16+ - just clone repository to $GOPATH
# if you want to save space add --depth=1 or --single-branch
git clone https://github.com/wal-g/wal-g $(go env GOPATH)/src/github.com/wal-g/wal-g

cd $(go env GOPATH)/src/github.com/wal-g/wal-g

# optional exports (see above)
export USE_BROTLI=1
export USE_LIBSODIUM=1
export USE_LZO=1

make deps
make pg_build
main/pg/wal-g --version

curl -L "https://github.com/wal-g/wal-g/releases/download/v0.2.15/wal-g.linux-amd64.tar.gz" -o "wal-g.linux-amd64.tar.gz"
tar -xzf wal-g.linux-amd64.tar.gz
mv wal-g /usr/local/bin/
```

Пытаемся создать файл конфигурации, по идее его можно создать где угодно,  просто нужно указать его путь при запуске команды wal-g backup-push.

Создаем в домашней директории пользователя postgres
```
sudo touch /home/user/walg.json
sudo nano /home/user/walg.json
```
парметры хранилищ https://github.com/wal-g/wal-g/blob/master/docs/STORAGES.md

вся настроцйка в постгресе: https://github.com/wal-g/wal-g/blob/master/docs/PostgreSQL.md

 /var/backups/postgresql/ - куда я буду складывать бэкапы

 Создаем каталог для бэкапов ``` sudo mkdir /var/backups/postgresql ```


```
    WALG_FILE_PREFIX =  "/var/backups/postgresql/",
    WALG_COMPRESSION_METHOD: "brotli",
    WALG_DELTA_MAX_STEPS: "5",
    PGDATA: "/var/lib/postgresql/14/main",
    PGHOST: "/var/run/postgresql/.s.PGSQL.5432"
```

с таким конфигом выдается ошибка:

```
/var/run/postgresql$ sudo wal-g backup-push /var/lib/postgresql/14/main --config /home/user/walg.json
ERROR: 2024/08/15 10:54:56.150550 failed to configure folder: No storage is configured now, please set one of following settings: [WALG_S3_PREFIX WALG_FILE_PREFIX WALG_GS_PREFIX WALG_AZ_PREFIX WALG_SWIFT_PREFIX]
```

совет чата gpt:

конфиг файл слежующий:
```
   {
     "WALG_FILE_PREFIX": "/var/backups/postgresql",
     "WALG_COMPRESSION_METHOD": "brotli",
     "WALG_DELTA_MAX_STEPS": "5",
     "PGDATA": "/var/lib/postgresql/14/main",
     "PGHOST": "/var/run/postgresql/.s.PGSQL.5432",
     "WALG_LOCAL_PREFIX": "/var/backups/postgresql"
   }
   
```
После этого не выдавал такую же ошибку, начал выдавать другую ошибку:

```
sudo wal-g backup-push /var/lib/postgresql/14/main --config /home/user/walg.json
WARNING: 2024/08/15 11:16:49.025849 WALG_LOCAL_PREFIX is unknown
WARNING: 2024/08/15 11:16:49.025934 We found that some variables in your config file detected as 'Unknown'.
  If this is not right, please create issue https://github.com/wal-g/wal-g/issues/new
INFO: 2024/08/15 11:16:49.027050 Couldn't find previous backup. Doing full backup.
ERROR: 2024/08/15 11:16:49.033098 FATAL: role "root" does not exist (SQLSTATE 28000)
ERROR: 2024/08/15 11:16:49.033211 Failed to connect using provided PGHOST and PGPORT, trying localhost:5432
ERROR: 2024/08/15 11:16:49.062994 Connect: postgres connection failed: unexpected message type
```


На всякий случай раскоментировал параметры по прослушке локального хоста и unix сокета

``` 
# - Connection Settings -

listen_addresses = 'localhost'          # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
port = 5432                             # (change requires restart)
max_connections = 100                   # (change requires restart)
#superuser_reserved_connections = 3     # (change requires restart)
unix_socket_directories = '/var/run/postgresql' # comma-separated list of directories
                                        # (change requires restart)
unix_socket_group = ''                  # (change requires restart)
unix_socket_permissions = 0777          # begin with 0 to use octal notation
                                        # (change requires restart)

```
еще строчки, которые я менял по совету сайта https://habr.com/ru/articles/506610/

```
wal_level=replica
archive_mode = on
archive_command = '/usr/local/bin/wal-g wal-push \"%p\" >> /var/log/postgresql/archive_command.log 2>&1'
archive_timeout = 60
restore_command = '/usr/local/bin/wal-g wal-fetch \"%f\" \"%p\" >> /var/log/postgresql/restore_command.log 2>&1'
```

пробуем не через unix сокет


   {
     "WALG_FILE_PREFIX": "/var/backups/postgresql",
     "WALG_COMPRESSION_METHOD": "brotli",
     "WALG_DELTA_MAX_STEPS": "5",
     "PGDATA": "/var/lib/postgresql/14/main",
     "PGHOST": "10.0.1.132",
     "PGPORT": "5432",
     "WALG_LOCAL_PREFIX": "/var/backups/postgresql"
   }

Это гавно заработало, как только я раздал права на папки внутри /var/backups/postgresql

конфиг в итоге сработал этот

   {
     "WALG_FILE_PREFIX": "/var/backups/postgresql",
     "WALG_COMPRESSION_METHOD": "brotli",
     "WALG_DELTA_MAX_STEPS": "5",
     "PGDATA": "/var/lib/postgresql/14/main",
     "PGHOST": "/var/run/postgresql/.s.PGSQL.5432",
     "WALG_LOCAL_PREFIX": "/var/backups/postgresql"
   }
   

Бэкапы автоматически создаются:


В папке wal-005
```
postgres@ubuntu-postgres:/var/backups/postgresql/wal_005$ ls -l
total 1596
-rw------- 1 postgres postgres 1502857 Aug 18 16:10 000000010000000000000001.br
-rw------- 1 postgres postgres     199 Aug 18 16:15 000000010000000000000002.br
-rw------- 1 postgres postgres   38110 Aug 18 16:30 000000010000000000000003.br
-rw------- 1 postgres postgres     616 Aug 18 16:31 000000010000000000000004.br
-rw------- 1 postgres postgres     154 Aug 18 16:35 000000010000000000000005.br
-rw------- 1 postgres postgres     104 Aug 19 07:32 000000010000000000000006.br
-rw------- 1 postgres postgres     176 Aug 19 07:32 000000010000000000000007.br
-rw------- 1 postgres postgres     175 Aug 19 07:33 000000010000000000000008.br
-rw------- 1 postgres postgres     178 Aug 19 07:34 000000010000000000000009.br
-rw------- 1 postgres postgres     155 Aug 19 07:41 00000001000000000000000A.br
-rw------- 1 postgres postgres     106 Aug 19 07:42 00000001000000000000000B.br
-rw------- 1 postgres postgres     194 Aug 19 07:42 00000001000000000000000C.00000028.backup.br
-rw------- 1 postgres postgres     194 Aug 19 07:42 00000001000000000000000C.br
-rw------- 1 postgres postgres     174 Aug 19 07:47 00000001000000000000000D.br
-rw------- 1 postgres postgres   27616 Aug 19 07:52 00000001000000000000000E.br
-rw------- 1 postgres postgres     186 Aug 19 07:53 00000001000000000000000F.br
-rw------- 1 postgres postgres     193 Aug 19 08:22 000000010000000000000010.00000028.backup.br
-rw------- 1 postgres postgres     177 Aug 19 08:22 000000010000000000000010.br
```


в папке basebackups_005
```
postgres@ubuntu-postgres:/var/backups/postgresql/wal_005$ ls -l
total 1596
-rw------- 1 postgres postgres 1502857 Aug 18 16:10 000000010000000000000001.br
-rw------- 1 postgres postgres     199 Aug 18 16:15 000000010000000000000002.br
-rw------- 1 postgres postgres   38110 Aug 18 16:30 000000010000000000000003.br
-rw------- 1 postgres postgres     616 Aug 18 16:31 000000010000000000000004.br
-rw------- 1 postgres postgres     154 Aug 18 16:35 000000010000000000000005.br
-rw------- 1 postgres postgres     104 Aug 19 07:32 000000010000000000000006.br
-rw------- 1 postgres postgres     176 Aug 19 07:32 000000010000000000000007.br
-rw------- 1 postgres postgres     175 Aug 19 07:33 000000010000000000000008.br
-rw------- 1 postgres postgres     178 Aug 19 07:34 000000010000000000000009.br
-rw------- 1 postgres postgres     155 Aug 19 07:41 00000001000000000000000A.br
-rw------- 1 postgres postgres     106 Aug 19 07:42 00000001000000000000000B.br
-rw------- 1 postgres postgres     194 Aug 19 07:42 00000001000000000000000C.00000028.backup.br
-rw------- 1 postgres postgres     194 Aug 19 07:42 00000001000000000000000C.br
-rw------- 1 postgres postgres     174 Aug 19 07:47 00000001000000000000000D.br
-rw------- 1 postgres postgres   27616 Aug 19 07:52 00000001000000000000000E.br
-rw------- 1 postgres postgres     186 Aug 19 07:53 00000001000000000000000F.br
-rw------- 1 postgres postgres     193 Aug 19 08:22 000000010000000000000010.00000028.backup.br
-rw------- 1 postgres postgres     177 Aug 19 08:22 000000010000000000000010.br
```



ПРОБУЕМ СОЗДАТЬ СКРИПТ ДЛЯ РАНДОМНОГО НАПОЛНЕНИЯ ТАБЛИЦЫ


в psql:

```

-- Создаем БД
CREATE DATABASE test_script;

-- Подключаемся к базе данных

\c test_script

-- Создаем таблицу data с тремя столбцами
CREATE TABLE data (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITHOUT TIME ZONE DEFAULT NOW(),
    value1 INTEGER,
    value2 TEXT
);
```

Создадим bash файл, который будет генерировать раз в минуту рандомные данные, а старые удалять

```
#!/bin/bash

# Настройки подключения к базе данных
DB_HOST="localhost"
DB_NAME="test_script"
DB_USER="postgres"
DB_PASS="postgres"

# Команда для вставки данных
INSERT_CMD="psql -h $DB_HOST -d $DB_NAME -U $DB_USER -c \"INSERT INTO data (value1, value2) VALUES ($RANDOM, '$RANDOM');\""

# Команда для удаления старых данных
DELETE_CMD="psql -h $DB_HOST -d $DB_NAME -U $DB_USER -c \"DELETE FROM data WHERE timestamp < (NOW() - INTERVAL '1 minute');\""

while true; do
  # Вставка данных
  $INSERT_CMD

  # Удаление старых данных
  $DELETE_CMD

  # Пауза на 1 минуту
  sleep 60
done
```



НОВЫЙ СКРИПТ С УДАЛЕНИЕМ 1 РАЗ В 5 МИНУТ
```
#!/bin/bash

# Настройки подключения к базе данных
DB_HOST="localhost"
DB_NAME="test_script"
DB_USER="postgres"
DB_PASS=""

# Команда для вставки данных
INSERT_CMD="psql -h $DB_HOST -d $DB_NAME -U $DB_USER -c \"INSERT INTO data (value1, value2) VALUES ($RANDOM, '$RANDOM');\""

# Команда для удаления старых данных
DELETE_CMD="psql -h $DB_HOST -d $DB_NAME -U $DB_USER -c \"DELETE FROM data WHERE timestamp < (NOW() - INTERVAL '5 minutes');\""

while true; do
  # Вставка данных
  $INSERT_CMD

  # Удаление старых данных (раз в 5 минут)
  if (( $(date +%M) % 5 == 0 )); then
    $DELETE_CMD
  fi

  # Пауза на 1 минуту
  sleep 60
done
```


еще 1 вариант

#!/bin/bash

# Настройки подключения к базе данных
DB_HOST="localhost"
DB_NAME="test_script"
DB_USER="postgres"
DB_PASS="postgres"

# Команда для вставки данных
INSERT_CMD="psql -h $DB_HOST -d $DB_NAME -U $DB_USER -c 'INSERT INTO data (value1, value2) VALUES ($RANDOM, '$RANDOM');'"

# Команда для удаления старых данных
DELETE_CMD="psql -h $DB_HOST -d $DB_NAME -U $DB_USER -c 'DELETE FROM data WHERE timestamp < (NOW() - INTERVAL '5 minutes');'"

while true; do
  # Вставка данных
  $INSERT_CMD

  # Удаление старых данных (раз в 5 минут)
  if (( $(date +%M) % 5 == 0 )); then
    $DELETE_CMD
  fi

  # Пауза на 1 минуту
  sleep 60
done



Пока закомментим строки, но в будущем сделать их

#"WALG_UPLOAD_CONCURRENCY" = 16
Количество потоков, по умолчанию равно 16



#"WALG_UPLOAD_DISK_CONCURRENCY" = 1 
количество параллельных потоков, считывающих диск


# "TOTAL_BG_UPLOADED_LIMIT" = 32
Количество WAL файлов загруженных после одного сканирования




#"WALG_PREVENT_WAL_OVERWRITE" = ? 
 Предотвращение перезаписи WAL файлов




#"WALG_DELTA_MAX_STEPS" = 0
№ Определяет, сколько дельта-резервных копий может быть между полными резервными копиями. По умолчанию 0.



# WALG_DELTA_ORIGIN = "LATEST"
Для настройки базы для следующего резервного копирования дельты (только если WALG_DELTA_MAX_STEPSне превышено). WALG_DELTA_ORIGINможет быть LATEST (цепочка приращений), LATEST_FULL (для баз, где изменчивая часть компактна и цепочка не имеет смысла — дельты перезаписывают друг друга). По умолчанию LATEST



# WALG_TAR_SIZE_THRESHOLD = 1 << 30 - 1
Размер одного резервного пакета. По умолчанию 1Гб



WALG_ALIVE_CHECK_INTERVAL
Для управления частотой проверки WAL-G активности Postgres во время backup-push. Если проверка не пройдена, backup-push прекращается.

Примеры:

0- отключить проверку активности
1m- проверка каждую минуту (значение по умолчанию)
10s- проверять каждые 10 секунд
10m- проверять каждые 10 минут

#WALG_STOP_BACKUP_TIMEOUT
Тайм-аут для вызова pg_stop_backup(). По умолчанию тайм-аут отсутствует.

Примеры:
0- отключить тайм-аут (значение по умолчанию)
10s- тайм-аут 10 секунд
10m- 10 минут тайм-аута


##Распаковка:
Прямая распаковка с помощью backup-fetch

```wal-g backup-fetch ~/extract/to/here example-backup```


##Обратная дельта-распаковка
```wal-g backup-fetch /path LATEST --reverse-unpack```



# Работа в режиме демона
daemon
Архивирует и извлекает все сегменты WAL в фоновом режиме. Работает с архивной библиотекой PostgreSQL walg_archiveили walg-daemon-client.

Использование:

wal-g daemon path/to/socket-descriptor

указывается путь до сокета postgres

```wal-g daemon /var/run/postgresql/postgresql.s.5432```


Конфигурация:

WALG_DAEMON_WAL_UPLOAD_TIMEOUT
Для настройки временного лимита для каждого архива WAL в демоне. При зависании на более длительное время операции будут прерваны. Значение по умолчанию — 60 с.


archive_timeout отвечает за создание дельта копийц wal-g


archive_timeout (integer)
Команда archive_command вызывается только для завершённых сегментов WAL. Поэтому, если ваш сервер записывает мало данных WAL (или это наблюдается в некоторые периоды времени), от завершения транзакции до надёжного сохранения её в архивном хранилище может пройти довольно много времени. Для ограничения времени существования неархивированных данных можно установить значение archive_timeout, чтобы сервер периодически переключался на новый файл сегмента WAL. Когда этот параметр больше нуля, сервер будет переключаться на новый файл сегмента, если с момента последнего переключения на новый файл прошло заданное число секунд, и наблюдалась какая-то активность базы данных, даже если это была просто контрольная точка. (Для сокращения числа ненужных контрольных точек в простаивающей системе можно увеличить checkpoint_timeout.) Заметьте, что архивируемые файлы, закрываемые раньше из-за принудительного переключения, всё равно будут иметь тот же размер, что и полностью заполненные. Поэтому устанавливать для archive_timeout очень маленькое значение неразумно — это ведёт к замусориванию архивного хранилища. Обычно для archive_timeout имеет смысл задавать значение около минуты. Если вам нужно, чтобы данные копировались с главного сервера быстрее, вам следует подумать о переходе от архивации к потоковой репликации. Этот параметр можно задать только в postgresql.conf или в командной строке при запуске сервера.

Настройка бэкапов cron
pg_lsclusters - узнать че вообще есть и че запущено

Команда вручную сделать бэкап
su - postgres -c '/usr/local/bin/wal-g backup-push /var/lib/postgresql/12/main'
