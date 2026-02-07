### На primary:
```sql
SELECT * FROM pg_stat_replication;
-- sent_lsn, write_lsn, flush_lsn, replay_lsn, state
```

### На standby:
```sql
SELECT * FROM pg_stat_wal_receiver;
-- received_lsn, last_msg_send_time

-- Отставание в байтах:
SELECT pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes
FROM pg_stat_replication;

-- Или в удобном виде:
SELECT pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn)) AS lag;
```
