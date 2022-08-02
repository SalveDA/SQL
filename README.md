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















![page_5_6](https://github.com/SalveDA/SQL/blob/main/page_5_6.png)

<br/>

![page_7_8](https://github.com/SalveDA/SQL/blob/main/page_7_8.png)

<br/>

![page_9_10](https://github.com/SalveDA/SQL/blob/main/page_9_10.png)
