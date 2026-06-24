# Amazon-Sales-Analysis-SQL-project

## project Overview
I have worked on analyzing a dataset of over 20,000 sales records from an Amazon-like e-commerce platform. This project involves extensive querying of customer behaviour, perduct performance, and sales trends using postgreSQL. Through this project, I have tackled various SQL problems, including revenue analysis, customer segmentation, and inventory management.

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

### 4. Monthly Sales Trend Query monthly total sales over the past year. Challenge: Display the sales trend, grouping by month, return current_month sale, last month sale!

```sql
select 
		year,
		month,
		total_sale as current_month_sale,
		lag(total_sale, 1) over(order by year,month) as last_month_sale
from
(
select 
		extract(month from o.order_date) as month,
		extract(year from o.order_date) as year,
		round(
				sum(oi.total_sales::numeric
				,2) as total_sale
from orders as o
JOIN
order_items as oi
on oi.order_id = o.order_id
where o.order_date >= current_date - interval '1 year'
group by 1,2
order by 1,2
) as t1
```

### 5. Customers with No Purchases Find customers who have registered but never placed an order. Challenge: List customer details and the time since their registration.

```sql
-- Approach 1
select * from customers
where customer_id not in (select
					distinct customer_id
				from orders
				);
```

```sql
-- Approach 2
select *
from customers as c
left join
orders as o
on c.customer_id = o.customer_id
where o.customer_id is null
```

### 6. Least-Selling Categories by State Identify the least-selling product category for each state. Challenge: Include the total sales for that category within each state.

```sql
with ranking_table
as
(
select 
		cx.state,
		c.category_name,
		sum(oi.total_sales) as total_sale,
		rank()over(partition by cx.state order by sum(oi.total_sales)asc) as rank
from orders as o
join
customers as cx
on o.customer_id = cx.customer_id
join
order_items as oi
on o.order_id = oi.order_id
join
products as p
on oi.product_id = p.product_id
join
category as c
on p.category_id = c.category_id
group by 1,2
)
select
*
from ranking_table
where rank = 1
```

### 7. Customer Lifetime Value (CLTV) Calculate the total value of orders placed by each customer over their lifetime. Challenge: Rank customers based on their CLTV.

```sql
select 
		c.customer_id,
		concat(c.first_name, ' ',c.last_name) as full_name,
		sum(oi.total_sales) as cltv,
		dense_rank()over(order by sum(oi.total_sales)desc) as cx_ranking
from customers as c
join
orders as o
on o.customer_id = c.customer_id
JOIN
order_items as oi
on oi.order_id = o.order_id
group by 1,2
```

### 8. Inventory Stock Alerts Query products with stock levels below a certain threshold (e.g., less than 10 units). Challenge: Include last restock date and warehouse information.

```sql
select 
		i.inventory_id,
		p.product_name,
		i.stock as current_stock_left,
		i.last_stock_date,
		i.warehouse_id
from inventory as i
JOIN
products as p
on i.product_id = p.product_id
where stock < 10
```

### 9. Shipping Delays Identify orders where the shipping date is later than 3 days after the order date. Challenge: Include customer, order details, and delivery provider.

```sql
select
		cx.*,
		o.*,
		s.shipping_providers,
		s.shipping_date - o.order_date as days_took_to_ship
from shippings as s
join
orders as o
on s.order_id = o.order_id
join
customers as cx
on cx.customer_id = o.order_id
where s.shipping_date - o.order_date > 3
```

### 10. Payment Success Rate  Calculate the percentage of successful payments across all orders. Challenge: Include breakdowns by payment status (e.g., failed, pending).

```sql
select
		p.payment_status,
		count(*) as total_cnt,
		count(*)::numeric/(select count(*) from payments)::numeric * 100
from orders as o
join
payments as p
on o.order_id = p.order_id
group by 1
```

### 11. Top Performing Sellers Find the top 5 sellers based on total sales value. Challenge: Include both successful and failed orders, and display their percentage of successful orders.

```sql
with top_seller
as
(
SELECT	
		s.seller_id,
		s.seller_name,
		sum(oi.total_sales) as total_sale
from orders as o
join
sellers as s
on o.seller_id = s.seller_id
join
order_items as oi
on oi.order_id = o.order_id
group by 1,2
order by 3 desc
limit 5
),
seller_order_status
as
(
select
		o.seller_id,
		ts.seller_name,
		o.order_status,
		count(*) as total_orders
from orders as o
JOIN
top_seller as ts
on ts.seller_id = o.seller_id
where o.order_status not in  ('Inprogress', 'Returned') 
group by 1,2,3
)
select 
		seller_id,
		seller_name,
		sum(case
			when order_status = 'Completed' then total_orders
			else 0
		end) as completed_orders,
		sum(case
			when order_status = 'Cancelled' then total_orders
			else 0
		end) as cancelled_orders,
		sum(total_orders) as t_o,
		sum(case
			when order_status = 'Completed' then total_orders
			else 0
		end)::numeric/ sum(total_orders)::numeric * 100 as successful_orders_percentage
from seller_order_status
group by 1,2
```

### 12. Product Profit Margin Calculate the profit margin for each product (difference between price and cost of goods sold). Challenge: Rank products by their profit margin, showing highest to lowest.

```sql
select 
		p.product_id,
		p.product_name,
		sum(oi.total_sales - (p.cogs * oi.quantity)) as profit,
		sum(oi.total_sales - (p.cogs * oi.quantity))/sum(oi.total_sales)*100 as profit_margin,
		dense_rank()over(order by sum(oi.total_sales - (p.cogs * oi.quantity))/sum(oi.total_sales)*100 desc) as product_rank
from products as p
JOIN
order_items as oi
on p.product_id = oi.product_id
JOIN
orders as o
on o.order_id = oi.order_id
group by 1,2
```

### 13. Most Returned Products Query the top 10 products by the number of returns. Challenge: Display the return rate as a percentage of total units sold for each product.

```sql
select 
		p.product_id,
		p.product_name,
		count(*) as total_unit_sold,
		sum(case
			when o.order_status ='Returned' then 1 else 0 end) as total_returned,
		sum(case
			when o.order_status ='Returned' then 1 else 0 end)::numeric/count(*)::numeric *100 as returned_percentage
from products as p
JOIN
order_items as oi
on p.product_id = oi.product_id
join
orders as o
on o.order_id = oi.order_id
group by 1,2
order by 5 desc
limit 10
```

### 14. Inactive Sellers Identify sellers who haven’t made any sales in the last 6 months. Challenge: Show the last sale date and total sales from those sellers.

```sql
with cte1 -- as these seller has not done any sale in last 6 months
as
(select * from sellers
where seller_id not in (select seller_id from orders where order_date >= current_date - interval '6 month')
)
SELECT
		o.seller_id,
		max(o.order_date) as last_sale_date,
		max(oi.total_sales) as last_sale_amount
from orders as o
join
cte1
on cte1.seller_id = o.seller_id
join
order_items as oi
on o.order_id = oi.order_id
group by 1
```

### 15. IDENTITY customers into returning or new if the customer has done more than 5 return categorize them as returning otherwise new Challenge: List customers id, name, total orders, total returns

```sql
SELECT	
		customer_full_name as customer,
		total_orders,
		total_returned,
		case
			when total_returned > 5 then 'returning_customer' else 'new'
		end as cx_category
from
(
select 
		cx.customer_id,
		concat(cx.first_name, ' ', cx.last_name) as customer_full_name,
		count(o.order_id) as total_orders,
		sum(case when o.order_status = 'Returned' then 1 else 0 end) as total_returned
from customers as cx
join
orders as o
on cx.customer_id = o.customer_id
JOIN
order_items as oi
on oi.order_id = o.order_id
group by 1,2
)
```

### 16. Top 5 Customers by Orders in Each State Identify the top 5 customers with the highest number of orders for each state. Challenge: Include the number of orders and total sales for each customer.

```sql
select * from
(
SELECT
		cx.state,
		cx.customer_id,
		concat(cx.first_name, ' ', cx.last_name) as customer_full_name,
		count(o.*) as total_orders,
		sum(total_sales) as total_sale,
		dense_rank() over(partition by cx.state order by count(o.*)desc) as rank
from customers as cx
JOIN
orders as o
on cx.customer_id = o.customer_id
JOIN
order_items as oi
on oi.order_id = o.order_id
group by 1,2
) as t1
where rank <= 5
```

### 17. Revenue by Shipping Provider Calculate the total revenue handled by each shipping provider. Challenge: Include the total number of orders handled and the average delivery time for each provider.

```sql
select
		s.shipping_id,
		s.shipping_providers,
		sum(oi.total_sales) as total_sale,
		count(o.*) as order_handels,
		coalesce(avg(s.return_date - s.shipping_date),0) avg_days
from orders as o
join
order_items as oi
on oi.order_id = o.order_id
join
shippings as s
on s.order_id = oi.order_id
group by 1
```

### 18. Top 10 product with highest decreasing revenue ratio compare to last year(2022) and current_year(2023). Challenge: Return product_id, product_name, category_name, 2022 revenue and 2023 revenue decrease ratio at end Round the result, Note: Decrease ratio = cr-ls/ls* 100 (cs = current_year ls=last_year)

```sql
with last_year_sale
as
(
SELECT
		p.product_id,
		p.product_name,
		sum(oi.total_sales) as total_revenue
from orders as o
join
order_items as oi
on oi.order_id = o.order_id
JOIN
products as p
on p.product_id = oi.product_id
where extract(year from o.order_date) = 2022
group by 1,2
),

current_year_sale
as
(
SELECT
		p.product_id,
		p.product_name,
		sum(oi.total_sales) as total_revenue
from orders as o
join
order_items as oi
on oi.order_id = o.order_id
JOIN
products as p
on p.product_id = oi.product_id
where extract(year from o.order_date) = 2023
group by 1,2
)

select
		 ls.product_id,
	 	ls.total_revenue as last_year_revenue,
	 	cs.total_revenue as current_year_revenue,
	 	ls.total_revenue - cs.total_revenue as rev_diff,
		round((cs.total_revenue - ls.total_revenue)::numeric/ls.total_revenue::numeric * 100,2) as revenue_dec_retio
from last_year_sale as ls
join
current_year_sale as cs
on ls.product_id = cs.product_id
where ls.total_revenue > cs.total_revenue
--group by 1
order by 5 desc
limit 10
```

### 19. Store procedure create a new store procedure that, when a product is sold, performs the following actions: Inserts a new sales records into the orders and order_items table. Update the inventory table to reduce the stock based on the product and quantity purchased. The procedure should ensure that the stock is adjusted immediately after recording the sales.

```sql
create or replace procedure add_sales
(
p_order_id int,
p_customer_id int,
p_seller_id int,
p_order_item_id int,
p_product_id int,
p_quantity int
)
language plpgsql
as $$

declare
-- all variable
v_count int;
v_price float;
v_product varchar(50);
begin
-- fetching product name and price based on product_id entered
	select 
		price, product_name
		INTO
		v_price, v_product
	from products
	where product_id = p_product_id;

-- checking stock and product availablity in inventory	
	SELECT
		count(*)
		INTO
		v_count
	from inventory
	WHERE
		product_id = p_product_id
		and
		stock >= p_quantity;
		
	if v_count > 0 then
		--add into orders and order_items table
		--update inventory
		insert into orders(order_id, order_date, customer_id, seller_id)
		VALUES
		(p_order_id, current_date, p_customer_id, p_seller_id);

		-- adding into order list
		
		insert into order_items(order_item_id, order_id, product_id, quantity, price_per_unit, total_sales)
		VALUES
		(p_order_item_id, p_order_id, p_product_id, p_quantity, v_price, v_price*p_quantity);

		--updating inventory
		update inventory
		set stock = stock - p_quantity
		where product_id = p_product_id;

		raise notice 'thank you product: % sale has been added also inventory stock updates', v_product;
	else
		raise notice 'thank you for your info the product: % is not available', v_product;
		
	end if;
	
end;
$$
```

#### Testing Store Procedure call add-sales (25009,2,9,25006,1,15);

---------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------

## Learning Outcomes

This project enabled me to:
	* Design and implement a normalized database schema.
	* Clean and preprocess real-world datasets for analysis.
	* Use advanced SQL techniques, including window functions, subqueries, and joins.
	* Conduct in-depth business analysis using SQL.
	* Optimize query performance and handle large datasets efficiently.

## Conclusion

This advanced SQL project successfully demonstrates my ability to solved real-world e-commerce problems using structured queries. From improving customer retention to optimizing inventory and logistics, the project provides valuable insights into operational challenges and solutions.

By completing this project, i have gained a deeper understanding of how SQL can be used to tackle complex data problems and drive business decision-making.

## Contact Me

📄 **[Resume](#)**  
📧 **[Email](jtusarkant@gmail.com)**  
📞 **Phone**: +91-7684-028977  
	



















```
