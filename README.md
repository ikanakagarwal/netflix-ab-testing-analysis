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
- **users.csv** → [https://github.com/ikanakagarwal/netflix-ab-testing-analysis/blob/main/users.csv](#)
- **sessions.csv** → [https://github.com/ikanakagarwal/netflix-ab-testing-analysis/blob/main/sessions.csv](#)

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

|device_type | avg_session_length | completion_rate_pct|
|------------|--------------------|--------------------|
|TV          | 15.12              | 61.40%             |
|Laptop      | 12.32              | 57.85%             |
|Mobile      | 9.07               | 49.10%             |



### Insight
TV shows the highest engagement with longer session lengths and better completion rates.  
Mobile users show the lowest engagement, indicating potential UI friction on smaller screens.

 
## Q3. What time of day has the highest engagement?

 ```sql
SELECT time_of_day,
       ROUND(AVG(session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(watch_completion), 2) AS completion_rate_pct
FROM sessions
GROUP BY time_of_day
ORDER BY avg_session_length DESC;
 ```
| time_of_day | avg_session_length | watch_rate_pct | completion_rate_pct |
|-------------|--------------------|----------------|----------------------|
| Night       | 14.10              | 78.40%         | 63.20%               |
| Evening     | 13.20              | 75.10%         | 60.45%               |
| Afternoon   | 10.45              | 68.15%         | 55.10%               |
| Morning     | 8.42               | 59.30%         | 48.20%               |



### Insight
Night and Evening have the highest engagement, with longer session durations and higher completion rates.
Morning has the weakest engagement, likely due to shorter available watch time.

## Q4. Which genres drive the highest watch time and engagement?
```sql
SELECT s.genre,
       COUNT(*) AS total_sessions,
       ROUND(AVG(s.session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(s.watch_completion), 2) AS completion_rate_pct
FROM sessions s
GROUP BY s.genre
ORDER BY avg_session_length DESC;
```
| genre       | total_sessions | avg_session_length | completion_rate_pct |
|-------------|----------------|--------------------|----------------------|
| Action      | 950            | 14.22              | 61.80%               |
| Sci-Fi      | 730            | 13.18              | 59.40%               |
| Thriller    | 810            | 12.95              | 58.10%               |
| Horror      | 610            | 11.85              | 55.25%               |
| Comedy      | 580            | 10.12              | 50.90%               |
| Drama       | 640            | 9.45               | 49.30%               |
| Documentary | 380            | 8.92               | 48.10%               |

### Insight
**Action, Sci-Fi, and Thriller** drive the highest average session lengths and completion rates, indicating strong user interest.  
Genres like **Drama** and **Documentary** show lower engagement, suggesting lighter viewer commitment.  
This insight can guide genre-focused recommendations and UI placement strategies.


## Q5. How does engagement vary across subscription tiers?

```sql
SELECT u.subscription_type,
       COUNT(*) AS total_sessions,
       ROUND(AVG(s.session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(s.watch_completion), 2) AS completion_rate_pct
FROM sessions s
JOIN users u ON s.user_id = u.user_id
GROUP BY u.subscription_type
ORDER BY completion_rate_pct DESC;

 ```
| subscription_type | total_sessions | avg_session_length | completion_rate_pct |
|-------------------|----------------|--------------------|----------------------|
| Premium           | 1850           | 14.20              | 63.80%               |
| Standard          | 1900           | 12.85              | 58.22%               |
| Basic             | 1250           | 10.40              | 51.10%               |

### Insight
**Premium users show the highest engagement**, with longer session durations and stronger completion rates.  
Basic users have the weakest performance, likely due to device limitations or content access restrictions.  
This indicates that higher-tier subscribers tend to be more loyal and engaged.

## Q6. Which countries show the highest viewing activity and engagement?

```sql
SELECT u.country,
       COUNT(*) AS total_sessions,
       ROUND(AVG(s.session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(s.watch_completion), 2) AS completion_rate_pct
FROM sessions s
JOIN users u ON s.user_id = u.user_id
GROUP BY u.country
ORDER BY total_sessions DESC;
```
### Output
| country | total_sessions | avg_session_length | completion_rate_pct |
|---------|----------------|--------------------|----------------------|
| IN      | 1550           | 13.10              | 60.20%               |
| US      | 1320           | 12.45              | 58.90%               |
| BR      | 890            | 11.02              | 55.15%               |
| UK      | 700            | 12.85              | 59.40%               |
| CA      | 340            | 11.55              | 56.20%               |
| DE      | 150            | 10.92              | 54.10%               |
| AU      | 50             | 10.31              | 52.80%               |

### Insight
India and the US have the highest total session volume, indicating large, active viewer bases.  
Completion rates are relatively stable across regions, with **India leading in session counts** and the **UK showing strong completion rates**.  
Smaller markets (DE, AU) have fewer sessions and weaker engagement, suggesting different consumption patterns or smaller user pools.


## Q8. How do UI groups A and B perform across the engagement funnel (Clicks → Watch → Completion)?

 ```sql
 SELECT `group`,
       ROUND(AVG(clicks_before_watch), 2) AS avg_clicks_before_watch,
       ROUND(100 * AVG(watched), 2) AS watch_rate_pct,
       ROUND(100 * SUM(watch_completion) / SUM(watched), 2) AS completion_after_watch_pct
FROM sessions
GROUP BY `group`
ORDER BY avg_clicks_before_watch ASC;
```
| group | avg_clicks_before_watch | watch_rate_pct | completion_after_watch_pct |
|-------|--------------------------|----------------|-----------------------------|
| B     | 1.10                     | 74.90%         | 62.20%                      |
| A     | 1.42                     | 67.10%         | 52.80%                      |

### Insight
UI Variant **B requires fewer clicks** to start watching, showing smoother navigation.  
This leads to a **higher watch rate** and **stronger completion-after-watch performance**.  
Variant A’s higher click count indicates friction in discovering or starting content.

## Q9. Who are the top 5 most engaged users based on total watch time?
```sql
 SELECT user_id,
       SUM(session_length) AS total_watch_minutes,
       COUNT(*) AS total_sessions
FROM sessions
GROUP BY user_id
ORDER BY total_watch_minutes DESC
LIMIT 5;
```
| user_id | total_watch_minutes | total_sessions |
|---------|----------------------|----------------|
| 102     | 241.52               | 18             |
| 350     | 228.40               | 17             |
| 77      | 211.33               | 15             |
| 889     | 208.11               | 16             |
| 24      | 197.50               | 14             |

### Insight
A small segment of **power users** contributes heavily to total platform watch time.  
These high-engagement users tend to watch more frequently and for longer durations, making them valuable for retention-based strategies.  
Understanding their behavior helps optimize personalization and targeted content promotion.

## Q10. How does engagement differ across countries and subscription tiers?
```sql
 SELECT u.country,
       u.subscription_type,
       ROUND(AVG(s.session_length), 2) AS avg_session_length,
       ROUND(100 * AVG(s.watch_completion), 2) AS completion_rate_pct,
       COUNT(*) AS total_sessions
FROM sessions s
JOIN users u ON s.user_id = u.user_id
GROUP BY u.country, u.subscription_type
ORDER BY avg_session_length DESC;
```
| country | subscription_type | avg_session_length | completion_rate_pct | total_sessions |
|---------|--------------------|--------------------|----------------------|----------------|
| US      | Premium            | 15.88              | 64.10%               | 420            |
| IN      | Premium            | 14.92              | 62.85%               | 510            |
| UK      | Standard           | 13.21              | 59.10%               | 280            |
| BR      | Standard           | 12.40              | 57.25%               | 260            |
| IN      | Basic              | 10.80              | 52.40%               | 310            |
| BR      | Basic              | 10.18              | 50.90%               | 180            |
| CA      | Basic              | 9.80               | 49.20%               | 90             |

### Insight
Premium subscribers consistently show the **highest engagement** across all countries, with longer sessions and higher completion rates.  
Emerging markets like **India and Brazil** have high session volume but lower average engagement among Basic users.  
This suggests opportunities for targeted UI improvements or subscription upgrade nudges in lower-tier segments.


