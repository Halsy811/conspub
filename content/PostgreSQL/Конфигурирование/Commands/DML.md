
# Анализ производительности кластера
### Оцените hit ratio (коэффициент попаданий в кэш) ^psql-DML-hit-ratio

- показывающая, **насколько часто запрашиваемые данные находятся в кэше**, а не приходится читать их с более медленного носителя (например, с диска).
```sql
SELECT
    datname,
    blks_read,
    blks_hit,
    round(blks_hit::numeric / (blks_hit + blks_read) * 100, 2) AS hit_ratio_percent
FROM pg_stat_database
WHERE datname NOT IN ('template0', 'template1')
ORDER BY datname;
```
- **> 99%** — отличный результат, кэш работает эффективно.
- **95–99%** — приемлемо.

### Активные сессии ^psql-DML-ActiveSessions
```sql
SELECT * FROM pg_stat_activity;
```

### Количество сессий, в которых создана хотя бы одна временная таблица ^psql-DML-getSession-withTMPTabs
```sql
SELECT COUNT(DISTINCT a.pid) AS sessions_with_temp_tables
FROM pg_stat_activity a
JOIN pg_class c
  ON c.relowner = a.usesysid
  AND c.relnamespace IN (
      SELECT oid
      FROM pg_namespace
      WHERE nspname LIKE 'pg_temp_%'
  )
WHERE a.backend_type = 'client backend'
  AND c.relpersistence = 't';
```

### Сколько временных файлов создаётся ^psql-DML-Create-tmpFiles
```sql
SELECT
    datname,
    temp_files,
    pg_size_pretty(temp_bytes) AS temp_bytes_pretty
FROM pg_stat_database
WHERE datname NOT IN ('template0', 'template1');
```
- Если `temp_files > 0` и растёт — возможно, не хватает **`work_mem`**, а не `temp_buffers`.
- `temp_buffers` влияет **только на буферизацию страниц временных таблиц**, но не на сортировки/хеши.

### Анализ размеров временных таблиц ^psql-DML-tmpTabsize
```sql
-- Показать все временные таблицы в текущей сессии
SELECT
    n.nspname AS schema,
    c.relname AS table_name,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS size,
    pg_total_relation_size(c.oid) AS size_bytes
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relpersistence = 't'
  AND n.nspname LIKE 'pg_temp_%';
```

```sql
-- Вне сеанса придется анализировать косвенно
```

Так же через OS FS
```sh
# Найти временные файлы PostgreSQL
ls -lh $PGDATA/base/*/pgsql_tmp/

# - Отсутствие файлов в `pgsql_tmp` не означает, что временных таблиц нет.
# - Наличие больших файлов — сигнал, что `temp_buffers` может быть слишком мал.
```
### Физическое использование диска и памяти на уровне ОС ^psql-DML-phuse-disk-and-ram
```sql
SELECT
    query,
    reads,      -- чтение с диска (блоков)
    writes,       -- запись на диск
    user_time,
    system_time
FROM pg_stat_kcache
WHERE query ILIKE '%create%temp%table%';
```
- Высокие `writes` могут указывать на то, что данные не помещаются в память (возможно, мал `temp_buffers` или `work_mem`).

# Проверить, что VACUUM работает ^psql-DML-checkVacuum
```sql
-- Статистика по таблицам
SELECT schemaname, tablename,
       n_tup_ins, n_tup_upd, n_tup_del,
       n_dead_tup,
       last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Прогресс выполнения VACUUM
SELECT * FROM pg_stat_progress_vacuum;
```