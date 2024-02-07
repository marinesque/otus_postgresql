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
 
 ![Read committed](https://github.com/marinesque/otus_postgresql/assets/97790878/fe5ad205-3754-4ee9-9fc3-235f32796480)

  *Session1:*

	insert into persons(first_name, second_name) values('sergey', 'sergeev');

  *Session2:*
  
	select * from persons;
  
![ Запись не видна для сессии 2 ](https://github.com/marinesque/otus_postgresql/assets/97790878/43743659-d4a8-469a-bbe6-458f11f778b8)


Запись, вставленная сессией 1 не видна сессии 2, т.к. на уровне изоляции read committed вины только успешно завершенные транзакции,а транзакций сессии 1 еще не выполнила commit.

  *Session1:*
  
	commit;
	  
  *Session2:*
  
	select * from persons;

![ Запись видна для сессии 2 ](https://github.com/marinesque/otus_postgresql/assets/97790878/e22bcb90-5654-46f4-af49-8c352fd51fd4)


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

![Уровень изоляции](https://github.com/marinesque/otus_postgresql/assets/97790878/ba03b641-c8c2-469a-81a5-7349aaf1af34)


  *Session1:*
  
	insert into persons(first_name, second_name) values('sveta', 'svetova');

  *Session2:*
  
	select * from persons;

![Запись, вставленная сессией 1, не видна для сессии 2](https://github.com/marinesque/otus_postgresql/assets/97790878/b239c662-ab4b-427b-90da-9bd406845ab1)


  *Session1:*
  
	commit;
	  
  *Session2:*
  
	select * from persons;

![Запись, вставленная сессией 1, до сих пор не видна для сессии 2](https://github.com/marinesque/otus_postgresql/assets/97790878/a1496c0a-6f7a-4c9d-b52b-8d00340e1371)


Запись, вставленная сессией 1, до сих пор не видна для сессии 2, потому что на уровне repeatable read обеспечивается согласованность данных на протяжении всей транзакции. Изменения других транзакций не видны. Виден снимок данных на начало транзакции. ([PostreSQL.Transaction Isolation - Official documentation](https://www.postgresql.org/docs/15/transaction-iso.html) )
