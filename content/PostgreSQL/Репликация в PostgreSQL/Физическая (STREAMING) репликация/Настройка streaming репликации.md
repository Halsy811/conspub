
# Без WAL archiving и Replication slots ^psql-phRepl-not-SlotAndWAL
- Не настраиваем `archive_mode` и `archive_command`

- При потере связи более чем на `wal_keep_size` (ранее `wal_keep_segments`) реплика может "отстать" и потребовать пересоздания
Рекомендуется установить `wal_keep_size` на primary, чтобы дать standby "буфер времени":
```sh
wal_keep_size = 1GB   # или больше, в зависимости от нагрузки
```
Или использовать **replication slots** (как в `pg_basebackup --slot=...`), которые автоматически предотвращают удаление нужных WAL-файлов.

Параметр **`wal_keep_size`** в PostgreSQL определяет **объём WAL-файлов (Write-Ahead Logging), которые сервер будет хранить в каталоге `pg_wal` даже после их "логического" завершения**, чтобы дать отстающим standby-серверам время на получение этих данных.
##### Primary
```sh
nano /etc/postgresql/<версия>/main/postgresql.conf
# nano /var/lib/pgsql/<версия>/data/postgresql.conf
	listen_addresses = '*'
	wal_level = replica # минимальный уровень для streaming replication.
	max_wal_senders = 1
	hot_standby = on
	wal_keep_size = 1GB
```

```sh
nano /etc/postgresql/<версия>/main/pg_hba.conf
	# TYPE  DATABASE        USER            ADDRESS                 METHOD
	host    replication     replicator      192.168.1.20/32         md5
```
- `replication` — специальное ключевое слово для репликации
- `replicator` — имя пользователя, который будет использоваться для репликации
- `192.168.1.20` — IP адрес `psql2` (standby)

Создайте пользователя для репликации
```sh
sudo -u postgres psql
```
```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'Stasees';
```

```sh
sudo systemctl restart postgresql
```

##### Standby
```sh
sudo systemctl stop postgresql
```
Очистите каталог данных (если он не пуст)
```sh
sudo rm -rf /var/lib/postgresql/18/main
```
Скопируйте данные с primary с помощью [[pg_basebackup#^psql-pgbasebackup-straightStandby|pg_basebackup]]
```sh
sudo -u postgres pg_basebackup \
  -h primary_host \
  -U replicator \
  -D /var/lib/postgresql/18/main \
  -P -v \
  -R \
  --wal-method=stream
# Проверьте
ls -l /var/lib/postgresql/18/main/standby.signal
cat /var/lib/postgresql/18/main/postgresql.auto.conf
#	primary_conninfo = ... primary_host
sudo systemctl start postgresql
```

[[Базовое АДМИНИСТРИРОВАНИЕ PostgreSQL#^psql-test-replication|Проверка физической репликации]]

---

# Настройка Physical Replication Slot ^psql-phRepl-add-slot

```sql
-- на primary
-- Создать слот с именем 'standby1'
SELECT pg_create_physical_replication_slot('standby1');
-- Проверьте
SELECT slot_name, slot_type, active FROM pg_replication_slots;

-- ### Автоматически через `pg_basebackup` (рекомендуется) с ключом
--slot=standby1
```

```sh
### на standby 
# Добавить в postgresql.auto.conf
	# primary_conninfo = 'user=replicator password=... host=192.168.1.10 port=5432
	primary_slot_name = 'standby1'
	
# «Подключайся к primary и используй replication slot с именем `standby1`».


```
- **Используйте `pg_stat_replication` + `pg_replication_slots`** для комплексного мониторинга.
[[Базовое АДМИНИСТРИРОВАНИЕ PostgreSQL#^psql-test-slotrepl|Проверка работы слота "Physical Replication Slot"]]
[[Базовое АДМИНИСТРИРОВАНИЕ PostgreSQL#^psql-del-slotrepl|Удаление слота "Physical Replication Slot"]]

# Настройка WAL archiving ^psql-phRepl-add-walArch

- Добавить WAL archiving на смонтированный диск к primary в /mnt/wal_archive

```sh
### Нстройка `archive_mode` 
### на primary

#Подготовить папку:
chown -R postgres:postgres /mnt/wal_archive
chmod 700 /mnt/wal_archive


# postgresql.conf
	wal_level = replica
	archive_mode = on
	archive_command = 'cp %p /mnt/wal_archive/%f'
		# более надёжная команда с проверкой
		archive_command = 'test ! -f /mnt/wal_archive/%f && cp %p /mnt/wal_archive/%f'
		# Или с логированием ошибок
		archive_command = 'cp %p /mnt/wal_archive/%f && echo "Archived %f at $(date)" >> /var/log/postgresql/wal_archive.log || exit 1'
	# `%f` → имя WAL-файла (например, `0000000100000000000000A5`)
	# `%p` → локальный путь, куда его нужно положить (в `pg_wal/`)
	
# Перезагрузите конфигурацию на primary
sudo systemctl reload postgresql
# ИЛИ внутри psql:
#	SELECT pg_reload_conf();
```

#### Настройка standby для использования WAL archive (на случай очень долгого разрыва репликации)

- `restore_command` работает **только в режиме recovery** (standby или PITR).

Если streaming replication прервётся, standby может догнать primary, используя архивные WAL-файлы
Для этого на standby, нужно указать `restore_command`.
```sh
### на standby
restore_command = 'cp /mnt/wal_archive/%f %p'

# --------------------- более интересно...
# Забирать wal с primary через scp
### Настройка scp на подключение без пароля
sudo -u postgres ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa # тут '~' = root
sudo -u postgres ssh-keygen -t rsa -b 4096 -N "" -f /var/lib/postgresql/.ssh/id_rsa
		# getent passwd postgres # точно узнать домашний каталог
		#	 postgres:x:102:110:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
	# или руками под самим пользователем, но скорее всего будет ошибка подключения
	ssh-keygen
	ssh-copy-id postgres@<primary_host>
sudo -u postgres bash -c 'ssh-keyscan <primary_host> >> /var/lib/postgresql/.ssh/known_hosts' # Добавить хост в known_hosts
# Скопировать ключ на Primary
sudo -u postgres ssh-copy-id postgres@10.0.10.91

# Проверьте
sudo -u postgres ssh postgres@10.0.10.91 "echo OK"



# на standby добавить в конфиг
restore_command = 'scp postgres@10.0.10.91:/mnt/wal_archive/%f %p'
	# `%f` → имя WAL-файла (например, `0000000100000000000000A5`)
	# `%p` → локальный путь, куда его нужно положить (в `pg_wal/`)

# с таймаутом и логированием (рекомендуется)
restore_command = 'timeout 30 scp postgres@10.0.10.91:/mnt/wal_archive/%f %p && echo "$(date): restored %f" >> /var/log/postgresql/restore.log || exit 1'
# --------------------- 
	
	
# Проверьте логи standby:
tail -f /var/log/postgresql/postgresql-*.log
#	restored log file "0000000100000000000000A5" from archive
```

---

- Каталог `/mnt/wal_archive` должен быть доступен на primary для записи и на standby для чтения.
- Регулярно очищайте архив (например, через `pg_archivecleanup`), чтобы не заполнить диск
```sh
# Пример: оставить последние 10 WAL-файлов
pg_archivecleanup -d /mnt/wal_archive/ 0000000100000000000000B0
```
- Или используйте скрипт на основе `pg_controldata` или мониторинга LSN.

1. **Replication slots + WAL archiving** — отличная комбинация:
    - Слоты защищают от разрыва при краткосрочных отключениях
    - Архив спасает при долгих простоях или сбое слота

# Отключение streaming репликации ^psql-phRepl-clear-streamingReplConf
```sh
# Удаление слотов репликации
	### Primary
	# для просмотра слотов
	SELECT * FROM pg_replication_slots; 
	# для удаления
	SELECT pg_drop_replication_slot('slot_name'); 
	
	# Очистка конфигурационных файлов на Standby
	/var/lib/postgresql/18/main/postgresql.auto.conf
	/etc/postgresql/18/main/conf.d/psql.conf
		# коментируем primary_conninfo, primary_slot_name, restore_command
```
[[Базовое АДМИНИСТРИРОВАНИЕ PostgreSQL#^psql-test-replication|Проверка]]