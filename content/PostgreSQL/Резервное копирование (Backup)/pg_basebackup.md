
Физический backup кластера.

- Копирует файловую структуру кластера (все базы и все данные)
- Копирует содержимое каталога данных (`PGDATA`) на момент запуска.
- Работает через **репликационный протокол** (требуется подключение с правами репликации).
- Бэкап получается **согласованным во времени** (consistent), то есть пригоден для восстановления без повреждений. ==Подходит для **горячего резервного копирования** (без остановки СУБД).==
Опционально может:
- Включить **WAL-файлы**, необходимые для восстановления до момента завершения бэкапа (с флагом `-X stream` или `-X fetch`).
- Создать **standby-файл** (`standby.signal`) для немедленного запуска реплики.
- Сжать вывод (`--gzip`, начиная с PostgreSQL 15).
- Записать в tar-архивы (`-Ft`) или в распакованном виде (`-Fp`, по умолчанию).

**Требования:**
1. Сервер должен быть запущен
2. В `postgresql.conf` включено:
```sh
wal_level = replica # или logical
```
3. Пользователь с правами **репликации** (или суперпользователь).
4. В `pg_hba.conf` разрешено репликационное подключение:
```sh
host replication replicator 192.168.1.0/24 trust
```

## Опции командной строки

| Опция                                                        | Назначение                                                                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| `-h host`                                                    | Хост основного сервера                                                                                          |
| `-p port`                                                    | Порт (по умолчанию 5432)                                                                                        |
| `-U username`                                                | Пользователь с правами репликации                                                                               |
| `-D /path/to/backup`                                         | Каталог назначения                                                                                              |
| `-F p` или `--format=plain`                                  | Формат: распакованные файлы (по умолчанию)                                                                      |
| `-F t` или `--format=tar`                                    | Формат: tar-архивы (по одному на табличное пространство)                                                        |
| `-X stream`                                                  | Передавать WAL "на лету" во время бэкапа                                                                        |
| `-X fetch`                                                   | Скачать нужные WAL после завершения бэкапа (может занять много места)                                           |
| `-P`                                                         | Показывать прогресс                                                                                             |
| `-v`                                                         | Подробный вывод                                                                                                 |
| `--wal-method=...`                                           | То же, что `-X` (в новых версиях)                                                                               |
| `--label="..."`                                              | Метка бэкапа (записывается в `backup_label`)                                                                    |
| `--checkpoint=fast` (по умолчанию) или `--checkpoint=spread` | Как делать контрольную точку                                                                                    |
| `--slot=slot_name`                                           | Использовать существующий слот репликации                                                                       |
| `--create-slot`                                              | Создать новый слот репликации                                                                                   |
| `--no-slot`                                                  | Не использовать слот (WAL может быть удалён до применения!)                                                     |
| `-R`                                                         | Создать `standby.signal` и добавить строку `primary_conninfo` в `postgresql.auto.conf` — сразу готовая реплика! |
| `--gzip[=N]`                                                 | Сжать tar-файлы (только с `-Ft`, начиная с PostgreSQL 15)                                                       |

---
## Backup в директорию ^psql-pgbasebackup-coldbackup
```sh
pg_basebackup -h localhost -D /backup/pgdata -P -v
# вывод
tree /tmp/backup/pgdata -L 2
	/tmp/backup/pgdata
	├── backup_label
	├── backup_manifest
	├── base
	│   ├── 1
	│   ├── 16389
	│   ├── 4
	│   └── 5
	├── global
	│   ├── 1213
	│   ├── 1213_fsm
	│   ├── 1213_vm
	│   ├── 1214
	...
	│   ├── pg_control
	│   └── pg_filenode.map
	├── pg_commit_ts
	├── pg_dynshmem
	├── pg_logical
	│   ├── mappings
	│   ├── replorigin_checkpoint
	│   └── snapshots
	├── pg_multixact
	│   ├── members
	│   └── offsets
	├── pg_notify
	├── pg_replslot
	├── pg_serial
	├── pg_snapshots
	├── pg_stat
	├── pg_stat_tmp
	├── pg_subtrans
	├── pg_tblspc
	├── pg_twophase
	├── PG_VERSION
	├── pg_wal
	│   ├── 000000010000000000000002
	│   ├── archive_status
	│   └── summaries
	├── pg_xact
	│   └── 0000
	└── postgresql.auto.conf

```

#### Restore на localhost ^psql-pgbasebackup-restorelocal
```sh
mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main.bak
cp -r /tmp/backup/pgdata /var/lib/postgresql/18/main
chown -R  postgres:postgres /var/lib/postgresql/18/main
chmod -R 700 /var/lib/postgresql/18/main

### или запустить в копии pg_ctl или systemd
# Главное помянять порт в postgresql.conf
sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl \
  -D /tmp/backup/pgdata \
  -l /tmp/backup/pgdata/logfile \
  start

### Если хочешь быстро проверить бэкап — запусти его на другом порту:
# Подготовка
sudo chown -R postgres:postgres /tmp/backup/pgdata
sudo -u postgres sed -i "s/^#port = .*/port = 5433/" /tmp/backup/pgdata/postgresql.conf
# Запуск
sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl -D /tmp/backup/pgdata -l /tmp/pg.log start
```

#### Backup + Restore сразу на Standby ^psql-pgbasebackup-straightStandby
```sh
### нужны параметры postgresql.conf
# wal_level = replica
# max_wal_senders = 3
# hot_standby = on
## пользователь с правами репликации

### На primary
sudo nano /etc/postgresql/18/main/pg_hba.conf
	host    replication     postgres        standby_host/32           scram-sha-256
sudo systemctl reload postgresql

### На Standby
sudo pg_ctlcluster 18 main stop    # или systemctl stop postgresql # если pg_ctl
	# sudo systemctl stop postgresql
sudo pg_dropcluster 18 main --stop  # удаляет кластер(/etc/powtgresql/*conf) и каталог данных

sudo -u postgres pg_basebackup \
  -h primary_host \
  -U postgres \
  -D /var/lib/postgresql/18/standby \  # дир на Standby
  -P -v \
  --slot=standby1 \ # указывает имя слота
  --create-slot \ # создаёт новый слот на primary
  -R \  # автоматически создаёт `standby.signal` и добавляет строку `primary_conninfo` в `postgresql.conf` и `primary_slot_name`
  --wal-method=stream  # позволяет standby получать WAL напрямую, не дожидаясь архивации.

# Перемещаем временно в другое расположение что бы не затереть данные
sudo mv /var/lib/postgresql/18/standby /var/lib/postgresql/18/standby.bak

# Создаем кластер используя имеющиеся данные
sudo pg_createcluster \
  18 \
  standby \
  --datadir=/var/lib/postgresql/18/standby \
  #-- --unix_socket_directories='/var/run/postgresql' # Всё после `--` передаётся как параметры в `postgresql.conf` при старте (опционально, но полезно)

# Перемещаем обратно
sudo mv /var/lib/postgresql/18/standby.bak /var/lib/postgresql/18/standby

# На всякий
ls -l /var/lib/postgresql/18/standby/standby.signal
sudo chown -R postgres:postgres /var/lib/postgresql/18/standby
sudo chmod 700 /var/lib/postgresql/18/standby

sudo pg_ctlcluster 18 main start
# sudo systemctl start postgresql@18-main

```
- **Standby** — чистая машина с той же версией PostgreSQL, установленной из пакета (но кластер остановлен или удалён).
- `pg_createcluster` всегда вызывает `initdb`, а `initdb` **отказывается работать**, если каталог не пуст или не инициализирован.
- **Standby узнаёт о Primary из файла `postgresql.auto.conf`**, который автоматически создаётся командой `pg_basebackup` с флагом `-R`.
