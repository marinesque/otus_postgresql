Шпаргалку не качала и не смотрела, буду объяснять, как понимаю логику.

Исходные данные: 
У меня есть уже локально установленный кластер Postgresql 15 версии:

![Screenshot from 2024-02-24 11-21-24](https://github.com/marinesque/otus_postgresql/assets/97790878/9a3d5037-7ea3-4632-88f9-deb043211c6f)

Суперюзер postgres, зайдем под ним:

![Screenshot from 2024-02-24 11-20-58](https://github.com/marinesque/otus_postgresql/assets/97790878/a77be31f-01cb-432a-a3b1-9dd115b85a87)

Сохздаем базу testdb и схему testnm в ней:

![Screenshot from 2024-02-24 11-33-58](https://github.com/marinesque/otus_postgresql/assets/97790878/c0a5156f-8070-47c7-9c88-649e6cf64ffd)

Вот, что получилось:

![Screenshot from 2024-02-24 11-35-09](https://github.com/marinesque/otus_postgresql/assets/97790878/e8378480-caeb-45d1-9be2-a11528db86c1)

В созданной схеме создадим таблицу и вставим в нее значение:

![Screenshot from 2024-02-24 12-15-36](https://github.com/marinesque/otus_postgresql/assets/97790878/5ccda07b-918b-4575-87c8-ef83ee092389)

Создаем роль readonly со следующими правами:

![Screenshot from 2024-02-24 11-43-17](https://github.com/marinesque/otus_postgresql/assets/97790878/23c8219e-8106-45bc-80af-289d78158373)

Создадим пользователя testread со следующим паролем для входа:

![Screenshot from 2024-02-24 11-44-35](https://github.com/marinesque/otus_postgresql/assets/97790878/40b24392-2866-4fc6-9e24-4de7f11a900d)

Изменим членство роли:

![Screenshot from 2024-02-24 11-54-01](https://github.com/marinesque/otus_postgresql/assets/97790878/67ad9670-f09f-43e2-bc21-c9e767264137)

Авторизуемся под новым пользователем:

![Screenshot from 2024-02-24 11-56-00](https://github.com/marinesque/otus_postgresql/assets/97790878/23f93024-f4e3-4c51-bee9-0710b11b72a4)

Успешно:

![Screenshot from 2024-02-24 11-57-07](https://github.com/marinesque/otus_postgresql/assets/97790878/9c97bc34-074a-4a56-89a1-61d7ad3ac41e)

Сделаем выборку из созданной таблицы:

![Screenshot from 2024-02-24 11-58-13](https://github.com/marinesque/otus_postgresql/assets/97790878/b0147e3b-44a1-4efe-862b-0954c2535321)

Я так поняла, что выше нужно было создать таблицу в схеме public, где по умолчанию создалась бы таблица, если не указать схему явно.
Подумала, что надо было сразу в схеме testnm ее создать.
Ок, пробуем дропнуть старую таблицу и создать заново без указания схемы под пользователем postgres:

![Screenshot from 2024-02-24 12-21-07](https://github.com/marinesque/otus_postgresql/assets/97790878/3d4386bd-60ca-4ae4-aaca-46208cf43bba)

Пробуем сделать выборку из этой таблицы под пользователем testread:

![Screenshot from 2024-02-24 12-02-04](https://github.com/marinesque/otus_postgresql/assets/97790878/65a10cd1-d388-4fd9-bfb0-6332d8bd0637)

Неуспешно. Но мы в public никакие права пользователю testdb и роли readonly не давали.
Попробуем создать таблицу без указания схемы под пользователем testread:

![Screenshot from 2024-02-24 12-03-08](https://github.com/marinesque/otus_postgresql/assets/97790878/0bd4811f-9a9e-49e1-8154-7da44f09b011)

Также нет прав на public вообще.
А вот если дать роли readonly в схеме testnm помимо usage объектов еще и создавать объекты самой (исполняем под пользователем postgres):

![Screenshot from 2024-02-24 12-08-49](https://github.com/marinesque/otus_postgresql/assets/97790878/3e1b34ba-180a-4aec-b9c2-aa6b8947b4ce)

То testread участник роли readonly уже может в этой схеме еще и создать таблицу:

![Screenshot from 2024-02-24 12-09-24](https://github.com/marinesque/otus_postgresql/assets/97790878/d1471ee8-9fb0-4e44-a276-616573e12775)
