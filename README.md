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




![page_9_10](https://github.com/SalveDA/SQL/blob/main/page_9_10.png)
