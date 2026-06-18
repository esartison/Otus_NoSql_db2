# Домашнее задание Сартисона Евгения 2 #

установить MongoDB одним из способов: ВМ, докер;

заполнить данными;

написать несколько запросов на выборку и обновление данных


Задание повышенной сложности*

создать индексы и сравнить производительность.






## (1) установить MongoDB одним из способов: ВМ, докер; ВМ ##
Устанавливаем MongoDB на локальную VM и используем [Install MongoDB Community Edition on Ubuntu](https://www.mongodb.com/docs/v8.0/tutorial/install-mongodb-on-ubuntu/#install-mongodb-community-edition-on-ubuntu)


установка утилит gnupg и curl
>sudo apt-get install gnupg curl

импортировать публичный ключ
>curl -fsSL --insecure https://pgp.mongodb.com/server-8.0.asc |    sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg    --dearmor

создать список файлов
> echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

обновить базу
> sudo apt-get update -o Acquire::https::Verify-Peer=false
<img width="1037" height="265" alt="image" src="https://github.com/user-attachments/assets/74caba5f-2f92-45c5-9127-6f9eb6e15022" />

установка MongoDB 
> sudo apt-get install -y mongodb-org -o Acquire::https::Verify-Peer=false
<img width="951" height="583" alt="image" src="https://github.com/user-attachments/assets/6ad39f00-57fb-40a6-a30f-558e1292b1e7" />

старт MongoDB
>sudo systemctl start mongod
<img width="1883" height="292" alt="image" src="https://github.com/user-attachments/assets/6fb6c041-b3c2-4438-bf1c-1cc6ff306648" />

проверка подключения
<img width="1789" height="472" alt="image" src="https://github.com/user-attachments/assets/a15c7e16-5ff9-429f-9863-426df8f7d67e" />




## (2) заполнить данными ##
[Load Sample Data into MongoDB](https://restheart.org/docs/mongodb-rest/sample-data)

скачать data set
>curl --insecure https://atlas-education.s3.amazonaws.com/sampledata.archive -o sampledata.archive

сделать импорт баз
>mongorestore --archive=sampledata.archive
<img width="1882" height="234" alt="image" src="https://github.com/user-attachments/assets/ca5705b4-45d3-489c-913a-74a8da27726a" />

проверить базы данных
<img width="519" height="269" alt="image" src="https://github.com/user-attachments/assets/a0da79ba-91c4-46d2-add1-966edb867a81" />

подключиться к базе sample_restaurants
<img width="344" height="51" alt="image" src="https://github.com/user-attachments/assets/429f7d98-a26d-439e-a34b-87ead440d22d" />


проверка доступных коллекций
<img width="399" height="120" alt="image" src="https://github.com/user-attachments/assets/540269df-9dd3-42da-bfd5-eeff875ae66b" />


## (3) написать несколько запросов на выборку и обновление данных ##

Взял конфигуратор https://pgconfigurator.cybertec-postgresql.com/ и получил рекомендации по следующим параметрам, пробежался и рекомендации выглядят оправданными
```
postgres@otuspgperf01:~$ psql -c "show all"|egrep "shared_buffers|work_mem|maintenance_work_mem|max_connections|effective_cache_size|effective_io_concurrency|random_page_cost|shared_preload_libraries"
 autovacuum_work_mem                         | -1                                      | Sets the maximum memory to be used by each autovacuum worker process.
 effective_cache_size                        | 4GB                                     | Sets the planner's assumption about the total size of the data caches.
 effective_io_concurrency                    | 1                                       | Number of simultaneous requests that can be handled efficiently by the disk subsystem.
 hash_mem_multiplier                         | 2                                       | Multiple of "work_mem" to use for hash tables.
 logical_decoding_work_mem                   | 64MB                                    | Sets the maximum memory to be used for logical decoding.
 maintenance_io_concurrency                  | 10                                      | A variant of "effective_io_concurrency" that is used for maintenance work.
 maintenance_work_mem                        | 64MB                                    | Sets the maximum memory to be used for maintenance operations.
 max_connections                             | 100                                     | Sets the maximum number of concurrent connections.
 random_page_cost                            | 4                                       | Sets the planner's estimate of the cost of a nonsequentially fetched disk page.
 shared_buffers                              | 128MB                                   | Sets the number of shared memory buffers used by the server.
 shared_preload_libraries                    |                                         | Lists shared libraries to preload into server.
 work_mem                                    | 4MB                                     | Sets the maximum memory to be used for query workspaces.
 ```

Изменил параметры и перезапустил Postgres
```
ALTER SYSTEM SET  shared_buffers = '512 MB'                           ;
ALTER SYSTEM SET  work_mem = '32 MB'                                  ;
ALTER SYSTEM SET  maintenance_work_mem = '320 MB'                     ;
ALTER SYSTEM SET  max_connections = 100                               ;
ALTER SYSTEM SET  superuser_reserved_connections = 3                  ;
ALTER SYSTEM SET  effective_cache_size = '1 GB'                       ;
ALTER SYSTEM SET  effective_io_concurrency = 5                        ;
ALTER SYSTEM SET  random_page_cost = 4                                ;
ALTER SYSTEM SET  shared_preload_libraries = 'pg_stat_statements'     ;
```
> sudo -i -u postgres pg_ctlcluster 17 main stop && sync && echo 3 > /proc/sys/vm/drop_caches  && sudo -i -u postgres  pg_ctlcluster 17 main start


Проверка значений параметров после 
```
postgres@otuspgperf01:~$ psql -c "show all"|egrep "shared_buffers|work_mem|maintenance_work_mem|max_connections|effective_cache_size|effective_io_concurrency|random_page_cost|shared_preload_libraries"
 autovacuum_work_mem                         | -1                                      | Sets the maximum memory to be used by each autovacuum worker process.
 effective_cache_size                        | 1GB                                     | Sets the planner's assumption about the total size of the data caches.
 effective_io_concurrency                    | 5                                       | Number of simultaneous requests that can be handled efficiently by the disk subsystem.
 hash_mem_multiplier                         | 2                                       | Multiple of "work_mem" to use for hash tables.
 logical_decoding_work_mem                   | 64MB                                    | Sets the maximum memory to be used for logical decoding.
 maintenance_io_concurrency                  | 10                                      | A variant of "effective_io_concurrency" that is used for maintenance work.
 maintenance_work_mem                        | 320MB                                   | Sets the maximum memory to be used for maintenance operations.
 max_connections                             | 100                                     | Sets the maximum number of concurrent connections.
 random_page_cost                            | 4                                       | Sets the planner's estimate of the cost of a nonsequentially fetched disk page.
 shared_buffers                              | 512MB                                   | Sets the number of shared memory buffers used by the server.
 shared_preload_libraries                    | pg_stat_statements                      | Lists shared libraries to preload into server.
 work_mem                                    | 32MB                                    | Sets the maximum memory to be used for query workspaces.
```

## (4) создать индексы и сравнить производительность* ##

Решил попробовать поменять значения для 3х параметров: wal_level, synchronous_commit, fsync
```
postgres@otuspgperf01:~$ psql -c "show all"|egrep "wal_level|synchronous_commit|fsync"
 fsync                                       | on                                      | Forces synchronization of updates to disk.
 recovery_init_sync_method                   | fsync                                   | Sets the method for synchronizing the data directory before crash recovery.
 synchronous_commit                          | on                                      | Sets the current transaction's synchronization level.
 wal_level                                   | replica                                 | Sets the level of information written to the WAL.
 wal_skip_threshold                          | 2MB                                     | Minimum size of new file to fsync instead of writing WAL.
``` 
