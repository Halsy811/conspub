
СОДЕРЖАНИЕ
1. [[psql - Install|Установка]] +
2. АРХИТЕКТУРА PostgreSQL ([[Структурная схема psql.drawio]])
	- [[Общая информация]] +
[[Postmaster process|Postmaster process]] +
[[Backend processes|Backend processes]] +
[[Shared memory]] +
Subprocesses Postmaster:
	[[Checkpointer|Checkpointer]] +
	[[Background writer|Background writer]] +
	[[WAL writer|WAL writer]] +
	[[WAL archiver|WAL archiver]] +
	[[WAL sender|WAL sender]] +
	[[Syslogger|Syslogger]] +
	[[Autovacuum launcher - Autovacuum worker|Autovacuum launcher - Autovacuum worker]] +
		[[Autovacuum launcher - Autovacuum worker#^psql-autovacuum-worker|Autovacuum worker]] +
	[[Startup process - WAL receiver (standby)|Startup process]] (standby) +
		[[Startup process - WAL receiver (standby)#^psql-WAL-receiver|WAL receiver]] (standby) +
	[[Logical replication launcher (standby) - Logical replication worker|Logical replication launcher]] (standby) +
		[[Logical replication launcher (standby) - Logical replication worker#^psql-logical-replication-worker|Logical replication worker]] (standby) = Logical Apply Worker + Table Sync Worker +
	[[Stats collector|Stats collector]] +
	[[Еще подпроцессы|Еще подпроцессы:]] +
		Two-phase commit (2PC) recovery processes (условно)
		Parallel worker (условный процесс)

Структура каталогов кластера (etc, PGDATA) + ^back-content-configiration-PGDATA
	[[configiration, PGDATA#^conf-cluster-psql|Каталог конфигурация кластера /etc/postgresql]] - Древо файлов и каталогов, их применение и особенности
	[[configiration, PGDATA#^pgdata-cluster-cotalog-psql|Каталог данных кластера PGDATA]] - Древо файлов и каталогов
	[[configiration, PGDATA#^other-cotalog-psql|Другие каталоги]] - Файлы с PID процессом и др.
	Файлы конфигурации:
		[[configiration, PGDATA#^conf-cluster-environment|/etc/psql/.../environment]] - это **переменные окружения**, которые устанавливаются **для процесса `postmaster` и всех его дочерних процессов** (backend’ов, фоновых процессов и т.д.).
		[[configiration, PGDATA#^conf-cluster-pgctl-conf|/etc/psql/.../pg_ctl.conf]] - альтернатива `systemd`
		[[configiration, PGDATA#^conf-cluster-pghba-conf|/etc/psql/.../pg_hba.conf]] - Кто, откуда и как может подключиться к каким базам данных в кластере
		[[configiration, PGDATA#^conf-cluster-pgident-conf|/etc/psql/.../pg_ident.conf]] -  **файл отображения имён пользователей ОС в роли PostgreSQL**, используемый **только при аутентификации методом `ident`** (для TCP-подключений) или **`peer`** (для локальных Unix-сокетов, хотя для `peer` он обычно не нужен).
			[[configiration, PGDATA#^psql-conf-file-postgresql-conf|/etc/psql/.../postgresql.conf]] - postgresql.conf первичная информация - основной конфигурационный файл
				[[postgresql.conf default major-18]] - default конфиг [[configiration, PGDATA#^reboot-conf-psql|Применение изменений]]
				[[postgresql.conf|postgresql.conf info]] - Подробное описание настроек
				[[configiration, PGDATA#^psql-conf-confd-files|/etc/psql/.../conf.d]] - Доп. файлы конфигурации. Приоритеты файлов конфигурации
				[[configiration, PGDATA#^conf-psql-postgresqlauto-ALTER-SYSTEM-command|/var/lib/psql/.../postgresql.auto.conf]] - postgresql.auto.conf - ==✅ Это **рекомендуемый способ** изменения глобальных параметров в production.== Так же меняется через SQL-команду ==ALTER SYSTEM==
		[[configiration, PGDATA#^conf-cluster-start-conf|/etc/psql/.../start.conf]] - Файл содержит **ровно одно слово** — режим автозапуска: auto

		[[Принцип работы - WAL]] +
		[[MVCC механизм, tuple]] +
		[[Принцип работы - VACUUM, autoVACUUM]] +
		[[Принцип работы - ANALYZE (pg_statistic)]] +
		[[Принцип работы - Planner (планировщик)]] +

		Логические схемы простых операций (Select, Insert, Update)
			Select ([[Структурная схема psql.drawio|см. на схеме]]) +
			Insert ([[Структурная схема psql.drawio|см. на схеме]]) +
			Update ([[Структурная схема psql.drawio|см. на схеме]]) +

РЕПЛИКАЦИЯ в PostgreSQL
	Виды репликации:
		[[Физическая (STREAMING) репликация]] - синхронная/асинхронная +
			[[Настройка streaming репликации]] +
				[[Мониторинг streaming репликации]] +
		[[Логическая (PUBLICATION) репликация]]+
			[[Настройка publication репликации]]
				[[Мониторинг publication репликации]] +
			[[Сравнение STREAMING & PUBLICATION]] +

РЕЗЕРВНОЕ КОПИРОВАНИЕ в PostgreSQL (Backup)
	[[Физический (pg_basebackup) vs Логический (pg_dump)]] +
	[[Архивация WAL, Point-in-Time Recovery (PITR)]] +
		[[Архивация WAL, Point-in-Time Recovery (PITR)#^psql-PITR-WALarchiveRestore|Восстановление из Archive-WAL до определенной точки с помощью PITR]] +
		[[Скрипт автоматического backup и восстановление PITR]]
	Утилиты: 
		[[pg_basebackup]]  — Применение. Физический backup +
		[[pg_dump, pg_dumpall, pg_restore]] — Применение. Логический backup. Частичный backup +

	pg_upgrade — для быстрого обновления кластера между [[Общая информация#^psql-major-minor-ver|major-версиями]]

[[RoadMap - Commands|Навигация по командам]] - RoadMap
	[[Системные команды администрирования PostgreSQL]] - на уровне операционной системы
	[[DDL]], [[DML]], [[DCL]] - SQL
	[[Мета-команды psql]] - `\?`
	[[Базовое АДМИНИСТРИРОВАНИЕ PostgreSQL]] - необходимый минимум

[[RoadMap - Табличное пространство psql|Табличное пространство psql]] - <span class="rem">?</span>

---

МИГРАЦИЯ
	[[Миграция с MySQL на psql - Pgloader]] +
	Миграция между версиями. PG 14 → PG 16 с минимальным downtime (временем отключения).

	Распределение нагрузки, Проксирование
		[[PgPool-II - Определение|PgPool-II - что это?]] +
			[[PgPool-II - Балансировка нагрузки|Балансировка нагрузки (Load Balancing) / Чтение с реплик (Read Scaling)]]
			[[PgPool-II - Пул соединений|Пул соединений (Connection Pooling)]]
			[[PgPool-II - Автоматический переход на резервную систему|Автоматический переход на резервную систему (Failover)]]
			[[PgPool-II - Кэширование запросов|Кэширование запросов (Query Caching)]]

---

[[Дополнительная информация psql - Ссылки на ресурсы]]

---
# == Задачки на прокачку

- [ ] [[pg_basebackup#^psql-pgbasebackup-coldbackup|pg_basebackup - Сделать холодный физический backup кластера]] +
- [ ] [[pg_basebackup#^psql-pgbasebackup-straightStandby|pg_basebackup - Развернуть Standby используя]] +
- [ ] [[Базовое АДМИНИСТРИРОВАНИЕ PostgreSQL#^psql-test-replication|Проверить репликацию]] +
- [ ] [[Базовое АДМИНИСТРИРОВАНИЕ PostgreSQL#^psql-remove-Standby|Удалить Standby, отключить его и почистить конфиги]] +

- [ ] Настроить Физическую (streaming) репликацию (без WAL archiving и Repl. slot) и промониторить 
	- [ ] [[Настройка streaming репликации#^psql-phRepl-not-SlotAndWAL|Без WAL archiving и Replication slots]] +
	- [ ] [[Настройка streaming репликации#^psql-phRepl-add-slot|Добавить Physical Replication Slot]] +
	- [ ] [[Настройка streaming репликации#^psql-phRepl-add-walArch|Добавить WAL archive]] +
	- [ ] [[Настройка streaming репликации#^psql-phRepl-clear-streamingReplConf|Отключение streaming репликации]] +

- [ ] Настроить Логическую (publication) репликацию и промониторить
	- [ ] [[Настройка publication репликации#^psql-unification-data-in-standby|Объединение данных из нескольких источников]] +
	- [ ] [[Настройка publication репликации#^psql-repl-onlyone-tables-chem|Реплика одной определенной таблицы]] +

- [ ] Организовать автоматическое резервное копирование на основе "pg_basebackup", Архивации WAL" и протестировать восстановление "PITR"
	- [ ] [[Скрипт автоматического backup и восстановление PITR]]

- [ ] Как **вручную прочитать `t_xmin`/`t_xmax`** через системный каталог `heap_page_items`?
- [ ] Как **отследить visibility map** и понять, почему `VACUUM` пропустил страницу?
- [ ] Или как **ограничение 2^32 XID** приводит к **wraparound**, и почему `VACUUM` обязателен?
- [ ] Как **мониторить прогресс checkpoint’а** через `pg_stat_bgwriter`,
- [ ] Или как **настроить checkpoint’ы под SSD/NVMe**?
- [ ] Как **настроить autovacuum для таблицы с 1 млрд строк**,
- [ ] Или как **принудительно запустить freeze vacuum**?

--- 

# Ответы на вопросы

[[DML#^psql-DML-hit-ratio|Коэффициент попаданий в кэш]]
Сколько сейчас [[DML#^psql-DML-ActiveSessions|активных сеансов]] к базе и к кластеру в общем
[[DML#^psql-DML-getSession-withTMPTabs|Количество сессий]], в которых создана хотя бы одна временная таблица
[[DML#^psql-DML-Create-tmpFiles|Сколько временных файлов создаётся]]
[[DML#^psql-DML-tmpTabsize|Анализ размеров временных таблиц]]
Просмотреть [[DML#^psql-DML-phuse-disk-and-ram|физическое использование диска и памяти на уровне ОС]]
[[DML#^psql-DML-checkVacuum|Проверить, что VACUUM работает]]


