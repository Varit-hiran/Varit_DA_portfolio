# Improving Retail Sales Forecast and Markdown Strategy Using Data Analytics
## 📦 Dataset

The dataset contains historical retail sales data from 45  stores , each with various departments. It includes:

- **Weekly sales figures**
- **Store-level variables** (e.g., markdowns, economic indicators, holidays)
- **Store type and size classifications**

Promotional markdowns were applied ahead of major holidays like **Labor Day, Father’s Day, Mother’s Day, and Christmas**, and these **holiday weeks are weighted 5× higher** in evaluations due to their business impact.

The dataset is structured into three tabs:

1. **Variables Dataset** – Economic and promotional indicators by store and week
2. **Weekly Sales Data** – Sales per department per store per week
3. **Store Type & Size** – Metadata about each store

---

## 🛠 Tools used

- **Microsoft Excel** – Data cleansing, Data analysis
- Power BI – Interactive Dashboard, Corrplot (R script)

---

## 🎯 Objectives

- To analyse historical weekly sales data across 45 retail stores and identify key patterns in holiday and non-holiday periods
- To evaluate the impact of promotional markdowns and economic variables on total sales
- To visualise sales performance using interactive Power BI dashboards
- To apply linear regression for the relationship between markdown strategies and weekly sales
- To generate actionable insights that strengthen data-driven decision making for future promotional planning and store resource allocation

---

## 🧹 Data cleansing

“Variables Dataset” and “Weekly Sales Data” tables were merged by using “store” and “date” as primary keys

- JOIN data using Excel Power Query to combine into 1 main table
- Replace NA value with 0 to match with the assumption that NA value indicates no promotional activity or no recorded event during that week

---

## 📊 Descriptive analysis

![image.png](attachment:de6fdcc3-5a8c-4124-a0fa-2cd4fc23e941:image.png)

1). Main dashboard view showing overall retail performance across all stores and all time periods

![image.png](attachment:26f1da9e-40e1-4023-9de9-ba2a30848492:image.png)

2). Filtered dashboard view that show data only for **holiday weeks**

![image.png](attachment:4687d0be-8dc2-4827-914c-e7cd9c355beb:image.png)

3). Filtered dashboard view that show data only for non-**holiday weeks**

![Screenshot 2025-05-14 003232.png](attachment:a0a54b44-237c-4f24-a02c-a78066073d7d:Screenshot_2025-05-14_003232.png)

4). Least performing stores view that replaces the bar chart to highlight the **3 least performing stores by type**.

---

## 📌 Key Insights from descriptive analysis

**Total Sales** reached **$6.74B** with a **5.04% YTD growth rate in 2021**

- **Sales Trend by Month** illustrates seasonal peaks in March, June, and December
- **Store type A** accounts for the highest proportion of stores and contributes the most to sales
- Least-performing stores were from **Store Type C and B**, with sales between **$43M–$74M**
- **Holiday weeks** contribute to **over 90%** of total annual sales, underscoring the importance of promotional markdowns around major events
- **Non-holiday sales** are significantly lower, suggesting an opportunity to **boost non-holiday performance**

---

## 🔗Correlation analysis

In this section, the R Script Editor in PowerBI was implemented to create correlation heat map, demonstrating the importance of each variable that impact total sales

```r
# The following code to create a dataframe and remove duplicated rows is always executed and acts as a preamble for your script: 

# dataset <- data.frame(undefined)
# dataset <- unique(dataset)

# Paste or type your script code here:
require("corrplot")
library(corrplot)

colnames(dataset) <- c("Total Sales", "Temperature", "Unemployment", "Mark Down 1", "Mark Down 2", "Mark Down 3", "Mark Down 4", "Mark Down 5", "Fuel Price", "CPI")

M <- cor(dataset)
corrplot(M, method = "color", tl.cex=1.5, tl.srt = 45, tl.col = "#010a4f")
```

Results:

![image.png](attachment:70e75da0-3ccd-48d7-8448-c5a108157698:image.png)

From above results, it can be seen that:

- Mark Down 1-5 has moderate positive correlation with **Total Sales**
- Unemployment has a slightly negative correlation with **Total Sales**, which align with the prediction that when unemployment rate increase, consumer tend to spend lower
- CPI and Fuel Price contribute minimal impact to the **Total Sales** in this dataset
- Temperature shows weak correlation to the **Total Sales**, which means seasonality is not the main driver

---

## 📢 Linear Regression Analysis

In this section, Linear Regression was conducted in the Excel tool to find the correlation between variables and Total Sales

![image.png](attachment:f202e812-43f6-42e3-9bc9-344919a2ecde:image.png)

- R Square results indicates that this model may only explain the relationship of variables to Total Sales only **0.007%**
- From P-Value results:
    - every variables has impact to the total sales except for Mark Down 4 that has least impact to Total Sales
- From Coefficients results:
    - Temperature, Mark Down 1, Mark Down 2, Mark Down 3 and Mark Down 5 have positive impact to Total Sales
    - Fuel Price, Mark Down 4, CPI and Unemployment rate have negative impact to Total Sales

### Conclusion for the Linear Regression analysis

- Temperature = low positive impact and low significant
- Fuel Price = High negative impact and significant
- Mark Down 1 = Low positive impact but significant
- Mark Down 2 = Low positive impact and low significance
- Mark Down 3 = High positive impact and high significance
- Mark Down 4 = Low negative impact and low significance
- Mark Down 5 = Low positive impact but significant
- CPI rate = Moderate negative impact and significant
- Unemployment rate = High negative impact and high significance

![image.png](attachment:5d3994ad-d819-43f0-93a3-71eb47408c8e:image.png)

This visualisation illustrates a predicted sales value for Holiday Week for store 20 (Most contributed to the Total Sales) using Slicer tool (Power BI) to exemplify the correlation between Total Sales and variables

Moreover, the predictions were made for 3 stores to represent each store type by comparing Total Sales of each week (holiday period) in November 

![image.png](attachment:7be1ea08-ae50-4c96-95b0-5b14bfada6a1:image.png)

![image.png](attachment:79c77622-0dba-4c00-b9d3-721209ea27a8:image.png)

![image.png](attachment:fff2c331-58ee-44d9-bea4-46ac075685ef:image.png)

### 📌 Key insights from the visualisation:

- Store type A contributed the most holiday Total Sales (30.15 M) compared to type B (23.19 M) and type C (6.32 M)
- Although, the first 3 week of 2021 moderately reduce from previous years, but week 4th maintained highest sales compared to previous years

---

## 🔮 What can be done further ?

Based on findings from this report, these are key recommendations to increase business value and improve decision-making:

1. Holiday Mark Down planning
    1. Since holiday week contribute over 90% of total sales, there are key mark downs that show positive correlation with revenue. Meaning that company should focus on particular Mark Down strategies in holiday periods
    2. The detailedly segmentation of high-performing vs low-performing stores can be done to improve resource allocation for specific stores
2. Regression integration into strategy planning
    1. The coefficients can guide the Mark Down prioritisation to strengthen the usage of Mark Down for specific strategies
    2. The significance of economic variables indicates the impact of each variable to handle the future possibilites
