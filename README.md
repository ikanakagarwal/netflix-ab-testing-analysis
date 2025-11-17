# 📺 Netflix A/B Testing & User Engagement Analysis (Python + SQL + AWS)

This project analyzes how changes in Netflix’s UI design impact user engagement using a simulated A/B test.  
I generated realistic synthetic user/session datasets using Python, stored them on AWS S3, modeled them in AWS RDS MySQL, and performed SQL-based analysis to measure how UI Variant B performs compared to Variant A.

The analysis reveals that **Variant B significantly improves watch completion (+18%) and reduces bounce rate (–12%)**, especially for mobile users and during night-time viewing.

---

# 🎯 1. Problem Statement

Netflix frequently runs A/B experiments to improve the user experience.  
This project simulates an A/B test where two different UI designs (A & B) influence:

- Watch completion  
- Bounce rate  
- Session length  
- Device-level behavior  
- Genre preferences  
- Time-of-day engagement  

The goal is to identify whether UI Variant **B** leads to improved engagement and identify behavioral patterns across segments.

---

# 🔍 2. Objectives

- Generate realistic, production-like datasets using Python  
- Build a normalized relational schema (users & sessions)  
- Store data in AWS S3 and model it in AWS RDS (MySQL)  
- Write advanced SQL queries for A/B testing and segmentation  
- Extract actionable product insights  
- Document a professional, end-to-end analytics workflow  

---
# 3.Synthetic Dataset Generation

A synthetic dataset of **~1,000 users** and **5,000 sessions** was created using Python (Pandas, NumPy).  
 ### 🔗 **Datasets**
- **users.csv** → [Link to file](#)
- **sessions.csv** → [Link to file](#)

### 🔗 **Python Code**
- **generate_data.ipynb** → [Link to file](#)

---

# ☁️ 5. AWS Integration — S3 → RDS → SQL

To simulate a real production pipeline:

### Step 1 — Upload to AWS S3  
- Created an S3 bucket  
- Uploaded both CSV datasets  
- Used boto3 to verify file access  

### Step 2 — Create AWS RDS (MySQL) Instance  
- Provisioned MySQL 8.0 on RDS  
- Configured inbound rules and security groups  
- Created database & tables schema  

### Step 3 — Connect Python → S3 → RDS  
Using boto3 + mysql-connector-python:  
- Downloaded CSVs from S3  
- Inserted rows into RDS MySQL  
- Validated schema  

### Step 4 — SQL Analysis  
After connecting through Workbench, all SQL queries were executed against normalized tables.
 
---
# SQL Questions, Queries, Outputs & Insights

Below are the 10 key analytical questions answered in this project along with their SQL queries, representative outputs, and insights derived.

---

## **Q1. Which UI Variant Performed Better (A vs B)?**
```sql
SELECT `group`,
       ROUND(AVG(session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(watched), 2) AS watch_rate_pct,
       ROUND(100 * SUM(watch_completion) / SUM(watched), 2) AS completion_rate_pct
FROM sessions
GROUP BY `group`;
```

| group | avg_session_length | watch_rate_pct | completion_rate_pct |
|-------|---------------------|----------------|----------------------|
| A     | 11.42               | 67.10%         | 52.80%              |
| B     | 13.54               | 74.90%         | 62.20%              |


### Insight
UI Variant B performs better across all engagement metrics (+18% completion uplift).

## Q2. Which device type shows higher engagement?

 ```sql
 SELECT device_type,
       ROUND(AVG(session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(watch_completion), 2) AS completion_rate_pct
FROM sessions
GROUP BY device_type
ORDER BY avg_session_length DESC;
```

device_type | avg_session_length | completion_rate_pct|
------------------------------------------------------|
TV          | 15.12              | 61.40%             |
Laptop      | 12.32              | 57.85%             |
Mobile      | 9.07               | 49.10%             |



### Insight
TV shows the highest engagement with longer session lengths and better completion rates.  
Mobile users show the lowest engagement, indicating potential UI friction on smaller screens.

 
## Q3. What time of day has the highest engagement?

### SQL Query
```sql
SELECT time_of_day,
       ROUND(AVG(session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(watch_completion), 2) AS completion_rate_pct
FROM sessions
GROUP BY time_of_day
ORDER BY avg_session_length DESC;
 ```
 time_of_day | avg_session_length | completion_rate_pct
-------------------------------------------------------
Night       | 14.10               | 63.20%
Evening     | 13.20               | 60.45%
Afternoon   | 10.45               | 55.10%
Morning     | 8.42                | 48.20%
Insight
Night and Evening have the highest engagement, with longer session durations and higher completion rates.
Morning has the weakest engagement, likely due to shorter available watch time.


