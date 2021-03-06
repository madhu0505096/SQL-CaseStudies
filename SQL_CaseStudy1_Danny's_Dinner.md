# SQL-CaseStudy1 Danny's Dinner
![image](https://user-images.githubusercontent.com/78327987/130469461-8c7c876d-57e4-4875-bb24-161a38b4d952.png)

The below case study is given by [Danny ma](https://8weeksqlchallenge.com/case-study-1/) and there are a total of 8 case studies; here I go through the first case study Danny's Dinner.

Danny has a awesome course named Serious SQL, I highly recommend to check out his course [here.](https://www.datawithdanny.com/courses/serious-sql)

____

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
We can execute the SQL using [dbfiddle]("https://www.db-fiddle.com/f/mcnNXgFYpuDXB5fFQGhogs/1"] )

__

## __Question 1: What is the total amount each customer spent at the restaurant?__


```
select 

sales.customer_id
,sum(menu.price) as Total_Amount_Spent

from 

dannys_diner.sales

join

dannys_diner.menu

on  sales.product_id  = menu.product_id

group by sales.customer_id;

```

| customer_id | total_amount_spent |
| ----------- | ------------------ |
| B           | 74                 |
| C           | 36                 |
| A           | 76                 |

---
Customer A has spent $76  
Customer B has spent $74  
Customer C has spent $36  

## __Question 2: How many days has each customer visited the restaurant?__

Group the data by customer_id and use count aggregate function on customer_id to get the number of days a customer has visited the restaurant.
```
    select 
    
    sales.customer_id
    ,count(sales.order_date) as no_of_days_visited
    
    from 
    
    dannys_diner.sales
    
    
    group by sales.customer_id;
```

| customer_id | no_of_days_visited |
| ----------- | ----- |
| B           | 6     |
| C           | 3     |
| A           | 6     |


But wait what if the customer has visited the restaurant more than once in a day so, let's check whether there are any records as such.

```
    select 
    
    sales.customer_id
    ,sales.order_date
    ,count(*)
    from     
    dannys_diner.sales  
    
    group by sales.customer_id,sales.order_date;
```

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


As you can see Customers A nd C has visited the restaurant twice in the same date so this should not be counted.

To get the desired result, we can modify our first query by including just a distinct while counting the sales.order_date 

```
    select 
        
        sales.customer_id
        ,count(distinct sales.order_date) as no_of_days_visited
        
        from 
        
        dannys_diner.sales
        
        
        group by sales.customer_id;

```

| customer_id | no_of_days_visited |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

1.Customer A visited 4 times.  
2.Customer B visited 6 times.  
3.Customer C visited 2 times.  


## __Question3: What was the first item from the menu purchased by each customer?__

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
```

| customer_id | order_date               | product_name | order |
| ----------- | ------------------------ | ------------ | ----- |
| A           | 2021-01-01T00:00:00.000Z | curry        | 1     |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 1     |
| B           | 2021-01-01T00:00:00.000Z | curry        | 1     |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 1     |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 1     |

If you don't want to see all the columns, remove '*' from the select statement and specify the required columns to be displayed.   

## __Question4: What is the most purchased item on the menu and how many times was it purchased by all customers?__

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

```

| product_name | count |
| ------------ | ----- |
| ramen        | 8     |
| sushi        | 3     |
| curry        | 4     |




Ramen is the most frequent and it was purchased 8 times by all customers combined.

curry is the second most frequent and it was purchased 4 times by all customers combined. 

sushi is the third most frequent and it was purchased 3 times by all customers combined. 


## __Question5: Which item was the most popular for each customer?__

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

---

| product_name | customer_id | numberoforders |
| ------------ | ----------- | -------------- |
| ramen        | A           | 3              |
| curry        | B           | 2              |
| sushi        | B           | 2              |
| ramen        | B           | 2              |
| ramen        | C           | 3              |

Customer A and C’s favourite item is ramen.  
Customer B tried all items in the menu.    

## __Question6: Which item was purchased first by the customer after they became a member?__  

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


1.The customer C DOESN'T have a membership so excluded.
2. Customer A bought 'curry' as the first dish after the membership  
3. Customer B bought 'Sushi' as the first dish after the membership  

## __Question7: Which item was purchased just before the customer became a member?__

It's always important to understand the business question.  

So what are we trying to achieve here?

If a customer has purchased food on let's say day 1,2,3,7,10 and if he/she get's membership on day 7 then we need to find what the customer bought on Day 3.  


Use where clause  to filter only the dates before membership and order it by desending and pick the first one.  

Use rank() function which helps us to align the order date for each customer in the way we want and the where clause helps to filter only the dates.   

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
        ,rank()over(partition by sales.customer_id order by order_date desc)
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
| B           | sushi        | 2021-01-04T00:00:00.000Z | 2021-01-09T00:00:00.000Z |

Just before the customers became a member,

Customer A --> Ordered Sushi and Curry on the same day

Customer B --> Ordered sushi 

Customer C --> Doesn't have a membership, so C is excluded

## __Question8: What is the total items and amount spent for each member before they became a member?__

Similar to previous question the where clause helps to concentrate only on the data before they became a member.  

Then, use aggregate function along with group by to find the desired result.  

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

## __Question9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?__
  
Every one dollar spent equals 10 points so we need to multiply price of every product(other than sushi) by 10   
Similarly, since sushi has a 2x points multiplier on points we multiply price of sushi) by 20  

This can be done using Case when statement and then  to find total points for each customer we can use aggregate functions along with group by.  

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

## __Question10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?__

Similar to other questions we use a case when statement to construct the logic, and consider every possible scenarios.  
```
    select 
        
        sales.customer_id as cust_id
 ,sum(case when (product_name = 'sushi' and order_date<join_date) or 
         (product_name = 'sushi' and extract( month from order_date)=1 and order_date>(join_date+6)  ) 
          then 20*price
         when (product_name not in ('sushi') and order_date<join_date) or 
         (extract( month from order_date)=1 and order_date>(join_date+6)) then 		10*price
          when  (order_date>= join_date and order_date<= (join_date+6) ) then 20*price  end) as points
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
| B       | 820    |


___

## Bonus Question  

![image](https://user-images.githubusercontent.com/78327987/130464615-4e427be9-6e16-4fe2-87b2-511f0ffced62.png)

![image](https://user-images.githubusercontent.com/78327987/130464579-1d6944d4-39b4-40d3-aef2-2d20bf179f0f.png)

To get the desired answer as shown above we need to follow the below steps,  
1.We need all the three tables, so join all three but while joining the member table use a left join or else customer C records would be dropped  
2.Use a case expression to create the member column  
3.Order the order_date in ascending and price in descending  
4.Inorder to remove the extra characters in the order_Date field use substr function to fetch only the ten characters, also remember to cast the order date as a string before applying substr function as substr only works on strings.

```
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

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.  

We can solve this by many ways here I will be taking two methods one the easy simple method and the other lengthy one.  

1.Create a CTE with member/non-member details as above  
2.Use  case when along with the rank window function  

```
WITH SUMMARY_CTE  AS 
(
 SELECT SALES.CUSTOMER_ID, SALES.ORDER_DATE, MENU.PRODUCT_NAME, MENU.PRICE,
  CASE
  WHEN MEMBERS.JOIN_DATE > SALES.ORDER_DATE THEN 'N'
  WHEN MEMBERS.JOIN_DATE <= SALES.ORDER_DATE THEN 'Y'
  ELSE 'N' END AS MEMBER
 FROM DANNYS_DINER.SALES AS SALES
 LEFT JOIN DANNYS_DINER.MENU 
  ON SALES.PRODUCT_ID = MENU.PRODUCT_ID
 LEFT JOIN DANNYS_DINER.MEMBERS 
  ON SALES.CUSTOMER_ID = MEMBERS.CUSTOMER_ID
) SELECT * 
, CASE
 WHEN MEMBER = 'N' THEN NULL
 ELSE
  RANK () OVER(PARTITION BY CUSTOMER_ID, MEMBER
  ORDER BY ORDER_DATE) END AS RANKING
  FROM SUMMARY_CTE
```

 **Another way of solving**  

To get rank for only the members and null vaue for non-members we create two seperate queries and union them.  This query would do the job but it is would have poor performance.   

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
**Union**  
 
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


Null values are displayed as blank in the above table which can be seen while executed in fiddle.

![image](https://user-images.githubusercontent.com/78327987/130297439-83d1f6a2-22c6-4a7b-b159-8a87393e6973.png)

**Have any SQL doubts or need mentoring please feel free to contact me at madhuprasath556@gmail.com or [Linkedln](https://www.linkedin.com/in/madhu-prasath/)**
