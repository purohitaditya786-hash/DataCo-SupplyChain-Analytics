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

##  SQL Analysis Highlights
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

SELECT 
    Product_Name,
    COUNT(Order_Id) AS Total_Orders,
    ROUND(SUM(Sales), 2) AS Total_Revenue,
    ROUND(SUM(Order_Profit_Per_Order), 2) AS Net_Profit
FROM dataco_supply_chain
GROUP BY Product_Name
HAVING SUM(Order_Profit_Per_Order) < 0
ORDER BY Net_Profit ASC;

SELECT 
    Order_City,
    Market,
    COUNT(Order_Id) AS Total_Delayed_Orders
FROM dataco_supply_chain
WHERE Delivery_Status = 'Late delivery'
GROUP BY Order_City, Market
ORDER BY Total_Delayed_Orders DESC
LIMIT 10;

SELECT 
    YEAR(order_date) AS Order_Year,
    COUNT(DISTINCT Order_Id) AS Total_Orders,
    ROUND(SUM(Sales), 2) AS Total_Revenue,
    ROUND(SUM(Order_Profit_Per_Order), 2) AS Total_Profit
FROM dataco_supply_chain
GROUP BY YEAR(order_date)
ORDER BY Order_Year ASC;

##  Dashboard Preview & Page Breakdown

Below are the executive previews of the 3-page interactive Power BI report:

### 1. Executive Overview (Page 1)
High-level operational metrics covering global sales distribution, order fulfillment status, and regional revenue growth.
![Executive Overview](Screenshot%20\(291\).jpg)

### 2. Logistics & Operations (Page 2)
Deep dive into shipping delays, late delivery rate trends, top delayed cities, and total shipping loss metrics.
![Logistics & Operations](Screenshot%20\(292\).png)

### 3. Product Performance & Profitability (Page 3)
Item-level profit analysis, discount impact tracking, zero-demand stock isolation, and loss-making order identification.
![Product Performance](Screenshot%20\(293\).png)
