Работаю с Postgresql 15.

1. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

Предполагаю, что данных настроек хватит:

![Screenshot from 2024-03-05 22-19-08](https://github.com/marinesque/otus_postgresql/assets/97790878/0a213e2e-3952-4316-a2b4-5735a39e1189)

Рестартую Postgresql:

![Screenshot from 2024-03-05 22-19-24](https://github.com/marinesque/otus_postgresql/assets/97790878/65eeb761-f06f-4fcc-9526-e608209ad200)

Посмотрела логи, а их нет. Включила логирование и еще раз рестартунулась:

![Screenshot from 2024-03-05 22-50-26](https://github.com/marinesque/otus_postgresql/assets/97790878/058e4e02-318a-4ea8-8ccc-6b91336e4263)

2. Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах.
   Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны.
   Пришлите список блокировок и объясните, что значит каждая.

Создала таблицу для тестов. Запустила из одной сессии обновление:

![Screenshot from 2024-03-05 22-26-31](https://github.com/marinesque/otus_postgresql/assets/97790878/ffbe4869-4971-416f-abb9-4ddf0307513b)

Запустила обновление из другой сессии:

![Screenshot from 2024-03-05 22-26-48](https://github.com/marinesque/otus_postgresql/assets/97790878/0f30505d-d061-4439-bd87-c64df23f1ac2)

Посмотрела логи, увидела локи:

![Screenshot from 2024-03-05 22-41-15](https://github.com/marinesque/otus_postgresql/assets/97790878/fe5894e5-c44e-4f47-b27f-220d66b5da51)

![Screenshot from 2024-03-05 22-55-32](https://github.com/marinesque/otus_postgresql/assets/97790878/3072c54d-bcba-427b-9b36-c172bf1f5306)

В блокировках увидела занятный лок второй транзакции, которая пыталась обновить строку:

![Screenshot from 2024-03-05 22-36-56](https://github.com/marinesque/otus_postgresql/assets/97790878/fc4d6cfe-0665-40e4-a470-1f41d84b7247)

Почему именно transactionid тип sharelock granted - false?
Остальные понятно, это блокировки номеров транзакций и блокировка строки в таблице.

3. Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

Сессия 1:

![Screenshot from 2024-03-07 11-18-31](https://github.com/marinesque/otus_postgresql/assets/97790878/f6588bd6-01e4-46b5-8442-03f499143bbd)

Сессия 2:

![Screenshot from 2024-03-07 11-18-35](https://github.com/marinesque/otus_postgresql/assets/97790878/d5f6783c-9683-4ffb-9791-17fa1e8db26f)

Сессия 3:

![Screenshot from 2024-03-07 11-18-54](https://github.com/marinesque/otus_postgresql/assets/97790878/27228906-5d73-48f2-9790-96d2043ff53e)

Результат - убило транзакцию сессии 3:

![Screenshot from 2024-03-07 11-17-06](https://github.com/marinesque/otus_postgresql/assets/97790878/c0680131-8896-4c94-922f-74790d732167)


В логах отражено, кто ждал, кто убит, что хотел выполнить, как5им процессом блокировался:

![Screenshot from 2024-03-07 12-14-39](https://github.com/marinesque/otus_postgresql/assets/97790878/c5da9b68-226f-4d63-877e-bf50ddfd1d5f)

4. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

Если команды исполняются единственными в транзакции, скорее всего не смогут, кто-то так или иначе заблокируется первым. На таком уровне обычно блокировки эскалируются до таблицы.

А если в транзакциях будет заблокирована часть строк звзаимоисключающая друг друга, более мелкая грануляция блокировки, то update смогут заблокировать друг друга.

Сессия 1:

![Screenshot from 2024-03-07 12-33-22](https://github.com/marinesque/otus_postgresql/assets/97790878/44324608-ef7a-4393-bee9-8496f3266e27)

Сессия 2:

![Screenshot from 2024-03-07 12-33-32](https://github.com/marinesque/otus_postgresql/assets/97790878/419854d2-cde0-4967-ac68-5417d0deb7da)

Результат:

![Screenshot from 2024-03-07 12-30-09](https://github.com/marinesque/otus_postgresql/assets/97790878/6feb9b36-2c50-4fa8-8cc6-1b866f43a9db)








