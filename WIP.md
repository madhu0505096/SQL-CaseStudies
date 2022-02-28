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

## 3 How many successful orders were delivered by each runner?  

If an order is not cancelled by the restaurant or the customer it denotes a succesful order and this information is in runner_orders table.

The only problem is for few records null is hardcoded, so we need to include this scenario in our where filter.

```

    SELECT count(*) as Number_Sucessful_Orders
    FROM  pizza_runner.runner_orders 
    where cancellation is null or cancellation='null';
```

| number_sucessful_orders |
| ----------------------- |
| 6                       |



[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
## 4.How many of each type of pizza was delivered?  

Join the `customer_orders` and `runner_orders` on `order_id` to get the data of `PIZZA_ID` and `CANCELLATION` in one place.

Then use Where clause to filter out cancelled orders.  

Then use Count along with group by to find how many pizza's were delivered in each type

```
SELECT 
PIZZA_ID
,COUNT(PIZZA_ID)
FROM 
PIZZA_RUNNER.CUSTOMER_ORDERS
INNER JOIN 
PIZZA_RUNNER.RUNNER_ORDERS
ON CUSTOMER_ORDERS.ORDER_ID = RUNNER_ORDERS.ORDER_ID
WHERE CANCELLATION IS NULL OR CANCELLATION = 'null'  
GROUP BY PIZZA_ID
```

| pizza_id | count |
| -------- | ----- |
| 1        | 7     |
| 2        | 3     |

**Answer**
1. Pizza_id 1(Meat Lovers) was delivered 7 times  
2. Pizza_id 2(Vegetarian) was delivered 3 times

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
    WHERE (cancellation = 'null' or cancellation is NULL)
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

Use case when statement to logically decide whether the order has a change or not and enclose this in a CTE.

Then find the number of orders which had the change using count() aggregate function along with group by.  

```
    with cte as(
        select customer_id,
             case when (exclusions ='' or exclusions is null or exclusions ='null') 
          	and (extras ='' or extras ='null' or extras is null) then 'No Change' else 'Change' end as test       
            
            FROM 
            PIZZA_RUNNER.CUSTOMER_ORDERS
            INNER JOIN 
            PIZZA_RUNNER.runner_orders
            ON CUSTOMER_ORDERS.ORDER_ID = runner_orders.order_id
            inner join 
            PIZZA_RUNNER.pizza_names
            on
            pizza_names.pizza_id = CUSTOMER_ORDERS.pizza_id
            WHERE (cancellation NOT IN ('Restaurant Cancellation','Customer Cancellation') 	   or cancellation is NULL)
        ) 
        select
        customer_id
        ,test
        ,count(test)
        from cte
        group by test,customer_id;
```

| customer_id | test      | count |
| ----------- | --------- | ----- |
| 104         | Change    | 2     |
| 103         | Change    | 3     |
| 104         | No Change | 1     |
| 105         | Change    | 1     |
| 101         | No Change | 2     |
| 102         | No Change | 3     |



[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

## 8.How many pizzas were delivered that had both exclusions and extras?

As usual we would remove all the cancelled orders via where filter.  

Then to find the orders with both exclusions and extras, we should make sure there is any number as a value using regular expression.

After applying the regular expression, the CTE would have only the orders with numbers either in exclusions and extras.  

Then on top of the CTE we filter only the records which has number in both exclusions and extras.

Finally the count of the result set would give the desired answer.  
```
    with cte as(
    select
        
          regexp_matches(exclusions,'[0-9]') as exclusions
      	 ,regexp_matches(extras,'[0-9]') as extras
        
        
        FROM 
        PIZZA_RUNNER.CUSTOMER_ORDERS
        INNER JOIN 
        PIZZA_RUNNER.runner_orders
        ON CUSTOMER_ORDERS.ORDER_ID = runner_orders.order_id
        inner join 
        PIZZA_RUNNER.pizza_names
        on
        pizza_names.pizza_id = CUSTOMER_ORDERS.pizza_id
        WHERE (cancellation NOT IN ('Restaurant Cancellation','Customer Cancellation') 	   or cancellation is NULL )
        
    ) 
    select
    count(*) as Count_Of_Exclusions_And_Extras
    from cte
    where (exclusions is not null and extras is not null);
```
| count_of_exclusions_and_extras |
| ------------------------------ |
| 1                              |

**Answer**
There was only one order which had both exclusion and extras  

## 9.What was the total volume of pizzas ordered for each hour of the day?  

We can extract only the hour from the time stamp and check how many orders were placed for each individual hour.

Extract function will help us to sperate the hour from the timestamp and count() aggregate function coupled with group by would give us the desired result.
```
    SELECT extract(hour from order_time)as individualHour, count(*) as NumberOfPizzaOrdered
    from pizza_runner.customer_orders	
    group by individualHour;
```
| individualhour | numberofpizzaordered |
| -------------- | -------------------- |
| 18             | 3                    |
| 23             | 3                    |
| 21             | 3                    |
| 11             | 1                    |
| 19             | 1                    |
| 13             | 3                    |

From above table we can see that mostly 3 pizzas are sold each hour and a few times only one pizza is sold per hour.


## 10.What was the volume of orders for each day of the week?

To get each day of the week we can use `isdow` function which would return a number between 1 to 7.  

The below table shows the results from `isdow` and it's asscoicated day in the week.  
|from isdow| Corresponding Day|
|-----|------|
|1 | Monday|
|2 | Tuesday|
|3 | Wednesday|
|4 | Thursday|
|5 | Friday|
|6 | Saturday|
|7 | Sunday|


```
    SELECT 
    case when extract(isodow  from order_time) = 1 then 'Monday' 
    when extract(isodow  from order_time) = 2 then 'Tuesday' 
    when extract(isodow  from order_time) = 3 then 'Wednesday' 
    when extract(isodow  from order_time) = 4 then 'Thursday' 
    when extract(isodow  from order_time) = 5 then 'Friday' 
    when extract(isodow  from order_time) = 6 then 'Saturday' 
    when extract(isodow  from order_time) = 7 then 'Sunday' 
    end as Day
    , count(*) as NumberOfPizzaOrdered
    from pizza_runner.customer_orders	
    group by Day;

```

| day       | numberofpizzaordered |
| --------- | -------------------- |
| Thursday  | 3                    |
| Friday    | 1                    |
| Saturday  | 5                    |
| Wednesday | 5                    |


**Answer**  
On wednesady there were 5 orders  
On Thursday there were 3 orders  
On Friday there were 1 orders  
On Saturday there were 5 orders  

We can infer that on  Mondays and Tuesdays the Pizza runners don't work and Wednesdays and Saturdays are the bussiest.

#  B.Runner and Customer Experience


## 1.How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
To solve this we will use the date part function to find the weeks for each specific dates.  
```
SELECT *, date_part('week',registration_date) as week_period
FROM pizza_runner.runners;
```


| runner_id | registration_date        | week_period |
| --------- | ------------------------ | ----------- |
| 1         | 2021-01-01T00:00:00.000Z | 53          |
| 2         | 2021-01-03T00:00:00.000Z | 53          |
| 3         | 2021-01-08T00:00:00.000Z | 1           |
| 4         | 2021-01-15T00:00:00.000Z | 2           |

As per the result the date 2021-01-01 belongs to week 53 but it should be 1.so we tweak the query a bit to make 2021-01-01 as the first week.  

```
 SELECT *, 
 case when date_part('week',registration_date)+1 =54 then 1 else 
 date_part('week',registration_date)+1 end as week_period
 FROM pizza_runner.runners
 ```
| runner_id | registration_date        | week_period |
| --------- | ------------------------ | ----------- |
| 1         | 2021-01-01T00:00:00.000Z | 1           |
| 2         | 2021-01-03T00:00:00.000Z | 1           |
| 3         | 2021-01-08T00:00:00.000Z | 2           |
| 4         | 2021-01-15T00:00:00.000Z | 3           |

The result is as expected now we only need to count the number of registration per each week.  

```
    SELECT  
     case when date_part('week',registration_date)+1 =54 then 1 else 
     date_part('week',registration_date)+1 end as week_period
     ,count(*)
     FROM pizza_runner.runners
     group by week_period
     order by week_period;
```
| week_period | count |
| ----------- | ----- |
| 1           | 2     |
| 2           | 1     |
| 3           | 1     |

On week 1 two persons registered, on week 2 and 3 only one person registered.  

## 2.What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?  

We need order time and pickup time to find the answer and both of these attributes are in different tables.So first we need to join the tables together and find the average time by calculating the difference.  

```
--Use epoch to get the seconds from the pickup time and ordertime difference
SELECT  
AVG(CAST( 
  EXTRACT(EPOCH FROM (CAST(PICKUP_TIME AS TIMESTAMP) - ORDER_TIME  ) )
  	AS INTEGER) 
  /60)
 ,RUNNER_ID

FROM 
PIZZA_RUNNER.RUNNER_ORDERS

INNER JOIN 
PIZZA_RUNNER.CUSTOMER_ORDERS

ON RUNNER_ORDERS.ORDER_ID = CUSTOMER_ORDERS.ORDER_ID
WHERE PICKUP_TIME != 'NULL'
GROUP BY RUNNER_ID;		
```
![image](https://user-images.githubusercontent.com/78327987/155070117-a6b72b25-e037-4566-9e5f-28f832496038.png)

## 3.Is there any relationship between the number of pizzas and how long the order takes to prepare?  

Using ntile analytical function we can split the preparation time into two buckets.


``` 
    select  
    runner_orders.order_id
    ,count(*) as total_orders
    ,ntile(2) over (order by 
    cast( 
      extract(epoch from (cast(PICKUP_TIME as timestamp) - order_time  ) )
      	as integer) 
      /60) as bucket
      ,cast( 
      extract(epoch from (cast(PICKUP_TIME as timestamp) - order_time  ) )
      	as integer) 
      /60 as preptime
      
    FROM 
    PIZZA_RUNNER.RUNNER_ORDERS
    INNER JOIN 
    PIZZA_RUNNER.CUSTOMER_ORDERS
    ON RUNNER_ORDERS.ORDER_ID = CUSTOMER_ORDERS.ORDER_ID
    WHERE PICKUP_TIME != 'null'
    group by RUNNER_ORDERS.order_id, RUNNER_ORDERS.PICKUP_TIME, customer_ORDERS.order_time
    order by total_orders desc;
```
| order_id | total_orders | bucket | preptime |
| -------- | ------------ | ------ | -------- |
| 4        | 3            | 2      | 29       |
| 10       | 2            | 2      | 15       |
| 3        | 2            | 2      | 21       |
| 5        | 1            | 1      | 10       |
| 2        | 1            | 1      | 10       |
| 8        | 1            | 2      | 20       |
| 7        | 1            | 1      | 10       |
| 1        | 1            | 1      | 10       |

**Answer**

If there are more than one orders usually the prep time is more than 15 minutes.

Yes there is a relationship between the number of orders and the preparation time.

## 4.What was the average distance travelled for each customer?  

Joining the customer and runner orders table would give us the required data to answer the question.  

The distance attribute is of type varchar and inorder to do calcualtions on it we need to clean it.  

Some fields has 'km' at the trailing end. We can use regexp_replace to replace the 'km' with blanks.  

Then the attribute is casted as float inorder to do allow computation and then let's use ceiling to round it up to the highest value close to the decimal point.  

---

    select 
    customer_id
    ,ceiling(avg(cast(regexp_replace(runner_orders.distance,'km','','g')  as float))) -- g represents global which removes all matches if there are multiple'km' in the same attribute 
    
    from 
    PIZZA_RUNNER.customer_orders
    inner join 
    PIZZA_RUNNER.runner_orders
    on customer_orders.order_id = runner_orders.order_id
    where distance != 'null'
    group by customer_id;

| customer_id | ceiling |
| ----------- | ------- |
| 101         | 20      |
| 103         | 24      |
| 104         | 10      |
| 105         | 25      |
| 102         | 17      |

---


## 5.What was the difference between the longest and shortest delivery times for all orders?

The duration field gives the time taken by the driver to deliver the order when it is picked up from the headquaters.

```
    select 
    cast(max(substring(duration,1,2)) as int) - cast( min(substring(duration,1,2)) as int) as Difference
    from 
    PIZZA_RUNNER.runner_orders
    where duration != 'null';
```
| difference |
| ---------- |
| 30         |


If the delivery time is calculated from the point in time where an order is placed to the order being delivered to the customer i.e order placed + driver picking up order + delivery

 we need to include the time taken for the driver to reach the headquaters to pickup the order because delivery time should be the time captured when the customer places the order and the food getting deliverd to the customer.  

So we need to combine the customer_orders table and make use of order_time attribute.  

Subtract order time and pickup time and convert it into seconds then add it to duration( in secs).  



## 6.What was the average speed for each runner for each delivery and do you notice any trend for these values?

```

    select 
        
       avg(( cast(regexp_replace(runner_orders.distance,'km','','g') as float) /cast(substring(duration,1,2) as int) )*60)
        ,count(*) as number_of_orders
        
        ,runner_orders.runner_id
        from 
        PIZZA_RUNNER.runner_orders
        where duration != 'null'
        group by 
        runner_orders.runner_id
        order by number_of_orders;

| avg               | number_of_orders | runner_id |
| ----------------- | ---------------- | --------- |
| 40                | 1                | 3         |
| 62.9              | 3                | 2         |
| 45.53611111111111 | 4                | 1         |

```

