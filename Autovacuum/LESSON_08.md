Работаю с VirtualBox.

Машина 1:

![Screenshot from 2024-03-04 20-19-53](https://github.com/marinesque/otus_postgresql/assets/97790878/0fb86116-4855-45a4-81b8-081e69d44914)


Ставлю Postgresql 15:

![Screenshot from 2024-03-04 19-25-18](https://github.com/marinesque/otus_postgresql/assets/97790878/1dacf840-899c-4867-bf88-7b14a6006bbf)


![Screenshot from 2024-03-04 19-27-20](https://github.com/marinesque/otus_postgresql/assets/97790878/d8924905-5174-4e1b-800e-69155f59b264)


![photo_5420368282717902998_x](https://github.com/marinesque/otus_postgresql/assets/97790878/18071162-8d76-446e-ab08-2f02ed377ac5)


![photo_5420368282717902999_x](https://github.com/marinesque/otus_postgresql/assets/97790878/5c655680-471a-4511-946c-93f6f92eb98d)


Инициализация pgbench:

![photo_5420368282717903001_x](https://github.com/marinesque/otus_postgresql/assets/97790878/3ff39027-e102-4d73-98ea-2684343a6401)


![photo_5420368282717903002_x](https://github.com/marinesque/otus_postgresql/assets/97790878/c37986f2-a242-4e29-9556-48c522ec82ad)


Машина 2:

![Screenshot from 2024-03-05 21-09-39](https://github.com/marinesque/otus_postgresql/assets/97790878/5751b70c-f6c2-4702-b425-b97859e51829)


Ставлю Postgresql 11:

![photo_5420368282717902906_x](https://github.com/marinesque/otus_postgresql/assets/97790878/b84f1f6f-4904-4995-9a16-80b161136e43)


Половину конфигов нашла в файле:

![photo_5420368282717902905_x](https://github.com/marinesque/otus_postgresql/assets/97790878/17cbdeb3-071f-4081-9d6f-d9f6fcf2b8e4)


Половину изменила в бд:

![photo_5420368282717902907_x](https://github.com/marinesque/otus_postgresql/assets/97790878/bda07716-ae57-4021-b8ed-11949638ba89)


Перезапустила postgresql:

![photo_5420368282717902908_x](https://github.com/marinesque/otus_postgresql/assets/97790878/37e4d746-92fe-4631-a9e6-83c6d3e73299)


![photo_5420368282717902909_x](https://github.com/marinesque/otus_postgresql/assets/97790878/382b8da8-c6be-458e-a818-cd69ec797925)

Тут я задумалась, зачем нужно было создавать таблицу для тестов? Вероятно, нужно было туда писать тесты pgbench.
Также, если бы машины были на ssd/hdd, вероятно, это повлияло бы на скорость записи и чтения с дисков и показатели значительно разнились бы.

----------------------------------------------------------------------------------------------------------------------------------------

Postgresql 15:

Создала таблицу, добавила миллион сгенерированных рандомно строк. Пока писала скрипты, пришел автовакуум и все почистил.
Поэтому я отключила на таблице его, чтобы успеть запечатлеть армию мертвецов.
Размер файла после включения автовакуума не изменился, т.к. никакой реструктуризации строк на страницах не произошло.

![Screenshot from 2024-03-05 21-36-01](https://github.com/marinesque/otus_postgresql/assets/97790878/14ed1e0c-81f3-4571-855c-cf86a3216503)

![Screenshot from 2024-03-05 21-35-19](https://github.com/marinesque/otus_postgresql/assets/97790878/759a8c1c-50e4-4623-9050-4799372bc382)

![Screenshot from 2024-03-05 21-28-16](https://github.com/marinesque/otus_postgresql/assets/97790878/f29b5ad3-c68c-45bb-bb4d-b131e43a4ac5)


