~~~

drop table if exists t1;
create temp table t1(map integer, col1 text);
drop table if exists t2;
create temp table t2(map integer, col2 text);
drop table if exists t3;
create temp table t3(id integer,  col3 text);

insert into t1(map, col1)
values
  (1, 'A'),
  (2, 'B'),
  (3, 'C'),
  (4, 'A'),
  (5, 'B'),
  (6, 'C'),
  (7, 'A');

insert into t2(map, col2)
values
  (0, 'A'),
  (2, 'B'),
  (8, 'C'),
  (4, 'D'),
  (5, 'A'),
  (9, 'B'),
  (7, 'C');

insert into t3(id, col3)
values
  (0, 'A'),
  (11, 'B'),
  (22, 'C'),
  (33, 'D'),
  (44, 'A'),
  (55, 'B'),
  (66, 'C');

--Запись по map есть и в t1 и в t2
select *
from t1
join t2 on t1.map = t2.map;

--Запись по map есть или в t1 или в t2
select *
from t1
full outer join t2 on t1.map = t2.map;

--Запись из t1 и соответствующая запись из t3, если она есть, или пусто, если нет
select *
from t1
left join t2 on t1.map = t2.map
left join t3 on t2.col2 = t3.col3;

--Запись из t1 и t2 по полному совпадению и/или запись t3 при совпадении col-колонок
select *
from t1
inner join t2 on t1.map = t2.map
full outer join t3 on t2.col2 = t3.col3;

drop table if exists tab;
create temp table tab(col text);
drop table if exists multiplier;
create temp table multiplier(id integer);

insert into tab(col)
values ('A'), ('B'), ('C'), ('D');

insert into multiplier (id)
values (1), (2);

select *
from tab t
cross join multiplier m;


~~~
