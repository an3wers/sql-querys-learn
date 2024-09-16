-- basic

select count (product_id) from products

select * from products

select count (distinct country) from employees

select company_name, contact_name, phone, country from customers where country like '%US%'

select * from products where unit_price > 20

select count (*) from products where unit_price > 10

select * from customers where city <> 'Berlin'

select * from orders where order_date > '1998-03-01'

-- and, or

select * from products where unit_price > 25 and units_in_stock > 40

select * from customers where city = 'Berlin' or city = 'London'

select * from orders where shipped_date > '1998-04-30' and (freight < 75 or freight > 150)

-- интервал

select count (*) from orders where freight >= 20 and freight <= 40

select count (*) from orders where freight between 20 and 40

-- интервал дат

select * from orders where order_date between '1998-03-30' and '1998-04-03'

-- in, not in

select * from customers where country = 'USA' or country = 'Mexico' or country = 'Canada'

select * from customers where country in ('USA', 'Mexico', 'Canada')

select * from customers where country not in ('USA', 'Mexico', 'Canada')

-- order

select distinct country from customers

select distinct country from customers order by country
select distinct country from customers order by country asc

select distinct country from customers order by country desc

select distinct country, city from customers order by country desc, city desc

select distinct country, city from customers order by country desc, city asc

-- 

select ship_city, order_date from orders where ship_city = 'London' order by order_date desc

select min(order_date) from orders where ship_city = 'London'

select max(order_date) from orders where ship_city = 'London'

-- вычесление средней цены

select avg(unit_price) from products where discontinued <> 1

-- сумма

select sum(units_in_stock) from products where discontinued <> 1

-- like

select last_name, first_name from employees where first_name like '%n'

select last_name, first_name from employees where first_name like 'A%'

-- limit

select * from products where discontinued <> 1 order by unit_price desc limit 10

-- check on null

select * from orders where ship_region is null

select * from orders where ship_region is not null

-- group by

select ship_country, count (*) from orders where freight > 50 group by ship_country order by count (*) desc

select category_id, sum(units_in_stock) from products group by category_id order by sum(units_in_stock) desc limit 5

-- having повторный фильтр

select category_id, sum(unit_price * units_in_stock) from products where discontinued <> 1
group by category_id having sum(unit_price * units_in_stock) > 5000 order by sum(unit_price * units_in_stock) desc

-- union

select country from customers
union
select country from employees

--  intersect

select country from customers
intersect
select country from suppliers

-- except

select country from customers
except
select country from suppliers


-- join
-- inenr join

select products.product_id, products.product_name, suppliers.company_name, products.units_in_stock
from products
inner join suppliers on products.supplier_id = suppliers.supplier_id
order by units_in_stock

-- с алиасами к таблице
select p.product_id, p.product_name, s.company_name, p.units_in_stock
from products as p
inner join suppliers as s
on p.supplier_id = s.supplier_id
order by units_in_stock


select categories.category_name, sum(products.units_in_stock)
from products
inner join categories on products.category_id = categories.category_id
group by categories.category_name
order by sum(products.units_in_stock) desc


select categories.category_name, sum(products.unit_price * units_in_stock)
from products
inner join categories on products.category_id = categories.category_id
where discontinued <> 1
group by categories.category_name
having sum(products.unit_price * units_in_stock) > 5000
order by sum(products.unit_price * units_in_stock) desc


select orders.order_id, orders.customer_id, employees.first_name, employees.last_name, employees.title
from orders
inner join employees on orders.employee_id = employees.employee_id
limit 10

-- объединение трех таблиц

select o.order_date, o.ship_country, pr.product_name, pr.unit_price, od.quantity, od.discount
from orders as o
inner join order_details as od
on o.order_id = od.order_id
inner join products as pr
on od.product_id = pr.product_id

-- left join

select products.product_id, products.product_name, suppliers.company_name, products.units_in_stock
from products
left join suppliers
on products.supplier_id = suppliers.supplier_id
order by units_in_stock

select company_name, order_id
from customers
join orders
on orders.customer_id = customers.customer_id
where order_id is null

select company_name, order_id
from customers
left join orders
on orders.customer_id = customers.customer_id
where order_id is null

-- right
select company_name, order_id
from orders
right join customers
on orders.customer_id = customers.customer_id
where order_id is null

--

select last_name, order_id
from employees
left join  orders
on orders.employee_id = employees.employee_id
where order_id is null

select count(*)
from employees
left join  orders
on orders.employee_id = employees.employee_id
where order_id is null

-- using

select o.order_date, o.ship_country, pr.product_name, pr.unit_price, od.quantity, od.discount
from orders o
inner join order_details od using(order_id) --on o.order_id = od.order_id
inner join products pr using(product_id) -- on od.product_id = pr.product_id

-- as

select count(*) as employees_count
from employees

select count(distinct country) as country_count
from employees

select category_id, sum(units_in_stock) as units_in_stock
from products
group by category_id
order by units_in_stock desc
limit 5

select category_id, sum(unit_price * units_in_stock) as total
from products
where discontinued <> 1
group by category_id
having sum(unit_price * units_in_stock) > 5000 -- алиас total с having не будет работать
order by total desc

-- подзапросы

-- задача: определить страны в которые делают заказы заказчики

select * from suppliers 

select company_name, country
from suppliers
where country
in (
	select distinct country
	from customers
)

-- то же, без подзапроса

select distinct s.company_name
from suppliers s
join customers using(country)

-- 

select category_name, sum(units_in_stock) as units_in_stock
from products
inner join categories using(category_id)
group by category_name
order by units_in_stock desc
limit(select min(product_id) + 4 from products)

--

select avg(units_in_stock) from products

select product_name, units_in_stock
from products
where units_in_stock > (select avg(units_in_stock) from products)
order by units_in_stock

-- where exists

select company_name, contact_name
from customers
where exists (
	select customer_id from orders
	where customer_id = customers.customer_id
	and freight between 50 and 100
)

select company_name, contact_name
from customers
where not exists (
	select customer_id from orders
	where customer_id = customers.customer_id
	and order_date between ('1995-02-01') and ('1996-02-15')
)


-- Задача: продукты, который не продавали за определенный период

select product_name, unit_price
from products
where not exists (
	select orders.order_id from orders
	join order_details using(order_id)
	where order_details.product_id = product_id
	and order_date between '1995-02-01' and '1995-02-15'
)
order by unit_price desc

-- подзапросы с any, all

select distinct company_name
from customers
join orders using(customer_id)
join order_details using(order_id)
where quantity > 40

-- то же с подзапросом

select * from orders

select customer_id from orders
join order_details using(order_id)
where quantity > 40

select distinct company_name
from customers
where customer_id = any(
	select customer_id from orders
	join order_details using(order_id)
	where quantity > 40
)

-- 

select avg(quantity) from order_details

select distinct product_name, quantity
from products
join order_details using(product_id)
where quantity > (
	select avg(quantity) from order_details
)
order by quantity

-- all

select avg (quantity) from order_details
group by product_id

select distinct product_name, quantity
from products
join order_details using(product_id)
where quantity > all (
	select avg (quantity) from order_details group by product_id
)
order by quantity

-- ddl

-- создание таблицы
create table students
(
	student_id serial,
	first_name: varchar(255),
	last_name: varchar(255),
	bithday: date,
	phone varchar(32)
)

create table cathedras
(
	cathedra_id: serial
	cathedra_name: varchar(255),
	dean: varchar(255),
)

-- добавление колонок
alter table students
add column middle_name varchar(255)

alter table students
add column rating float

alter table students
add column enrolled date

-- удаление колонок
alter table students
drop column middle_name

-- переименовать таблицу
alter table cathedras
rename to chairs

-- переименовать столбец
alter table chairs
rename cathedra_id to chair_id

alter table chairs
rename cathedra_name to chair_name

-- поменять тип данных колонки
alter table students
alter column first_name set data type varchar(60)

alter table students
alter column last_name set data type varchar(60)

-- удаление данных из таблицы

create table faculties
(
	faculty_id: serial,
	faculty_name: varchar
)

insert into faculties (faculty_name)
values
('faculty 1'),
('faculty 2'),
('faculty 3');

-- удаляет данные из таблицы, но оставляет таблицу
truncate table faculties
truncate table faculties restart identity -- рестарт id

-- удалить таблицу полностью
drop table faculties


-- PK
create table faculties
(
	faculty_id: serial primary key,
	faculty_suid: int unique not null,
	faculty_name: varchar,
	dean: varchar
)

create table faculties
(
	faculty_id: serial,
	faculty_suid: int unique not null,
	faculty_name: varchar,
	dean: varchar
	dean_id: int
	
	-- еще один способ записать primary key
	constraint pk_faculties_faculty_id primary key(faculty_id)
)

-- primary key на всю таблицу может быть один

-- добавить primary key к таблице

alter table faculties
add primary key(faculty_id)

-- удалить ограничение
alter table faculties
drop constraint pk_faculties_faculty_id

-- update, delete, returning

select * from employees_usa order by employee_id
update employees_usa set last_name = 'Bavolio' where employee_id = '1'

-- delete

select * from employees
delete from employees where employee_id = '10' 

delete from employees_usa -- удаление всех строк

truncate table employees_usa -- тоже удаление всех строк, работает быстрее и без логов

-- returning

select * from employees_usa order by employee_id

insert into employees_usa (employee_id, last_name, first_name)
values(10, 'Jhon', 'Smith')
returning *

-- можно использовать также при update

-- FK

alter table faculties
add constraint fk_faculties_dean foreign key(dean_id) references deans(dean_id)

-- условия
 -- case when

SELECT PRODUCT_NAME,
	UNIT_PRICE,
	UNITS_IN_STOCK,
	CASE
					WHEN UNITS_IN_STOCK >= 100 THEN 'lots of'
					WHEN UNITS_IN_STOCK >= 50
										AND UNITS_IN_STOCK < 100 THEN 'average'
					WHEN UNITS_IN_STOCK < 50 THEN 'low number'
					ELSE 'unknown'
	END AS AMOUNT
FROM PRODUCTS
ORDER BY UNITS_IN_STOCK --

SELECT ORDER_ID,
	ORDER_DATE,
	CASE
					WHEN DATE_PART('month',

											ORDER_DATE) BETWEEN 3 AND 5 THEN 'spring'
					WHEN DATE_PART('month',

											ORDER_DATE) BETWEEN 6 AND 8 THEN 'summer'
					WHEN DATE_PART('month',

											ORDER_DATE) BETWEEN 9 AND 11 THEN 'autumn'
					ELSE 'wunter'
	END AS SEASON
FROM ORDERS --

SELECT PRODUCT_NAME,
	UNIT_PRICE,
	CASE
					WHEN UNIT_PRICE >= 30 THEN 'Expensive'
					WHEN UNIT_PRICE < 30 THEN 'inexpensive'
					ELSE 'Unknown'
	END AS PRICE_DESCRIPTION
FROM PRODUCTS -- coalesce, nullif

SELECT *
FROM ORDERS
LIMIT 10
SELECT ORDER_ID,
	ORDER_DATE,
	COALESCE(SHIP_REGION,

		'unknown') AS SHIP_REGION
FROM ORDERS
LIMIT 10
SELECT LAST_NAME,
	FIRST_NAME,
	COALESCE(REGION,

		'N/A') AS REGION
FROM EMPLOYEES

SELECT *
FROM CUSTOMERS

SELECT CONTACT_NAME,
	COALESCE(NULLIF(CITY,

											''),

		'Unknown') AS CITY
FROM CUSTOMERS
