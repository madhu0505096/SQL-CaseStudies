# SQL-CaseStudies

## Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.  

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.  

## Problem Statement  
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:  

1.sales  
2.menu  
3.members  

![image](https://user-images.githubusercontent.com/78327987/129861800-7a993e1e-b198-49a3-a813-9e6a78a4e455.png)

___

__Example Datasets__

![image](https://user-images.githubusercontent.com/78327987/129862101-f424b93b-f524-413f-8efe-6a6bee8377e4.png)

![image](https://user-images.githubusercontent.com/78327987/129862169-1571a57e-12c3-4457-9a74-bad076f6639e.png)

___

__DDL Statements to create the data needed for analysis __

```
Schema SQL Query SQL ResultsEdit on DB Fiddle
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```
We can execute the SQL using dbfiddle["https://www.db-fiddle.com/f/mcnNXgFYpuDXB5fFQGhogs/1"]  

__

__Question 1: What is the total amount each customer spent at the restaurant?__


```
select 

sales.customer_id
,sum(menu.price)

from 

dannys_diner.sales

join

dannys_diner.menu

on  sales.product_id  = menu.product_id

group by sales.customer_id;

```
__Question 2. How many days has each customer visited the restaurant?__

Group the data by customer_id and use count aggregate function on customer_id to get the number of days a customer has visited the restaurant.
```
    select 
    
    sales.customer_id
    ,count(sales.order_date)
    
    from 
    
    dannys_diner.sales
    
    
    group by sales.customer_id;

| customer_id | count |
| ----------- | ----- |
| B           | 6     |
| C           | 3     |
| A           | 6     |

```
But wait what if the customer has visited the restaurant more than once in a day so, let's check whether there are any records as such.

```
    select 
    
    sales.customer_id
    ,sales.order_date
    ,count(*)
    from     
    dannys_diner.sales  
    
    group by sales.customer_id,sales.order_date;

| customer_id | order_date               | count |
| ----------- | ------------------------ | ----- |
| C           | 2021-01-07T00:00:00.000Z | 1     |
| B           | 2021-01-01T00:00:00.000Z | 1     |
| B           | 2021-02-01T00:00:00.000Z | 1     |
| A           | 2021-01-01T00:00:00.000Z | 2     |
| B           | 2021-01-16T00:00:00.000Z | 1     |
| A           | 2021-01-10T00:00:00.000Z | 1     |
| A           | 2021-01-07T00:00:00.000Z | 1     |
| B           | 2021-01-02T00:00:00.000Z | 1     |
| A           | 2021-01-11T00:00:00.000Z | 2     |
| B           | 2021-01-04T00:00:00.000Z | 1     |
| C           | 2021-01-01T00:00:00.000Z | 2     |
| B           | 2021-01-11T00:00:00.000Z | 1     |

```
As you can see Customers A nd C has visited the restaurant twice in the same date so this should not be counted.

To get the desired result, we can modify our first query by including just a distinct while counting the sales.order_date 

```
    select 
        
        sales.customer_id
        ,count(distinct sales.order_date)
        
        from 
        
        dannys_diner.sales
        
        
        group by sales.customer_id;

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

```

__Question 3. What was the first item from the menu purchased by each customer?__

The first item from the menu which was purchased by the customers can be found out by looking at the first order_date for each customer.
As we know some customers came more than once on the same day we should make sure our query covers this scenario.

```

--Create a temprory table or CTE to hold the first ordered date result

    with temp as (
        select
        sales.customer_id,sales.order_date,menu.product_name
        ,rank()over(partition by sales.customer_id order by sales.order_date ) as order
       
       from 
        
        dannys_diner.sales
        
        join
        
        dannys_diner.menu
        
        on sales.product_id = menu.product_id
        
       )
       
       --Query the result from the temp table with desired filter
       
       select  * from temp where temp.order =1;

| customer_id | order_date               | product_name | order |
| ----------- | ------------------------ | ------------ | ----- |
| A           | 2021-01-01T00:00:00.000Z | curry        | 1     |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 1     |
| B           | 2021-01-01T00:00:00.000Z | curry        | 1     |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 1     |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 1     |

```

__Question 4:What is the most purchased item on the menu and how many times was it purchased by all customers?__

Join the sales and menu table using a inner join and use count aggregate function.

```


    select
        menu.product_name
        ,count(*)
       
       
       from 
        
        dannys_diner.sales
        
        join
        
        dannys_diner.menu
        
        on sales.product_id = menu.product_id
    
     	group by menu.product_name;

| product_name | count |
| ------------ | ----- |
| ramen        | 8     |
| sushi        | 3     |
| curry        | 4     |

```


Ramen is the most frequent and it was purchased 8 times by all customers combined.

__Question5:Which item was the most popular for each customer?__

The most popular item is the one which was bought by the customer the highest number of times.

---



    select 
         	final.product_name
           ,final.customer_id
           ,final.NumberOfOrders
     		from
     
     		(
        
      	    select 
      	    a.product_name
           ,a.customer_id
           ,a.count as NumberOfOrders
           ,rank()over(partition by a.customer_id order by a.count desc) as freq
                                    
           from 
           
           (  
          
           select 
           menu.product_name
           ,sales.customer_id
           ,count(*) as count
                  
            from 
            
            dannys_diner.sales
            
            inner join
            
            dannys_diner.menu
            
            on sales.product_id = menu.product_id
            
            group by menu.product_name,sales.customer_id 
            
            order by customer_id asc,count desc
         
            
            )a
           
           )final where final.freq=1;

| product_name | customer_id | numberoforders |
| ------------ | ----------- | -------------- |
| ramen        | A           | 3              |
| curry        | B           | 2              |
| sushi        | B           | 2              |
| ramen        | B           | 2              |
| ramen        | C           | 3              |

---

__Question6: Which item was purchased first by the customer after they became a member?__  

```
    SELECT 
    cust_id as customer_id
    ,temp.prod_name as product_name
    
    FROM 
    (
    SELECT
    *,
    rank()over(partition by sales.customer_id order by order_date)
    ,sales.customer_id as cust_id
    ,menu.product_name as prod_name
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
    inner join 
    dannys_diner.members
    on sales.customer_id = members.customer_id
    where join_date<=order_date
    order by sales.customer_id,order_date
    )TEMP
    WHERE RANK = 1;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |


1.The customer C hasn't purchased anything after his/her membership  
2. Customer A bought 'Ramen' as the first dish after the membership  
3. Customer B bought 'Sushi' as the first dish after the membership  

__Question7: Which item was purchased just before the customer became a member?__

```

    SELECT 
    cust_id as customer_id
    ,temp.prod_name as product_name
    ,order_date
    ,join_date as membership_date
    FROM 
    (
    SELECT
    order_date
    ,join_date 
    ,rank()over(partition by sales.customer_id order by order_date)
    ,sales.customer_id as cust_id
    ,menu.product_name as prod_name
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
    inner join 
    dannys_diner.members
    on sales.customer_id = members.customer_id
    where join_date>order_date
    order by sales.customer_id,order_date
    )TEMP
    WHERE RANK = 1;
```
| customer_id | product_name | order_date               | membership_date          |
| ----------- | ------------ | ------------------------ | ------------------------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| A           | curry        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| B           | curry        | 2021-01-01T00:00:00.000Z | 2021-01-09T00:00:00.000Z |


Just before the customers became a member,

Customer A --> Ordered Sushi and Curry on the same day

Customer B --> Ordered Curry on the 

Customer C --> Doesn't have a membership yet

__Question8: What is the total items and amount spent for each member before they became a member?__

```

    select 
    sales.customer_id as cust_id
    ,count(product_name) as number_Of_Items
    ,sum(price) as Total_Amount_Spent_B4_Membership
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
    inner join 
    dannys_diner.members
    on sales.customer_id = members.customer_id
    where join_date>order_date
    group by cust_id;
 ```
 
| cust_id | number_of_items | total_amount_spent_b4_membership |
| ------- | --------------- | -------------------------------- |
| B       | 3               | 40                               |
| A       | 2               | 25                               |


Before becoming a member, customer A has spent $25 on 2 times and customer B has spent $40 on 3 items.

__Question 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?__
  
Every one dollar spent equals 10 points so we need to multiply price of every product(other than sushi) by 10   
Similarly, since sushi has a 2x points multiplier on points we multiply price of sushi) by 20  

```
    select 
    sales.customer_id as cust_id
    ,sum(case when product_name = 'sushi' then 20*price
    else 10*price end)
    as points
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
   group by cust_id;
```

| cust_id | points |
| ------- | ------ |
| B       | 940    |
| C       | 360    |
| A       | 860    |

We can infer the points for each customer from above table.  

__Question 10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?__

```
    select 
        
        sales.customer_id as cust_id
        ,sum(
        case when (product_name = 'sushi' and order_date<join_date) then 20*price
        	 when (product_name not in ('sushi') and order_date<join_date) then 10*price
          	 when  (order_date>= join_date and order_date< (join_date+6) ) then 20*price
          	 when ( (order_date>(join_date+7)) and (order_date<'2021-01-31') ) then 1
             end) as points
        FROM 
        dannys_diner.menu
        inner join 
        dannys_diner.sales
        on sales.product_id = menu.product_id
        inner join 
        dannys_diner.members
        on sales.customer_id = members.customer_id
        group by cust_id;

```
| cust_id | points |
| ------- | ------ |
| A       | 1370   |
| B       | 700    |


____

Bonus Question  
TO get the desired answer as shown we need to follow the below steps,  
1.We need all the three tables, so join all three but while joining the member table use a left join or else customer C records would be dropped  
2.Use a case expression to create the member column  
3.Order the order_date in ascending and price in descending  
4.Inorder to remove the extra characters in the order_Date field use substr function to fetch only the ten characters, also remember to cast the order date as a string before applying substr function as substr only works on strings.

        SELECT
        sales.customer_id as customer_id
        ,substr(cast(order_date as varchar),1,10) as order_date
        ,menu.product_name as product_name
        ,price    
        ,case when order_date<join_date then 'N'
        when order_date>=join_date then 'Y'
        when join_date is null then 'N'
        end as member     
        FROM 
        dannys_diner.menu
        inner join 
        dannys_diner.sales
        on sales.product_id = menu.product_id
        left join 
        dannys_diner.members
        on sales.customer_id = members.customer_id   
        order by sales.customer_id,order_date,price desc;

```
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | Y      |
| C           | 2021-01-07 | ramen        | 12    | Y      |




___
**Rank All The Things**

To get rank for only the members and null vaue for non-members we create two seperate queries and union them.


**Non-Members**
```
	SELECT 
	customer_id
	,order_date
	,product_name
	,price
	,member
	,cast(NULL as int) as ranking

	from (
	
     SELECT
    sales.customer_id as customer_id
    ,substr(cast(order_date as varchar),1,10) as order_date
    ,menu.product_name as product_name
    ,price    
   ,case when order_date<join_date then 'N'
    when order_date>=join_date then 'Y'
    when join_date is null then 'N'
    end as member    
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
    left join 
    dannys_diner.members
    on sales.customer_id = members.customer_id   
	)mem
    where mem.member = 'N'
 ```
 
 **Members**
 ```
 SELECT 
	customer_id
	,order_date
	,product_name
	,price
	,member
	,rank()over(partition by mem.customer_id order by order_date,mem.price desc) as ranking

	from (
	
     SELECT
    sales.customer_id as customer_id
    ,substr(cast(order_date as varchar),1,10) as order_date
    ,menu.product_name as product_name
    ,price    
	 ,case when order_date<join_date then 'N'
    when order_date>=join_date then 'Y'
    when join_date is null then 'N'
    end as member        
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
    inner join 
    dannys_diner.members
    on sales.customer_id = members.customer_id   
	)mem
    where mem.member = 'Y'
 ```
 Now we combine both of the queries to get the desired result.
 
 ```
 
select * from (

select * from (	
--Only memebers
	
	
	SELECT 
	customer_id
	,order_date
	,product_name
	,price
	,member
	,rank()over(partition by mem.customer_id order by order_date,mem.price desc) as ranking

	from (
	
     SELECT
    sales.customer_id as customer_id
    ,substr(cast(order_date as varchar),1,10) as order_date
    ,menu.product_name as product_name
    ,price    
	 ,case when order_date<join_date then 'N'
    when order_date>=join_date then 'Y'
    when join_date is null then 'N'
    end as member        
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
    inner join 
    dannys_diner.members
    on sales.customer_id = members.customer_id   
	)mem
    where mem.member = 'Y'
	)memberonly
	
	union all 
--Non-members
select * from 

(

	SELECT 
	customer_id
	,order_date
	,product_name
	,price
	,member
	,cast(NULL as int) as ranking

	from (
	
     SELECT
    sales.customer_id as customer_id
    ,substr(cast(order_date as varchar),1,10) as order_date
    ,menu.product_name as product_name
    ,price    
   ,case when order_date<join_date then 'N'
    when order_date>=join_date then 'Y'
    when join_date is null then 'N'
    end as member    
    FROM 
    dannys_diner.menu
    inner join 
    dannys_diner.sales
    on sales.product_id = menu.product_id
    left join 
    dannys_diner.members
    on sales.customer_id = members.customer_id   
	)mem
    where mem.member = 'N'
	
)nonmember
)final
order by final.customer_id asc, order_date asc, price desc;
```
| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | curry        | 15    | N      |         |
| A           | 2021-01-01 | sushi        | 10    | N      |         |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |         |
| B           | 2021-01-02 | curry        | 15    | N      |         |
| B           | 2021-01-04 | sushi        | 10    | N      |         |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-07 | ramen        | 12    | N      |         |





