# Amazon-Sales-Analysis-SQL-project

## project Overview
i have worked on analyzing a dataset of over 20,000 sales records from an Amazon-like e-commerce platform. This project involves extensive querying of customer behaviour, perduct performance, and sales trends using postgreSQL. Through this project, I have tackled various SQL problems, including revenue analysis, customer segmentation, and inventory management.

This project also focuses on data cleaning, handlling null values, and solving real-world business problems using structured queries.

### An ERD diagram is include to visually represent the database schema and relationships between tables.

![](https://github.com/Tusarkant05/Amazon-Sales-Analysis-SQL-project/blob/main/Amazon%20ERD.png)

![](https://github.com/Tusarkant05/Amazon-Sales-Analysis-SQL-project/blob/main/Updated%20ERD%20-%20Amazon.png)

## Database Setup & Design

### Schema Structure

```sql
create table category
( 
  category_id int primary key,
  category_name varchar(20)
);
```

```sql
create table customers
( 
  customer_id int primary key,
  first_name varchar(20),
  last_name varchar(20),
  state varchar(20),
  address varchar(5) default ('xxxx')
);
```

```sql
create table sellers
(
  seller_id int primary key,
  seller_name varchar(25),
  origin varchar(5)
);
```

```sql
  alter table sellers
  alter COLUMN origin type varchar(10)
;
```

```sql
create table products
(
  product_id int primary key,
  product_name varchar(50),
  price float,
  cogs float,
  category_id int, --fk
  constraint product_fk_category foreign key(category_id) references category(category_id)
);
```

```sql
create table orders
(
  order_id int primary key,
  order_date date,
  customer_id int, --fk
  seller_id int, --fk
  order_status varchar(15),
  constraint order_fk_customers foreign key (customer_id) references customers(customer_id),
  constraint order_fk_sellers foreign key (seller_id) references sellers(seller_id)
);
```

```sql
create table order_items
(
  order_item_id int primary key,
  order_id int, --fk
  product_id int, --fk
  quantity int,
  price_per_unit float,
  constraint order_items_fk_orders foreign key (order_id) references orders(order_id),
  constraint order_items_fk_products foreign key (product_id) references products(product_id)
);
```

```sql
create table payments
(
  payment_id int PRIMARY key,
  order_id int, --fk
  payment_date date,
  payment_status varchar(20),
  CONSTRAINT payments_fk_order foreign key (order_id) REFERENCES orders(order_id)
);
```

```sql
CREATE TABLE shippings
(
  shipping_id	INT PRIMARY KEY,
  order_id	INT, -- FK
  shipping_date DATE,	
  return_date	 DATE,
  shipping_providers	VARCHAR(15),
  delivery_status VARCHAR(15),
  CONSTRAINT shippings_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

```sql
CREATE TABLE inventory
(
  inventory_id INT PRIMARY KEY,
  product_id INT, -- FK
  stock INT,
  warehouse_id INT,
  last_stock_date DATE,
  CONSTRAINT inventory_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

## Task: Data Cleaning

### I cleaned the dataset by:

  * Removing Duplicates: Duplicates in the customer and order tables were identified and removed.
  * Handling missing values: Null values in critical fields (e.g., customer address, payment status) were eithre filled with default value or handled using appropriate methods.

## Handling Null Values

### Null values were handled based on their context:

  * Customer addresses: Missing addresses were assigned default placeholder values.
  * Payment statuses: Orders with null payment statuses were categorized as "pending".
  * Shipping Information: Null return dates were left as is, as not all shipments are returned.

## Objective

The primary objective of this project is to showcase SQL proficiency through complex queries that address real-world e-commerce business challenges. The analysis covers various aspects of e-commerce operations, including:

  * Customer behavior
  * Sales trend
  * Inventory management
  * Payment and shipping analysis
  * Forecasting and product performance

## Identifying Business Problems

Key business problems identified:

  1. Low product availablity due to inconsistent restocking.
  2. High return rates for specific product categories.
  3. Significant delays in shipments and inconsistencies in delivery times.
  4. High customer acquisition costs with a low customer retention rate.

## Solving Business Problems

#### Solutions Implemented:

### 1. Top Selling Products Query the top 10 products by total sales value. Challenge: Include product name, total quantity sold, and total sales value.

```sql
select * from order_items;

-- Createing new COLUMNS
alter table order_items
add column total_sales float;

-- updating price qty * price per unit
update order_items
set total_sales = quantity * price_per_unit;
```

### Ans
```sql
select 
	  oi.product_id,
	  p.product_name,
	  sum(oi.total_sales) as total_sales,
	  count(o.order_id) as total_orders
from products as p
join
order_items as oi
on p.product_id = oi.product_id
join
orders as o
on oi.order_id = o.order_id
group by 1,2
order by 3 desc
limit 10 
```

### 2. Revenue by Category Calculate total revenue generated by each product category. Challenge: Include the percentage contribution of each category to total revenue.

```sql

select 
  	p.category_id,
  	c.category_name,
  	sum(oi.total_sales) as total_sale,
  	sum(oi.total_sales)/
				  	(select sum(total_sales)from order_items)
				  	*100 
  	as percent_contribution
from order_items as oi
join
products as p
on p.product_id = oi.product_id
left join
category as c
on c.category_id = p.category_id
group by 1,2
order by 3 desc
```

### 3. Average Order Value (AOV) Compute the average order value for each customer. Challenge: Include only customers with more than 5 orders.

```sql
select 
    c.customer_id,
	  concat(c.first_name, ' ',c.last_name) as full_name,
	  sum(oi.total_sales)/ count(o.order_id) as aov,
	  count(o.order_id) as total_orders
from customers as c
join
orders as o
on o.customer_id = c.customer_id
JOIN
order_items as oi
on oi.order_id = o.order_id
group by 1,2
having count(o.order_id) > 5
```


















```
