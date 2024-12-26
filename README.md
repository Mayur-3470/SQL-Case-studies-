# SQL-Case-studies-


# Danny's Dinner

Situation:  
Danny's Diner, specializing in sushi, curry, and ramen, seeks assistance in leveraging basic customer data to enhance business operations. Danny aims to understand customer visiting patterns, expenditure, and favorite menu items to personalize the dining experience. Insights will guide decisions on expanding the loyalty program.

TASK:  
Danny has shared three crucial datasets for our case study: sales, menu, and members. Due to privacy concerns, he provided samples of overall customer data. We aim to craft fully functioning SQL queries using these examples to address Danny's questions and improve his restaurant's operations.

ACTION:  

<img width="449" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/f0a34e75-f759-47c5-a3d8-c5681d7c481e">  

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

Result:  
-- What is the total amount each customer spent at the restaurant?  
SELECT s.customer_id, SUM(m.price)  
FROM sales s  
JOIN menu m  
ON s.product_id=m.product_id  
GROUP BY s.customer_id;  

<img width="138" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/1b836d61-30d0-4566-8c70-3ff72c6c2f2e">  

-- How many days has each customer visited the restaurant?  

SELECT customer_id, COUNT(DISTINCT order_date) AS no_of_visit  
FROM sales  
GROUP BY customer_id;  

<img width="130" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/d10ad891-4f9e-4bfa-8e4f-85d4be0d0911">


-- What was the first item from the menu purchased by each customer?  

WITH cte AS (SELECT s.customer_id, s.order_date, m.product_name,  
dense_rank() OVER(PARTITION BY customer_id ORDER BY order_date) AS r  
FROM sales s  
JOIN menu m  
ON s.product_id=m.product_id  
Group by s.customer_id, m.product_name, s.order_date)  

SELECT cte.customer_id, cte.product_name  
FROM cte  
WHERE cte.r = 1;  

<img width="140" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/fb1968ec-1611-4da6-a1f3-fb12bdc74011">


-- What is the most purchased item on the menu and how many times was it purchased by all customers?  
SELECT m.product_name, COUNT(s.product_id) AS purchase_count  
FROM sales s  
JOIN menu m  
ON s.product_id=m.product_id  
GROUP BY m.product_name  
ORDER BY purchase_count DESC  
LIMIT 1;  

<img width="158" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/cb8b4784-372a-4e74-9964-9096a85600d1">


-- Which item was the most popular for each customer?  
WITH cte AS (SELECT s.customer_id, m.product_name, COUNT(s.product_id) AS purchase_count,  
dense_rank() OVER(PARTITION BY customer_id ORDER BY(SELECT COUNT(s.product_id) AS purchase_count)) AS r  
FROM sales s  
JOIN menu m  
ON s.product_id=m.product_id  
GROUP BY s.customer_id, m.product_name)  

SELECT cte.customer_id, cte.product_name  
FROM cte  
WHERE r=1;  

<img width="143" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/d554e867-c139-4a69-8d49-fc9a2d068565">


-- Which item was purchased first by the customer after they became a member?  
WITH cte AS(SELECT DISTINCT s.customer_id, s.product_id, s.order_date,  
dense_rank() OVER(PARTITION BY customer_id ORDER BY order_date) AS r  
FROM sales s  
JOIN members m  
ON s.order_date>=m.join_date  
WHERE s.customer_id IN (SELECT customer_id FROM members)),  

cte2 AS(SELECT product_id, product_name FROM menu)  

SELECT cte.customer_id, cte2.product_name, cte.order_date  
FROM cte  
JOIN cte2 ON cte.product_id=cte2.product_id  
WHERE r=1  
ORDER BY cte.customer_id;  

<img width="200" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/c1a0220c-8a80-4cee-a06d-32fa01c1950f">


-- Which item was purchased just before the customer became a member?  
WITH cte AS(SELECT s.customer_id, s.product_id, s.order_date,  
dense_rank() OVER(PARTITION BY customer_id ORDER BY order_date DESC) AS r  
FROM sales s  
JOIN members m  
ON s.customer_id=m.customer_id  
WHERE s.customer_id IN (SELECT customer_id FROM members) AND s.order_date < m.join_date),  

cte2 AS(SELECT product_id, product_name FROM menu)  

SELECT cte.customer_id, cte2.product_name, cte.order_date  
FROM cte  
JOIN cte2 ON cte.product_id=cte2.product_id  
WHERE r=1  
ORDER BY cte.customer_id;  

<img width="202" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/b2c93cc7-e4aa-4355-8232-f4319e9bc55f">


-- What is the total items and amount spent for each member before they became a member?  
SELECT s.customer_id, COUNT(s.product_id) AS total_item, SUM(m.price) AS total_amount  
FROM sales s  
JOIN menu m  
ON s.product_id=m.product_id  
JOIN members mem  
ON s.customer_id=mem.customer_id  
WHERE s.customer_id IN (SELECT customer_id FROM members) AND s.order_date < mem.join_date  
GROUP BY s.customer_id  
ORDER BY s.customer_id;  

<img width="194" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/212bf8fe-7920-4007-a1fd-418a14d270d7">


-- If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?  
WITH cte AS(SELECT s.customer_id, s.product_id, m.price,  
CASE when s.product_id=1 THEN m.price*10*2 ELSE m.price*10 END AS points  
FROM sales s  
JOIN menu m  
ON s.product_id=m.product_id)  

SELECT cte.customer_id, SUM(cte.points) AS total_points  
FROM cte  
GROUP BY cte.customer_id;  

<img width="142" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/2e50d8be-a90d-42e7-9cea-703fcb1a2db4">


-- In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?  
WITH cte AS(SELECT s.customer_id, s.product_id, m.price, s.order_date,  
CASE when s.product_id=1 THEN m.price*10*2  
WHEN s.order_date BETWEEN mem.join_date AND DATE_ADD(mem.join_date, INTERVAL 6 DAY) THEN m.price*10*2  
ELSE m.price*10 END AS points  
FROM sales s  
JOIN menu m  
ON s.product_id=m.product_id  
JOIN members mem  
ON s.customer_id=mem.customer_id  
WHERE MONTH(order_date)= 1)  

SELECT cte.customer_id, SUM(cte.points) AS total_points  
FROM cte  
GROUP BY cte.customer_id  
ORDER BY customer_id;  

<img width="134" alt="image" src="https://github.com/Sweta-Sah/Case_Study-1--SQL/assets/132820867/3b07a865-6d21-4b51-92fc-6ecde2cbb424">





*SQL Questions with solutions.

1) All movies released in the year 2022?
Ans: SELECT * FROM movies where release_year = 2022

2)Now all the movies released after 2020?
Ans: SELECT * FROM movies where release_year >= 2020

3) All movies after the year 2020 that have more than 8 rating?
Ans: select * from movies where release_year>2020 and imdb_rating>8

4) Select all movies that are by Marvel studios and Hombale Films?
Ans: Select * from movies where studio IN ("Marvel studios" , "Hombale films")

5)Select all THOR movies by their release year?
Ans: Select title, release_year  from movies where title like "%Thor%" order by release_year asc ( Only title & Release year)
     Select *  from movies where title like "%Thor%" (All details from movies)

6)Select all movies that are not from Marvel Studios?
Ans: Select *  from movies where studio = "Marvel studios" 

7)Print all movies in the order of their release year (latest first)?
Ans: Select *  from movies order by release_year DESC

8)Print all movie titles and release year for all Marvel Studios movies
Ans: SELECT title, release_year from movies where studio="Marvel Studios"

9)Print all movies that have Avenger in their name
Ans: SELECT * from movies where title LIKE '%Avenger%'

10)Print the year in which "The Godfather" move was released
Ans: SELECT release_year from movies where title="The Godfather"

11)Print all distinct movie studios on Bollywood industry
Ans: SELECT DISTINCT studio from movies where industry="Bollywood"

12)How many movies were released between 2015 and 2022?
Ans: Select
     count(studio) from movies
     where release_year
     Between 2015 AND 2022 
     Group by studio

13)Print the max and min movie release year?
Ans: Select 
     min(release_year) as Min_year,
     max(release_year) as Max_year from movies

14)Print a year and how many movies were released in that year starting with the latest year?
Ans: select 
     Release_year , Count(*) as movie_count 
     from movies group by  release_year 
     order by release_year DESC

15)Print all the years with more than 2 movies were released?
Ans: Select Release_year , Count(*) as movie_count
     From Movies
     Group by release_year 
     Having movie_Count>2
     Order by movie_count DESC

16)How to add Current date/Year column in MY SQL?
Ans: Select Year(CURDATE());

17)Print profit % for all the movies?
Ans: select *, 
    (revenue-budget) as profit, 
    (revenue-budget)*100/budget as profit_pct 
    from financials

18)You have a table with cricket scores of players and you want to retrieve second, third and fourth highest scores. Select the correct query for this?
Ans: SELECT * from player_stats 
     ORDER BY score DESC LIMIT 3 OFFSET 1

19)For movies table, write a query to print (1) count of distinct imdb_rating (2) standard deviation of imdb_rating (HINT: Use your google skills)?
Ans: SELECT count(distinct imdb_rating), 
     STDDEV(imdb_rating) from movies

20)Show all the movies with their language names?
Ans: SELECT
     title, studio ,l.language_id , name 
     FROM movies m
     JOIN languages l  
     ON m.language_id = l.language_id

21)Show all Telugu movie names (assuming you don't know the language_id for Telugu)?
Ans: SELECT title,name	
     FROM movies m 
     LEFT JOIN languages l 
     ON m.language_id=l.language_id
     WHERE l.name="Telugu";

22)Show the language and number of movies released in that language?
Ans: Select 
     name , COUNT(m.movie_id) as no_moviecount
     from languages l
     LEFT JOIN movies m
     USING (language_id)
     group by language_id
     order by no_moviecount DESC

23)How to Cross join 2 tables in one new tables?
Ans: SELECT *,
     CONCAT(name,  " - " ,variant_name) AS full_name,
     (price + variant_price) As full_price
     FROM food_db.items
     CROSS JOIN variants 

24)how to convert profit into millions & billions?
Ans: SELECT 
     m.movie_id, title , budget , revenue , currency , unit ,
     CASE 
     WHEN unit = "thousand" then ROUND ((revenue - budget)/1000,1)
     WHEN unit = "Billions" then ROUND((revenue - Budget)*1000,1)
     ELSE ROUND((revenue-budget),1)
     END as profit_mln 
     FROM movies m
     JOIN financials f
     ON m.movie_id = f.movie_id
     WHERE industry = "Bollywood"
     order by profit_mln DESC

25)Generate a report of all Hindi movies sorted by their revenue amount in millions.
Ans:  SELECT 
      m.title, revenue , currency , unit ,
      CASE 
      WHEN unit = "thousands" then ROUND(revenue/1000,2)
      WHEN unit = "Billions" then ROUND(revenue*1000,2)
      ELSE ROUND(revenue,2) 
      END as revenue_mln 
      FROM movies m
      JOIN financials f 
      ON m.movie_id = f.movie_id
      JOIN languages l
      ON m.language_id = l.language_id
      where l.name = "Hindi"
      order by revenue_mln DESC

26)In customers table, there are two columns: first_name and last_name. Now you want to print those records where first_name + last_name is duplicate along with their count. Here duplicate means all outputs whose count is greater than 1. Which query can give you this result?
Ans: SELECT first_name, last_name, 
     count(*) as cnt from customers 
     GROUP BY first_name, last_name 
     HAVING cnt>1

27)You have two tables, pricing and discounts. Common link between these tables is product_code which can have null value in both the tables. A null product_code record in pricing table is considered a match with discounts table record with NULL product_code value. Which query can perform a correct inner join on all the records including NULL records? (HINT: Use your Google skills)?
Ans: SELECT * from pricing p 
     JOIN discounts d ON 
     IFNULL(p.product_code, 1) = IFNULL(d.product_code,1)

28)Following query is used to join pricing and discounts table where both pricing and discount amount is calculated using a combination of customer_code and product_code?
Ans: SELECT * from pricing 
     JOIN discounts 
     USING (customer_code, product_code)

29)product_code is a common link between pricing and discounts tables. Write a query that prints product_code along with its final price after discounts. Here are the names of the columns along with their purpose: discount_pct -> percentage discount (value is between 0 to 1), gross_price -> gross or original price of a product?
Ans: SELECT p.product_code, (1 - d.discount_pct)*p.gross_price as final_price 
     from pricing p 
     JOIN discounts d 
     USING (product_code)

30)How many movies each actor acted in the film?
Ans: SELECT a.actor_id , name , count(*) as movie_count                                                              
     FROM movie_actor ma
     JOIN actors a
     ON a.actor_id = ma.actor_id
     group by a.actor_id 
     order by movie_count DESC


     SELECT actor_id,name , 
     (Select count(*) 
     from movie_actor 
     where actor_id = actors.Actor_id ) as movie_count
     from actors
     order by movie_count DESC

31)Select all the movies with minimum and maximum release_year. Note that there can be more than one movie in min and a max year hence output rows can be more than 2?
Ans: Select * from movies where release_year 
     in((select MIN(release_year) from movies),
     (select max(release_year) from movies))

32)select all the rows from movies table whose imdb_rating is higher than the average rating?
Ans: SELECT * FROM movies
     Where imdb_rating > (select avg (imdb_rating) 
     from movies)

33)Select all Hollywood movies released after the year 2000 that made more than 500 million $ profit or more profit. Note that all Hollywood movies have millions as a unit hence you don't need to do the unit conversion. Also, you can write this query without CTE as well but you should try to write this using CTE only?
Ans: with cte as (select title, release_year, (revenue-budget) as profit
     from movies m
     join financials f
     on m.movie_id=f.movie_id
     where release_year>2000 and industry="hollywood")
     select * from cte where profit>500

34)Which clause executes the condition when at least one of the values in a list meets the condition?
Ans: SELECT DISTINCT(imdb_rating) FROM movies ORDER BY imdb_rating DESC LIMIT 1 OFFSET 5

35)Write a query to select most youngest and oldest 3 actors from our movies database?
Ans: (SELECT name, birth_year FROM actors ORDER BY birth_year ASC LIMIT 3) 
     UNION 
     (SELECT name, birth_year FROM actors ORDER BY birth_year DESC LIMIT 3)

36)For the salaries table below, write an SQL query that can print all the employees whose salary is greater than the average salary in the department?
emp_id	   name	        department	salary
10	Virat Koli	Engineering	10200
11	Bo Jiden	    HR	        12500
12	Mariah Darey	Engineering	23100
13	Babar Kazam	    HR	        14200
Ans: WITH avg_salary_table as (SELECT department, AVG(salary) as avg_salary 
     FROM 
     salaries GROUP BY deparment) 
     SELECT * from salaries s JOIN avg_salary_table a 
     USING (department) 
     WHERE s.salary > a.avg_salary

37)To create fiscal_year quarter in MYSQL (function)?
Ans: CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_quarter`(
     calendar_date date
     ) 
     RETURNS char(2) CHARSET utf8mb4
     DETERMINISTIC
     BEGIN
     DECLARE M TINYINT;
     DECLARE qtr CHAR(2);
     SET M = MONTH(calendar_date);
     CASE 
     WHEN M IN (9,10,11) THEN set qtr ="Q1";
     WHEN M IN (12,1,2) THEN set qtr ="Q2";
     WHEN M IN (3,4,5) THEN set qtr ="Q3";
     ELSE set qtr ="Q4";
     END CASE;
     RETURN qtr;
     END

38)To create Fiscal_year in MYSQL (function)?
Ans: CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_year`(
     calendar_date date 
     ) 
     RETURNS int
     DETERMINISTIC
     BEGIN
     Declare fiscal_year INT;
     SET fiscal_year = YEAR(date_add(calendar_date , interval 4 month));
     RETURN fiscal_year;
     END

39)Generate a yearly report for Croma India where there are two columns
1. Fiscal Year?
2. Total Gross Sales amount In that year from Croma?
Ans: SELECT 
     get_fiscal_year(date) as fiscal_year,
     SUM(ROUND(g.gross_price*s.sold_quantity,2)) as total_gross_price
     FROM fact_sales_monthly S
     JOIN fact_gross_price G
     ON g.product_code=s.product_code
     AND
     g.fiscal_year=get_fiscal_year(s.date)
     where customer_code=90002002 
     GROUP BY get_fiscal_year(date)
     ORDER BY fiscal_year

40) how to get store procedure in MY SQL on market bases?
Ans: CREATE DEFINER=`root`@`localhost` PROCEDURE `get_market_badge`(
     IN in_market VARCHAR(45),
     IN in_fiscal_year year,
     OUT out_badge varchar(45)
     )
     BEGIN
     declare qty int default 0;

     # reterive total qty for a given market
     Select 
     SUM(sold_quantity) into qty
     from fact_sales_monthly S
     JOIN dim_customer C
     on S.customer_code=C.customer_code
     where get_fiscal_year(s.date) = in_fiscal_year 
     AND c.market = in_market
     group by c.market;

     # determine market badge + fiscal year

     IF qty > 5000000 then 
     set out_badge = "gold";
     else 
     set out_badge = "Silver";
     end if;
     END 

41)Full form of DDL and DML are? 
Ans: DDL: Data Definition Language
     DML: Data Manipulation Language









