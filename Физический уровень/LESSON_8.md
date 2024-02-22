Отправная точка:
У меня уже есть виртуальная машина в Virtual B
ox c Ubuntu и Postgresql 14:
![Screenshot from 2024-02-22 19-48-20](https://github.com/marinesque/otus_postgresql/assets/97790878/27d96dfe-9078-4c97-bb92-95898a501c95)

![Screenshot from 2024-02-22 19-50-37](https://github.com/marinesque/otus_postgresql/assets/97790878/12fafe76-5cf8-4208-beaa-4fc8cfe2ec17)

Выключаю виртуальную машину и создаю еще один диск под нее на 10 гигабайтов:
![Screenshot from 2024-02-22 20-47-24](https://github.com/marinesque/otus_postgresql/assets/97790878/5e46ce7d-78d4-45ce-9023-3bd3f24b782b)

Включаю вм снова и проверяю, что диск приаттачился:
![Screenshot from 2024-02-22 19-50-37](https://github.com/marinesque/otus_postgresql/assets/97790878/b9c2ab0a-7688-4082-b2fe-3ac3853dcf73)

Воспользовалась ссылкой и установила утилиту parted, чтобы дальше работать с диском:
![Screenshot from 2024-02-22 21-16-56](https://github.com/marinesque/otus_postgresql/assets/97790878/8e751c93-5464-4bf5-a002-6a0248c54f0f)

Создаю раздел, оставила один цельный раздел, но на 2048 байтов не делится полностью, "хвостик" откусило:
![Screenshot from 2024-02-22 21-24-12](https://github.com/marinesque/otus_postgresql/assets/97790878/569750ab-eece-40e3-9051-1e532d2a2b98)

Но сам раздел успешно создался:
![Screenshot from 2024-02-22 21-24-56](https://github.com/marinesque/otus_postgresql/assets/97790878/39c30a87-3298-4431-b3be-18c5fb317a85)

Я забыла определить ext4 для fs, монтирование упало с ошибкой несоместимости, поэтому пришлось доделать:
![Screenshot from 2024-02-22 21-33-32](https://github.com/marinesque/otus_postgresql/assets/97790878/29044f56-fb3b-48b3-ba62-66f7fb849d60)

![Screenshot from 2024-02-22 21-35-34](https://github.com/marinesque/otus_postgresql/assets/97790878/ffe1a6f7-2158-4741-acee-076e7fca5dd9)

Правда label назвала не datapartition, а patapartition):
![Screenshot from 2024-02-22 21-36-04](https://github.com/marinesque/otus_postgresql/assets/97790878/ab285a23-a131-4b66-9884-703587beb65b)

Теперь все нормально примонтировалось:
![Screenshot from 2024-02-22 21-36-30](https://github.com/marinesque/otus_postgresql/assets/97790878/f6944c8a-e45e-44ac-b7e0-290d99eddc89)

Меняю права:
![Screenshot from 2024-02-22 21-36-57](https://github.com/marinesque/otus_postgresql/assets/97790878/02b5d3a0-62a4-447f-b3e0-50cb44bae79b)

Попробовала создать табличное пространство:
![Screenshot from 2024-02-22 21-40-42](https://github.com/marinesque/otus_postgresql/assets/97790878/fb9d8045-da4d-4b59-aaff-b9979cab01d2)

![Screenshot from 2024-02-22 21-40-58](https://github.com/marinesque/otus_postgresql/assets/97790878/69b4b7b2-c7ae-44ff-a03e-dc6923c4c023)

Попыталась сменить овнера на каталог:
![Screenshot from 2024-02-22 21-53-00](https://github.com/marinesque/otus_postgresql/assets/97790878/3f46cb92-72be-48c7-b0fd-58234942f417)

Думала, что проблема в пароле, поэтому попробовала овнером поставить себя, но все равно получила ошибку:
![Screenshot from 2024-02-22 22-00-56](https://github.com/marinesque/otus_postgresql/assets/97790878/f09b7825-935b-4946-ad39-916373afb2fe)

Что делаю не так?
