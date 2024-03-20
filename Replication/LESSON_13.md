Не буду распиисывать подробности, т.к. работала над задачей в несколько заходов в разные дни и просто не успела сделать подробные скриншоты.

Исходные данные:
Две виртуальные машины с Ubuntu 22.04. На машине 1 (192.168.0.4) - Postgresql 15, 2 (192.168.0.5) - Postgresql 16.

Первое, что я пыталась сделать - подружить эти две вм. Чтобы они могли видеть друг друга и общаться.

Я объединила их в NAT сеть, чтобы у каждой был доступ к глобальной сети, но чтобы они пинговались по 192.168.0.ХХХ.


Удостоверилась, что вм друг друга видят с помощью ping.


Затем я создала на каждой из вм таблицы

VM1:
~~~
create table public.test(col text);
insert into public.test(col) values ('This is from machine 1');

create table public.test2(col text);
~~~

VM2:
~~~
create table public.test2(col text);
insert into public.test2(col) values ('This is from machine 2');

create table public.test(col text);
~~~

VM1:
~~~
create publication from1to2pub for table public.test;
~~~

VM2:
~~~
create publication from2to1pub for table public.test2;
~~~

VM1:
~~~
create subscription from2to1sub connection 'host=192.168.0.4 port=5432 user=postgres password=postgres dbname=postgres' publication from2to1pub;
~~~

VM2:
~~~
create subscription from1to2sub connection 'host=192.168.0.4 port=5432 user=postgres password=postgres dbname=postgres' publication from1to2pub;
~~~

При попытке создания подписки сначала я получала ошибки подключения к Postgresql другой машины.
Пошла копаться в конфигах. Прослушка локалхоста была заменена на '*'.
Изменила wal_level на logical.
Добавила ip в pg_hba.conf. Сначала забыла указать, как будет аутентификация происходить. Получила пару ошибок на этот счет.

![Screenshot from 2024-03-20 21-39-36](https://github.com/marinesque/otus_postgresql/assets/97790878/e9302817-ccc2-4e17-aa79-bee1ee1c972f)

/etc/postgresql/15/main/postgresql.conf:

VM1:

![Screenshot from 2024-03-20 21-33-03](https://github.com/marinesque/otus_postgresql/assets/97790878/4c6f6ab4-682d-4646-8aac-467afb885fd9)

/etc/postgresql/15/main/pg_hba.conf:

![Screenshot from 2024-03-20 21-35-51](https://github.com/marinesque/otus_postgresql/assets/97790878/46665924-1282-44d3-9abe-cbe2163a1bcb)

~~~
alter system set wal_level = logical;
~~~

~~~
systemctl restart postgresql.service
~~~

В результате данные пришли на дружественные машинки в целости и сохранности:

VM1:

![Screenshot from 2024-03-20 21-31-22](https://github.com/marinesque/otus_postgresql/assets/97790878/499c9154-e24a-4494-a6f5-2a57bd6ac7a5)

VM2:

![Screenshot from 2024-03-20 21-44-04](https://github.com/marinesque/otus_postgresql/assets/97790878/34ae2e9f-97e7-4215-8e99-c5cafc314deb)

