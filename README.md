# Домашнее задание №5

Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
 
 ![image](https://user-images.githubusercontent.com/130083589/235350629-5e09d7ad-1b10-4722-96b9-e515568ede0e.png)


Установить на него PostgreSQL 15 с дефолтными настройками

>sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-15
 
 ![image](https://user-images.githubusercontent.com/130083589/235350653-f775280e-a5cc-46ec-a490-1ed65c7f9149.png)


Создать БД для тестов: выполнить pgbench -i postgres
sudo -u postgres pgbench -i postgres
 
 ![image](https://user-images.githubusercontent.com/130083589/235350663-6b0643c2-8977-481a-a935-fa58653b8387.png)


Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
 
 ![image](https://user-images.githubusercontent.com/130083589/235350667-2e2bd819-c19b-49fa-92ab-25ccb25882fb.png)


Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
- выполняем sudo nano /etc/postgresql/15/main/postgresql.conf и редактируем файл, после чего перезагружаем пострге ( sudo systemctl restart postgresql )

|Параметр|Исходное значение|Новое значение|
|--------|-----------------|--------------|
|max_connections|100|40|
|shared_buffers|128MB|1GB|
|effective_cache_size|4GB|3GB|
|maintenance_work_mem|64MB|512MB|
|checkpoint_completion_target|0.9|0.9|
|wal_buffers|-1|16MB|
|default_statistics_target|100|500|
|random_page_cost|4|4|
|effective_io_concurrency|1|2|
|work_mem|4MB|6553kB|
|min_wal_size|80MB|4GB|
|max_wal_size|1GB|16GB|

Протестировать заново

![image](https://user-images.githubusercontent.com/130083589/235350926-af1d1343-1933-4c48-8bfe-743f7b13082f.png)

 
Что изменилось и почему?

>Из-за того, что увеличили shared_buffers, work_mem и maintenance_work_mem, увеличилась скорость транзакций с tps = 648 до tps = 717.

Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
create table tbl1 (str varchar(100));
insert into tbl1 (str) select substr(gen_random_uuid()::text, 1, 10) from generate_Series(1, 1000000);
 
 ![image](https://user-images.githubusercontent.com/130083589/235350936-2ce08ada-685e-4576-8830-b274f95479f8.png)


Посмотреть размер файла с таблицей

>SELECT pg_size_pretty( pg_total_relation_size( 'tbl1' ) );

![image](https://user-images.githubusercontent.com/130083589/235350950-b424751b-794b-4bc8-8bd5-c38648c4f96d.png)

 
5 раз обновить все строчки и добавить к каждой строчке любой символ

>update tbl1 set str = str ||  '_'  || ceil(random()*9);

![image](https://user-images.githubusercontent.com/130083589/235350954-18e5ac0a-6afb-4a71-bacf-c9dfa503ac73.png)

 
Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум

>select relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum from pg_stat_user_tables where relname = 'tbl1';
 
 ![image](https://user-images.githubusercontent.com/130083589/235350965-907d04a1-0128-4393-b9c8-b452367de136.png)


Подождать некоторое время, проверяя, пришел ли автовакуум

>Нет, т.к. мертвых строк нет

5 раз обновить все строчки и добавить к каждой строчке любой символ

![image](https://user-images.githubusercontent.com/130083589/235350986-b277f396-313c-439f-a379-d5e4f615fff5.png)

 
Посмотреть размер файла с таблицей

![image](https://user-images.githubusercontent.com/130083589/235350990-29c1b663-eeb5-45e1-b1d1-959e13412839.png)

 
Отключить Автовакуум на конкретной таблице

>ALTER TABLE tbl1 SET (autovacuum_enabled = false);

![image](https://user-images.githubusercontent.com/130083589/235350996-db0ce3a0-72d8-4b8e-8a4a-50610f682cde.png)


 
10 раз обновить все строчки и добавить к каждой строчке любой символ

DO $$

BEGIN

FOR i IN 1..10 

  LOOP

  update tbl1 set str = str ||  '_'  || ceil(random()*9);

  END LOOP;

END $$;

![image](https://user-images.githubusercontent.com/130083589/235351001-b3b31998-6e5f-4d4b-a87d-e8b65012a4be.png)

 
Посмотреть размер файла с таблицей

![image](https://user-images.githubusercontent.com/130083589/235351005-b5ba35df-1812-4b6b-a66d-84e072b420c5.png)

 

Не забудьте включить автовакуум)

>ALTER TABLE tbl1 SET (autovacuum_enabled = true);

![image](https://user-images.githubusercontent.com/130083589/235351018-0da0e7e9-895b-4be6-a8f7-e4d1609cbfa2.png)

 
Объясните полученный результат

>Можно уменьшить размер таблицы, если выполнить команду «vacuum full».

![image](https://user-images.githubusercontent.com/130083589/235351068-0429715c-1c1a-47a1-8cb1-03bb90951ac5.png)

 
Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.

>create table tbl2 (str varchar(100));

>insert into tbl2 (str) select substr(gen_random_uuid()::text, 1, 10) from generate_Series(1, 10000);


>DO $$

>BEGIN

>FOR i IN 1..10 

>  LOOP

>    update tbl2 set str = str ||  '/'  || ceil(random()*9);

>    RAISE NOTICE 'step %', i;

>  END LOOP;

>END $$;

![image](https://user-images.githubusercontent.com/130083589/235351086-0cba76db-6edd-4fa5-a08c-b878a3e98042.png)

