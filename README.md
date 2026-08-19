# 📊 Uzbekistan & Samarkand Tourism Data Analysis (SQL Project)
### *O'zbekiston va Samarqand viloyati turizm sohasi tahlili*

[![SQL](https://img.shields.io/badge/Language-SQL%20%7C%20PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Database](https://img.shields.io/badge/Database-Star%20Schema-green?style=for-the-badge)](https://github.com/fboynazarova001-design/ProjectWork)
[![Data Source](https://img.shields.io/badge/Data-stat.uz%20%7C%20data.egov.uz-blue?style=for-the-badge)](https://stat.uz)

---

## 📌 Introduction / Kirish

Welcome to the **Uzbekistan & Samarkand Tourism SQL Analysis Project**! 🇺🇿✨

This project dives deep into the dynamic tourism economy of Uzbekistan, with a specialized focus on **Samarkand**—the legendary Pearl of the Silk Road. Using relational database modeling and advanced SQL queries, this project uncovers **top-visited regions, international source markets, high-spending tourist segments, and strategic sweet spots** to drive tourism growth and investment.

🔍 **SQL Queries Folder:** Check out the complete SQL scripts in the [project_sql/](/project_sql/) directory.  
🗄️ **Database Schema & Loaders:** Check table definitions and import scripts in [sql_load/](/sql_load/).

---

## 🏛️ Background & Motivation / Loyiha haqida

Driven by Uzbekistan's rapid tourism expansion and open-visa policy reforms, this project analyzes the volume and economic impact of foreign visitors. 

Data is modeled following official statistical methodologies and indicators from:
- **[Agency of Statistics of Uzbekistan (stat.uz)](https://stat.uz)** (Inbound tourist flows, purposes of visit, expenditure metrics).
- **[Open Data Portal of Uzbekistan (data.egov.uz)](https://data.egov.uz)** (Regional infrastructure, accommodation facilities, and heritage indicators).

### The 5 Business Questions Answered via SQL:
1. **Top-Visited Regions:** Which regions attract the highest number of foreign tourists, and how does Samarkand compare to Tashkent, Bukhara, and Khorezm?
2. **Samarkand Tourism Segments:** What are the primary purposes of visit to Samarkand (Cultural Heritage, Ziyorat/Pilgrimage, Business/MICE, Leisure)?
3. **Top Source Countries:** Which nations are the biggest source markets of inbound tourists to Uzbekistan?
4. **Highest-Spending Markets:** Which foreign visitor segments have the highest average spend ($/tourist) and longest stay durations?
5. **Optimal High-Value Markets:** Which inbound markets represent the optimal "Sweet Spot" (combining high volume with high average spend)?

---

## 🛠️ Tools I Used / Foydalanilgan vositalar

- **SQL & PostgreSQL:** The core engine used to design a Star Schema database, execute CTEs, aggregations, window functions, and extract strategic insights.
- **Visual Studio Code / DBeaver / pgAdmin:** Tools for database query execution and script development.
- **Python (Matplotlib & Pandas):** Utilized to render high-resolution visual charts from query results for executive reporting.
- **Git & GitHub:** Version control, collaboration, and portfolio showcase.

---

## 🏗️ Relational Database Schema (Star Schema)

The database is designed with 1 Fact Table and 4 Dimension Tables:
- 	ourist_arrivals_fact: Granular monthly records of tourist arrivals, average expenditure ($), and stay duration.
- 
egions_dim: 14 administrative regions of Uzbekistan with capitals and UNESCO World Heritage counts.
- countries_dim: Inbound source markets, continents, and visa regimes (Visa-Free, E-Visa, Visa Required).
- 	ourism_types_dim: Classification of trip purposes (Heritage, Ziyorat, Leisure, Business, Medical, etc.).
- ccommodations_dim: Hotel and accommodation facility metrics, ADR ($), and occupancy rates.

---

## 📈 The Analysis & Key Insights / Tahlil va Natijalar

### 1. Top Visited Regions in Uzbekistan (2024)
To evaluate the regional distribution of foreign visitors, this query aggregates arrival volumes, market share percentages, average stay lengths, and estimated tourism revenue across all regions of Uzbekistan.

`sql
SELECT 
    r.region_id,
    r.region_name_uz,
    r.region_name_en,
    r.unesco_sites_count,
    SUM(f.visitor_count) AS total_visitors_2024,
    ROUND(SUM(f.visitor_count) * 100.0 / SUM(SUM(f.visitor_count)) OVER(), 2) AS market_share_percentage,
    ROUND(AVG(f.avg_length_of_stay_days), 1) AS avg_stay_days,
    ROUND(SUM(f.visitor_count * f.avg_spend_usd) / 1000000.0, 2) AS estimated_revenue_million_usd
FROM 
    tourist_arrivals_fact f
JOIN 
    regions_dim r ON f.region_id = r.region_id
WHERE 
    f.arrival_year = 2024
GROUP BY 
    r.region_id,
    r.region_name_uz,
    r.region_name_en,
    r.unesco_sites_count
ORDER BY 
    total_visitors_2024 DESC;
`

**Key Findings:**
- **Tashkent City & Samarkand Region Dominate:** Together they account for over **60% of all inbound tourists** entering Uzbekistan.
- **Samarkand as the Cultural Capital:** Samarkand generated over **.8B+ in estimated economic impact**, driven by high international interest in Registan, Shah-i Zinda, and the Silk Road Samarkand complex.
- **Historic Triangle:** Samarkand, Bukhara, and Khorezm form the core cultural Silk Road route, showing longer average stays (5.5 – 7.2 days).

![Top Visited Regions](assets/1_top_visited_regions.png)

---

### 2. Samarkand: Purpose of Visit & Tourism Segments
This query investigates the distribution of travel motivations specifically for Samarkand to determine the balance between Cultural, Religious (Ziyorat), and Business tourism.

`sql
WITH samarkand_visits AS (
    SELECT 
        f.purpose_id,
        f.visitor_count,
        (f.visitor_count * f.avg_spend_usd) AS total_spend,
        f.avg_length_of_stay_days
    FROM 
        tourist_arrivals_fact f
    WHERE 
        f.region_id = 2 AND -- Samarqand viloyati
        f.arrival_year = 2024
)
SELECT 
    t.purpose_name_uz,
    t.purpose_name_en,
    t.category,
    SUM(sv.visitor_count) AS total_visitors,
    ROUND(SUM(sv.visitor_count) * 100.0 / SUM(SUM(sv.visitor_count)) OVER(), 2) AS share_pct,
    ROUND(AVG(sv.avg_length_of_stay_days), 1) AS avg_length_of_stay,
    ROUND(SUM(sv.total_spend) / 1000000.0, 2) AS total_revenue_million_usd
FROM 
    samarkand_visits sv
JOIN 
    tourism_types_dim t ON sv.purpose_id = t.purpose_id
GROUP BY 
    t.purpose_name_uz,
    t.purpose_name_en,
    t.category
ORDER BY 
    total_visitors DESC;
`

**Key Findings:**
- **Cultural & Heritage Tourism Leads (55.4%):** Ancient monuments (Registan, Gur-e-Amir, Bibi-Khanym) remain the primary magnet for international travelers.
- **Ziyorat (Pilgrimage) Tourism (22.8%):** Strong growth from Turkey, Malaysia, Indonesia, and Middle Eastern countries visiting Imam al-Bukhari and historic shrines.
- **Emerging MICE & Business Tourism (11.2%):** Accelerated by international congresses and summits held at Silk Road Samarkand.

![Samarkand Tourism Types](assets/2_samarkand_tourism_types.png)

---

### 3. Top Source Countries for Inbound Tourism
To understand where visitors originate, this query ranks the top 10 international origin markets and assesses the correlation with visa policies.

`sql
SELECT 
    c.country_id,
    c.country_name_uz,
    c.country_name_en,
    c.continent,
    c.visa_regime,
    SUM(f.visitor_count) AS total_tourists_2024,
    ROUND(SUM(f.visitor_count) * 100.0 / SUM(SUM(f.visitor_count)) OVER(), 2) AS share_of_inbound_tourism_pct,
    DENSE_RANK() OVER (ORDER BY SUM(f.visitor_count) DESC) AS rank_by_volume
FROM 
    tourist_arrivals_fact f
JOIN 
    countries_dim c ON f.country_id = c.country_id
WHERE 
    f.arrival_year = 2024
GROUP BY 
    c.country_id,
    c.country_name_uz,
    c.country_name_en,
    c.continent,
    c.visa_regime
ORDER BY 
    total_tourists_2024 DESC
LIMIT 10;
`

**Key Findings:**
- **Central Asia & CIS form the Volume Backbone:** Kazakhstan, Russia, Tajikistan, and Kyrgyzstan represent the highest visitor volume due to geographical proximity and visa-free travel.
- **Rapid Growth from Turkey, China & India:** Boosted by simplified 10-day visa-free and e-visa regimes.

![Top Source Countries](assets/3_top_source_countries.png)

---

### 4. Highest-Spending Inbound Markets
This query filters for countries with substantial volume and identifies which markets yield the highest average expenditure per traveler and longest hotel stays.

`sql
SELECT 
    c.country_name_uz,
    c.country_name_en,
    c.continent,
    ROUND(AVG(f.avg_spend_usd), 2) AS avg_spend_per_tourist_usd,
    ROUND(AVG(f.avg_length_of_stay_days), 1) AS avg_stay_duration_days,
    SUM(f.visitor_count) AS total_visitors_2024,
    ROUND(SUM(f.visitor_count * f.avg_spend_usd) / 1000000.0, 2) AS total_market_revenue_million_usd
FROM 
    tourist_arrivals_fact f
JOIN 
    countries_dim c ON f.country_id = c.country_id
WHERE 
    f.arrival_year = 2024
GROUP BY 
    c.country_name_uz,
    c.country_name_en,
    c.continent
HAVING 
    SUM(f.visitor_count) >= 5000
ORDER BY 
    avg_spend_per_tourist_usd DESC;
`

**Key Findings:**
- **High-Yield Markets:** Travelers from **Saudi Arabia (,950+ avg spend)**, **UAE (,890+)**, **USA (,720+)**, and **Germany/Italy/UK (,400 - ,580)** spend significantly more per trip.
- **Extended Stay Duration:** European, American, and East Asian tourists stay an average of **7.0 to 8.5 days**, booking higher-category accommodation and cultural tours.

![Top Spending Markets](assets/4_top_spending_markets.png)

---

### 5. Optimal High-Value Markets ("Sweet-Spot" Matrix)
By combining volume analysis and spending power using Common Table Expressions (CTEs), this query identifies strategic market tiers to maximize tourism revenue ROI.

`sql
WITH volume_analysis AS (
    SELECT 
        c.country_id,
        c.country_name_uz,
        c.country_name_en,
        c.continent,
        SUM(f.visitor_count) AS total_visitors
    FROM 
        tourist_arrivals_fact f
    JOIN 
        countries_dim c ON f.country_id = c.country_id
    WHERE 
        f.arrival_year = 2024
    GROUP BY 
        c.country_id,
        c.country_name_uz,
        c.country_name_en,
        c.continent
),
spending_analysis AS (
    SELECT 
        f.country_id,
        ROUND(AVG(f.avg_spend_usd), 2) AS avg_spend_usd,
        ROUND(AVG(f.avg_length_of_stay_days), 1) AS avg_stay_days
    FROM 
        tourist_arrivals_fact f
    WHERE 
        f.arrival_year = 2024
    GROUP BY 
        f.country_id
)
SELECT 
    va.country_name_uz,
    va.country_name_en,
    va.continent,
    va.total_visitors,
    sa.avg_spend_usd,
    sa.avg_stay_days,
    ROUND((va.total_visitors * sa.avg_spend_usd) / 1000000.0, 2) AS total_economic_impact_million_usd,
    CASE 
        WHEN sa.avg_spend_usd >= 1200 AND va.total_visitors >= 30000 THEN '🌟 Tier 1: High Volume & High Spend'
        WHEN sa.avg_spend_usd >= 1200 THEN '💎 Tier 2: Premium Niche Market'
        WHEN va.total_visitors >= 100000 THEN '📈 Tier 3: Mass Volume Market'
        ELSE '🔍 Tier 4: Developing Market'
    END AS strategic_market_tier
FROM 
    volume_analysis va
JOIN 
    spending_analysis sa ON va.country_id = sa.country_id
ORDER BY 
    total_economic_impact_million_usd DESC;
`

**Strategic Matrix Breakdown:**
- **Tier 1 (High Volume + High Spend):** China, Turkey, and Russia represent the most lucrative markets overall.
- **Tier 2 (Premium Niche):** Germany, France, Italy, UK, USA, Saudi Arabia, and Japan represent high-spending cultural and pilgrimage travelers. Targeted marketing here yields maximum revenue per visitor.

![Optimal Markets](assets/5_optimal_markets.png)

---

## 🎯 Strategic Recommendations / Tavsiyalar

1. **Focus Marketing on Premium & Ziyorat Segments:** Expand targeted campaigns in GCC (Saudi Arabia, UAE), Southeast Asia (Malaysia, Indonesia), and Western Europe.
2. **Promote Multi-City Silk Road Passes:** Connect Samarkand, Bukhara, and Khorezm via Afrosiyob high-speed rail packages to increase average stay from 4.5 to 7+ days.
3. **Winter & Off-Peak Incentives:** Address winter seasonality in Samarkand with MICE conferences, winter cultural festivals, and business travel incentives.

---

## 💻 How to Run This Project Locally / Loyihani ishga tushirish

### 1. Clone the repository:
`ash
git clone https://github.com/fboynazarova001-design/ProjectWork.git
cd ProjectWork
`

### 2. Set up PostgreSQL Database:
Open PostgreSQL terminal or pgAdmin / DBeaver:
`sql
-- 1. Create the database
CREATE DATABASE uzbekistan_tourism_db;

-- 2. Execute schema creation script
-- Run sql_load/2_create_tables.sql

-- 3. Load dataset from CSV files
-- Run sql_load/3_modify_tables.sql
`

### 3. Run Analytics Queries:
Execute any query script inside the project_sql/ directory to explore insights.

---

## 👤 Author & Acknowledgments

- **Project Lead:** [Farangiz Boynazarova](https://github.com/fboynazarova001-design)
- **Data Sources:** [stat.uz](https://stat.uz) & [data.egov.uz](https://data.egov.uz)
- **Course Inspiration:** Luke Barousse SQL Analytics Course format.

⭐ *If you found this project insightful, feel free to star the repository!*
