
```sh
sudo systemctl stop postgresql
	# pg_ctlcluster 18 main stop
sudo systemctl start postgresql
	# pg_ctlcluster 18 main start
sudo systemctl reload postgresql
sudo systemctl reload postgresql@18-main
	# sudo pg_ctlcluster 18 main reload

# запуск клвстера вручную
sudo -u postgres /usr/lib/postgresql/18/bin/postgres \
  -D /var/lib/postgresql/18/standby \
  -c unix_socket_directories='/var/run/postgresql' \
  -p 5432


# Посмотреть логи текущего кластера (например, 18/main)
sudo journalctl -u postgresql@18-main
# Или для кластера с именем "standby"
sudo journalctl -u postgresql@18-standby
# Следить в реальном времени
sudo journalctl -u postgresql@18-standby -f

# Процессы
ps aux | grep postgres
```

### Чтобы не вводить пароль каждый раз, создайте файл `~/.pgpass`:
```sh
touch ~/.pgpass
	localhost:5432:mydb:postgres:mypassword
	
chmod 600 ~/.pgpass
```

