
Включение логической репликации
```sh
### На Publisher
nano /etc/postgresql/18/main/conf.d/psql_listen.conf
	wal_level = logical
	
# В файле `pg_hba.conf` добавьте строку, разрешающую подключение от целевого сервера для репликации. Например:
#    # TYPE  DATABASE        USER            ADDRESS                 METHOD
#    host    replication     replicator      <IP_Target>/32          md5

# Создайте пользователя для репликации:
	CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'strong_password';
```


## Реплика одной определенной таблицы ^psql-repl-onlyone-tables-chem
Требования
1. **PostgreSQL версии 10 или выше** (логическая репликация появилась в v10).
2. Доступ к обоим серверам (источнику и приёмнику).
3. Права суперпользователя или как минимум `REPLICATION` и `CREATE` на обеих сторонах.
4. Сетевая доступность между серверами.
5. На источнике: таблица должна иметь **первичный ключ (PRIMARY KEY)** или **уникальный индекс, включающий NOT NULL-столбцы** (это обязательно для UPDATE/DELETE в логической репликации).

```sql
-- ### На Publisher
-- Убедиться, что таблица имеет первичный ключ
-- Пример таблицы
CREATE TABLE public.sales (
    id SERIAL PRIMARY KEY,
    product_name TEXT NOT NULL,
    amount NUMERIC(10,2),
    sale_date DATE
);
-- Если PK отсутствует — добавьте его:
	ALTER TABLE public.sales ADD PRIMARY KEY (id);

-- Создать публикацию только для нужной таблицы
CREATE PUBLICATION pub_sales FOR TABLE public.sales;

-- По умолчанию публикация реплицирует **INSERT, UPDATE, DELETE**.  
	-- Если нужно, можно явно указать типы операций:
	-- CREATE PUBLICATION pub_sales FOR TABLE public.sales WITH (publish = 'insert,update');
	

-- ### На Subscriber
-- Убедиться, что структура таблицы совпадает
	-- На приёмнике **вручную создайте** такую же таблицу:
		CREATE TABLE public.sales (
		    id SERIAL PRIMARY KEY,
		    product_name TEXT NOT NULL,
		    amount NUMERIC(10,2),
		    sale_date DATE
		);
-- Имя таблицы, схема, столбцы и типы данных **должны совпадать**.
-- Первичный ключ **обязателен** — иначе репликация не сможет корректно обрабатывать UPDATE/DELETE.

-- Создать подписку
CREATE SUBSCRIPTION sub_sales_from_main
CONNECTION 'host=192.168.1.50 port=5432 dbname=maindb user=replicator password=your_secure_password'
PUBLICATION pub_sales;
	-- При создании подписки PostgreSQL автоматически:
	-- 1. Запускает **начальную синхронизацию** (копирует все существующие строки из `sales` на источник),
	-- 2. Затем переходит в режим **непрерывной репликации** (применяет новые изменения в реальном времени).
```

#### Как проверить, что всё работает?
```sql
-- ### На Publisher
-- Посмотреть активные публикации
SELECT * FROM pg_publication;
-- Посмотреть, какие таблицы в публикации
SELECT * FROM pg_publication_tables WHERE pubname = 'pub_sales';

-- ### На Subscriber
-- Статус подписок
SELECT subname, subenabled, subconninfo, subpublications FROM pg_subscription;
-- Статус синхронизации (если ещё идёт)
SELECT * FROM pg_stat_subscription;
-- Проверить данные
SELECT COUNT(*) FROM public.sales;

```

## Объединение данных из нескольких источников ^psql-unification-data-in-standby
**Ключевое требование:** Чтобы данные из разных источников не конфликтовали при вставке в одну таблицу на целевом сервере, **первичный ключ (Primary Key) должен быть уникальным глобально**, а не только в рамках одного источника. Самый простой способ — использовать составной первичный ключ, где одним из полей будет идентификатор самого источника.
```sql
-- ### На Publisher
-- Убедитесь, что у вашей таблицы есть первичный ключ. Для нашего примера предположим, что таблица выглядит так:
-- Пример для primary1
    CREATE TABLE events (
        source_id TEXT NOT NULL DEFAULT '<primary1>', -- Идентификатор источника, для 2го источника <primary2>
        event_id SERIAL,
        event_data TEXT,
        PRIMARY KEY (source_id, event_id) -- Составной PK
-- Создайте публикацию для этой таблицы:
CREATE PUBLICATION pub_events FOR TABLE events;

-- ### На Subscriber
-- Создать структуру таблицы
-- Таблица на целевом сервере должна быть идентична таблицам на источниках, но без значений по умолчанию для `source_id`, чтобы избежать путаницы.
CREATE TABLE events (
        source_id TEXT NOT NULL,
        event_id INTEGER NOT NULL,
        event_data TEXT,
        PRIMARY KEY (source_id, event_id)
    );
    
-- Теперь нужно создать две отдельные подписки, каждая из которых будет слушать свой источник.
-- Подписка на <primary1>:
CREATE SUBSCRIPTION sub_from_source_a
    CONNECTION 'host=<primary1> port=5432 dbname=your_db user=replicator password=strong_password'
    PUBLICATION pub_events;
-- Подписка на <primary2>:
CREATE SUBSCRIPTION sub_from_source_b
    CONNECTION 'host=<primary2> port=5432 dbname=your_db user=replicator password=strong_password'
    PUBLICATION pub_events;
    
-- После выполнения этих команд PostgreSQL автоматически начнёт процесс копирования существующих данных (initial data sync) из каждой публикации в соответствующую подписку, а затем будет применять все новые изменения (INSERT, UPDATE, DELETE) в реальном времени.
```

**Важные моменты и ограничения**
*  **Уникальность данных:** Как уже упоминалось, глобальная уникальность через составной ключ (`source_id`, `event_id`) является критически важной. Если два источника попытаются вставить строку с одинаковым первичным ключом, возникнет конфликт, и репликация для этой подписки остановится.
*  **Операции DDL:** Логическая репликация **не реплицирует** операции определения данных (DDL), такие как `CREATE TABLE`, `ALTER TABLE`. Любые изменения в структуре таблицы на источнике нужно вручную применять на целевом сервере до того, как изменения данных начнут поступать.
*  **Управление подписками:** Вы можете управлять подписками с помощью команд `ALTER SUBSCRIPTION` (для отключения/включения потока данных) и `DROP SUBSCRIPTION` (для полного удаления).
*  **Мониторинг:** Состояние подписок можно отслеживать в системном каталоге `pg_stat_subscription`.
- По умолчанию тригггеры **не срабатывают** при репликации. Чтобы они работали, используйте `ENABLE REPLICA TRIGGER`.
- При конфликте (например, дубликат PK) подписка **останавливается**. Нужно вручную исправлять данные и возобновлять: `ALTER SUBSCRIPTION ... ENABLE;`
- Логическая репликация потребляет больше CPU и WAL-пространства, чем физическая.


## Полезно
```sql
-- Отключить подписку (без удаления):
ALTER SUBSCRIPTION sub_sales_from_main DISABLE;
-- Возобновить
ALTER SUBSCRIPTION sub_sales_from_main ENABLE;

-- Удалить подписку
DROP SUBSCRIPTION sub_sales_from_main;

-- Изменить подключение (например, сменить пароль):
ALTER SUBSCRIPTION sub_sales_from_main
CONNECTION 'host=... user=... password=new_pass';
```

