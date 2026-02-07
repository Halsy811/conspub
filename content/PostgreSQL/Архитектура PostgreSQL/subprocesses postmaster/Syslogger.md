
**Syslogger** — это фоновый процесс PostgreSQL, который **собирает все внутренние сообщения СУБД (логи)** и **записывает их в файлы** на диске, если включена опция `logging_collector = on`.

> Он **не является обязательным**, но крайне полезен для диагностики, мониторинга и аудита в production-средах.

1. Все процессы PostgreSQL (backend, checkpointer, autovacuum и др.) **пишут лог-сообщения в общий лог-буфер** (в shared memory или через IPC). **Перехватывает лог-сообщения** от всех процессов кластера:
    - backend’ов (ошибки, медленные запросы),
    - фоновых процессов (checkpointer, autovacuum, WAL writer и др.),
    - самого `postmaster` (старт, остановка, ошибки подключения).
2. **Форматирует сообщения** в соответствии с `log_line_prefix`.
3. **Записывает их в файлы** в каталоге `log_directory` (по умолчанию — `pg_log/` внутри `PGDATA` или заданный явно). (Записывает в текущий лог-файл (например, `postgresql-2025-12-24.log`),)
4. **Выполняет ротацию логов**:
    - по размеру (`log_rotation_size`),
    - по времени (`log_rotation_age`).

---

- **Syslogger запускается только если `logging_collector = on`**.
- Если вы используете **systemd**, логи могут дублироваться в `journalctl` — но файлы дают **более гибкую ротацию и архивацию**.
- **Не влияет на производительность напрямую**, но:
    - `log_statement = 'all'` → огромный I/O,
    - `log_min_duration_statement = 0` → логируется всё.
- **Не шифрует и не сжимает логи** — для этого нужны внешние инструменты (`logrotate`, `rsyslog`, `fluentd`).

---

### Ключевые параметры логирования

| Параметр                     | Пример                    | Роль                                                   |
| ---------------------------- | ------------------------- | ------------------------------------------------------ |
| `logging_collector`          | `on`                      | Включает Syslogger                                     |
| `log_directory`              | `log`                     | Каталог для лог-файлов                                 |
| `log_filename`               | `postgresql-%Y-%m-%d.log` | Шаблон имени файла                                     |
| `log_rotation_age`           | `1d`                      | Ротация по времени                                     |
| `log_rotation_size`          | `100MB`                   | Ротация по размеру                                     |
| `log_statement`              | `all` / `mod` / `none`    | Что логировать: все запросы, только модификации и т.д. |
| `log_min_duration_statement` | `1000`                    | Логировать медленные запросы (>1 сек)                  |
| `log_line_prefix`            | `%t [%p]:`                | Префикс строки: время, PID и т.д.                      |

---

В `postgresql.conf`:
```sh
logging_collector = on          # обязательно
log_directory = 'log'           # каталог (относительно PGDATA или абсолютный)
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_rotation_size = 100MB
log_min_messages = WARNING      # уровень логирования (DEBUG5, INFO, NOTICE, WARNING, ERROR, LOG, FATAL, PANIC)
log_statement = 'none'          # или 'ddl', 'mod', 'all'
log_min_duration_statement = 1000  # логировать запросы дольше 1 сек
log_line_prefix = '%t [%p]: %u@%d '  # префикс: время, PID, пользователь, БД
```
Обновить конфигурацию
```sql
SELECT pg_reload_conf();
```
### Логи хранятся
- По умолчанию: `$PGDATA/log/` или `$PGDATA/pg_log/`
- В Debian/Ubuntu: часто `/var/log/postgresql/`

