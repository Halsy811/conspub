#PostgreSQL  #psql 

# Debian
###  Default install

```sh
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh # Automated repository configuration
apt search "postgresql-18$"
apt install postgresql-18 # или др версия

systemctl status postgresql 
```

### Manually install

Чтобы вручную настроить репозиторий Apt
```sh
sudo apt install curl ca-certificates  
sudo install -d /usr/share/postgresql-common/pgdg  
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc  
. /etc/os-release  
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"  
sudo apt update
```

```sh
sudo apt install postgresql-18
```

---

Репозиторий содержит:

| postgresql-client-18     | клиентские библиотеки и двоичные файлы клиента                    |
| ------------------------ | ----------------------------------------------------------------- |
| postgresql-18            | основной сервер базы данных                                       |
| postgresql-doc-18        | documentation                                                     |
| libpq-dev                | библиотеки и заголовки для разработки интерфейса на языке Си      |
| postgresql-server-dev-18 | библиотеки и заголовки для разработки серверной части на языке Си |
