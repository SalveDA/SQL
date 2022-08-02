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






![page_1_2](https://github.com/SalveDA/SQL/blob/main/page_1_2.png)

<br/>

![page_3_4](https://github.com/SalveDA/SQL/blob/main/page_3_4.png)

<br/>

![page_5_6](https://github.com/SalveDA/SQL/blob/main/page_5_6.png)

<br/>

![page_7_8](https://github.com/SalveDA/SQL/blob/main/page_7_8.png)

<br/>

![page_9_10](https://github.com/SalveDA/SQL/blob/main/page_9_10.png)
