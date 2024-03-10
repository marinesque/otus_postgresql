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
Latest checkpoint's redo - lsn записи начала контрольной точки, на который гарантировано восстановление состояния бд

Проверила, меняется ли lsn с течением времени:
  select pg_current_wal_lsn();
  
  --0/842DA978
  select pg_walfile_name('0/842DA978'); --000000010000000000000084
  
  --0/842DAAC0
  select pg_walfile_name('0/842DAAC0'); --000000010000000000000084
  
  --0/B0566700
  select pg_walfile_name('0/B0566700'); --0000000100000000000000B0

Также 
![Screenshot from 2024-03-10 20-22-34](https://github.com/marinesque/otus_postgresql/assets/97790878/22c8e4e6-7e48-4931-b9c8-175c5d818963)
