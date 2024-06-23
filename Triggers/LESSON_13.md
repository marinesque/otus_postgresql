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

alter table good_sum_mart add constraint uq_good_sum_mart_good_name unique (good_name);

truncate table good_sum_mart;
truncate table sales;
select * from good_sum_mart;
select * from sales;


create or replace function trigger_for_sales()
returns trigger
as $$
declare
    v_good_id integer;
    v_good_price numeric(12, 2) := 0;
begin
    v_good_id := case TG_OP when 'DELETE' then OLD.good_id else NEW.good_id end;
    v_good_price := coalesce((select good_price from goods where goods_id = v_good_id), 0);

    insert into good_sum_mart (good_name, sum_sale)
    select g.good_name, coalesce(good_price * NEW.sales_qty, 0)
    from goods g
    where g.goods_id = v_good_id
    on conflict (good_name)
    do update
    set sum_sale = coalesce(good_sum_mart.sum_sale +
        case TG_OP when 'DELETE' then - v_good_price * OLD.sales_qty
                   when 'UPDATE' then - v_good_price * OLD.sales_qty + v_good_price * NEW.sales_qty
                   when 'INSERT' then + v_good_price * NEW.sales_qty end, 0);

    return NEW;
end;
$$
language plpgsql;


create trigger tg_sales
after insert or update or delete
on sales for row execute function trigger_for_sales();
~~~

Собственно, все бы ничего, но если у нас меняется цена товара, мы не сможем учесть это в витрине. И не знаем, сколько товара продано по этой цене.
Надо или вести историчность и время изменения данных (scd2, например), или фиксировать цену продаци в sales.
