# 🚚 Supply Chain Logistics Analysis - SQL Project  

## 📌 Overview  
This **SQL-based Supply Chain Logistics Analysis Project** aims to optimize **order management, warehouse utilization, shipping efficiency, freight cost analysis, and carrier performance**.  
The project provides insights into:  
✅ **Total orders and weight analysis by plant**  
✅ **Product demand trends**  
✅ **Warehouse capacity and order volume management**  
✅ **Freight cost and shipping efficiency analysis**  
✅ **Carrier performance evaluation**  

Using **SQL queries**, this project allows businesses to **improve supply chain operations, reduce costs, and enhance delivery efficiency**.  

---

## 📂 Dataset Details  
This project uses **multiple datasets** related to supply chain logistics:  

1. **OrderList.csv**  
   - Contains **order transactions** with details like product, plant, weight, shipping, and cost.  
   - **Key columns:** `Order_ID`, `Plant_Code`, `Product_ID`, `Unit_Quantity`, `Weight`, `Origin_Port`, `Destination_Port`, `Carrier`, `Ship_Late_Day_count`  

2. **FreightRates.csv**  
   - Defines **shipping costs, weight limits, and carrier-specific rates**.  
   - **Key columns:** `orig_port_cd`, `dest_port_cd`, `Carrier`, `rate`, `minm_wgh_qty`, `max_wgh_qty`, `minimum_cost`  

3. **PlantPorts.csv**  
   - Maps **plants to valid shipping ports**.  
   - **Key columns:** `Plant_Code`, `Port`  

4. **ProductsPerPlant.csv**  
   - Shows **product distribution across plants**.  
   - **Key columns:** `Plant_Code`, `Product_ID`, `Product_Name`  

5. **WarehouseCapacity.csv**  
   - Defines **warehouse capacity limits** for order processing.  
   - **Key columns:** `Plant_ID`, `Daily_Capacity`  

6. **WarehouseCosts.csv**  
   - Contains **storage cost per unit for warehouses**.  
   - **Key columns:** `WH`, `Cost_unit`  

7. **VmiCustomers.csv**  
   - Lists **customers in Vendor-Managed Inventory (VMI) programs**.  
   - **Key columns:** `Customer_ID`, `Customer_Name`  

---

## 🏗️ Data Processing & Key SQL Queries  

### 1️⃣ **Total Orders and Weight by Plant**  
✅ **Identifies plants with the highest order volume** and **total shipping weight**.  

```sql
SELECT 
    o.Plant_Code,
    COUNT(*) AS Total_Orders,
    SUM(o.Unit_quantity) AS Total_Units,
    SUM(o.Weight) AS Total_Weight
FROM OrderList o
GROUP BY o.Plant_Code
ORDER BY Total_Orders DESC;
```

🔹 **Findings:**  
- **Plant 03 has the highest order volume**, indicating a **high demand center**.  
- **Weight distribution across plants** helps in **balancing logistics load efficiently**.  

---

### 2️⃣ **Top 5 Most Ordered Products**  
✅ **Determines the highest demand products by units and order count.**  

```sql
SELECT TOP 5 Product_ID, COUNT(*) AS OrderCount
FROM OrderList
GROUP BY Product_ID
ORDER BY OrderCount DESC;
```

🔹 **Findings:**  
- **Product ID 1684862 is the most ordered item**, showing high demand.  
- **Insights help in inventory planning** and **warehouse stocking strategies**.  

---

### 3️⃣ **Warehouse Utilization Analysis**  
✅ **Checks if warehouses are exceeding their daily capacity limits.**  

```sql
SELECT
  o.Plant_Code,
  CAST(o.Order_Date AS DATE) AS OrderDay,
  SUM(o.Unit_quantity) AS TotalUnits,
  wc.Daily_Capacity,
  CASE 
    WHEN SUM(o.Unit_quantity) > wc.Daily_Capacity THEN 'Over Capacity' 
    ELSE 'Under Capacity' 
  END AS CapacityStatus
FROM OrderList o
JOIN WarehouseCapacity wc
  ON o.Plant_Code = wc.Plant_ID
GROUP BY o.Plant_Code, CAST(o.Order_Date AS DATE), wc.Daily_Capacity;
```

🔹 **Findings:**  
- **Some warehouses exceed daily capacity**, leading to **logistics bottlenecks**.  
- **Optimizing distribution** can **reduce warehouse overload issues**.  

---

### 4️⃣ **On-Time Shipping Performance by Carrier**  
✅ **Evaluates carrier reliability by calculating on-time shipping percentages.**  

```sql
SELECT Carrier,
	CAST(SUM(CASE WHEN Ship_Late_Day_count = 0 THEN 1 ELSE 0 END) * 100.00
    / COUNT(*) AS DECIMAL (10, 2)) AS OnTimeRate
FROM OrderList
GROUP BY Carrier
ORDER BY OnTimeRate DESC;
```

🔹 **Findings:**  
- **Carrier V44_3 has the highest on-time shipping rate**, making it the most **reliable logistics partner**.  
- **Carrier V444_0 struggles with delays**, requiring **further investigation into operational inefficiencies**.  

---

### 5️⃣ **Orders Exceeding Weight Limits**  
✅ **Identifies shipments that exceed freight weight restrictions.**  

```sql
WITH shipping_table AS(
	SELECT o.Order_ID, o.Origin_Port, o.Destination_Port, o.Weight, f.max_wgh_qty,
		   CASE 
			   WHEN o.Weight > f.max_wgh_qty THEN 'EXCEEDED'
			   ELSE 'WITHIN LIMIT'
		   END AS Shipping_Status
	FROM OrderList o
	JOIN FreightRates f ON o.Origin_Port = f.orig_port_cd 
						AND o.Destination_Port = f.dest_port_cd
)
SELECT DISTINCT *
FROM shipping_table
WHERE Shipping_Status = 'EXCEEDED'
ORDER BY [Weight] DESC;
```

🔹 **Findings:**  
- **Several orders exceed shipping limits**, leading to **additional costs or shipment delays**.  
- **Optimizing weight distribution** across orders can **reduce cost overruns**.  

---

### 6️⃣ **Logistics Cost Calculation**  
✅ **Determines total logistics cost per order, including freight and warehouse storage costs.**  

```sql
SELECT
    o.Order_ID,
    -- Freight Cost
    CASE 
        WHEN o.Weight < f.minm_wgh_qty THEN f.minimum_cost
        ELSE ROUND(o.Weight * f.rate, 2)
    END AS Freight_Cost,

    -- Warehouse Cost
    ROUND(o.Unit_quantity * wc.Cost_unit, 2) AS Warehouse_Cost,

    -- Summation
    CASE 
        WHEN o.Weight < f.minm_wgh_qty THEN f.minimum_cost
        ELSE ROUND(o.Weight * f.rate, 2)
    END 
    + ROUND((o.Unit_quantity * wc.Cost_unit), 2) AS Total_Logistics_Cost
FROM OrderList AS o
INNER JOIN FreightRates AS f
    ON  o.Carrier = f.Carrier
    AND o.Origin_Port = f.orig_port_cd
    AND o.Destination_Port = f.dest_port_cd
    AND o.Weight BETWEEN f.minm_wgh_qty AND f.max_wgh_qty
INNER JOIN WarehouseCosts AS wc
    ON O.Plant_Code = wc.WH;
```

🔹 **Findings:**  
- **Freight costs contribute significantly** to total logistics expenses.  
- **Optimizing warehouse selection and carrier choice** can **reduce total supply chain costs**.  

---

## 🚀 Future Improvements  
🔹 **Incorporate predictive analytics** to **forecast demand trends and optimize inventory**.  
🔹 **Use machine learning models** to **predict late shipments and reduce delays**.  
🔹 **Integrate real-time tracking** to **monitor warehouse capacity and freight utilization**.  

---

## 🤝 Connect with Me  
📧 **Email:** eden.vietnguyen@gmail.com  
🔗 **LinkedIn:** [www.linkedin.com/in/eden-nguyen](https://www.linkedin.com/in/eden-nguyen)  
🌐 **Portfolio Website:** [eden-nguyen.vercel.app](https://eden-nguyen.vercel.app/)  

### 🔥 **Enhancements in This README:**
✅ **Well-structured SQL queries and insights**  
✅ **Explained trends and findings from each analysis**  
✅ **Recommended future improvements**  

Let me know if you need **any modifications or additional details!** 🚀
