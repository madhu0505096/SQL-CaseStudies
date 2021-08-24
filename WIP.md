# Case Study #2 - Pizza Runner  

![image](https://user-images.githubusercontent.com/78327987/130497647-affc6707-99a1-455f-b672-8a9e58d8ecf6.png)

## Introduction  
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.  

## Available data
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

All datasets exist within the pizza_runner database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.  

![image](https://user-images.githubusercontent.com/78327987/130498155-b7c37d11-9a69-4cef-951b-7edef229290f.png)

![image](https://user-images.githubusercontent.com/78327987/130498200-101bcd23-78e8-4327-a255-b1164410e946.png)

![image](https://user-images.githubusercontent.com/78327987/130498311-39eb12ac-d236-4379-8c1b-b97258f74a4b.png)

![image](https://user-images.githubusercontent.com/78327987/130498376-7cbf812c-4f1c-4d5e-83bc-3495d865d292.png)

![image](https://user-images.githubusercontent.com/78327987/130498420-a88e2cf6-705a-4baa-b972-73f74881e3bf.png)

![image](https://user-images.githubusercontent.com/78327987/130498485-8aaf77eb-8ced-49d9-a8e8-8633d463756c.png)

![image](https://user-images.githubusercontent.com/78327987/130498545-8403be2a-d79d-4b45-a5ed-1b328fe51318.png)

![image](https://user-images.githubusercontent.com/78327987/130498571-d39c30ff-5fdc-408b-8260-febd3a2ae266.png)

## Case Study Questions  
![image](https://user-images.githubusercontent.com/78327987/130499146-fa030cd5-f513-4c3c-a3cf-8446dd253905.png)

This case study has LOTS of questions - they are broken up by area of focus including:

1.Pizza Metrics  
2.Runner and Customer Experience  
3.Ingredient Optimisation  
4.Pricing and Ratings  
5.Bonus DML Challenges (DML = Data Manipulation Language)  
6.Each of the following case study questions can be answered using a single SQL statement. 

Again, there are many questions in this case study - please feel free to pick and choose which ones you’d like to try!

Before you start writing your SQL queries however - you might want to investigate the data, you may want to do something with some of those null values and data types in the customer_orders and runner_orders tables!  
___  
## A.Piza Metrics  

## 1.How many pizzas were ordered?  

To find out this we first need to know which table has this data.  

By reading the description and looking at the data it is evident that the customer table has all the pizza which was ordered irrespective of whether the customer has cancelled it or not.  

From the data we can see that if the same customer orders pizza at the same time, the  order number remains the same.  

So, if we find out the number of records in this table that will equal the number of Pizza ordered so far.  


```

    SELECT COUNT(*) AS TOTAL_PIZZA_ORDERED FROM 
    PIZZA_RUNNER.CUSTOMER_ORDERS;
```

| total_pizza_ordered |
| ------------------- |
| 14                  |

**Answer**  
A total of 5  pizzas were ordered so far.  

## 2.How many unique customer orders were made?  
I am assuming that the question asks how many unique customers ordered pizza so far.  
To find this we just need the unique customer ids.  
![image](https://user-images.githubusercontent.com/78327987/130566589-41fb6d09-5ac1-4ce0-bd75-66cc2a2aa20d.png)  

```
SELECT COUNT(DISTINCT CUSTOMER_ID) AS UNIQUE_CUSTOMER_ORDERS FROM 
PIZZA_RUNNER.CUSTOMER_ORDERS;
```
| UNIQUE_CUSTOMER_ORDERS  |
| ------------------------|
| 5                       |


**Answer**
The  number of unique customer orders is 5  


## 4.How many of each type of pizza was delivered?  

Join the `customer_orders` and `runner_orders` on `order_id` to get the data of `PIZZA_ID` and `CANCELLATION` in one place.

Then use Where clause to filter out cancelled orders.  

Then use Count along with group by to find how may pizza's were delivered in each type

```
SELECT
PIZZA_ID
,COUNT(PIZZA_ID)
FROM 
PIZZA_RUNNER.CUSTOMER_ORDERS
INNER JOIN 
PIZZA_RUNNER.RUNNER_ORDERS
ON CUSTOMER_ORDERS.ORDER_ID = RUNNER_ORDERS.ORDER_ID
WHERE CANCELLATION NOT IN ('RESTAURANT CANCELLATION','CUSTOMER CANCELLATION')
GROUP BY PIZZA_ID;
```

| pizza_id | count |
| -------- | ----- |
| 2        | 1     |
| 1        | 5     |

**Answer**
1. Pizza_id 1 was delivered 5 times  
2. Pizza_id 2 was delivered 1 time

## 5.How many Vegetarian and Meatlovers were ordered by each customer?  

The vegetarian and meatlovers data is aviailable in the `pizza_names` table, so we join `pizza_names` with the existing SQL from previous question.  

Since we want all the orders we should remove the where clause.  

Then for each group of pizza_names we can find the count.  
```
    SELECT
    pizza_names.pizza_name
    ,count(CUSTOMER_ORDERS.pizza_id)
    FROM 
    PIZZA_RUNNER.CUSTOMER_ORDERS
    INNER JOIN 
    PIZZA_RUNNER.runner_orders
    ON CUSTOMER_ORDERS.ORDER_ID = runner_orders.order_id
    inner join 
    PIZZA_RUNNER.pizza_names
    on
    pizza_names.pizza_id = CUSTOMER_ORDERS.pizza_id
    
    group by pizza_names.pizza_name;
```
| pizza_name | count |
| ---------- | ----- |
| Meatlovers | 10    |
| Vegetarian | 4     |

## 6.What was the maximum number of pizzas delivered in a single order?

A single order correpsonds to an order placed at the same time.If I have ordered 2 pizzas at time A and if it's delivered then the count is 2.

From the date we can see that all the single orders have duplicate values.  
![image](https://user-images.githubusercontent.com/78327987/130576684-8166bfe0-a27d-4ed2-8463-fcb1347925c0.png)

So we can get the desired result by removing all cancelled orders and counting the number of duplicate order_ids.  


```
with cte as(
    select
      CUSTOMER_ORDERS.ORDER_ID
    ,count(CUSTOMER_ORDERS.ORDER_ID) as cnt
    FROM 
    PIZZA_RUNNER.CUSTOMER_ORDERS
    INNER JOIN 
    PIZZA_RUNNER.runner_orders
    ON CUSTOMER_ORDERS.ORDER_ID = runner_orders.order_id
    inner join 
    PIZZA_RUNNER.pizza_names
    on
    pizza_names.pizza_id = CUSTOMER_ORDERS.pizza_id
    WHERE (cancellation NOT IN ('Restaurant Cancellation','Customer Cancellation') or cancellation is NULL)
    group by CUSTOMER_ORDERS.ORDER_ID
    )
    select order_id,cnt from cte where cnt in (
      select max(cnt) from cte);
```


| order_id | cnt |
| -------- | --- |
| 4        | 3   |


**Answer**  
In a single order the maximum number of pizzas delivered is 3 and the corresponfing order_id is 4  


## 7.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
Firstly, to get only the delivered pizzas we can use the where filter on `cancellation` column.  

Then to know whether the pizza had a change or not we should look into `exclusions` and `extras` column. 

## 8.How many pizzas were delivered that had both exclusions and extras?

## 9.What was the total volume of pizzas ordered for each hour of the day?

## 10.What was the volume of orders for each day of the week?
