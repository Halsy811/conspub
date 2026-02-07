#psql #commands
# `psql` - command

## Подключиться к БД с указанием хоста, порта, пользователя и имени БД
^query-get-psql-using
```sh
# Подключиться к БД с указанием хоста, порта, пользователя и имени БД
psql -h localhost -p 5432 -U myuser -d mydb
# Просмотреть доступные базы в кластере
psql -l
```

## Проверка: действительно ли это standby?
```sh
sudo -u postgres psql -c "SELECT pg_is_in_recovery();"
# t - standby
# f - primary
```


## Проверка физической репликации ^psql-test-replication
#### Проверьте процесс приёма WAL
```sh
# Есть ли подключенные...

### Standby
sudo -u postgres psql -c "SELECT * FROM pg_stat_wal_receiver;"
#	Если всё в порядке, вы увидите **одну строку** с колонками:
# `status = 'streaming'` — получает данные в реальном времени
# `slot_name` — имя слота репликации (если используется)
# `sender_host`, `sender_port` — откуда идёт поток
# `latest_end_lsn` — последний полученный LSN
  
# Если таблица **пустая** → standby **не подключён** к primary.

### Primary
sudo -u postgres psql -c "SELECT * FROM pg_stat_replication;"
```
#### Сравните LSN (Log Sequence Number) с primary
LSN показывает, **до какой точки из WAL-логов уже применены изменения** на этом сервере.
```sh
# Сравните значения вывода на Primary и Standby
sudo -u postgres psql -c "SELECT pg_last_wal_replay_lsn();"
# или в цыкле
sudo -u postgres watch -n 2 "psql -U postgres -c \"SELECT pg_last_wal_replay_lsn();\""
```
#### Или можно увидить описание Primary в postgresql.auto.conf
```sh
# postgresql.auto.conf	# Файл для ALTER SYSTEM — автоматически управляемый конфиг
sudo -u postgres cat /var/lib/postgresql/18/standby/postgresql.auto.conf
	# primary_conninfo = ...
```

### Проверка работы слота "Physical Replication Slot" ^psql-test-slotrepl
```sh
### На primary
sudo -u postgres psql -c "
	SELECT
	  slot_name,
	  slot_type,
	  active,
	  active_pid,
	  restart_lsn,
	  pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS lag
	FROM pg_replication_slots;
"

#`active = t` — standby подключён и использует слот
# `lag` — примерное отставание в байтах (можно перевести в МБ/ГБ)


### На standby
# Убедитесь, что вы в режиме восстановления
sudo -u postgres psql -c "SELECT pg_is_in_recovery();" # true
# Посмотрите статус приёма WAL
sudo -u postgres psql -c "SELECT * FROM pg_stat_wal_receiver;"
```
###### Удаление слота "Physical Replication Slot"  ^psql-del-slotrepl
```sh
### На primary
sudo -u postgres psql -c "SELECT pg_drop_replication_slot('standby1');"
# Проверьте
sudo -u postgres psql -c "SELECT * FROM pg_replication_slots;"

# Контролируйте размер `pg_wal`
du -sh /var/lib/postgresql/14/main/pg_wal/
```
## Удаление / Отключение Standby ^psql-remove-Standby
```sh
### На Standby
systemctl stop postgresql
rm /var/lib/postgresql/18/standby/standby.signal # Переводим в режим Primary

# Следующее опционально:
rm /var/lib/postgresql/18/standby/postgresql.auto.conf # Standby узнаёт о Primary из файла `postgresql.auto.conf`, который автоматически создаётся командой `pg_basebackup` с флагом `-R`.


### На primary
# отключить wal трансляцию
```

## Мониторинг архивации WAL ^psql-monArchWAL
```sh
# Проверить, что файлы появляются
ls -lt /archive/wal/ | tail -10
```

```sql
-- Последний заархивированный WAL
SELECT last_archived_wal, last_archived_time, failed_count FROM pg_stat_archiver;

-- Ошибки архивации
SELECT last_failed_wal, last_failed_time, last_archive_error FROM pg_stat_archiver;

-- Создайте контрольную точку перед важными операциями
-- Потом можно восстановиться именно до этой точки.
SELECT pg_create_restore_point('before_monthly_cleanup');
```

💡 **Правило админа**:  
Если у вас нет PITR — у вас **нет настоящего резервного копирования**.