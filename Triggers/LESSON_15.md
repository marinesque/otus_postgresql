Исходное задание:

 ~~~
-- ДЗ тема: триггеры, поддержка заполнения витрин

DROP SCHEMA IF EXISTS pract_functions CASCADE;
CREATE SCHEMA pract_functions;

SET search_path = pract_functions, public

-- товары:
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозайственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);

-- Продажи
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);

INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);

-- отчет:
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;

-- с увеличением объёма данных отчет стал создаваться медленно
-- Принято решение денормализовать БД, создать таблицу
CREATE TABLE good_sum_mart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);

-- Создать триггер (на таблице sales) для поддержки.
-- Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE

-- Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
-- Подсказка: В реальной жизни возможны изменения цен.
~~~

Функция и триггер:

~~~
create or replace function trigger_for_sales()
returns trigger
as $$
begin

case tg_op
when 'INSERT' then

    insert into good_sum_mart (good_name, sum_sale)
    select g.good_name, (g.good_price * NEW.sales_qty) as sum_sale
    from goods g
    where g.goods_id = NEW.goods_id;

    return NEW;

when 'UPDATE' then

    with t as (
        select g.good_name as name, (g.good_price * NEW.sales_qty) as sum
        from goods g
        where g.goods_id = NEW.goods_id
    )
    update good_sum_mart
    set sum_sale = t.sum
    from t
    where good_name = t.name;

    return new;

when 'DELETE' then

    with t as (select g.good_name as name
               from goods g
               where g.goods_id = OLD.goods_id
    )
    delete from good_sum_mart
    using t
    where good_name = t.name;

end case;

end;
$$
language plpgsql;


create trigger tg_sales
after insert or update or delete
on sales for row execute function trigger_for_sales();
~~~
