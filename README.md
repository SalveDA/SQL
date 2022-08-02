<h1 align="center">Solving business problems with SQL</a>

####

<h2 align="center">Analysis of the activities of the company “Pens and Pencils”</a>  

#### GOAL:
Analyze the work of the company in terms of its efficiency and give recommendations for scaling the business, namely, in which state it is better to open an offline store.

**TASKS:**
1.	Evaluate the dynamics of sales and the distribution of revenue by product.  
2.	Make a portrait of the client, and for this - find out which clients bring in the most revenue.
3.	Control the logistics of the company (determine whether all orders are delivered on time and in which state it is better to open an offline store).

To analyze the company's activities, the following data is presented:  
-	customer directory;
-	product directory;
-	a table with a list of products in the order, their quantities, as well as a discount for each product;
-	a table with information about orders and their delivery.

Orders according to the provided data were carried out in the period from 01/03/2017 to 12/30/2020. During this period, 5009 orders were completed for a total amount of $1,446,157 (including discounts on certain items).  

**1.	Dynamics of sales and distribution of revenue by goods**  
1.1 Let's analyze the dynamics of the company's revenue distribution by months, for this we will display the necessary data (date and volume of revenue) and build a graph for better visual perception:  

***Monthly income:***  
*select  
    date_trunc ('month', sd.order_date)::date as date,  
    round (sum ((sp.price - sp.price * sc.discount) * sc.quantity)) as revenue  
from sql.store_carts as sc  
    join sql.store_products as sp on sc.product_id = sp.product_id  
    join sql.store_delivery as sd on sd.order_id = sc.order_id  
group by date  
order by date*  

Query result:  

![revenue](https://github.com/SalveDA/SQL/blob/main/revenue%20dynamics.png)

Based on the data obtained, it can be seen that the company's income as a whole is growing, but there is an increase and decrease in revenue depending on the season.  

1.2 Let's determine which products are in the greatest demand, for this we will rank the sales of various products in various categories and subcategories:  

***The amount of revenue for various categories and subcategories:***  
*select<br/>
    sp.category,<br/>
    sp.subcategory,<br/>
    round(sum ((sp.price - sp.price * sc.discount) * sc.quantity)) as revenue<br/>
from sql.store_carts as sc<br/>
    join sql.store_products as sp on sc.product_id = sp.product_id<br/>
group by category, subcategory<br/>
order by revenue desc*<br/>

Query result:  

![product_category](https://github.com/SalveDA/SQL/blob/main/product_category.png)<br/>

From the table we see that the top three sales leaders include armchairs, telephones and storage space for office supplies.  

1.3 Consider which of the products bring the most profit and what share of the total revenue is their sales. To do this, we will rank the top 25 products by revenue, and also calculate the number of goods sold and their share of total revenue as a percentage:  

***Top 25 products by revenue:***  
*select<br/>
    sp.product_nm,<br/>
    round ((sum (quantity * price * (1 - discount))), 2) as revenue,<br/>
    sum (sc.quantity) as quantity,<br/>
    round ((sum (quantity * price * (1 - discount)) / (select
            sum (quantity * price * (1 - discount))<br/>
            from sql.store_products as p<br/>
            join sql.store_carts as c on p.product_id = c.product_id) * 100), 2) as percent_from_total<br/>
from sql.store_carts as sc<br/>
    join sql.store_products as sp on sc.product_id = sp.product_id<br/>
group by product_nm<br/>
order by revenue desc<br/>
limit 25*<br/>

Query result:  

![name_of_product](https://github.com/SalveDA/SQL/blob/main/name_of_product.png)

**2.	Client portrait**  
2.1 Let's determine how many B2B (Corporate) and B2C (Consumer) customers the company has and what share of the total revenue they bring.  
To do this, we calculate the number of customers and revenue by customer category.  

***Number of clients and revenue by client category:***  
*select<br/>
    scu.category,<br/>
    count (distinct scu.cust_id) as cust_cnt,<br/>
    round(sum((sp.price - sp.price * sc.discount) * sc.quantity)) as revenue<br/>
from sql.store_delivery as sd<br/>
    join sql.store_customers as scu on sd.cust_id = scu.cust_id<br/>
    join sql.store_carts as sc on sd.order_id = sc.order_id<br/>
    join sql.store_products as sp on sp.product_id = sc.product_id<br/>
group by scu.category<br/>
order by revenue desc*<br/>

Query result:  

![number_of_clients](https://github.com/SalveDA/SQL/blob/main/number_of_clients.png)

According to the data obtained, it can be seen that there are more corporate clients and the revenue from them is higher than from retail buyers.  

2.2 Let's analyze the dynamics of new B2B clients by months. At the same time, we will find out whether revenue is growing due to an increase in sales for old customers or by attracting new ones.  

***Number of new corporate clients by month:***  
*select<br/>
    date_trunc ('month', date)::date as month,<br/>
    count (cust_id) as new_custs<br/>
from (<br/>
select<br/>
    min (sd.order_date) as date,<br/>
    scu.cust_id<br/>
from sql.store_delivery as sd<br/>
join sql.store_customers as scu on sd.cust_id = scu.cust_id<br/>
where scu.category = 'Corporate'<br/>
group by scu.cust_id) as cust_date<br/>
group by month<br/>
order by month*<br/>

Query result:

![new_corporate](https://github.com/SalveDA/SQL/blob/main/new%20corporate.png)

From the diagram, it can be seen that the number of new customers is decreasing, which means that the company's revenue is growing at the expense of old customers.  

2.3 Let's study the main indicators for corporate clients, for this we calculate what is the average number of different goods in orders, the average amount of orders, the average number of different offices for corporate clients:  
***-- the average number of different products in orders and the average amount of orders from corporate customers:***  
*with quantity_and_sum_order as (<br/>
select<br/>
    distinct  scu.cust_nm as cust_nm,<br/>
    count (distinct sc.order_id) as order_id,<br/>
    count (distinct sp.product_nm) as qnt,<br/>
    sum ((sp.price - sp.price * sc.discount) * sc.quantity) as revenue<br/>
from sql.store_delivery as sd<br/>
    join sql.store_customers as scu on sd.cust_id = scu.cust_id<br/>
    join sql.store_carts as sc on sd.order_id = sc.order_id<br/>
    join sql.store_products as sp on sp.product_id = sc.product_id<br/>
where scu.category = 'Corporate'<br/>
group by cust_nm<br/>
order by cust_nm),*<br/>

***-- average number of different offices for corporate clients:***  
qnt_office as<br/>
(select<br/>
    distinct  scu.cust_nm as cust_nm,<br/>
   count (distinct sd.zip_code) as cnt_office<br/>
from sql.store_delivery as sd<br/>
    join sql.store_customers as scu on sd.cust_id = scu.cust_id<br/>
where scu.category = 'Corporate'<br/>
group by cust_nm<br/>
order by cust_nm)<br/>

select<br/>
    round (sum (qnt) / sum (order_id), 1) as avg_qnt_product,<br/>
    round (sum (revenue) / sum (order_id), 1) as avg_sum_order,<br/>
    round (avg (cnt_office), 1) as avg_cnt_office<br/>
from quantity_and_sum_order as qs<br/>
join qnt_office as qo on qs.cust_nm = qo.cust_nm<br/>  

Query result:  

![average_number](https://github.com/SalveDA/SQL/blob/main/average_number.png)  

Based on the data obtained, we can conclude that corporate clients are the most valuable for the company, because they account for a large share of revenue, but the number of new customers is steadily falling, and since the number and amount of goods in an order is on average very low, then, accordingly, and revenue will fall. In this case, it is necessary to take actions to attract new corporate clients and increase their average bill.  

**3.	Company logistics**  
3.1 In order to assess the current picture of delivery logistics, it is necessary to determine how efficiently orders are fulfilled and how deliveries and revenues are distributed by states and cities. To do this, we determine what proportion of orders are completed on time.  

***Type of delivery, total number of orders, number of orders not delivered on time and share of orders completed on time:***  
*with delivery_time as<br/>
(select<br/>
ship_mode,<br/>
    count (order_id)::numeric as orders_cnt,<br/>
    count (case when ship_mode = 'Standard Class' and (ship_date - order_date) > 6 then order_id<br/>
        when ship_mode = 'Second Class' and (ship_date - order_date) > 4 then order_id<br/>
        when ship_mode = 'First Class' and (ship_date - order_date) > 3 then order_id<br/>
        when ship_mode = 'Same Day' and (ship_date - order_date) > 0 then order_id<br/>
        end)::numeric as late_orders_cnt<br/>
from sql.store_delivery<br/>
group by ship_mode)<br/>

select<br/>
    ship_mode,<br/>
    orders_cnt,<br/>
    late_orders_cnt,<br/>
    ROUND ((orders_cnt - late_orders_cnt)*100 / orders_cnt, 2) AS "% success"<br/>
from delivery_time<br/>
order by "% success"*<br/>

Query result:  

![Delivery_type](https://github.com/SalveDA/SQL/blob/main/Delivery_type.png)  

Based on the results of the data obtained, we can conclude that orders sent by the second class (Second Class) most often arrive late.  

3.2 It is necessary to find out how systematically this happens. Perhaps the delivery service only had problems for a limited period. To do this, we calculate ***the proportion of orders sent by the second class, which were delivered late, by quarter:***  
with delivery_time_sc as<br/>
(select<br/>
	ship_mode,<br/>
	date_trunc ('quarter', order_date)::date as quarter,<br/>
    count (order_id)::numeric as orders_cnt,<br/>
    count (case when (ship_date - order_date) > 4 then order_id<br/>
                end)::numeric as late_orders_cnt<br/>
from sql.store_delivery<br/>
group by ship_mode, quarter)<br/>

select<br/>
    quarter,<br/>
    ROUND ((late_orders_cnt)*100 / orders_cnt, 2) AS "% unsuccess"<br/>
from delivery_time_sc<br/>
where ship_mode = 'Second Class'<br/>
order by quarter<br/>  

Query result:  

![late_orders](https://github.com/SalveDA/SQL/blob/main/late%20orders.png)  

The graph shows that orders sent by “Second Class” are systematically delivered late.  

3.3 Let's analyze to which state and city the most deliveries are made.  

***Calculate the number of deliveries by state and visualize the result using a map:***  
*select<br/>
    state,<br/>
    count(distinct order_id)<br/>
from sql.store_delivery<br/>
group by 1<br/>
order by 2 desc*<br/>

Query result:  











