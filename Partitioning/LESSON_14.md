Самый напрашивающийся вариант - сексионирование по годам бронирований.

~~~

create table bookings_partitioned_by_year (
  book_ref bpchar(6) not null,
  book_date timestamptz not null,
  total_amount numeric(10, 2) not null
)
partition by range(book_date);

create table bookings_partitioned_by_year_2016 partition of bookings_partitioned_by_year
  for values from ('2016-01-01'::timestamptz) to ('2017-01-01'::timestamptz);

create table bookings_partitioned_by_year_2017 partition of bookings_partitioned_by_year
  for values from ('2017-01-01'::timestamptz) to ('2018-01-01'::timestamptz);

insert into bookings_partitioned_by_year(book_date, book_ref, total_amount) select book_date, book_ref, total_amount from bookings;

select tableoid::regclass, count(*) cnt from bookings_partitioned_by_year group by tableoid;

explain
select count(*) from bookings_partitioned_by_year where book_date >= '2016-10-01'::timestamptz and book_date < '2016-11-01'::timestamptz;

explain
select count(*) from bookings_partitioned_by_year where extract(year from book_date) = 2016 and extract(month from book_date) = 11;

~~~

Итого в первом запросе пользуемся тем, что сразу знаем, в какой секции искать:

![Screenshot from 2024-06-23 12-41-30](https://github.com/marinesque/otus_postgresql/assets/97790878/1718976d-aa01-4d47-ad54-402aa49ccaf6)

А во втором секционирование не будет использовано,т.к. дата завернула в функции, напрямую их нельзя вытащить под поиск секций:

![Screenshot from 2024-06-23 12-41-36](https://github.com/marinesque/otus_postgresql/assets/97790878/d49607bf-a2d0-4430-8b03-b0910d048063)


