# Exploratory-Sales-Analysis---Superstore-Dataset.

This project demonstrates how I cleaned and analyzed the Superstore dataset using MySQL. 
It highlights how I solved date inconsistencies and derived actionable insights, such as regional sales and delivery efficiency.

## Problem Statement

The dataset contained sales records with `order_date` and `ship_date` stored as VARCHAR. 
Some dates were in mixed formats (e.g., 4/28/2018 vs 11/20/2019), causing SQL conversion errors.
The challenge was to clean and convert these dates into proper DATE types for analysis.

## Approach

1. **Preserve raw data**: I created a separate table `superstore_clean` from the original `superstore` CSV to avoid modifying raw data.

2. **Inspect the raw data**: I checked the `order_date` and `ship_date` columns for inconsistencies such as mixed-length dates, hidden characters, or invalid formats.

3. **Sanitize the data**: I cleaned the date columns in the new table using `TRIM` and `REGEXP_REPLACE` to remove spaces and non-numeric characters.

4. **Safe conversion to DATE type**: I created new columns in `superstore_clean` and converted the cleaned strings to proper `DATE` types using `STR_TO_DATE`. This approach avoids strict-mode errors and preserves invalid or corrupt rows as `NULL`.

5. **Validation**: I verified that all dates are valid calendar dates, ensuring there are no NULLs (except for previously corrupted rows).

6. **Analysis**: I computed delivery days (`DATEDIFF(ship_date_clean, order_date_clean)`), aggregated sales, delivery days, and orders by region, and visualized insights using charts.

## Key SQL Queries
Step 1: Clean hidden characters
-- UPDATE superstore_data
-- SET order_date = REGEXP_REPLACE(order_date, '[^0-9/]', ''),
--     ship_date  = REGEXP_REPLACE(ship_date,  '[^0-9/]', '');

Step 2: Create clean date columns
-- ALTER TABLE superstore_data
-- ADD COLUMN order_date_clean DATE,
-- ADD COLUMN ship_date_clean DATE;

Step 3: Attempt string-to-date conversion
-- UPDATE superstore_data
-- SET order_date_clean = STR_TO_DATE(order_date, '%m,%d,%Y'),
-- ship_date_clean = STR_TO_DATE(ship_date, '%m, %d, %Y')

Step 4: Create a clean analytical table
-- CREATE TABLE superstore_clean AS
-- SELECT
--     *,
--     STR_TO_DATE(order_date, '%m/%d/%Y') AS order_date_cleaned,
--     STR_TO_DATE(ship_date,  '%m/%d/%Y') AS ship_date_cleaned
-- FROM superstore_data;

Step 5: Remove raw date columns and finalize schema
-- ALTER TABLE superstore_clean
-- DROP COLUMN order_date,
-- DROP COLUMN order_date_clean,
-- DROP COLUMN ship_date_clean,
-- DROP COLUMN ship_date;

-- ALTER TABLE superstore_clean
-- CHANGE order_date_cleaned order_date DATE,
-- CHANGE ship_date_cleaned ship_date DATE;

Final Analysis Query: Regional Performance & Delivery Time
SELECT 
	region,
    ROUND(SUM(sales),2) AS total_sales,
    ROUND(COUNT(order_id),0) AS total_orders,
	ROUND(AVG(DATEDIFF(ship_date, order_date)), 2) AS avg_delivery_days
FROM superstore_clean
GROUP BY region

## Insights

- All dates successfully converted to DATE type for accurate analysis.
- Average delivery days per region:
  - West: 3.93 days
  - East: 3.91 days
  - Central: 4.06 days
  - South: 3.96 days
  
- Regional sales performance:
  - The West region recorded the highest total sales, indicating strong market demand.
  - Central and East regions showed lower total sales in comparison, suggesting regional differences in customer volume or purchasing power.

- Business implication:
  - Regions with high sales but slower delivery times may benefit from operational improvements.
  - Clean temporal data allows decision-makers to align logistics performance with sales growth trends.
    
- The dataset is now ready for further analysis, such as sales trends, top products, and customer behavior.


