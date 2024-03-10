Буду работать на имеющейся виртуальной машине с установленной Ubuntu 22.04 и Postgresql 15.

![Screenshot from 2024-03-10 17-27-48](https://github.com/marinesque/otus_postgresql/assets/97790878/e899ffc2-d645-4bb0-861c-2b3cba45d2dc)

Попробую рассчитать оптимальные настройки для Postgresql 15 с помощью сервиса Cybertec (https://pgconfigurator.cybertec.at/).

Предлоположу, что с субд будет работать до 5000 пользователей, будет wal_level уровня logical.

      # DISCLAIMER - Software and the resulting config files are provided AS IS - IN NO EVENT SHALL
      # BE THE CREATOR LIABLE TO ANY PARTY FOR DIRECT, INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL
      # DAMAGES, INCLUDING LOST PROFITS, ARISING OUT OF THE USE OF THIS SOFTWARE AND ITS DOCUMENTATION.
      
      # Connectivity
      max_connections = 5000
      superuser_reserved_connections = 3
      
      # Memory Settings
      shared_buffers = '1024 MB'
      work_mem = '32 MB'
      maintenance_work_mem = '320 MB'
      huge_pages = off
      effective_cache_size = '3 GB'
      effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
      random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)
      
      # Monitoring
      shared_preload_libraries = 'pg_stat_statements'    # per statement resource usage stats
      track_io_timing=on        # measure exact block IO times
      track_functions=pl        # track execution times of pl-language procedures if any
      
      # Replication
      
      max_wal_senders = 10
      synchronous_commit = off
      
      # Checkpointing: 
      checkpoint_timeout  = '15 min' 
      checkpoint_completion_target = 0.9
      max_wal_size = '1024 MB'
      min_wal_size = '512 MB'
      
      # WAL archiving
      archive_mode = on # having it on enables activating P.I.T.R. at a later time without restart›
      archive_command = '/bin/true'  # not doing anything yet with WAL-swal_level = logical
            
      
      # WAL writing
      wal_compression = on
      wal_buffers = -1    # auto-tuned by Postgres till maximum of segment size (16MB by default)
      
      
      # Background writer
      bgwriter_delay = 200ms
      bgwriter_lru_maxpages = 100
      bgwriter_lru_multiplier = 2.0
      bgwriter_flush_after = 0
      
      # Parallel queries: 
      max_worker_processes = 1
      max_parallel_workers_per_gather = 1
      max_parallel_maintenance_workers = 1
      max_parallel_workers = 1
      parallel_leader_participation = on
      
      # Advanced features 
      enable_partitionwise_join = on 
      enable_partitionwise_aggregate = on
      jit = on
      max_slot_wal_keep_size = '1000 MB'
      track_wal_io_timing = on
      maintenance_io_concurrency = 100
      wal_recycle = on
      
      
      # General notes:
      # Note that not all settings are automatically tuned.
      #   Consider contacting experts at 
      #   https://www.cybertec-postgresql.com 
      #   for more professional expertise.

Создала файл myconfig.conf в /etc/postgresql/15/main, добавила туда настройки, убедилась, что чтение файла сконфигурировано в настройках кластера:

![Screenshot from 2024-03-10 16-21-11](https://github.com/marinesque/otus_postgresql/assets/97790878/c3dd5e06-ce7e-4e25-b13f-61aaa80001ae)

Провела тестирование до и после изменения настроек, получила профит в tps более, чем в 1,5 раза.

![Screenshot from 2024-03-10 16-22-51](https://github.com/marinesque/otus_postgresql/assets/97790878/a3942e29-fb17-4a83-9aa3-3f6a5bfb2020)

![Screenshot from 2024-03-10 16-37-11](https://github.com/marinesque/otus_postgresql/assets/97790878/ad4fcf54-97c4-4411-98ca-1f9d1bf2ff68)

![Screenshot from 2024-03-10 16-38-14](https://github.com/marinesque/otus_postgresql/assets/97790878/4a925522-b291-43b0-a862-e1e9cf1bdc1c)

![Screenshot from 2024-03-10 16-42-40](https://github.com/marinesque/otus_postgresql/assets/97790878/c57fcdf7-7870-48ca-aed0-68f55ab2ba05)
