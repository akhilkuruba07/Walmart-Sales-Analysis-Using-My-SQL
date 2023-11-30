# Walmart-Sales-Analysis-Using-My-SQL

-- Create database
CREATE DATABASE IF NOT EXISTS walmartSales;
Use walmartsales;
-- Create table
CREATE TABLE IF NOT EXISTS sales(
	invoice_id VARCHAR(30) NOT NULL PRIMARY KEY,
    branch VARCHAR(5) NOT NULL,
    city VARCHAR(30) NOT NULL,
    customer_type VARCHAR(30) NOT NULL,
    gender VARCHAR(30) NOT NULL,
    product_line VARCHAR(100) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    quantity INT NOT NULL,
    tax_pct FLOAT(6,4) NOT NULL,
    total DECIMAL(12, 4) NOT NULL,
    date DATETIME NOT NULL,
    time TIME NOT NULL,
    payment VARCHAR(15) NOT NULL,
    cogs DECIMAL(10,2) NOT NULL,
    gross_margin_pct FLOAT(11,9),
    gross_income DECIMAL(12, 4),
    rating FLOAT(2, 1)
);
select * from sales;
alter table sales add column Vat double after cogs ;
update sales set Vat = 0.05*cogs;
alter table sales add column time_of_day varchar(20);
SET SQL_SAFE_UPDATES = 0;

-- -----------------------------------Feature Engineering-------------------------------------------------

alter table sales add column time_of_day varchar(20);
SET SQL_SAFE_UPDATES = 0;
UPDATE sales
SET time_of_day = (
    CASE
        WHEN `time` BETWEEN '00:00:00' AND '12:00:00' THEN 'Morning'
        WHEN `time` BETWEEN '12:01:00' AND '16:00:00' THEN 'Afternoon'
        ELSE 'Evening'
    END
    );
-- Day Name
alter table sales add column Day_name varchar(10);
SET SQL_SAFE_UPDATES = 0;
UPDATE sales
SET Day_name= 
dayname(date);
-- MonthName
alter table sales add column Month_name varchar(10);
SET SQL_SAFE_UPDATES = 0;
UPDATE sales
SET Month_name= 
monthname(date);
-- -----------------------------------Generic-------------------------------------------------

-- How many unique cities does the data have?
select distinct(city) as "Unique Cities" from sales; 
-- In which city is each branch?
select distinct(city),branch from sales;
--  --------------------------------------Product-------------------------------------------------
-- How many unique product lines does the data have?
select count(distinct(product_line)) from sales;
-- What is the most common payment method?
select payment,count(payment) as cnt from sales group by payment order by cnt desc;
-- What is the most selling product line?
select product_line,count(product_line) as ProductCount from sales group by product_line order by ProductCount Desc;
-- What is the total revenue by month?
select Month_name as Months,sum(total) as Total_Revenue from sales group by Months order by Total_Revenue desc;
-- What month had the largest COGS?
select Month_name as Months , sum(cogs) as Counts from sales group by Months order by Counts desc; 
-- What product line had the largest revenue?
select product_line,sum(total) as Total_Revenue from sales group by product_line order by Total_Revenue desc;
-- What is the city with the largest revenue?
select city,branch, sum(total) as Revenue  from sales group by city,branch order by Revenue Desc;
-- What product line had the largest VAT?
select product_line,avg(tax_pct) as Vat from sales group by product_line order by Vat desc;
-- Which branch sold more products than average product sold?
select branch, sum(quantity) as QTY from sales group by branch having sum(quantity) >(select avg(quantity) from sales);
-- What is the most common product line by gender?
select gender,product_line,count(gender) as Total_count from sales group by gender,product_line order by Total_count Desc;
-- What is the average rating of each product line?
select product_line,avg(rating) as Avg_Rating from sales group by product_line order by Avg_rating desc;
-- -----------------------------------Sales-------------------------------------------------
-- Number of sales made in each time of the day per weekday
select time_of_day,count(*) As Total_Count from sales where day_name = "Sunday" group by time_of_day order by Total_Count desc;
-- Which of the customer types brings the most revenue?
select customer_type,sum(total) as Revenue from sales group by customer_type order by Revenue desc;
-- Which city has the largest tax percent/ VAT (Value Added Tax)?
select city,avg(vat) As Avg_Vat from sales group by city order by Avg_Vat desc;
-- Which customer type pays the most in VAT?
select customer_type,sum(vat) as Total_Vat from sales group by customer_type order by Total_Vat desc;
-- -----------------------------------Customer-------------------------------------------------
-- How many unique customer types does the data have?
select count(distinct(customer_type)) as Unique_Customer_type from sales;
-- How many unique payment methods does the data have?
select count(distinct(payment)) as Unique_Payment_type from sales;
-- What is the most common customer type?
select customer_type, count(*) as Most_type from sales group by customer_type order by Most_type desc;
-- What is the gender of most of the customers?
select gender,count(*) as CNT from sales group by gender order by CNT desc;
-- What is the gender distribution per branch?
select branch,gender,count(*) as Gender_count from sales group by branch,gender order by Gender_count Desc;
-- Which time of the day do customers give most ratings?
select time_of_day,avg(rating) as Avg_Rating from sales group by time_of_day order by  Avg_Rating desc;
-- Which time of the day do customers give most ratings per branch?
select time_of_day,branch,avg(rating) as Avg_Rating from sales group by time_of_day,branch order by  Avg_Rating desc;
-- Which day fo the week has the best avg ratings?
select day_name,avg(rating) as Avg_rating from sales group by day_name order by Avg_rating desc;
-- Which day of the week has the best average ratings per branch?
select day_name,branch,avg(rating) as Avg_rating from sales group by day_name,branch order by Avg_rating desc;
