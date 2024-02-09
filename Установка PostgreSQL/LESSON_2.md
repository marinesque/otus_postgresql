Исходные данные:
Домашняя ОС Ubuntu 22.04.3 LTS
Установлен Docker version 25.0.3, build 4debf41
Пробую поднять два контейнера (сервер, клиент) и подключиться с клиента к серверу и с домашнего ПК к серверу.

Создаю сеть:

	sudo docker network create cherrypaha-net

Создаю контейнер с сервером:

![Контейнер с сервером](https://github.com/marinesque/otus_postgresql/assets/97790878/43504882-a9ed-4c48-9c6e-a709c137a696)

Создала на 5433 порте, т.к. 5432 занят у меня локальным домашним/учебным кластером.

Проверяю, что контейнер успешно поднят:

![Контейнер с сервером поднят](https://github.com/marinesque/otus_postgresql/assets/97790878/6cbaf0ae-b57a-475c-9035-9b93d699829c)

Поднимаем контейнер с клиентом и подключаемся к базе в контейнере с сервером:

![Контейнер с клиентом](https://github.com/marinesque/otus_postgresql/assets/97790878/097ab72b-e753-4cd1-8b06-2cab4c04784c)

Создаю базу данных otus:

![Create database](https://github.com/marinesque/otus_postgresql/assets/97790878/6ca8c5d8-da92-4a17-867f-770aaed78206)

Создаю таблицу marina:

![Create table](https://github.com/marinesque/otus_postgresql/assets/97790878/20477ff1-a720-42fa-9065-a15160e5b618)

Заполняю таблицу фиктивным значением:

![Insert into table](https://github.com/marinesque/otus_postgresql/assets/97790878/f39f9543-edb3-47e4-91ad-8348d0fac8df)

Выхожу и проверяю, что у меня поднято два контейнера:

![SDocker ps](https://github.com/marinesque/otus_postgresql/assets/97790878/88399911-ac0f-43d0-9aa2-1b9370384806)

Стопаю все контейнеры, а контейнер с клиентом еще и удаляю:

![Docker stop](https://github.com/marinesque/otus_postgresql/assets/97790878/923e76d8-7c85-44cd-8885-745ad076c5f0)

![Docker rm](https://github.com/marinesque/otus_postgresql/assets/97790878/bb3ec9d0-db6a-4b8f-a999-458686e15cb8)

Проверяю, что поднятых контейнеров нет:

![Docker ps](https://github.com/marinesque/otus_postgresql/assets/97790878/f7f2dd10-1a4d-4c9e-b575-ce9649137341)

Пытаюсь поднять контейнер снова:

![Контейнер с сервером](https://github.com/marinesque/otus_postgresql/assets/97790878/7ef315b0-60bc-4510-a5b7-a10f34369c34)

И пытаюсь подключиться к базе:

	psql -h localhost -U postgres -d postgres
 
Здесь я подключилась не к локальному postgres, а не к контейнеру с сервером. Начинаю думать, где я ошиблась?

Попинговавшись в 5433 порт, поняла, что с контейнер я тоже предполагала подключиться к 5433 порту, а там нет никого):

	psql -h localhost -U postgres -d postgres -p 5433

Тушу контейнер с сервером, переподнимаю его с другими портами хоста и контейнера:

![Контейнер с сервером](https://github.com/marinesque/otus_postgresql/assets/97790878/8877617b-aa90-416d-9a97-435c278f2f57)

Благополучно нахожу свою базу otus. Иду искать таблицу marina, но не нахожу. 
Понимаю в этот момент, что я не переключилась на базу otus, когда подключалась из контейнера с клиентом.
Значит моя таблица должна быть в базе, к которой я подключилась, когда только зашла. И в самом деле:

![Where is a table?](https://github.com/marinesque/otus_postgresql/assets/97790878/7995241e-2f8b-47b8-ae30-507a5f8d8ff0)

