# Домашнее задание Сартисона Евгения 2 #

установить MongoDB одним из способов: ВМ, докер;

заполнить данными;

написать несколько запросов на выборку и обновление данных


Задание повышенной сложности*

создать индексы и сравнить производительность.






## (1) установить MongoDB одним из способов: ВМ, докер; ВМ ##
Устанавливаем MongoDB на локальную VM и используем [Install MongoDB Community Edition on Ubuntu](https://www.mongodb.com/docs/v8.0/tutorial/install-mongodb-on-ubuntu/#install-mongodb-community-edition-on-ubuntu)

установил Postgres с настройками по умолчанию
>sudo apt install curl ca-certificates

>sudo install -d /usr/share/postgresql-common/pgdg

>sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

>sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

>sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt noble-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

>sudo apt update

>sudo apt install postgresql-13 postgresql-client-13
![image](https://github.com/user-attachments/assets/5fcc0e98-235a-4fc7-9380-842f2606bbc2)

создал базу pg_test_perf для тестирования


![image](https://github.com/user-attachments/assets/601417dd-f301-4296-a879-4b1bef8cf09a)


## (2) заполнить данными ##
выполнилить базовый ран(от него будем отталкиваться) против базы pg_test_perf, перед каждым раном будем перезапускать экземпляр и чистить кэш на OS
> sudo -i -u postgres pg_ctlcluster 17 main stop && sync && echo 3 > /proc/sys/vm/drop_caches  && sudo -i -u postgres  pg_ctlcluster 17 main start
> sudo -i -u postgres pgbench -i -s 150 pg_test_perf
![image](https://github.com/user-attachments/assets/f51b27e2-1099-4c80-b96d-6bb1ba119e4f)



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
