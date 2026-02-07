
**Postmaster** (главный процесс) - «диспетчер»
- Запускается **первым** при старте PostgreSQL.
- Слушает порт (обычно 5432) и **принимает подключения** от клиентов.
- При каждом новом подключении **порождает дочерний процесс** (backend).
- Также запускает **фоновые процессы**
	- [[Backend processes|Вackend]] (как сказано выше)
	- [[WAL writer]]
	- [[WAL archiver]]
	- [[WAL sender]]
	- [[Background writer]]
	- [[Checkpointer]]
	- [[Syslogger]]
	- [[Autovacuum launcher - Autovacuum worker]]
	- [[Stats collector]]
	- [[Startup process - WAL receiver (standby)|Startup process]]
	- Logical replication launcher [[Логическая (PUBLICATION) репликация]]
		- Logical replication worker
	- и еще [[Еще подпроцессы|некоторые процессы]] выполняемые в не тривиальных ситуациях
- Инициализирует всю область Shared Memory в оперативной памяти, остальные процессы только управляют/работают с ней

