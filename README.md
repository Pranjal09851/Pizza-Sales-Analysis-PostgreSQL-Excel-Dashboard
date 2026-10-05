# Pizza-Sales-Analysis-PostgreSQL-Excel-Dashboard
An end-to-end sales analysis of a pizza restaurant. I used PostgreSQL to calculate KPIs and trends with SQL, and built an Excel dashboard using PivotTables and charts to present the results.

📌 Business Questions
How much revenue did the restaurant make, and how many orders and pizzas did it sell?
What is the average order value, and how many pizzas does a typical order contain?
Which days and hours are the busiest?
Which pizza categories and sizes contribute the most to sales?
What are the best and worst selling pizzas?
🗂️ Dataset
Detail	Value
Period	1 Jan 2015 – 31 Dec 2015
Rows (pizza line items)	48,620
Unique orders	21,350
Unique pizza names	32

Key columns: order_id, order_date, order_time, pizza_name, pizza_category, pizza_size, quantity, unit_price, total_price

🛠️ Tools Used
PostgreSQL: querying and KPI calculation
Microsoft Excel: PivotTables, charts, dashboard
SQL concepts: aggregation, GROUP BY, COUNT(DISTINCT), subqueries, date/time functions, ORDER BY ... LIMIT
📊 Dashboard Highlights
KPI	Value
Total Revenue	$817,860
Total Orders	21,350
Total Pizzas Sold	49,574
Average Order Value	$38.31
Average Pizzas per Order	2.32
<img width="1917" height="1018" alt="image" src="https://github.com/user-attachments/assets/9d029391-5121-49a8-aad1-6ef84ed735f5" />

🔍 Key Insights
Friday is the busiest day (3,538 orders), about 35% more than Sunday, the slowest day (2,624).
Two clear rush periods: lunch (12–2 PM) and evening (5–8 PM) together account for about 55% of all orders. Almost nothing happens before 11 AM.
Large pizzas drive nearly half of revenue (45.9%), followed by Medium (30.5%) and Regular (21.8%). X-Large and XX-Large together contribute under 2%.
Classic is the top category by volume (14,888 pizzas, about 30% of units). Revenue is spread fairly evenly across all four categories (23.7% – 26.9%).
Best sellers: The Classic Deluxe (2,453), The Barbecue Chicken (2,432), The Hawaiian (2,422), The Pepperoni (2,418) and The Thai Chicken (2,371).
Worst sellers: The Brie Carre sells only 490 units, roughly half of the next-lowest pizza (The Mediterranean, 934), making it a candidate for a menu review.

🧮 SQL Queries (PostgreSQL)

Table: pizza_sales

A. KPIs
sql
-- Total Revenue
SELECT SUM(total_price) AS total_revenue
FROM pizza_sales;

-- Average Order Value
SELECT ROUND(SUM(total_price)::numeric / COUNT(DISTINCT order_id), 2) AS avg_order_value
FROM pizza_sales;

-- Total Pizzas Sold
SELECT SUM(quantity) AS total_pizzas_sold
FROM pizza_sales;

-- Total Orders
SELECT COUNT(DISTINCT order_id) AS total_orders
FROM pizza_sales;

-- Average Pizzas per Order
SELECT ROUND(SUM(quantity)::numeric / COUNT(DISTINCT order_id), 2) AS avg_pizzas_per_order
FROM pizza_sales;
B. Daily Trend for Total Orders
sql
SELECT TRIM(TO_CHAR(order_date, 'Day')) AS order_day,
       COUNT(DISTINCT order_id)         AS total_orders
FROM pizza_sales
GROUP BY 1, EXTRACT(ISODOW FROM order_date)
ORDER BY EXTRACT(ISODOW FROM order_date);
C. Hourly Trend for Orders
sql
SELECT EXTRACT(HOUR FROM order_time)::int AS order_hour,
       COUNT(DISTINCT order_id)           AS total_orders
FROM pizza_sales
GROUP BY 1
ORDER BY 1;
D. % of Sales by Pizza Category
sql
SELECT pizza_category,
       ROUND(SUM(total_price)::numeric, 2) AS total_revenue,
       ROUND((SUM(total_price) * 100 / (SELECT SUM(total_price) FROM pizza_sales))::numeric, 2) AS pct
FROM pizza_sales
GROUP BY pizza_category
ORDER BY pct DESC;
E. % of Sales by Pizza Size
sql
SELECT pizza_size,
       ROUND(SUM(total_price)::numeric, 2) AS total_revenue,
       ROUND((SUM(total_price) * 100 / (SELECT SUM(total_price) FROM pizza_sales))::numeric, 2) AS pct
FROM pizza_sales
GROUP BY pizza_size
ORDER BY pct DESC;
F. Total Pizzas Sold by Category
sql
SELECT pizza_category,
       SUM(quantity) AS total_quantity_sold
FROM pizza_sales
GROUP BY pizza_category
ORDER BY total_quantity_sold DESC;
G. Top 5 Best Sellers
sql
SELECT pizza_name, SUM(quantity) AS total_pizzas_sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY total_pizzas_sold DESC
LIMIT 5;
H. Bottom 5 Sellers
sql
SELECT pizza_name, SUM(quantity) AS total_pizzas_sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY total_pizzas_sold ASC
LIMIT 5;
Filtering by month or quarter

Built from the cleaned dataset using PivotTables and charts:

KPI cards: revenue, orders, pizzas sold, average order value, average pizzas per order
Daily and hourly order trends
% of sales by pizza category and by pizza size
Pizzas sold by category
Top 5 and bottom 5 best-selling pizzas

Technique used: a helper column total_orders (1 ÷ number of pizzas in that order) lets a PivotTable sum to the correct number of distinct orders, since PivotTables don't count distinct values by default.

📁 Suggested Repository Structure
pizza-sales-analysis/
├── README.md
├── sql/
│   └── pizza_sales_queries.sql
├── excel/
│   └── pizza_sales_dashboard.xlsx
├── data/
│   └── pizza_sales.csv
└── images/
    └── dashboard.png

▶️ How to Reproduce
Create a table pizza_sales in PostgreSQL and import the dataset (pgAdmin → right-click table → Import/Export).
Run the queries from the sql/ folder and compare results with the dashboard.
Open the Excel file and use Data → Refresh All to update the PivotTables.

