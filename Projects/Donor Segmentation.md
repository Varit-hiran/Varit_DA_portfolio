# Donor Segmentation
## 📦 Dataset

<img src="https://i.imgur.com/DQEfBYZ.png" width="600"/>

- Simulated CRM donor dataset (~5,000 records)
- Fields include: name, city, state, donation history, engagement score, event participation

🔗 Source:  [Kaggle – CRM Donor Dataset](https://www.kaggle.com/datasets/gaurobsaha/customer-relationship-management-dataset)

---

## 🛠 Tools Used

- **SQLite** – Data storage & querying
- **Power BI** – Dashboard creation & visual storytelling
- [Power BI interactive dashboard](https://autuni-my.sharepoint.com/:u:/g/personal/cxt2452_autuni_ac_nz/EfaZzmPhFl1Eq2zhFVPezTgBhlIcqwHxC79vz8dWGktJRA?e=hqgwgk)

---

## 🥇 Objective & Solution Plan

The marketing manager of a nonprofit organization needs a data-driven approach to identify and target their most valuable donors for upcoming fundraising campaigns.

Instead of relying on basic lists and intuition, the manager aims to use data analytics to create detailed donor profiles based on their behavior, engagement levels, and giving history.

The goal is to identify donors who are most likely to donate again and who contribute the most value over time—those who participate in events, have high engagement scores, or have made multiple and significant donations in the past.

To make strategic decisions, the manager requires clear insights into donor performance, segmentation, and historical contribution patterns.

**Solution Plan**

To help the marketing team identify and target high-value donors, I will use SQL and Power BI to clean, transform, and analyze the donor dataset.

Using SQL, I will extract key metrics such as:

- Recency of last donation
- Total amount donated
- Event participation status
- Engagement score

These metrics will form the basis for segmenting donors and uncovering patterns of behavior that correlate with high-value contributions.

Once the data is structured and insights are extracted, I will build an interactive Power BI dashboard that presents:

- **Key performance indicators** (total donors, average donation, highest individual donation)
- RFM-based donor segments
- **Time-series visualizations** to reveal donation trends by quarter
- **Donor-level table views** for reviewing top contributors and exporting contact lists
- Comparative segment views

This solution will allow the marketing manager to explore and target key donor segments for future campaigns with confidence, improving campaign effectiveness and overall fundraising outcomes.

 

---

## 🔍 SQL execution (SQLite)

Create view table for essential donor fields

```sql
CREATE VIEW 	donors_main_table AS
  SELECT		
  		DonorID AS ID,
  		FirstName, 
  		LastName, 
  		City,
  		LastDonationDate AS Last_do_date,
  		TotalGifts AS Total_do,
  		TotalAmountDonated	AS Total_amount,
  		EventParticipation AS Event_part,
  		EngagementScore AS Donor_score
  FROM			donors;
```

### Question 1: Which donors should be prioritized for re-engagement campaigns?

```sql
SELECT *
FROM donors_main_table
WHERE Event_part = 'Yes'
  AND donor_score > 70
ORDER BY Total_amount DESC
LIMIT 200;

CREATE VIEW high_engaged_donors AS
	SELECT *
  FROM donors_main_table
WHERE Event_part = 'Yes'
  AND donor_score > 70
ORDER BY Total_amount DESC
LIMIT 200;
```

Query result:

![image.png](https://i.imgur.com/fKMpEcx.png[/img])

> This query displays the top 200 donors who have:
- participated in events
- An engagement score greater than 70
> 

The results is sorted by total amount donated in descending order to prioritise high-value, high-engagement donors for next campaigns.

### Question 2: Do donors who participate in events show different engagement and donation behavior compared to those who don’t?

```sql
SELECT 	event_part,
				ROUND(AVG(donor_score), 2) 	AS avg_engangement,
        ROUND(AVG(total_amount), 2) 	AS AVG_total
FROM donors_main_table
GROUP BY event_part;
```

Query result:

![image.png](https://i.imgur.com/NMvujpe.png[/img])

The results contrary to expectations where donors who participated in events do not significantly higher engagement or donation value compared to those who haven’t  

Therefore, the event participation alone may not be a reliable indicator of donor value.

### Question 3: Are there donors who donated a lot but have low participation score?

Building upon Question 2, this analysis aims to uncover high-value donors who have low participation score, to identify potential re-engagement opportunities beyond participation.

```sql
SELECT *
FROM donors_main_table
WHERE donor_score < 30
	AND total_amount > (
    SELECT AVG(total_amount) * 1.5
    FROM donors_main_table
    )
ORDER BY total_amount
;
```

Query result:

![image.png](https://i.imgur.com/TA8K09Z.png[/img])

Despite the low engagement of donors who have financially outperformed the average donor by more than 1.5x.

This segment have been overlooked but presents a potential opportunity for personalised recognition or retention.

---

## 📈RFM analysis by using SQL execution

Based on the SQL execution results, the RFM analysis becomes a more suitable and data-driven approach to segmenting donors, enabling to identify high-value individuals regardless of their participation and tailor engagement strategies accordingly

```sql
WITH rfm_metrics AS(
SELECT *,
-- Recency Score
CASE
	WHEN (SELECT MAX(Last_do_date) FROM donors_main_table) - Last_do_date <= 30 THEN 5
  WHEN (SELECT MAX(Last_do_date) FROM donors_main_table) - Last_do_date <= 90 THEN 4
  WHEN (SELECT MAX(Last_do_date) FROM donors_main_table) - Last_do_date <= 180 THEN 3
  WHEN (SELECT MAX(Last_do_date) FROM donors_main_table) - Last_do_date <= 365 THEN 2
  ELSE 1
END AS R_score,

-- Monetary Score
Case
	WHEN total_amount > (
    SELECT AVG(total_amount) * 1.5
    FROM donors_main_table
    )
  	THEN 5
  WHEN total_amount > (
    SELECT AVG(total_amount) * 1.25
    FROM donors_main_table
    )
  	THEN 4
  WHEN total_amount > (
    SELECT AVG(total_amount) * 1
    FROM donors_main_table
    )
  	THEN 3
  WHEN total_amount > (
    SELECT AVG(total_amount) * 0.75
    FROM donors_main_table
    )
  	THEN 2
 	ELSE 1
 END AS M_score,

-- Frequency Score
CASE
	WHEN Total_do >= (SELECT MAX(Total_do) FROM donors_main_table) * 0.8 THEN 5
  WHEN Total_do >= (SELECT MAX(Total_do) FROM donors_main_table) * 0.6 THEN 4
  WHEN Total_do >= (SELECT MAX(Total_do) FROM donors_main_table) * 0.4 THEN 3
  WHEN Total_do >= (SELECT MAX(Total_do) FROM donors_main_table) * 0.2 THEN 2
  ELSE 1
END AS F_score
FROM donors_main_table
),

--RFM segmentation
rfm_segmentation AS(
SELECT *,
	CASE
    WHEN R_score = 5 AND F_score >= 4 AND M_score >= 4 THEN 'Champion'
    WHEN R_score >= 4 AND F_score >= 3 THEN 'Loyal'
    WHEN R_score >= 3 AND F_score >= 2 THEN 'Potential Loyalist'
    WHEN R_score = 5 AND F_score = 1 THEN 'New Customer'
    WHEN R_score <= 2 AND F_score >= 4 AND M_score >= 4 THEN 'Cannot Lose'
    ELSE 'Others'
  END AS rfm_segment
FROM rfm_metrics
)

SELECT *
FROM rfm_segmentation
```

Query result:

![image.png](https://i.imgur.com/KqC2o8W.png[/img])

The RFM analysis is the classification that segments donors into meaningful classification including:

- Champions: Frequent, recent, and high-value donors
- Loyal: Consistent donors with good frequency
- Potential loyalist: positive signals of becoming loyal supporters
- New customers: recently acquired donors with low history
- Cannot lose: Formerly high-value donors who became inactive

---

## 📊 Data Visualisation

The Power BI tool were used to turn data into interactive dashboard for further analysis

![image.png](https://i.imgur.com/mysY6Ed.png[/img])

This dashboard showcases donor segmentation based on behavioral analysis using RFM metrics which provides key insights for personalised fundraising strategies

💡 Key Insights:

- **💰 Total Donation:** $61.75M from 5,000 donors
- **📈 Trend Over Time:** Donation peaked during 2023–2024, then gradually declined
- **📌 Slicer Panel:** Enables filtering by segment to analyze behavior across categories such as *Loyal*, *Cannot Lose*, and *New Customers*

![image.png](https://i.imgur.com/Mwi69Mf.png[/img])

This figure shows filtered view of donors categorised under the **“Cannot lose”** segment, defined as high donation frequency and monetary value but low recent engagement.

- **Total Number of Donors:** 251
- **Total Donation Value:** $3.22M
- **Average Donation per Donor:** $12.84K
- **Top Donation from This Segment:** $24.96K

Despite representing only a small portion of the total donor base, **"Cannot Lose"** donors contributed over 3 millions, emphasising their financial significance. The donation trend over time shows steady contributions from this segment through 2022 and 2023.

 

![image.png](https://i.imgur.com/cq4pZW4.png[/img])

This figure shows filtered view of donors categorised under the **“Potential Loyalist”** segment, defined as strong donation history and moderate recency

- Total donation from this segment exceeds most other segments, reflecting high monetary contribution potential
- The average donation per donor remains consistently strong
- The recency consistent suggesting stable commitment

This segment present an opportunity for elevation through specific strategy to build long-term loyalty

 

---

## 🎯 Conclusion

Based on dashboard and data, it becomes evident that:

- “Cannot lose” donors contribute significantly to the total donation amount. However, they haven’t donated recently. This highlights the potential of re-engagement strategies to prevent from lost high-value group
- Conversely,  “Potential Loyalist” represent ideal candidates for loyalty-building campaigns.

This insights suggests that the marketing team can prioritise donor targeting based on strategic directions through RFM segmentation.

By segmenting donor based on either monetary or recency, the marketing team can align their strategy with campaign goals—whether to reclaim lost revenue from valuable but inactive donors, or to nurture promising relationships with newly engaged supporters.

For example, marketers could select the top 200 for each segment to build connection and encourage future giving.
