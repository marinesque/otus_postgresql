Не буду распиисывать подробности, т.к. работала над задачей в несколько заходов в разные дни и просто не успела сделать подробные скриншоты.

Исходные данные:
Две виртуальные машины с Ubuntu 22.04. На машине 1 - Postgresql 15, 2 - Postgresql 16.

Первое, что я пыталась сделать - подружить эти две вм. Чтобы они могли видеть друг друга и общаться.

Я объединила их в NAT сеть, чтобыу каждой был доступ к глобальной сети, но чтобы они пинговались по 192.168.0.ХХХ.


Удостоверилась, что вм друг друга видят с помощью ping.


Затем я создала на каждой из вм таблицы

VM1:
create table public.test(col text);
insert into public.test(col) values ('This is from machine 1');

create table public.test2(col text);

VM2:
create table public.test2(col text);
insert into public.test2(col) values ('This is from machine 2');

create table public.test(col text);

VM1:
create publication ..

Сначала я получала ошибки подключения к Postgresql другой машины.
Пошла копаться в конфигах.

/etc/postgresql/15/main/postgresql.conf:

VM1:
![Screenshot from 2024-03-20 21-33-03](https://github.com/marinesque/otus_postgresql/assets/97790878/4c6f6ab4-682d-4646-8aac-467afb885fd9)

/etc/postgresql/15/main/pg_hba.conf:

![Screenshot from 2024-03-20 21-35-51](https://github.com/marinesque/otus_postgresql/assets/97790878/46665924-1282-44d3-9abe-cbe2163a1bcb)

alter system set wal_level = logical;

systemctl restart postgresql.service





VM1:
![Screenshot from 2024-03-20 21-31-22](https://github.com/marinesque/otus_postgresql/assets/97790878/499c9154-e24a-4494-a6f5-2a57bd6ac7a5)


