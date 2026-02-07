
Еще информация здесь: [[configiration, PGDATA#^psql-conf-file-postgresql-conf|Конфигурация и файлы базы]] - ссылки на содержание

Это **ядро конфигурации кластера**, и почти всё, что управляет поведением СУБД, задаётся именно здесь (или через SQL-команду ==ALTER SYSTEM==, которая вносит изменения в ==postgresql.auto.conf==).
### Применение изменений ^enable-conf-psql

```sh
# Перезапуск кластера
sudo systemctl restart postgresql@18-main
```

```sql
-- Перезагрузка конфигурации
SELECT pg_reload_conf();
```


**Поддерживаемые типы значений**

| **Type**         | **Exemple**                                 | Note                          |
| ---------------- | ------------------------------------------- | ----------------------------- |
| Числа            | max_connections = 100                       |                               |
| Строки           | cluster_name = 'prod-db'                    | в кавычках, если есть пробелы |
| Булевы           | fsync = on                                  |                               |
| Интервалы/размер | wal_keep_size = 1GB                         |                               |
| Время            | checkpoint_timeout = 15min                  |                               |
| Списки           | listen_addresses = 'localhost,192.168.1.10' |                               |
 
### postgresql.conf ^postgres-genegal-config

Перемещение по разделам:
[[postgresql.conf#^psql-conf-info|Описание конфига]]

[[postgresql.conf#^psql-conf-locatefile|РАСПОЛОЖЕНИЕ ФАЙЛОВ]]
[[postgresql.conf#^psql-conf-connect-and-auth|ПОДКЛЮЧЕНИЯ И АУТЕНТИФИКАЦИЯ]]
[[postgresql.conf#^psql-conf-resurse-using|ИСПОЛЬЗОВАНИЕ РЕСУРСОВ]]
[[postgresql.conf#^psql-conf-WAL-journal-recovery|ЖУРНАЛ ПРЕДВАРИТЕЛЬНОЙ ЗАПИСИ (WAL)]]
[[postgresql.conf#^psql-conf-replication-configure|РЕПЛИКАЦИЯ]]
[[postgresql.conf#^psql-conf-query-settings-configure|НАСТРОЙКА ЗАПРОСОВ]]
[[postgresql.conf#^psql-conf-report-and-logging|ОТЧЁТНОСТЬ И ЖУРНАЛИРОВАНИЕ]]
[[postgresql.conf#^psql-conf-statistics|СТАТИСТИКА]]
[[postgresql.conf#^psql-conf-vacuum-configure|VACUUM]]
[[postgresql.conf#^psql-conf-connect-client-default|НАСТРОЙКИ ПОДКЛЮЧЕНИЙ КЛИЕНТОВ ПО УМОЛЧАНИЮ]]
[[postgresql.conf#^psql-conf-lock-manager-configure|УПРАВЛЕНИЕ БЛОКИРОВКАМИ]]
[[postgresql.conf#^psql-conf-compatibility-ver-platform|СОВМЕСТИМОСТЬ ВЕРСИЙ И ПЛАТФОРМ]]
[[postgresql.conf#^psql-conf-debug-configure|ОБРАБОТКА ОШИБОК]]
[[postgresql.conf#^psql-conf-add-confd-files|ПОДКЛЮЧЕНИЕ ДОПОЛНИТЕЛЬНЫХ ФАЙЛОВ КОНФИГУРАЦИИ]]

#### postgresql.conf - ^psql-conf-info
```sh
# -----------------------------
# Файл конфигурации PostgreSQL
# -----------------------------
#
# Этот файл состоит из строк вида:
#
#   name = value
#
# (Символ "=" является необязательным.) Пробелы допускаются. Комментарии начинаются с "#"
# в любом месте строки. Полный список имён параметров и допустимых значений можно найти
# в документации PostgreSQL.
#
# Закомментированные настройки в этом файле отражают значения по умолчанию.
# Простое повторное закомментирование параметра НЕ возвращает его к значению по умолчанию;
# необходимо перезагрузить сервер.
#
# Этот файл считывается при запуске сервера и при получении сигнала SIGHUP.
# Если вы редактируете файл на работающей системе, необходимо отправить серверу сигнал SIGHUP,
# выполнить "pg_ctl reload" или "SELECT pg_reload_conf()", чтобы изменения вступили в силу.
# Некоторые параметры, отмеченные ниже, требуют остановки и перезапуска сервера.
#
# Любой параметр также может быть задан как опция командной строки сервера, например:
# "postgres -c log_connections=all". Некоторые параметры можно изменить во время выполнения
# с помощью команды SQL "SET".
#
# Единицы памяти:  B  = байты             Единицы времени:  us  = микросекунды
#                 kB = килобайты                       ms  = миллисекунды
#                 MB = мегабайты                       s   = секунды
#                 GB = гигабайты                       min = минуты
#                 TB = терабайты                       h   = часы
#                                                      d   = дни
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### РАСПОЛОЖЕНИЕ ФАЙЛОВ ^psql-conf-locatefile
```sh
#------------------------------------------------------------------------------
# РАСПОЛОЖЕНИЕ ФАЙЛОВ
#------------------------------------------------------------------------------
# Значения по умолчанию для этих переменных определяются опцией командной строки -D
# или переменной окружения PGDATA, здесь обозначенной как ConfigDir.
data_directory = '/var/lib/postgresql/18/main'		# использовать данные из другого каталога
# (требуется перезапуск)
hba_file = '/etc/postgresql/18/main/pg_hba.conf'	# файл аутентификации на основе хоста
# (требуется перезапуск)
ident_file = '/etc/postgresql/18/main/pg_ident.conf'	# файл конфигурации ident
# (требуется перезапуск)
# Если external_pid_file не задан явно, дополнительный PID-файл не создаётся.
external_pid_file = '/var/run/postgresql/18-main.pid'			# записать дополнительный PID-файл
# (требуется перезапуск)
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### ПОДКЛЮЧЕНИЯ И АУТЕНТИФИКАЦИЯ ^psql-conf-connect-and-auth
```sh
#------------------------------------------------------------------------------
# ПОДКЛЮЧЕНИЯ И АУТЕНТИФИКАЦИЯ
#------------------------------------------------------------------------------
# - Настройки подключения -
#listen_addresses = 'localhost'		# IP-адрес(а), на которых осуществляется прослушивание;
# список адресов, разделённых запятыми;
# по умолчанию 'localhost'; используйте '*' для всех
# (требуется перезапуск)
port = 5432				# (требуется перезапуск)
max_connections = 100			# (требуется перезапуск)
#reserved_connections = 0		# (требуется перезапуск)
#superuser_reserved_connections = 3	# (требуется перезапуск)
unix_socket_directories = '/var/run/postgresql' # список каталогов, разделённых запятыми
# (требуется перезапуск)
#unix_socket_group = ''			# (требуется перезапуск)
#unix_socket_permissions = 0777		# начинайте с 0 для восьмеричной записи
# (требуется перезапуск)
#bonjour = off				# рекламировать сервер через Bonjour
# (требуется перезапуск)
#bonjour_name = ''			# по умолчанию используется имя компьютера
# (требуется перезапуск)
# - Настройки TCP -
# см. "man tcp" для подробностей
#tcp_keepalives_idle = 0		# TCP_KEEPIDLE, в секундах;
# 0 означает использовать значение по умолчанию в системе
#tcp_keepalives_interval = 0		# TCP_KEEPINTVL, в секундах;
# 0 означает использовать значение по умолчанию в системе
#tcp_keepalives_count = 0		# TCP_KEEPCNT;
# 0 означает использовать значение по умолчанию в системе
#tcp_user_timeout = 0			# TCP_USER_TIMEOUT, в миллисекундах;
# 0 означает использовать значение по умолчанию в системе
#client_connection_check_interval = 0	# интервал между проверками разрыва соединения клиента
# во время выполнения запросов;
# 0 — никогда не проверять
# - Аутентификация -
#authentication_timeout = 1min		# от 1s до 600s
#password_encryption = scram-sha-256	# scram-sha-256 или md5
#scram_iterations = 4096
#md5_password_warnings = on
#oauth_validator_libraries = ''	# список доверенных модулей-валидаторов через запятую
# GSSAPI через Kerberos
#krb_server_keyfile = 'FILE:${sysconfdir}/krb5.keytab'
#krb_caseins_users = off
#gss_accept_delegation = off
# - SSL -
ssl = on
#ssl_ca_file = ''
ssl_cert_file = '/etc/ssl/certs/ssl-cert-snakeoil.pem'
#ssl_crl_file = ''
#ssl_crl_dir = ''
ssl_key_file = '/etc/ssl/private/ssl-cert-snakeoil.key'
#ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL'	# разрешённые шифры TLSv1.2
#ssl_tls13_ciphers = ''	# разрешённые наборы шифров TLSv1.3, оставьте пустым для значений по умолчанию
#ssl_prefer_server_ciphers = on
#ssl_groups = 'X25519:prime256v1'
#ssl_min_protocol_version = 'TLSv1.2'
#ssl_max_protocol_version = ''
#ssl_dh_params_file = ''
#ssl_passphrase_command = ''
#ssl_passphrase_command_supports_reload = off
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### ИСПОЛЬЗОВАНИЕ РЕСУРСОВ ^psql-conf-resurse-using
```sh
#------------------------------------------------------------------------------
# ИСПОЛЬЗОВАНИЕ РЕСУРСОВ (кроме WAL)
#------------------------------------------------------------------------------

# - Память -
shared_buffers = 128MB			# минимум 128kB # 25% от RAM (но не более 8–16 ГБ на больших серверах)
# (требуется перезапуск)
#huge_pages = try			# on, off или try # **огромные страницы памяти** (Huge Pages) для `shared_buffers` # Уменьшает overhead TLB, повышает производительность при больших `shared_buffers`.
# (требуется перезапуск)
#huge_page_size = 0			# ноль — использовать значение по умолчанию в системе # (например, `2MB`, `1GB`)
# (требуется перезапуск)
#temp_buffers = 8MB			# минимум 800kB
temp_buffers = 8MB			# минимум 800kB
	# Память на **сессию** для временных таблиц (`CREATE TEMP TABLE`).  
	# Выделяется **только при первом использовании**.  
	# Не влияет на обычные таблицы.  
	# **Можно менять "на лету"** (но применяется только в новых сессиях).
#max_prepared_transactions = 0		# ноль отключает функцию
# (требуется перезапуск)
# Внимание: не рекомендуется устанавливать max_prepared_transactions отличным от нуля,
# если вы не планируете активно использовать подготовленные транзакции.
#work_mem = 4MB				# минимум 64kB
work_mem = 4MB				# минимум 64kB
	# Память на **одну операцию** (сортировка, хеширование, bitmap scan).  
	# **Не на запрос и не на сессию!**  
	# Пример: запрос с 4 `ORDER BY` + 2 `HASH JOIN` может использовать до `6 × work_mem`.  
	# **Слишком высокое значение → OOM!**  
	# **Можно менять на лету**.
#hash_mem_multiplier = 2.0		# множитель от 1 до 1000.0 для work_mem хеш-таблиц
	# Множитель для `work_mem` **только для хеш-таблиц**.  
	# Позволяет хеш-операциям использовать больше памяти, чем сортировки.  
	# Диапазон: `1.0`–`1000.0`.  
	# Полезен при тяжёлых `JOIN` и `GROUP BY`.
#maintenance_work_mem = 64MB		# минимум 64kB
	# Память для **обслуживающих операций**: `VACUUM`, `CREATE INDEX`, `ALTER TABLE`, `CLUSTER`.  
	# **На процесс**, не на сессию.  
	# Рекомендация: 1–2 ГБ на сервер с 16+ ГБ RAM.  
	# **Можно менять на лету**.
#autovacuum_work_mem = -1		# минимум 64kB, или -1 для использования maintenance_work_mem
	# Память для **автоматического vacuum**.  
	# `-1` → использовать `maintenance_work_mem`.  
	# Явное значение — если хотите ограничить autovacuum отдельно.  
	# **Можно менять на лету**.
#logical_decoding_work_mem = 64MB	# минимум 64kB
	# Память для **буферизации изменений** при логической декодировке (логическая репликация).  
	# Увеличение ускоряет обработку, но повышает потребление RAM.  
	# **Можно менять на лету**.
#max_stack_depth = 2MB			# минимум 100kB  # Ограничение на **стек вызовов** backend-процесса.
#shared_memory_type = mmap		# по умолчанию используется первый поддерживаемый
# операционной системой вариант:
#   mmap — через файловый mapping,
#   sysv — классический (System V),
#   windows — только на Windows.
# (требуется перезапуск)
dynamic_shared_memory_type = posix	# по умолчанию обычно используется первый
# поддерживаемый операционной системой вариант:
#   posix
#   sysv
#   windows
#   mmap
# (требуется перезапуск)
#min_dynamic_shared_memory = 0MB	# (требуется перезапуск)
	# Минимальный объём **динамической shared memory**, выделяемой при старте.
	# `0` — выделять по мере необходимости.
	# Полезно, если много параллельных запросов → уменьшает задержки.
#vacuum_buffer_usage_limit = 2MB	# размер кольца буфера стратегии доступа при VACUUM и ANALYZE;
# 0 — отключить стратегию буфера VACUUM;
# диапазон от 128kB до 16GB

# Буферы SLRU (требуется перезапуск)
	#Эти параметры управляют **кэшированием служебных WAL-подобных данных** в shared memory.  
	#Увеличение снижает чтение с диска для системных операций.
#commit_timestamp_buffers = 0		# память для pg_commit_ts (0 = авто) # Буферы для `pg_commit_ts` (если включено `track_commit_timestamp`).
#multixact_offset_buffers = 16		# память для pg_multixact/offsets # Буферы для `pg_multixact/offsets` — мультитранзакции (SHARE-блокировки). # Увеличивайте при высокой конкуренции за строки
#multixact_member_buffers = 32		# память для pg_multixact/members # Буферы для `pg_multixact/members`
#notify_buffers = 16			# память для pg_notify # Буферы для `LISTEN`/`NOTIFY`.
#serializable_buffers = 32		# память для pg_serial # Буферы для сериализуемой изоляции (`pg_serial`).
#subtransaction_buffers = 0		# память для pg_subtrans (0 = авто) # Буферы для вложенных транзакций (`SAVEPOINT`).
#transaction_buffers = 0		# память для pg_xact (0 = авто) # Буферы для CLOG (`pg_xact`) — статусы транзакций.

# - Диск -
#temp_file_limit = -1			# ограничение на объём временных файлов на процесс
	# Макс. объём **временных файлов на процесс** (при превышении `work_mem`).  
	# Значение в **килобайтах**.  
	# Защита от заполнения диска "раздутыми" сортировками.  
	# **Можно менять на лету**.
	# в килобайтах, или -1 — без ограничений
#file_copy_method = copy		# copy, clone (если поддерживается ОС)
	# Как копировать файлы (например, при `CREATE DATABASE`):
	# - `copy` — постраничное копирование,
	# - `clone` — reflink (на Btrfs/XFS) → мгновенно и без дублирования данных.  
	#     **Требует перезапуска**.
#max_notify_queue_pages = 1048576	# ограничивает число страниц SLRU, выделяемых
	# Макс. объём очереди `LISTEN`/`NOTIFY` в страницах (8 КБ).  
	# Защита от неограниченного роста очереди.  
	# **Требует перезапуска**.

# - Ресурсы ядра -
#max_files_per_process = 1000		# минимум 64
	# Макс. число **дескрипторов файлов на процесс**.  
	# Должно быть **≥ числа таблиц + индексов + WAL + временных файлов**.  
	# **Требует перезапуска**.
# (требуется перезапуск)

# - Фоновый записывающий процесс -
#bgwriter_delay = 200ms			# от 10 до 10000 мс между раундами
	# Интервал между раундами записи.  
	# Меньше → чаще сбрасывает, но выше нагрузка.
#bgwriter_lru_maxpages = 100		# макс. число буферов, записываемых за раунд, 0 — отключить
	# Макс. число страниц, записываемых за раунд.  
	# `0` — полностью отключает background writer.
#bgwriter_lru_multiplier = 2.0		# множитель от 0 до 10.0 для количества сканируемых буферов за раунд
	# Множитель для "активности" — сколько страниц проверять за раунд.  
	# `2.0` = в 2 раза больше, чем было записано в прошлом раунде.
#bgwriter_flush_after = 512kB		# измеряется в страницах, 0 — отключить
	# Сбрасывать на диск после записи такого объёма (для SSD/NVMe).  
	# `0` — отключить.

# - Ввод-вывод -
#backend_flush_after = 0		# измеряется в страницах, 0 — отключить
	# Backend сам делает `write()` после такого объёма изменений.  
	# Полезен при высокой записи.  
	# `0` — отключить.
#effective_io_concurrency = 16		# от 1 до 1000; 0 — отключить выдачу нескольких одновременных запросов ввода-вывода
	# Сколько **асинхронных I/O-запросов** может выдавать планировщик.  
	# Для **HDD** — `1–2`, для **SSD/NVMe** — `100–200+`.  
	# Требует `posix_fadvise()` и поддержки ОС.
#maintenance_io_concurrency = 16	# от 1 до 1000; аналогично effective_io_concurrency # То же, но для `VACUUM`, `CREATE INDEX`.
#io_max_combine_limit = 128kB		# обычно от 1 до 128 блоков (зависит от ОС)
	# Оптимизация I/O для операций типа `fsync`.  
	# Зависит от ОС и диска.  
	# **Требует перезапуска**.
# (требуется перезапуск)
#io_combine_limit = 128kB		# обычно от 1 до 128 блоков (зависит от ОС)
#io_method = worker			# worker, io_uring, sync
	# Метод асинхронного I/O:
	# - `worker` — классический (background worker),
	# - `io_uring` — на Linux 5.1+ (высокая производительность),
	# - `sync` — синхронный (медленно).  
	# Требует перезапуска.
#io_max_concurrency = -1		# макс. число одновременных операций ввода-вывода на один процесс
# -1 — устанавливается на основе shared_buffers
# (требуется перезапуск)
#io_workers = 3				# от 1 до 32; Число фоновых worker’ов для асинхронного I/O.  Только при `io_method = worker`.

# - Рабочие процессы -
#max_worker_processes = 8		# (требуется перезапуск) Общее число фоновых worker’ов (параллельные запросы, логическая репликация и др.).  
#max_parallel_workers_per_gather = 2	# ограничено max_parallel_workers # Макс. число **параллельных worker’ов на один запрос** (`SELECT`).
#max_parallel_maintenance_workers = 2	# ограничено max_parallel_workers # Число worker’ов для `CREATE INDEX`, `VACUUM`.
#max_parallel_workers = 8		# число max_worker_processes, # Общее число worker’ов, которые можно использовать **в параллельных операциях**.
# которые могут использоваться в параллельных операциях
#parallel_leader_participation = on
	# Может ли **ведущий процесс** участвовать в выполнении (а не только собирать результаты).  
	# `on` — выше эффективность,  
	# `off` — меньше contention.
```
- **Не увеличивайте `work_mem` без мониторинга** — это частая причина OOM.
- Для **OLTP**: выше `shared_buffers`, ниже `work_mem`.
- Для **OLAP/аналитики**: выше `work_mem`, `effective_io_concurrency`.
- Всегда **тестируйте изменения** под реальной нагрузкой.
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### ЖУРНАЛ ПРЕДВАРИТЕЛЬНОЙ ЗАПИСИ (WAL) ^psql-conf-WAL-journal-recovery
```sh
#------------------------------------------------------------------------------
# ЖУРНАЛ ПРЕДВАРИТЕЛЬНОЙ ЗАПИСИ (WAL)
#------------------------------------------------------------------------------
# - Настройки -
#wal_level = replica			# minimal, replica или logical
# (требуется перезапуск)
#fsync = on				# сбрасывать данные на диск для защиты от сбоев
# (отключение может привести
# к необратимой порче данных)
#synchronous_commit = on		# уровень синхронизации:
# off, local, remote_write, remote_apply или on
#wal_sync_method = fsync		# по умолчанию используется первый поддерживаемый
# операционной системой метод:
#   open_datasync
#   fdatasync (по умолчанию на Linux и FreeBSD)
#   fsync
#   fsync_writethrough
#   open_sync
#full_page_writes = on			# восстанавливать после частичной записи страницы
#wal_log_hints = off			# также выполнять полную запись страниц при некритических обновлениях
# (требуется перезапуск)
#wal_compression = off			# включает сжатие полных записей страниц;
# off, pglz, lz4, zstd или on
#wal_init_zero = on			# заполнять новые WAL-файлы нулями
#wal_recycle = on			# повторно использовать WAL-файлы
#wal_buffers = -1			# минимум 32kB, -1 — устанавливается на основе shared_buffers
# (требуется перезапуск)
#wal_writer_delay = 200ms		# от 1 до 10000 миллисекунд
#wal_writer_flush_after = 1MB		# измеряется в страницах, 0 — отключить
#wal_skip_threshold = 2MB
#commit_delay = 0			# диапазон от 0 до 100000, в микросекундах
#commit_siblings = 5			# диапазон от 1 до 1000
# - Контрольные точки -
#checkpoint_timeout = 5min		# диапазон от 30s до 1d
#checkpoint_completion_target = 0.9	# целевая продолжительность контрольной точки, от 0.0 до 1.0
#checkpoint_flush_after = 256kB		# измеряется в страницах, 0 — отключить
#checkpoint_warning = 30s		# 0 — отключить
max_wal_size = 1GB
min_wal_size = 80MB
# - Предвыборка при восстановлении -
#recovery_prefetch = try	# предварительно загружать страницы, на которые есть ссылки в WAL?
#wal_decode_buffer_size = 512kB	# размер окна предварительного просмотра для предвыборки
# (требуется перезапуск)
# - Архивирование -
#archive_mode = off		# включает архивирование; off, on или always
# (требуется перезапуск)
#archive_library = ''		# библиотека, используемая для архивирования WAL-файла
# (пустая строка означает, что следует использовать archive_command)
#archive_command = ''		# команда для архивирования WAL-файла
# подстановки: %p = путь к архивируемому файлу
#              %f = только имя файла
# например: 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'
#archive_timeout = 0		# принудительно переключать WAL-файл после указанного
# количества секунд; 0 — отключить
# - Восстановление из архива -
# Используются только в режиме восстановления.
#restore_command = ''		# команда для восстановления архивированного WAL-файла
# подстановки: %p = путь к восстанавливаемому файлу
#              %f = только имя файла
# например: 'cp /mnt/server/archivedir/%f %p'
#archive_cleanup_command = ''	# команда, выполняемая в каждой точке перезапуска
#recovery_end_command = ''	# команда, выполняемая после завершения восстановления
# - Целевая точка восстановления -
# Указываются только при выполнении целенаправленного восстановления.
#recovery_target = ''		# 'immediate' — завершить восстановление сразу после
# достижения согласованного состояния
# (требуется перезапуск)
#recovery_target_name = ''	# имя точки восстановления, до которой будет выполняться восстановление
# (требуется перезапуск)
#recovery_target_time = ''	# временная метка, до которой будет выполняться восстановление
# (требуется перезапуск)
#recovery_target_xid = ''	# идентификатор транзакции, до которого будет выполняться восстановление
# (требуется перезапуск)
#recovery_target_lsn = ''	# LSN WAL, до которого будет выполняться восстановление
# (требуется перезапуск)
#recovery_target_inclusive = on	# указывает, следует ли останавливаться:
# сразу после указанной цели восстановления (on)
# сразу перед целью восстановления (off)
# (требуется перезапуск)
#recovery_target_timeline = 'latest'	# 'current', 'latest' или идентификатор временной шкалы
# (требуется перезапуск)
#recovery_target_action = 'pause'	# 'pause', 'promote', 'shutdown'
# (требуется перезапуск)
# - Суммаризация WAL -
#summarize_wal = off		# запускать ли процесс суммаризации WAL?
#wal_summary_keep_time = '10d'	# когда удалять старые файлы суммаризации, 0 — никогда
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### РЕПЛИКАЦИЯ ^psql-conf-replication-configure
```sh
#------------------------------------------------------------------------------
# РЕПЛИКАЦИЯ
#------------------------------------------------------------------------------
# - Отправляющие серверы -
# Указываются на первичном сервере и на любом резервном, который будет отправлять репликационные данные.
#max_wal_senders = 10		# максимальное количество процессов walsender
# (требуется перезапуск)
#max_replication_slots = 10	# максимальное количество репликационных слотов
# (требуется перезапуск)
#wal_keep_size = 0		# в мегабайтах; 0 — отключить
#max_slot_wal_keep_size = -1	# в мегабайтах; -1 — отключить
#idle_replication_slot_timeout = 0	# в секундах; 0 — отключить
#wal_sender_timeout = 60s	# в миллисекундах; 0 — отключить
#track_commit_timestamp = off	# собирать метки времени фиксации транзакций
# (требуется перезапуск)
# - Первичный сервер -
# Эти настройки игнорируются на резервном сервере.
#synchronous_standby_names = ''	# резервные серверы, обеспечивающие синхронную репликацию:
# метод выбора синхронных резервных серверов, количество синхронных резервных серверов
# и список application_name резервных серверов через запятую;
# '*' = все
#synchronized_standby_slots = ''	# имена слотов резервных серверов потоковой репликации,
# на которые будут ожидать логические процессы walsender
# - Резервные серверы -
# Эти настройки игнорируются на первичном сервере.
#primary_conninfo = ''			# строка подключения к отправляющему серверу
#primary_slot_name = ''			# репликационный слот на отправляющем сервере
#hot_standby = on			# "off" запрещает выполнение запросов во время восстановления
# (требуется перезапуск)
#max_standby_archive_delay = 30s	# максимальная задержка перед отменой запросов
# при чтении WAL из архива;
# -1 — разрешить неограниченную задержку
#max_standby_streaming_delay = 30s	# максимальная задержка перед отменой запросов
# при потоковом чтении WAL;
# -1 — разрешить неограниченную задержку
#wal_receiver_create_temp_slot = off	# создать временный слот, если primary_slot_name
# не указан
#wal_receiver_status_interval = 10s	# отправлять подтверждения как минимум с такой периодичностью
# 0 — отключить
#hot_standby_feedback = off		# отправлять информацию с резервного сервера для предотвращения
# конфликтов запросов
#wal_receiver_timeout = 60s		# время, в течение которого приёмник ждёт
# связи с первичным сервером
# в миллисекундах; 0 — отключить
#wal_retrieve_retry_interval = 5s	# время ожидания перед повторной попыткой
# получить WAL после неудачной попытки
#recovery_min_apply_delay = 0		# минимальная задержка применения изменений во время восстановления
#sync_replication_slots = off		# включает синхронизацию слотов на физическом резервном сервере от первичного
# - Подписчики -
# Эти настройки игнорируются на издателе.
#max_active_replication_origins = 10	# максимальное количество активных источников репликации
# (требуется перезапуск)
#max_logical_replication_workers = 4	# берётся из max_worker_processes
# (требуется перезапуск)
#max_sync_workers_per_subscription = 2	# берётся из max_logical_replication_workers
#max_parallel_apply_workers_per_subscription = 2	# берётся из max_logical_replication_workers
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### НАСТРОЙКА ЗАПРОСОВ ^psql-conf-query-settings-configure
```sh
#------------------------------------------------------------------------------
# НАСТРОЙКА ЗАПРОСОВ
#------------------------------------------------------------------------------
# - Конфигурация методов планировщика -
#enable_async_append = on
#enable_bitmapscan = on
#enable_gathermerge = on
#enable_hashagg = on
#enable_hashjoin = on
#enable_incremental_sort = on
#enable_indexscan = on
#enable_indexonlyscan = on
#enable_material = on
#enable_memoize = on
#enable_mergejoin = on
#enable_nestloop = on
#enable_parallel_append = on
#enable_parallel_hash = on
#enable_partition_pruning = on
#enable_partitionwise_join = off
#enable_partitionwise_aggregate = off
#enable_presorted_aggregate = on
#enable_seqscan = on
#enable_sort = on
#enable_tidscan = on
#enable_group_by_reordering = on
#enable_distinct_reordering = on
#enable_self_join_elimination = on
# - Константы стоимости планировщика -
#seq_page_cost = 1.0			# измеряется в произвольных единицах
#random_page_cost = 4.0			# та же шкала
#cpu_tuple_cost = 0.01			# та же шкала
#cpu_index_tuple_cost = 0.005		# та же шкала
#cpu_operator_cost = 0.0025		# та же шкала
#parallel_setup_cost = 1000.0		# та же шкала
#parallel_tuple_cost = 0.1		# та же шкала
#min_parallel_table_scan_size = 8MB
#min_parallel_index_scan_size = 512kB
#effective_cache_size = 4GB
#jit_above_cost = 100000		# выполнять JIT-компиляцию, если доступна
# и стоимость запроса выше этого значения;
# -1 — отключить
#jit_inline_above_cost = 500000		# встраивать небольшие функции, если стоимость запроса
# выше этого значения; -1 — отключить
#jit_optimize_above_cost = 500000	# использовать дорогие JIT-оптимизации, если
# стоимость запроса выше этого значения;
# -1 — отключить
# - Генетический оптимизатор запросов (GEQO) -
#geqo = on
#geqo_threshold = 12
#geqo_effort = 5			# диапазон от 1 до 10
#geqo_pool_size = 0			# выбирает значение по умолчанию на основе effort
#geqo_generations = 0			# выбирает значение по умолчанию на основе effort
#geqo_selection_bias = 2.0		# диапазон от 1.5 до 2.0
#geqo_seed = 0.0			# диапазон от 0.0 до 1.0
# - Другие опции планировщика -
#default_statistics_target = 100	# диапазон от 1 до 10000
#constraint_exclusion = partition	# on, off или partition
#cursor_tuple_fraction = 0.1		# диапазон от 0.0 до 1.0
#from_collapse_limit = 8
#jit = on				# разрешить JIT-компиляцию
#join_collapse_limit = 8		# 1 — отключает свёртывание явных JOIN-выражений
#plan_cache_mode = auto			# auto, force_generic_plan или
# force_custom_plan
#recursive_worktable_factor = 10.0	# диапазон от 0.001 до 1000000
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### ОТЧЁТНОСТЬ И ЖУРНАЛИРОВАНИЕ ^psql-conf-report-and-logging

```sh
### временно включить логирование в файл (для отладки)
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql.log'
log_statement = 'all'  # опционально, для деталей

sudo -u postgres mkdir /var/lib/postgresql/18/standby/log
```

```sh
#------------------------------------------------------------------------------
# ОТЧЁТНОСТЬ И ЖУРНАЛИРОВАНИЕ
#------------------------------------------------------------------------------
# - Куда писать журнал -
#log_destination = 'stderr'		# допустимые значения — комбинации
# stderr, csvlog, jsonlog, syslog и
# eventlog (в зависимости от платформы).
# csvlog и jsonlog требуют включения
# logging_collector.
# Используется при журналировании в stderr:
#logging_collector = off		# включить захват stderr, jsonlog и csvlog  ##### Если `logging_collector = off` → логи **только в systemd/journal**.
# в файлы журнала. Обязательно включить
# для csvlog и jsonlog.
# (требуется перезапуск)
# Используются только если logging_collector включён:
#log_directory = 'log'			# каталог для записи файлов журнала,
# может быть абсолютным или относительным к PGDATA
#log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'	# шаблон имени файла журнала,
# может содержать спецификаторы strftime()
#log_file_mode = 0600			# права создания файлов журнала,
# начинайте с 0 для восьмеричной записи
#log_rotation_age = 1d			# автоматическая ротация файлов журнала
# произойдёт по истечении этого времени. 0 — отключить.
#log_rotation_size = 10MB		# автоматическая ротация файлов журнала
# произойдёт после достижения этого объёма.
# 0 — отключить.
#log_truncate_on_rotation = off		# если включено, существующий файл журнала с тем же именем,
# что и новый, будет усечён, а не дополнен.
# Но усечение происходит только при ротации по времени,
# а не при перезапуске или ротации по размеру.
# По умолчанию — off, то есть всегда дописывать.
# Используются при журналировании в syslog:
#syslog_facility = 'LOCAL0'
#syslog_ident = 'postgres'
#syslog_sequence_numbers = on
#syslog_split_messages = on
# Используется только при журналировании в eventlog (Windows):
# (требуется перезапуск)
#event_source = 'PostgreSQL'
# - Когда писать в журнал -
#log_min_messages = warning		# значения в порядке убывания детализации:
#   debug5
#   debug4
#   debug3
#   debug2
#   debug1
#   info
#   notice
#   warning
#   error
#   log
#   fatal
#   panic
#log_min_error_statement = error	# значения в порядке убывания детализации:
#   debug5
#   debug4
#   debug3
#   debug2
#   debug1
#   info
#   notice
#   warning
#   error
#   log
#   fatal
#   panic (практически отключено)
#log_min_duration_statement = -1	# -1 — отключено, 0 — записывать все запросы
# и их длительность, >0 — только запросы,
# выполняющиеся не менее указанного числа
# миллисекунд
#log_min_duration_sample = -1		# -1 — отключено, 0 — записывать выборку запросов
# и их длительность, >0 — только выборку запросов,
# выполняющихся не менее указанного числа миллисекунд;
# доля выборки определяется log_statement_sample_rate
#log_statement_sample_rate = 1.0	# доля записываемых запросов, превышающих
# log_min_duration_sample; 1.0 — все такие запросы,
# 0.0 — никогда не записывать
#log_transaction_sample_rate = 0.0	# доля транзакций, запросы которых
# записываются независимо от длительности;
# 1.0 — записывать все запросы всех транзакций,
# 0.0 — никогда не записывать
#log_startup_progress_interval = 10s	# интервал между сообщениями о прогрессе
# для длительных операций запуска.
# 0 — отключить функцию, >0 — интервал в миллисекундах.
# - Что записывать в журнал -
#debug_print_parse = off
#debug_print_rewritten = off
#debug_print_plan = off
#debug_pretty_print = on
#log_autovacuum_min_duration = 10min	# журналировать активность autovacuum;
# -1 — отключить, 0 — записывать все действия и их длительность,
# >0 — только действия, выполняющиеся не менее этого числа миллисекунд.
#log_checkpoints = on
#log_connections = '' # журналировать аспекты установки соединения
# варианты: receipt, authentication, authorization,
# setup_durations и all — для записи всех аспектов
#log_disconnections = off
#log_duration = off # записывать длительность запроса
#log_error_verbosity = default		# краткие (terse), обычные (default) или подробные (verbose) сообщения
#log_hostname = off
log_line_prefix = '%m [%p] %q%u@%d '		# специальные значения:
#   %a = имя приложения
#   %u = имя пользователя
#   %d = имя базы данных
#   %r = удалённый хост и порт
#   %h = удалённый хост
#   %L = локальный адрес
#   %b = тип бэкенда
#   %p = идентификатор процесса
#   %P = идентификатор процесса лидера параллельной группы
#   %t = временная метка без миллисекунд
#   %m = временная метка с миллисекундами
#   %n = временная метка с миллисекундами (в виде Unix-эпохи)
#   %Q = идентификатор запроса (0, если отсутствует или не вычислен)
#   %i = тег команды
#   %e = SQL-состояние
#   %c = идентификатор сессии
#   %l = номер строки в сессии
#   %s = временная метка начала сессии
#   %v = виртуальный идентификатор транзакции
#   %x = идентификатор транзакции (0, если отсутствует)
#   %q = остановиться здесь в процессах, не связанных с сессией
#   %% = '%'
# например: '<%u%%%d> '
#log_lock_waits = off			# записывать ожидания блокировок >= deadlock_timeout
#log_lock_failures = off		# записывать сбои блокировок
#log_recovery_conflict_waits = off	# записывать ожидания конфликтов восстановления на резервном сервере
# >= deadlock_timeout
#log_parameter_max_length = -1		# при журналировании запросов ограничивать
# значения привязываемых параметров N байтами;
# -1 — выводить полностью, 0 — отключить
#log_parameter_max_length_on_error = 0	# при журналировании ошибки ограничивать
# значения привязываемых параметров N байтами;
# -1 — выводить полностью, 0 — отключить
#log_statement = 'none'			# none, ddl, mod, all
#log_replication_commands = off
#log_temp_files = -1			# записывать временные файлы, размер которых не меньше
# указанного (в килобайтах);
# -1 — отключить, 0 — записывать все временные файлы
log_timezone = 'Europe/Moscow'
# - Заголовок процесса -
cluster_name = '18/main'			# добавляется к заголовкам процессов, если не пусто
# (требуется перезапуск)
#update_process_title = on
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### СТАТИСТИКА ^psql-conf-statistics
```sh
#------------------------------------------------------------------------------
# СТАТИСТИКА
#------------------------------------------------------------------------------
# - Совокупная статистика по запросам и индексам -
#track_activities = on
#track_activity_query_size = 1024	# (требуется перезапуск)
#track_counts = on
#track_cost_delay_timing = off
#track_io_timing = off
#track_wal_io_timing = off
#track_functions = none			# none, pl, all
#stats_fetch_consistency = cache	# cache, none, snapshot
# - Мониторинг -
#compute_query_id = auto
#log_statement_stats = off
#log_parser_stats = off
#log_planner_stats = off
#log_executor_stats = off
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### VACUUM ^psql-conf-vacuum-configure
```sh
#------------------------------------------------------------------------------
# VACUUM
#------------------------------------------------------------------------------
# - Автоматическая очистка -
#autovacuum = on			# включить подпроцесс autovacuum? 'on'
# требует, чтобы track_counts также был включён.
autovacuum_worker_slots = 16	# количество выделяемых слотов рабочих процессов autovacuum
# (требуется перезапуск)
#autovacuum_max_workers = 3		# максимальное количество подпроцессов autovacuum
#autovacuum_naptime = 1min		# интервал между запусками autovacuum
#autovacuum_vacuum_threshold = 50	# минимальное количество обновлённых строк перед VACUUM
#autovacuum_vacuum_insert_threshold = 1000	# минимальное количество вставленных строк
# перед VACUUM; -1 — отключить VACUUM при вставке
#autovacuum_analyze_threshold = 50	# минимальное количество обновлённых строк перед ANALYZE
#autovacuum_vacuum_scale_factor = 0.2	# доля размера таблицы перед VACUUM
#autovacuum_vacuum_insert_scale_factor = 0.2	# доля незамороженных страниц
# перед VACUUM при вставке
#autovacuum_analyze_scale_factor = 0.1	# доля размера таблицы перед ANALYZE
#autovacuum_vacuum_max_threshold = 100000000    # максимальное количество обновлённых строк
# перед VACUUM; -1 — отключить максимум
#autovacuum_freeze_max_age = 200000000	# максимальный возраст XID перед принудительным VACUUM
# (требуется перезапуск)
#autovacuum_multixact_freeze_max_age = 400000000	# максимальный возраст multixact
# перед принудительным VACUUM
# (требуется перезапуск)
#autovacuum_vacuum_cost_delay = 2ms	# задержка стоимости VACUUM по умолчанию для autovacuum,
# в миллисекундах; -1 — использовать vacuum_cost_delay
#autovacuum_vacuum_cost_limit = -1	# ограничение стоимости VACUUM по умолчанию для autovacuum,
# -1 — использовать vacuum_cost_limit
# - Задержка VACUUM на основе стоимости -
#vacuum_cost_delay = 0			# от 0 до 100 миллисекунд (0 — отключить)
#vacuum_cost_page_hit = 1		# от 0 до 10000 кредитов
#vacuum_cost_page_miss = 2		# от 0 до 10000 кредитов
#vacuum_cost_page_dirty = 20		# от 0 до 10000 кредитов
#vacuum_cost_limit = 200		# от 1 до 10000 кредитов
# - Поведение по умолчанию -
#vacuum_truncate = on			# включить усечение после VACUUM
# - Заморозка -
#vacuum_freeze_table_age = 150000000
#vacuum_freeze_min_age = 50000000
#vacuum_failsafe_age = 1600000000
#vacuum_multixact_freeze_table_age = 150000000
#vacuum_multixact_freeze_min_age = 5000000
#vacuum_multixact_failsafe_age = 1600000000
#vacuum_max_eager_freeze_failure_rate = 0.03 # 0 — отключить активное сканирование
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### НАСТРОЙКИ ПОДКЛЮЧЕНИЙ КЛИЕНТОВ ПО УМОЛЧАНИЮ ^psql-conf-connect-client-default
```sh
#------------------------------------------------------------------------------
# НАСТРОЙКИ ПОДКЛЮЧЕНИЙ КЛИЕНТОВ ПО УМОЛЧАНИЮ
#------------------------------------------------------------------------------
# - Поведение операторов -
#client_min_messages = notice		# значения в порядке убывания детализации:
#   debug5
#   debug4
#   debug3
#   debug2
#   debug1
#   log
#   notice
#   warning
#   error
#search_path = '"$user", public'	# имена схем
#row_security = on
#default_table_access_method = 'heap'
#default_tablespace = ''		# имя табличного пространства, '' — использовать по умолчанию
#default_toast_compression = 'pglz'	# 'pglz' или 'lz4'
#temp_tablespaces = ''			# список имён табличных пространств, '' — использовать только по умолчанию
#check_function_bodies = on
#default_transaction_isolation = 'read committed'
#default_transaction_read_only = off
#default_transaction_deferrable = off
#session_replication_role = 'origin'
#statement_timeout = 0				# в миллисекундах, 0 — отключить
#transaction_timeout = 0			# в миллисекундах, 0 — отключить
#lock_timeout = 0				# в миллисекундах, 0 — отключить
#idle_in_transaction_session_timeout = 0	# в миллисекундах, 0 — отключить
#idle_session_timeout = 0			# в миллисекундах, 0 — отключить
#bytea_output = 'hex'			# hex или escape
#xmlbinary = 'base64'
#xmloption = 'content'
#gin_pending_list_limit = 4MB
#createrole_self_grant = ''		# set и/или inherit
#event_triggers = on
# - Локаль и форматирование -
datestyle = 'iso, mdy'
#intervalstyle = 'postgres'
timezone = 'Europe/Moscow'
#timezone_abbreviations = 'Default'	# выбрать набор доступных сокращений часовых поясов.
# Сейчас доступны:
#   Default
#   Australia (историческое использование)
#   India
# Вы можете создать собственный файл в
# share/timezonesets/.
#extra_float_digits = 1			# минимум -15, максимум 3; любое значение >0 фактически
# включает точный режим вывода
#client_encoding = sql_ascii		# на самом деле по умолчанию используется кодировка базы данных
# Эти настройки инициализируются initdb, но могут быть изменены.
lc_messages = 'en_US.UTF-8'		# локаль для системных сообщений об ошибках
lc_monetary = 'en_US.UTF-8'		# локаль для форматирования денежных значений
lc_numeric = 'en_US.UTF-8'		# локаль для форматирования чисел
lc_time = 'en_US.UTF-8'			# локаль для форматирования времени
#icu_validation_level = warning		# сообщать об ошибках проверки ICU-локали
# на указанном уровне
# конфигурация текстового поиска по умолчанию
default_text_search_config = 'pg_catalog.english'
# - Предварительная загрузка разделяемых библиотек -
#local_preload_libraries = ''
#session_preload_libraries = ''
#shared_preload_libraries = ''		# (требуется перезапуск)
#jit_provider = 'llvmjit'		# используемая JIT-библиотека
# - Прочие настройки по умолчанию -
#dynamic_library_path = '$libdir'
#extension_control_path = '$system'
#gin_fuzzy_search_limit = 0
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### УПРАВЛЕНИЕ БЛОКИРОВКАМИ ^psql-conf-lock-manager-configure
```sh
#------------------------------------------------------------------------------
# УПРАВЛЕНИЕ БЛОКИРОВКАМИ
#------------------------------------------------------------------------------
#deadlock_timeout = 1s
#max_locks_per_transaction = 64		# минимум 10
# (требуется перезапуск)
#max_pred_locks_per_transaction = 64	# минимум 10
# (требуется перезапуск)
#max_pred_locks_per_relation = -2	# отрицательные значения означают
# (max_pred_locks_per_transaction
#  / -max_pred_locks_per_relation) - 1
#max_pred_locks_per_page = 2		# минимум 0
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### СОВМЕСТИМОСТЬ ВЕРСИЙ И ПЛАТФОРМ ^psql-conf-compatibility-ver-platform
```sh
#------------------------------------------------------------------------------
# СОВМЕСТИМОСТЬ ВЕРСИЙ И ПЛАТФОРМ
#------------------------------------------------------------------------------
# - Предыдущие версии PostgreSQL -
#array_nulls = on
#backslash_quote = safe_encoding	# on, off или safe_encoding
#escape_string_warning = on
#lo_compat_privileges = off
#quote_all_identifiers = off
#standard_conforming_strings = on
#synchronize_seqscans = on
# - Другие платформы и клиенты -
#transform_null_equals = off
#allow_alter_system = on
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### ОБРАБОТКА ОШИБОК ^psql-conf-debug-configure
```sh
#------------------------------------------------------------------------------
# ОБРАБОТКА ОШИБОК
#------------------------------------------------------------------------------
#exit_on_error = off			# завершать сессию при любой ошибке?
#restart_after_crash = on		# переинициализировать после сбоя бэкенда?
#data_sync_retry = off			# повторять или завершать работу при невозможности выполнить fsync
# для данных?
# (требуется перезапуск)
#recovery_init_sync_method = fsync	# fsync, syncfs (Linux 5.8+)
```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]
###### ПОДКЛЮЧЕНИЕ ДОПОЛНИТЕЛЬНЫХ ФАЙЛОВ КОНФИГУРАЦИИ ^psql-conf-add-confd-files
```sh
#------------------------------------------------------------------------------
# ПОДКЛЮЧЕНИЕ ДОПОЛНИТЕЛЬНЫХ ФАЙЛОВ КОНФИГУРАЦИИ
#------------------------------------------------------------------------------
# Эти опции позволяют загружать настройки из файлов, отличных от
# стандартного postgresql.conf. Обратите внимание, что это директивы,
# а не присваивания переменных, поэтому их можно указывать несколько раз.
include_dir = 'conf.d'			# подключать файлы с расширением '.conf' из
# указанного каталога, например 'conf.d'
#include_if_exists = '...'		# подключить файл, только если он существует
#include = '...'			# подключить файл
#------------------------------------------------------------------------------
# ПОЛЬЗОВАТЕЛЬСКИЕ НАСТРОЙКИ
#------------------------------------------------------------------------------
# Добавьте сюда настройки для расширений

```
[[postgresql.conf#^postgres-genegal-config| к содержанию]]