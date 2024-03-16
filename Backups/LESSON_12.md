Работаю с ОС Ubuntu 22.04, установлен кластер Postgresql 15.

Попробовала создать папку ~/otus_backups/ и сделать овнером папки пользователя postgres.

![Screenshot from 2024-03-16 17-41-37](https://github.com/marinesque/otus_postgresql/assets/97790878/863af303-4358-464c-a2b0-57e27c591771)

![Screenshot from 2024-03-16 17-41-52](https://github.com/marinesque/otus_postgresql/assets/97790878/125f0e28-7207-4287-a99a-509fabae05ed)

Выдала chmod 777 на указанный путь.

![Screenshot from 2024-03-16 18-28-12](https://github.com/marinesque/otus_postgresql/assets/97790878/a67f9d81-0541-48b8-8e47-1f1900aa03a6)

Подключилась к бд под юзером postgres.

Работаю в одноименной бд postgres.

~~~
create schema otus;

create table otus.test12 (col text);

insert into otus.test12 (col)
select md5(random()::text) from generate_series(1,100);
~~~

Пробую сделать логический бэкап таблицы и получаю permission denied на папку.

![Screenshot from 2024-03-16 19-29-02](https://github.com/marinesque/otus_postgresql/assets/97790878/e860b759-7d1e-4faa-944b-5c19c484e66f)

Проверяю права на /home/marinesque/otus_backups.

![Screenshot from 2024-03-16 18-21-20](https://github.com/marinesque/otus_postgresql/assets/97790878/c1b3b12a-bc1b-4618-9b22-0d8a35296757)

![Screenshot from 2024-03-16 18-28-12](https://github.com/marinesque/otus_postgresql/assets/97790878/3a8c0570-5248-402f-912d-9145e780e613)

![Screenshot from 2024-03-16 18-32-34](https://github.com/marinesque/otus_postgresql/assets/97790878/a2366da6-b7f9-40a4-95b6-1117e2ae2bc1)

Прав нет. Не смогла победить раздачу прав.
Подошла с другой стороны. Куда у postgres точно есть права на запись и чтение? Конечно же, в папку postgresql.

![Screenshot from 2024-03-16 18-47-43](https://github.com/marinesque/otus_postgresql/assets/97790878/4c6ef0dc-0992-44fe-a807-5252b497feeb)

Создала папку для бэкапов. Пробую сделать бэкап и восстановить в другую таблицу.

~~~
\copy otus.test12 to '/var/lib/postgresql/otus_backups/backup12.sql';

-----------------------

create table otus.test12_restored (col text);

\copy otus.test12_restored from '/var/lib/postgresql/otus_backups/backup12.sql';
~~~

Проверяю, что бэкап создан.

![Screenshot from 2024-03-16 18-47-51](https://github.com/marinesque/otus_postgresql/assets/97790878/bb2760fa-7585-4eed-beb5-d946eaeb2797)

Пробую сделать кастомную резервную копию в сжатый gz файл.

![Screenshot from 2024-03-16 18-55-29](https://github.com/marinesque/otus_postgresql/assets/97790878/a03da6f5-1526-4e46-87e7-1a6539605256)

Выдал ошибку на битый файл таблицы из предыдущего урока) Поэтому таблицу старую я пока дропнула.

Файл создался.

![Screenshot from 2024-03-16 18-55-49](https://github.com/marinesque/otus_postgresql/assets/97790878/0ee38e05-69f1-4aea-97a5-eed02260df65)

Пробую восстановить только вторую таблицу в новую базу данных.

~~~
sudo -u postgres createdb otus
sudo -u postgres pg_restore -d otus -p 5432 --schema-only -j 2 /var/lib/postgresql/otus_backups/dump12custom.gz
sudo -u postgres pg_restore -d otus -p 5432 --schema otus --table test12_restored -j 2 /var/lib/postgresql/otus_backups/dump12custom.gz
~~~

Проверяю, что таблица создалась.

![Screenshot from 2024-03-16 19-12-01](https://github.com/marinesque/otus_postgresql/assets/97790878/53982994-94d5-40f9-ba40-2ed58a092248)

При этом первая таблица восстановлена только в части структуры, а данные не восстановлены.

![Screenshot from 2024-03-16 19-41-56](https://github.com/marinesque/otus_postgresql/assets/97790878/af31656e-6ebb-4283-bd48-a7fec3ecbd08)

