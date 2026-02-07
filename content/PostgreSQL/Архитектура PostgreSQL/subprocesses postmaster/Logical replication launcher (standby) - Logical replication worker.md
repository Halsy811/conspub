
# Logical replication launcher

Фоновый процесс PostgreSQL, который **управляет запуском рабочих процессов логической репликации (logical replication workers)** на сервере-подписчике (**subscriber**).

Это «диспетчер» логической репликации, аналог **Autovacuum Launcher** для подписок.

> Он **не передаёт данные**, а **координирует выполнение подписок**, определённых через `CREATE SUBSCRIPTION`.

### Главная задача

> **Периодически проверять все активные подписки (`pg_subscription`) и запускать для них рабочие процессы**, если они не работают или требуют синхронизации.

### Как это работает (по шагам)

1. **При старте кластера** (если есть хотя бы одна подписка) **postmaster** запускает **один процесс `logical replication launcher`**.
2. **Launcher каждые `wal_retrieve_retry_interval`** (по умолчанию — 5 сек) проверяет:
    - Есть ли активные подписки (`pg_subscription.active = true`),
    - Работают ли для них **apply worker’ы**,
    - Требуется ли **синхронизация таблиц** (initial sync).
3. **Если нужно** — он запускает:
    - **`Logical Apply Worker`** — для применения изменений из потока репликации,
    - **`Table Sync Worker`** — для начальной синхронизации таблиц (если `synchronous_commit = 'off'` или таблица ещё не синхронизирована).

- Автоматически, если в кластере есть хотя бы одна строка в `pg_subscription`.
- Если все подписки удалены — **launcher завершается** (в новых версиях) или **спит бесконечно**.
- **Не передаёт данные** — только управляет worker’ами.
- При падении worker’а **автоматически перезапускает его** (с экспоненциальной задержкой).
- Если publisher недоступен — **worker засыпает**, launcher периодически пытается перезапустить.
### Ключевые параметры

| Параметр                            | По умолчанию | Роль                                                                                                                                                |
| ----------------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `max_logical_replication_workers`   | `4`          | Макс. число **apply worker’ов** (включая sync). <br><br>Общее число worker’ов:  <br>**1 launcher + до `max_logical_replication_workers` worker’ов** |
| `max_sync_workers_per_subscription` | `2`          | Макс. число **table sync worker’ов на подписку**                                                                                                    |
| `wal_retrieve_retry_interval`       | `5s`         | Как часто launcher проверяет подписки                                                                                                               |
1. На **subscriber’е** создаётся запись в `pg_subscription`.
2. **Logical Replication Launcher** обнаруживает новую подписку.
3. Запускает:
    - **Table Sync Worker** для каждой таблицы в публикации (если нужно),
    - **Logical Apply Worker**, который:
        - Подключается к publisher’у,
        - Получает поток логических изменений,
        - Применяет `INSERT`/`UPDATE`/`DELETE` локально.

|Процесс|Роль|
|---|---|
|**Logical Replication Launcher**|Диспетчер: запускает и следит за worker’ами|
|**Logical Apply Worker**|Применяет поток изменений от publisher’а|
|**Table Sync Worker**|Выполняет начальную синхронизацию таблиц|

# Logical replication worker ^psql-logical-replication-worker

**«Logical replication worker»** — это общее название.  
	**«Logical Apply Worker»** — Конкретный тип logical replication worker’а, который применяет изменения из потока репликации.
	**«Table Sync Worker»** — Второй тип (для начальной синхронизации)

В `ps aux` оба будут отображаться как:
```
postgres: logical replication worker
```

Но в представлении `pg_stat_subscription` можно увидеть разницу:
```sql
SELECT 
  subname,
  pid,
  relid,
  received_lsn,
  last_msg_send_time
FROM pg_stat_subscription;
```

- Если `relid IS NULL` → это **Apply Worker** (работает с потоком, а не с конкретной таблицей).
- Если `relid IS NOT NULL` → это **Table Sync Worker** для таблицы с OID = `relid`.

### Жизненный цикл подписки

1. `CREATE SUBSCRIPTION ...`
2. **Logical Replication Launcher** запускает:
    - **Table Sync Worker** для каждой таблицы → копирует данные.
3. После синхронизации таблицы:
    - **Table Sync Worker завершается**.
4. **Apply Worker** запускается (или уже работает) → начинает применять **реальные изменения в реальном времени**.

> На одну подписку - **ровно один Apply Worker**, но **много Table Sync Worker’ов** (параллельно по таблицам, если разрешено `max_sync_workers_per_subscription`).

Общее число **всех logical replication worker’ов** (apply + sync) ограничено:
```sh
max_logical_replication_workers = 4  # по умолчанию
```
Число **одновременных sync-воркеров на одну подписку**:
```sh
max_sync_workers_per_subscription = 2
```


#### То есть: 

1. **Logical replication launcher** - **Запускается `postmaster`'ом** при старте кластера, **если есть хотя бы одна подписка** в `pg_subscription`.

2. **Table Sync Worker** - **Запускается Logical Replication Launcher’ом** (через `postmaster`) **для каждой таблицы**, требующей начальной синхронизации.
- **Копирует данные напрямую из publisher’а в локальные таблицы подписчика** через SQL-запросы (`COPY ...` или `SELECT`).
- **Данные пишутся сразу в настоящие таблицы подписчика** (в `base/.../`), **не во временную директорию**!
- Использует **специальную логику блокировок**, чтобы не мешать Apply Worker’у.

1. **Apply Worker** - **Запускается Logical Replication Launcher’ом** сразу после создания подписки (или при старте кластера).
	- **Но сначала "спит"**, пока Table Sync Worker не завершит синхронизацию таблицы.
	- После завершения синхронизации **начинает применять поток изменений (INSERT/UPDATE/DELETE)** из логической репликации.
	- **Тоже пишет напрямую в конечные таблицы** (в `base/.../`).

**Как это работает на уровне данных?**
Допустим, у вас есть таблица `public.users` на **publisher’е**.
1. **Table Sync Worker**:
    - Делает `COPY public.users TO STDOUT` на publisher’е,
    - Получает данные по сети,
    - Выполняет `COPY public.users FROM STDIN` на **subscriber’е** → данные попадают в файлы в `base/<OID_БД>/<RELFILENODE>`.
2. **Apply Worker**:
    - Получает логические изменения: `INSERT INTO public.users VALUES (...)`,
    - Выполняет их как обычный SQL → те же файлы `base/...`.

Оба worker’а **модифицируют одни и те же физические файлы таблицы** — но **никогда одновременно для одной таблицы**.  
PostgreSQL использует **флаги в `pg_subscription_rel`**, чтобы избежать конфликта.

> Про "директорию" — важное уточнение
> 	- **Нет промежуточного хранилища**.
> 	- Данные **сразу пишутся в конечные таблицы** в каталоге данных кластера:
> 		`/var/lib/postgresql/18/main/base/<OID_базы>/<RELFILENODE>`
> 	- Это **обычные файлы таблицы**, такие же, как при `INSERT` вручную.