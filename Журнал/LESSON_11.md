Работаю с локальным инстансом Postgresql 15 на домашней ОС Ubuntu 22.04.

Буду логировать контрольные точки и записывать их каждые 30 секунд:
  alter system set checkpoint_timeout = '30s';
  alter system set log_checkpoints = on;
  
  select pg_reload_conf();

Подаем нагрузку на кластер в течение 10 минут:

![Screenshot from 2024-03-10 20-20-46](https://github.com/marinesque/otus_postgresql/assets/97790878/4b69b417-3ed0-46b6-bce3-8ae6547f6b84)


Заглянула в статистику pg_controldata почитать информацию по контрольным точкам:

![Screenshot from 2024-03-10 20-18-57](https://github.com/marinesque/otus_postgresql/assets/97790878/d14e1737-09f5-4ea5-a328-a169eb502d86)

Latest checkpoint - lsn последней контрольной точки 

Latest checkpoint's redo - lsn записи начала контрольной точки, на который гарантировано, что все изменения сохранены

Проверила, меняется ли lsn с течением времени:
  select pg_current_wal_lsn();
  
  --0/842DA978
  select pg_walfile_name('0/842DA978'); --000000010000000000000084
  
  --0/842DAAC0
  select pg_walfile_name('0/842DAAC0'); --000000010000000000000084
  
  --0/B0566700
  select pg_walfile_name('0/B0566700'); --0000000100000000000000B0

Если заглянуть утилитой pg_waldump, то можно найти запись о контрольной точке, ее lsn и lsn ее начала:

![Screenshot from 2024-03-10 20-22-34](https://github.com/marinesque/otus_postgresql/assets/97790878/22c8e4e6-7e48-4931-b9c8-175c5d818963)

Объем байтов между ними:

![Screenshot from 2024-03-10 20-40-43](https://github.com/marinesque/otus_postgresql/assets/97790878/6b6d4640-7c29-4cc6-ac71-43c012e0d132)

Время начала не всегда равно 30 секунд.

![Screenshot from 2024-03-10 20-03-59](https://github.com/marinesque/otus_postgresql/assets/97790878/15fa8e6b-15d4-4b82-80e9-cc78b32e83be)


Статистика bgwriter:

![Screenshot from 2024-03-10 20-11-47](https://github.com/marinesque/otus_postgresql/assets/97790878/6b30e7f8-a1b7-4057-9a08-1a5b015de472)


  show synchronous_commit;
  alter system set synchronous_commit = off;
  
  select pg_reload_conf();

![Screenshot from 2024-03-10 21-14-59](https://github.com/marinesque/otus_postgresql/assets/97790878/3712a2d1-c05d-4f02-8694-be1355988bab)

В асинхронном режиме количество транзакций, которые успели обработаться, значительно увеличилось. 
А вот время транзакций также увеличилось, потому что завершались они в асинхронном режиме, коммит мог произойти гораздо позже фактического завершения работы транзакции.

  show data_checksums;
  alter system set data_checksums = on;
  
  show ignore_checksum_failure;
  alter system set ignore_checksum_failure = off;
  
  show wal_log_hints;
  alter system set wal_log_hints = on;
  
  
  select pg_reload_conf();

Создала таблицу:

  create table test11 (col text);
  
  insert into test11(col)
  select md5(random()::text) from generate_series(1,1000000);
    
  select pg_relation_filepath('test11'); --base/5/32928

Останавливаю кластер:

![Screenshot from 2024-03-11 11-41-16](https://github.com/marinesque/otus_postgresql/assets/97790878/54d7db76-f0e6-4f56-88b2-d2da8a379474)

Заменила 16 битов нулями в файле таблицы и поднимаю кластер:

![Screenshot from 2024-03-11 11-41-34](https://github.com/marinesque/otus_postgresql/assets/97790878/d55996bb-8269-41b4-8029-96ade4e98b6d)

Пробую подключиться к базе и прочитать данные из таблицы:

![Screenshot from 2024-03-11 10-15-14](https://github.com/marinesque/otus_postgresql/assets/97790878/24b937f5-459f-4b76-b38c-828de2694ef1)

Получаю ошибку. Теперь, я так понимаю, светит только восстановление из резервной копии.
