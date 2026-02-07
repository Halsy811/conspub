#psql #MySQL

# Миграция MySQL на PostgreSQL с помощью Pgloader

### Установка pgloader

https://github.com/dimitri/pgloader.git

Pgloader - это бесплатный инструмент для миграции баз данных, который позволяет пользователям осуществлять непрерывную миграцию из MySQL, SQLite и MS SQL Server в PostgreSQL.

```sh
# sudo apt install sbcl unzip libsqlite3-dev gawk curl make freetds-dev libzip-dev
sudo apt install -y sbcl curl make gcc libc6-dev git

# Склонируйте и соберите
git clone https://github.com/dimitri/pgloader.git
cd pgloader
make pgloader
```

```
pgloader --version
```

Создайте файл для настроек миграции
```sh
nano migrate.load
```
В него внесите:
обязательно нужно использовать пароли!
```sh
load database
     from mysql://user:password@10.0.10.95/db
     into postgresql://postgres:password@localhost/snquality
;
```
Начать миграцию:
```sh
./pgloader/build/bin/pgloader migrate.load
```


Пример вывода:
```sh
2025-12-26T12:34:41.184063+03:00 LOG report summary reset
                   table name     errors       rows      bytes      total time
-----------------------------  ---------  ---------  ---------  --------------
              fetch meta data          0         68                     0.116s
               Create Schemas          0          0                     0.004s
             Create SQL Types          0          0                     0.004s
                Create tables          0         36                     0.052s
               Set Table OIDs          0         18                     0.004s
-----------------------------  ---------  ---------  ---------  --------------
           db.sert_attachment          0       5112   702.0 kB          0.444s
           db.sert_sertnumber          0        830    14.4 kB          0.284s
                 db.sert_sert          0        826    86.4 kB          0.084s
                 db.sert_melt          0        598    70.7 kB          0.596s
           db.auth_permission          0         56     2.4 kB          0.596s
         db.django_migrations          0         19     1.2 kB          1.044s
          db.sert_signatories          0          9     0.6 kB          1.056s
                 db.auth_user          0          4     0.7 kB          1.544s
    db.auth_group_permissions          0          0                     1.504s
db.auth_user_user_permissions          0          0                     1.884s
               db.sert_kernel          0        743    57.3 kB          0.084s
          db.django_admin_log          0        355    27.0 kB          0.500s
            db.django_session          0         44    12.4 kB          0.552s
       db.django_content_type          0         14     0.2 kB          1.004s
           db.sert_conclusion          0          8     2.1 kB          1.008s
                db.auth_group          0          0                     1.456s
          db.auth_user_groups          0          0                     1.480s
            db.sert_guarantee          0          1     0.1 kB          1.816s
-----------------------------  ---------  ---------  ---------  --------------
      COPY Threads Completion          0          4                     2.228s
               Create Indexes          0         36                     0.228s
       Index Build Completion          0         36                     1.180s
              Reset Sequences          0         12                     0.080s
                 Primary Keys          0         18                     0.012s
          Create Foreign Keys          0         14                     0.024s
              Create Triggers          0          0                     0.000s
              Set Search Path          0          1                     0.000s
             Install Comments          0          0                     0.000s
-----------------------------  ---------  ---------  ---------  --------------
            Total import time          ✓       8619   977.5 kB          3.752s

```