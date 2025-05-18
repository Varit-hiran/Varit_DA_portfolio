# Grocery Inventory Optimisation
## 📦 Dataset

![image.png](https://i.imgur.com/sBRgbua.png[/img])

- Simulated grocery store dataset contains 990 products across various categories

🔗 Source: [Kaggle – Grocery Inventory and Sales Dataset](https://www.kaggle.com/datasets/salahuddinahmedshuvo/grocery-inventory-and-sales-dataset?)

## 1. Data cleaning and preprocessing

- 3 columns of date are in VARCHAR format, using ‘MODIFY COLUMN’ to transform to DATE format

```sql
ALTER TABLE grocery_inventory
MODIFY COLUMN Date_Received DATE,
MODIFY COLUMN Last_Order_Date DATE,
MODIFY COLUMN Expiration_Date DATE;
```

- Unit Price is in VARCHAR format, using ‘SET’ and ‘Replace’ to remove ‘$’ in the dataset
- Transform from VARCHAR to Decimal using ‘MODIFY COLUMN’ and set at 2 digits

```sql
UPDATE grocery_inventory
SET Unit_Price = REPLACE(Unit_Price, '$', '');
```

```sql
ALTER TABLE grocery_inventory
MODIFY COLUMN Unit_Price DECIMAL(10,2);
```

---

## 2. Data Exploration

### 📦 Stock & Reorder

```sql
SELECT *
FROM 		grocery_inventory
WHERE 	  Stock_quantity < Reorder_level
AND			  Status = 'Active';
```

- Explore which product stock level lower than reorder level, to immediately reorder

```sql
SELECT 	*
FROM 		grocery_inventory
WHERE 	Sales_Volume > (
  SELECT AVG(Sales_Volume)
  FROM grocery_inventory
)
AND status = 'Active';
```

- Explore products with high sales volume and almost stockout, to highlight profitable items that need urgent restocking

```sql
SELECT
		 Product_Name,
    Stock_Quantity,
    Reorder_Level,
    Sales_Volume,
    Status
FROM			grocery_inventory
WHERE			Status = 'Backordered';
```

- Explore backordered products that needs to be immediately restock to meet customer demand

```sql
SELECT 
  COUNT(*) AS Backorder_Count,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM grocery_inventory), 2) AS Backorder_Percent
FROM grocery_inventory
WHERE Status = 'Backordered';
```

- Count backordered products and find percentage of backorders from overall products

```sql
SELECT 
    Product_Name,
    Stock_Quantity,
    Reorder_Level,
    Sales_Volume,
    Inventory_Turnover_Rate,
    Status
FROM grocery_inventory
WHERE 
    Stock_Quantity > (Reorder_Level * 2) -- Overstock 2x more
    AND Inventory_Turnover_Rate < (
  SELECT AVG(Inventory_Turnover_Rate) -- Turnover rate lower than AVG
  FROM grocery_inventory
)
    AND Status = 'Active'
ORDER BY Inventory_Turnover_Rate ASC;
```

- Explore overstock products compare to reorder level and turnover rate is low than average, to detect items that unnecessarily holding in inventory.

### ⏳ Expiry & Discontinuation

```sql
SELECT 		Product_Name,
    			Stock_Quantity,
    			Date_Received,
    			Expiration_Date,
					DATEDIFF(Expiration_Date, Date_Received) AS Product_life
FROM			grocery_inventory
WHERE			DATEDIFF(Expiration_Date, Date_Received) <= 0
ORDER BY	Product_life DESC;
```

![image.png](https://i.imgur.com/MfCiA0o.png[/img])

- There are lots of products where expiry date on or before the date they were received, indicating that error in data entry or quality in the supply chain process
- This may specialise the promotion for specific products that shelf life is low

```sql
SELECT 
		 Product_Name,
    Stock_Quantity,
    Reorder_Level,
    Sales_Volume,
    Status
FROM			grocery_inventory
WHERE			Stock_quantity > 0
AND				Status = 'Discontinued';
```

- Explore the discontinued items that still left in stock, labelling items as obsolete for removal or clearance

---

## Data Summarisation

From above queries, it can be utilised by:

- Reducing issues related to both stockouts and overstocking to strengthen inventory management
- Using reorder levels and actual sales data to support decision-making
- Aligning warehouse space and resources allocation with product turnover rates
- Adjusting sales and promotional strategy particularly for items with low turnover rates but high stock levels
- Generating automated system that automatically reorder through machine learning systems
