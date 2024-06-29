# TechTronix-SQL-project.



# Project Title

### TechTronix Sales Data Analysis

#### Introduction

T.T Inc. is a leading company in the consumer electronics
sector. You have been asked by the Head of Supply Chain Management to present data
insights and optimization strategies for inventory management. Within the Supply Chain
Management team, your objectives are to:
● Optimize inventory levels to minimize overstock and understock situations.
● Understanding seasonal trends of sales for different products.
● Improve customer satisfaction by ensuring product availability.

#### Data Overview

Data Description

This case study contains 3 datasets and they are as
follows;
Sales Data

● Sales ID: Unique sales identifier.

● Product ID: Unique product identifier.

● Sales Date/month/year: Date/month/year of product sale
(Date)

● Sales Quantity (Units): Number of units sold (Units).

● Product Cost (USD per Unit): Cost per product unit in USD
(USD per Unit).

Product Information Data


● Product ID: Unique product identifier.

● Product Category: Product type.

● Promotions: Indicator of promotions.

External Information Data

● Sales Date/month/year: Date/month/year of product sale(Date)

● GDP (Gross Domestic Product) (USD): Economic data in USD (USD).

● Inflation Rate (%): Percentage change in prices.

● Seasonal Factor (Dimensionless): Index for seasonal effects.

#### Questions to help with analyzing the data.
a) What is the total number of units sold per product SKU?

b) Which product category had the highest sales volume last month?

c) How does the inflation rate correlate with sales volume for a specific month?

d) What is the correlation between the inflation rate and sales quantity for all products combined on a monthly basis over the last year?

e) Did promotions significantly impact the sales quantity of products?

f) What is the average sales quantity per product category?

g) How does the GDP affect the total sales volume?

h) What are the top 10 best-selling product SKUs?

i) How do seasonal factors influence sales quantities for different product categories?

j) What is the average sales quantity per product category, and how many products within each
category were part of a promotion?
l) Calculation of the year-to-year and month-to-month sales Growth.


### What is the total number of units sold per product SKU?
    SELECT productid,
    SUM(inventoryquantity) AS total_quantity
    FROM sales
    GROUP BY productid
    ORDER BY 2 DESC


### Which product category had the highest sales volume last month?

    SELECT p.productid AS pid,p.productcategory AS category,
    SUM(s.inventoryquantity) AS inventory
    FROM sales s
    JOIN product p
    ON s.productid = p.productid
    WHERE s.salesdate >= DATE_TRUNC('month','2022-12-10'::date)
    AND s.salesdate < DATE_TRUNC('month','2022-12-10'::date) + INTERVAL '1 month'
    GROUP BY 1,2
    ORDER BY 3 DESC


#### QUESTION 3 : How does the inflation rate correlate with sales volume for a specific month?

    -- Step 1: Aggregate sales data by month

    CREATE TEMP TABLE monthly_sales AS
    SELECT 
    DATE_TRUNC('month', s.salesdate) AS sales_month,
    SUM(s.inventoryquantity) AS total_sales_volume,
    AVG(f.inflationrate) AS avg_inflation_rate
    FROM 
    sales s
    JOIN 
    factors f ON DATE_TRUNC('month', s.salesdate) = DATE_TRUNC('month', f.salesdate)
    GROUP BY 1

-- Calculate the correlation between total sales volume and average inflation rate
    
    SELECT 
    corr(total_sales_volume, avg_inflation_rate) AS sales_inflation_correlation
    FROM 
    monthly_sales;
 OR 
-- Step 1: Aggregate sales data for Month of DEC 2021

    CREATE TEMP TABLE monthly_sales AS
    SELECT 
    s.salesdate >= DATE_TRUNC('month', '2021-12-10'::date)
    AND s.salesdate < DATE_TRUNC('month', '2021-12-10'::date) + INTERVAL '1 month' AS sales_month,
    SUM(s.inventoryquantity) AS total_sales_volume,
    AVG(f.inflationrate) AS avg_inflation_rate
    FROM 
    sales s
    JOIN 
    factors f ON DATE_TRUNC('month', s.salesdate) = DATE_TRUNC('month', f.salesdate)
    GROUP BY 1
    
-- Calculate the correlation between total sales volume and average inflation rate

    SELECT 
    corr(total_sales_volume, avg_inflation_rate) AS sales_inflation_correlation
    FROM 
    monthly_sales;


### Question 4 : What is the correlation between the inflation rate and sales quantity for all products combined on a
monthly basis over the last year

    -- Aggregate sales data by month and calculate correlation for the last year

    WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', s.salesdate) AS     sales_month,
        SUM(s.inventoryquantity) AS     total_sales_volume,
        AVG(f.inflationrate) AS avg_inflation_rate
    FROM 
        sales s
    JOIN 
        factors f ON DATE_TRUNC('month', s. salesdate) = DATE_TRUNC('month', f.salesdate)
    WHERE 
        s.salesdate >= DATE_TRUNC('month',  CURRENT_DATE) - INTERVAL '1 year'
    GROUP BY 1
    )

-- Calculate the correlation between total sales volume and average inflation rate

    SELECT 
    corr(total_sales_volume, avg_inflation_rate) AS sales_inflation_correlation
    FROM 
    monthly_sales;

-- Question 5 : Did promotions significantly impact the sales quantity of products?

    -- Aggregate sales data by promotion status
    SELECT 
    p.promotions,
    SUM(s.inventoryquantity) AS total_sales_quantity,
    AVG(s.inventoryquantity) AS avg_sales_quantity,
    COUNT(*) AS num_sales
    FROM 
    sales s
    JOIN 
    product p ON s.productid = p.productid
    GROUP BY 
    p.promotions;

-- Question 6  
What is the average sales quantity per product category?


    -- Calculate the average inventory quantity per product category
    SELECT 
    p.productcategory AS category,
    AVG(s.inventoryquantity) AS avg_inventory_quantity
    FROM 
    sales s
    JOIN 
    product p ON s.productid = p.productid
    GROUP BY 
    p.productcategory;


##### Question 7 : How does the GDP affect the total sales volume?


    WITH monthly_sales AS (
    SELECT 
    DATE_TRUNC('month', s.salesdate) AS sales_month,
    SUM(s.inventoryquantity) AS total_sales_quantity,
    AVG(f.gdp) AS avg_gdp
    FROM sales s
    JOIN 
    factors f ON DATE_TRUNC('month', s.salesdate) = DATE_TRUNC('month', f.salesdate)
    WHERE 
    s.salesdate >= CURRENT_DATE - INTERVAL '1 year'
    GROUP BY  1
    )
    SELECT 
    corr(total_sales_quantity, avg_gdp) AS sales_gdp_correlation
    FROM 
    monthly_sales;

    
### Question 8 : What are the top 10 best-selling product SKUs

    SELECT   p.productid,
    SUM(s.inventoryquantity) AS total_sales
    FROM sales s
    JOIN product p ON s.productid = p.productid
    GROUP BY p.productid
    ORDER BY total_sales DESC
    LIMIT 10;

### Question 9 : How do seasonal factors influence sales quantities for different product categories?

    SELECT f.seasonalfactor AS reason,
    p.productcategory AS category,
    SUM(s.inventoryquantity) AS total_sales,
    AVG(s.inventoryquantity) AS avg_sales
    FROM factors f
    JOIN sales s ON f.salesdate = s.salesdate
    JOIN product p ON s.productid = p.productid     Join with product table
    GROUP BY f.seasonalfactor, p.productcategory
    ORDER BY f.seasonalfactor, total_sales DESC;

### Question 10 : What is the average sales quantity per product category, and how many products within each category were part of a promotion?

    -- Calculate the average sales quantity per product category and count promoted products
    
    SELECT 
    p.productcategory AS category,
    AVG(s.inventoryquantity) AS avg_inventory_quantity,
    COUNT(DISTINCT CASE WHEN p.promotions = 'Yes' THEN s.productid END) AS num_promoted_products
    FROM 
    sales s
    JOIN 
    product p ON s.productid = p.productid
    GROUP BY 
    p.productcategory;


### Question 11 : Which product categories have the highest and lowest average sales quantities?

    p.productcategory AS category,
    AVG(s.inventoryquantity) AS avg_inventory_quantity
    FROM 
    sales s
    JOIN 
    product p ON s.productid = p.productid
    GROUP BY 
    p.productcategory
    ORDER BY 
    avg_inventory_quantity DESC;


### Question 12 : Calculate MoM% Revenue, YoY% Revenue
#### -- MoM% and YoY% Revenue Growth

    WITH monthly_revenue AS (
    SELECT 
    DATE_TRUNC('month', s.salesdate) AS sales_month,
    SUM(s.inventoryquantity * s.productcost) AS total_revenue
    FROM 
    sales s
    GROUP BY 
    DATE_TRUNC('month', s.salesdate)
    ),      
    monthly_revenue_with_lag AS (
    SELECT 
    sales_month,
    total_revenue,
    LAG(total_revenue, 1) OVER (ORDER BY    sales_month) AS prev_month_revenue,
    LAG(total_revenue, 12) OVER (ORDER BY sales_month) AS prev_year_revenue
    FROM 
    monthly_revenue
    )
    SELECT 
    sales_month,
    total_revenue,
    CASE 
    WHEN prev_month_revenue IS NULL THEN NULL
    ELSE ((total_revenue - prev_month_revenue) / prev_month_revenue) * 100
    END AS mom_percentage_change,
    CASE 
    WHEN prev_year_revenue IS NULL THEN NULL
    ELSE ((total_revenue - prev_year_revenue) / prev_year_revenue) * 100
    END AS yoy_percentage_change
    FROM 
    monthly_revenue_with_lag
    ORDER BY 
    sales_month;

### Number of Customers Ordering Last Year vs. This Year
-- YoY% Revenue Growth

    -- Customers Ordering Last Year
    SELECT
    COUNT(DISTINCT customerID) AS customers_last_year
    FROM
    NorthwindSales
    WHERE
    TO_CHAR(orderDate, 'YYYY') = TO_CHAR(SYSDATE, 'YYYY') - 1;

-- Customers Ordering This Year

    SELECT
    COUNT(DISTINCT customerID) AS customers_this_year
    FROM
    NorthwindSales
    WHERE
    TO_CHAR(orderDate, 'YYYY') = TO_CHAR(SYSDATE,   'YYYY');

### Average Days Between Order Date and Shipping Date.
    SELECT
    AVG(shippedDate - orderDate) AS     avg_days_to_ship
    FROM
    NorthwindSales;

### Distribution of Number of Orders by Order Month
    SELECT
    order_month,
    number_of_orders
    FROM (
    SELECT
    TO_CHAR(orderDate, 'YYYY-MM') AS        order_month,
    COUNT(orderID) AS number_of_orders
    FROM
    NorthwindSales
    GROUP BY
    TO_CHAR(orderDate, 'YYYY-MM')
    )  
    ORDER BY
    order_month;

### Key Findings

#### Monthly Revenue Trends
Total Monthly Revenue: By aggregating the total revenue per month, we observed the revenue trends over the past year.
Month-over-Month (MoM) Revenue Change: The MoM percentage change in revenue provided insights into how revenue fluctuated on a monthly basis.
Year-over-Year (YoY) Revenue Change: The YoY percentage change in revenue allowed us to compare the revenue performance of each month to the same month in the previous year.

#### Correlation Between Economic Factors and Sales.

GDP and Sales Volume: The analysis of the correlation between GDP and total sales volume indicated how economic conditions influenced sales. A positive correlation would suggest that higher GDP is associated with higher sales volume, while a negative correlation would suggest the opposite.
Active Products: Count and list of products still available for sale.

#### Impact of Promotions on Sales

Promotions and Sales Quantity: By analyzing the average sales quantity per product category and the number of products within each category that were part of a promotion, we could determine the effectiveness of promotions in boosting sales. Categories with a higher average sales quantity during promotions indicate successful promotional strategies.

#### Revenue by Product Category
Revenue Distribution: By calculating the monthly revenue per product category, we could identify which categories contributed the most to the overall revenue and how their performance varied throughout the year.



#### Inflation Rate and Sales Quantity
Economic Impact on Sales: The correlation analysis between the inflation rate and sales quantity provided insights into how changes in inflation influenced consumer purchasing behavior. A positive correlation would suggest that higher inflation rates are associated with higher sales quantities, possibly due to consumers purchasing more before anticipated price increases, while a negative correlation would suggest the opposite.


#### Recommendations

Growth Strategies: Identify months or periods with declining revenue and develop strategies to address the declines, such as new marketing campaigns or product launches.
Benchmarking: Use YoY comparisons to benchmark performance and set realistic sales targets.
Additional Recommendations
Data-Driven Decision Making:

Use the insights from these analyses to drive strategic business decisions, focusing on data-driven approaches for marketing, inventory management, and pricing strategies.
Continuous Monitoring:

Regularly update and analyze sales data to stay informed about changing trends and to quickly adapt strategies as needed.
Customer Insights:

Conduct additional analyses to understand customer behavior and preferences, such as segmenting customers based on purchase patterns and targeting them with personalized marketing.
Investment in High-ROI Areas:

Prioritize investments in product categories, promotions, and periods that show high returns on investment based on historical data.
By following these recommendations, the business can optimize sales performance, improve promotional effectiveness, and better adapt to economic conditions, ultimately leading to increased revenue and growth.

