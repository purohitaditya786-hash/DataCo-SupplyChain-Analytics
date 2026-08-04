# DataCoSupply Chain & Logistics Analytics 

An executive-grade Power BI dashboard and SQL analytics project designed to optimize supply chain efficiency, reduce shipping losses, and perform inventory rationalization using the DataCo Global Supply Chain dataset.

---

##  Executive Summary
This project analyzes **$37M+ in global sales** and **180K+ order transactions** to identify operational bottlenecks, shipping inefficiencies, and profit-draining products.

### Key Insights:
* **Financial Impact:** Identified **$4M in shipping losses** driven by negative profit orders.
* **Logistics Inefficiencies:** Analyzed **99K delayed orders** with a global **Late Delivery Rate of 55%**.
* **Inventory Optimization:** Isolated **34K loss-making orders** and flagged zero/low-demand products for portfolio rationalization.

---

##  Tech Stack & Tools Used
* **Business Intelligence:** Power BI (DAX, Power Query, Data Modeling, Custom Dark Theme UI)
* **Database & Querying:** SQL (CTEs, Aggregations, Filtering, Conditional Logic)
* **Data Processing:** Microsoft Excel / CSV

---

##  Dashboard Architecture
The Power BI report is structured into 3 interactive pages following an executive dark navy (`#0A2540`) UI standard:

1. **Executive Overview (`Page 1`):** Global revenue distribution, regional performance, fulfillment status, and key financial KPIs ($37M Revenue, $4M Net Profit).
2. **Logistics & Operations (`Page 2`):** Late delivery trends, scheduled vs. actual shipping days, top delayed cities, and total shipping loss metrics.
3. **Product Performance (`Page 3`):** Item-level revenue vs. profit analysis, discount impact, loss-making orders, and low-demand stock isolation.

---

## 🔍 SQL Analysis Highlights

Key analytical queries written to extract actionable business metrics directly from the dataset:

### 1. Late Delivery Rate & Loss by Shipping Mode
```sql
SELECT 
    Shipping_Mode,
    COUNT(Order_Id) AS Total_Orders,
    COUNT(CASE WHEN Delivery_Status = 'Late delivery' THEN 1 END) AS Late_Orders,
    ROUND(100.0 * COUNT(CASE WHEN Delivery_Status = 'Late delivery' THEN 1 END) / COUNT(Order_Id), 2) AS Late_Delivery_Rate_Pct,
    ROUND(SUM(CASE WHEN Order_Profit_Per_Order < 0 THEN Order_Profit_Per_Order ELSE 0 END), 2) AS Total_Loss
FROM dataco_supply_chain
GROUP BY Shipping_Mode
ORDER BY Late_Delivery_Rate_Pct DESC;

2. Loss-Bleeding Products Filter
SQL
SELECT 
    Product_Name,
    COUNT(Order_Id) AS Total_Orders,
    ROUND(SUM(Sales), 2) AS Total_Revenue,
    ROUND(SUM(Order_Profit_Per_Order), 2) AS Net_Profit
FROM dataco_supply_chain
GROUP BY Product_Name
HAVING SUM(Order_Profit_Per_Order) < 0
ORDER BY Net_Profit ASC;

3. Top Delayed Cities Identification
SQL
SELECT 
    Order_City,
    Market,
    COUNT(Order_Id) AS Total_Delayed_Orders
FROM dataco_supply_chain
WHERE Delivery_Status = 'Late delivery'
GROUP BY Order_City, Market
ORDER BY Total_Delayed_Orders DESC
LIMIT 10;

4. Yearly Sales & Profit Trend
SQL
SELECT 
    YEAR(order_date) AS Order_Year,
    COUNT(DISTINCT Order_Id) AS Total_Orders,
    ROUND(SUM(Sales), 2) AS Total_Revenue,
    ROUND(SUM(Order_Profit_Per_Order), 2) AS Total_Profit
FROM dataco_supply_chain
GROUP BY YEAR(order_date)
ORDER BY Order_Year ASC;

5. Average Delivery Variance (Scheduled vs. Actual Days)
SQL
SELECT 
    Shipping_Mode,
    ROUND(AVG(Days_for_shipping_real), 2) AS Avg_Actual_Days,
    ROUND(AVG(Days_for_shipment_scheduled), 2) AS Avg_Scheduled_Days,
    ROUND(AVG(Days_for_shipping_real - Days_for_shipment_scheduled), 2) AS Avg_Delay_Days
FROM dataco_supply_chain
GROUP BY Shipping_Mode
ORDER BY Avg_Delay_Days DESC;

6. Top 3 Profitable Categories per Market 
SQL
WITH CategoryRankings AS (
    SELECT 
        Market,
        Category_Name,
        ROUND(SUM(Sales), 2) AS Category_Revenue,
        ROUND(SUM(Order_Profit_Per_Order), 2) AS Category_Profit,
        DENSE_RANK() OVER(PARTITION BY Market ORDER BY SUM(Order_Profit_Per_Order) DESC) AS Profit_Rank
    FROM dataco_supply_chain
    GROUP BY Market, Category_Name
)
SELECT 
    Market,
    Category_Name,
    Category_Revenue,
    Category_Profit
FROM CategoryRankings
WHERE Profit_Rank <= 3;

7. Monthly Revenue Growth & Running Total 
SQL
WITH MonthlySales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS Year_Month,
        ROUND(SUM(Sales), 2) AS Monthly_Revenue
    FROM dataco_supply_chain
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    Year_Month,
    Monthly_Revenue,
    ROUND(SUM(Monthly_Revenue) OVER(ORDER BY Year_Month), 2) AS Running_Total_Revenue
FROM MonthlySales
ORDER BY Year_Month ASC;

8. High-Risk Regions (Late Delivery Rate > 50% & High Volume)
SQL
SELECT 
    Order_Region,
    COUNT(Order_Id) AS Total_Orders,
    COUNT(CASE WHEN Delivery_Status = 'Late delivery' THEN 1 END) AS Late_Orders,
    ROUND(100.0 * COUNT(CASE WHEN Delivery_Status = 'Late delivery' THEN 1 END) / COUNT(Order_Id), 2) AS Late_Rate_Pct
FROM dataco_supply_chain
GROUP BY Order_Region
HAVING COUNT(Order_Id) > 1000 AND Late_Rate_Pct > 50.0
ORDER BY Late_Rate_Pct DESC;

9. Discount Impact on Profitability by Customer Segment
SQL
SELECT 
    Customer_Segment,
    COUNT(Order_Id) AS Total_Orders,
    ROUND(SUM(Order_Item_Discount_Amount), 2) AS Total_Discount_Given,
    ROUND(AVG(Order_Item_Discount_Rate) * 100, 2) AS Avg_Discount_Rate_Pct,
    ROUND(SUM(Order_Profit_Per_Order), 2) AS Net_Profit
FROM dataco_supply_chain
GROUP BY Customer_Segment
ORDER BY Net_Profit DESC;

10. Repeat Order Frequency & Customer Lifetime Value Analysis
SQL
SELECT 
    Customer_Id,
    Customer_City,
    COUNT(DISTINCT Order_Id) AS Total_Orders_Placed,
    ROUND(SUM(Sales), 2) AS Lifetime_Value,
    ROUND(AVG(Sales), 2) AS Avg_Order_Value
FROM dataco_supply_chain
GROUP BY Customer_Id, Customer_City
HAVING COUNT(DISTINCT Order_Id) > 5
ORDER BY Lifetime_Value DESC
LIMIT 10;




