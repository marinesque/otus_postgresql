Открываем две сессии в одной базе данных.
Выключаем режим auto commit для обеих сессий.

  *Session1:*

	create table persons(id serial, first_name text, second_name text);
	
	commit;
	
	insert into persons(first_name, second_name) values('ivan', 'ivanov');
	insert into persons(first_name, second_name) values('petr', 'petrov'); 
	
	commit;
	
	------------------------------------------------------------------------
	
	show transaction isolation level;
	
	commit;

![ Read committed ](/home/marinesque/Pictures/Screenshots/Screenshot from 2024-02-07 12-23-52.png "RESULT")

  *Session1:*

	insert into persons(first_name, second_name) values('sergey', 'sergeev');

  *Session2:*
  
	select * from persons;
  
![ Запись не видна для сессии 2 ](file:///home/marinesque/Pictures/Screenshots/Screenshot%20from%202024-02-07%2019-14-22.png  "RESULT")

Запись, вставленная сессией 1 не видна сессии 2, т.к. на уровне изоляции read committed видны только успешно завершенные транзакции,а транзакций сессии 1 еще не выполнила commit.

  *Session1:*
  
	commit;
	  
  *Session2:*
  
	select * from persons;

![ Запись видна для сессии 2 ](/home/marinesque/Pictures/Screenshots/Screenshot from 2024-02-07 19-20-14.png  "RESULT")

Теперь запись видна для сессии 2, т.к. 1 сессия закоммитила свои изменения.

  *Session2:*
  
	commit;
	  
Устанавливаем для обеих сессий уровень изоляции транзакций repeatable read.

  *Session1, Session2:*

	set transaction isolation level repeatable read;
	
Убедимся, что уровень изоляции успешно изменен:

  *Session1, Session2:*

	show transaction isolation level;
	
	commit;

![Уровень изоляции](/home/marinesque/Pictures/Screenshots/Screenshot from 2024-02-07 19-28-31.png  "RESULT")

  *Session1:*
  
	insert into persons(first_name, second_name) values('sveta', 'svetova');

  *Session2:*
  
	select * from persons;

![Запись, вставленная сессией 1, не видна для сессии 2](/home/marinesque/Pictures/Screenshots/Screenshot from 2024-02-07 19-30-39.png  "RESULT")

  *Session1:*
  
	commit;
	  
  *Session2:*
  
	select * from persons;

![Запись, вставленная сессией 1, до сих пор не видна для сессии 2](/home/marinesque/Pictures/Screenshots/Screenshot from 2024-02-07 19-34-58.png  "RESULT")

Запись, вставленная сессией 1, до сих пор не видна для сессии 2, потому что на уровне repeatable read обеспечивается согласованность данных на протяжении всей транзакции. Изменения других транзакций не видны. Виден снимок данных на начало транзакции. ([PostreSQL.Transaction Isolation - Official documentation](https://www.postgresql.org/docs/15/transaction-iso.html) )