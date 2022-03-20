## 1.

> Список БД
```commandline
postgres=#            \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

```
> Подключение к БД
```commandline
postgres=# \c template1
You are now connected to database "template1" as user "postgres".
```

>  Вывод списка таблиц
```commandline
test_database=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | orders | table | postgres
(1 row)

```

> Описание таблиц
```commandline
test_database=# \d orders
                                   Table "public.orders"
 Column |         Type          | Collation | Nullable |              Default               
--------+-----------------------+-----------+----------+------------------------------------
 id     | integer               |           | not null | nextval('orders_id_seq'::regclass)
 title  | character varying(80) |           | not null | 
 price  | integer               |           |          | 0
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)

```

> Выход 
```commandline
test_database=# \q
```

## 2.
```commandline
test_database=# select attname, avg_width from pg_stats where avg_width = (select max(avg_width) from pg_stats where tablename = 'orders');
 attname | avg_width 
---------+-----------
 title   |        16
(1 row)

```

## 3.

```commandline
CREATE TABLE new_orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL,
    price integer DEFAULT 0
)PARTITION BY RANGE(price);

create table orders_2 PARTITION OF new_orders FOR VALUES FROM (MINVALUE) TO (500);
create table orders_1 PARTITION OF new_orders FOR VALUES FROM (500) TO (MAXVALUE);

insert into orders_1 (id,title,price) select id,title,price from orders where price>500;
insert into orders_2 (id,title,price) select id,title,price from orders where price <= 499;
ALTER TABLE orders RENAME TO ordesr_backup;
ALTER TABLE new_orders RENAME TO orders;
```

>При проектировании таблицы можно было предусмотреть заранее разбиение таблицы, можно использовать триггеры, скрипты, правила для вставки данных.

## 4.

```commandline
root@df3ce47da307:/# pg_dump -U postgres test_database > /tmp/backup_bd.sql
root@df3ce47da307:/# 
```

в бэкап добавил следующие органичения
```commandline
-- Name: orders_1 uniq_or1; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.orders_1
    ADD CONSTRAINT uniq_or1 UNIQUE (title);


--
-- Name: orders_2 uniq_or2; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.orders_2
    ADD CONSTRAINT uniq_or2 UNIQUE (title);


--
-- PostgreSQL database dump complete
--

```
