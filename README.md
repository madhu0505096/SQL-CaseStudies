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

