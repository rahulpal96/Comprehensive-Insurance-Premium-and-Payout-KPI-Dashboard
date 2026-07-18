# Insurance-Premium-and-Payout-KPI-Dashboard
  A production-grade, end-to-end Data Analytics project designed to track the complete insurance policy life cycle (2015–2025) for True Secure Credit Insurance. This project implements robust data modeling, dynamic DAX calculations, and Row-Level Security (RLS) to optimize premium collections, claim processing efficiency, and financial forecasting.

📌** Project Overview**

True Secure Credit Insurance provides specialized individual and business protection plans featuring credit- and debit-linked coverage. Spanning ten years of structural data (2015-2025), this project builds a central, production-ready operational intelligence environment. It automates reporting, eliminates manual reporting latency, tracks policy sign-ups across demographic segments, and helps stakeholders drill down into financial health metrics.

💼** Business Problem**

The executive leadership faced several major operational challenges:

**Data Fragmentation**: Financial records, claim metrics, and policy processing lifecycles were siloed across disjointed relational schemas, introducing substantial reporting lags.

**Customer Uptake Blindspots**: The business lacked clear visibility into which policy plans or regional sectors were underperforming.

**Claim Bottlenecks & Exposure**: There was no automated system to monitor active claim processing cycles or accurately project incoming liability payouts from mature investments.

**🎯** Project Goals****

**Unified Business View**: Design an interactive data model that unifies sales, claims, loans, and policy details.

**Granular Demographic Profiling**: Analyze policy selection trends by state, age at entry, and occupation.

**Financial Risk Tracking**: Monitor key metrics like revenue growth, historical payouts, and premium collection cycles.

**Operational Efficiency & Security**: Implement advanced bi-directional cross-filtering for deeper ad-hoc drill-downs and secure data access via dynamic Row-Level Security (RLS).

📊** Dataset Information**

The project runs on a relational star schema composed of 5 tables:

🧩** Dimension Tables**

**dim_customer_detail**: Demographics including customer_id, name, gender, age at entry, current age, occupation, smoking status, medical exam requirements, and home state.

**dim_policy_type**: Master categories (term, whole life, universal) mapped via policy_type_code.

**dim_policy_nam**e: Detailed policy names linked to parent policy classifications.

**dim_regional_security / dm_zonal_manager**: Mapping organizational management levels and emails for secure access controls.

**🪙 Fact Table**

fact_policy_lifecycle: Main transactional engine containing policy values, premium amounts, payment frequencies (Monthly, Quarterly, Half-Yearly, Yearly), loan eligibilities, cash values, surrender receivables, and sales agent codes.

🛠️** Tools & Technologies**

Data Warehouse / Storage: SQL Relational Database

ETL & Transformation: Power Query (M Formula Language)

Data Modeling & Visualization: Microsoft Power BI Desktop

Analytical Calculations: Advanced DAX (Data Analysis Expressions)

Security Framework: Dynamic Row-Level Security (RLS) & Power BI Service Cloud Deployment

🧼 Data Cleaning & Transformation (Power Query)
Data preparation followed a strict enterprise extract-transform-load (ETL) pipeline in Power Query:

Strict Type Binding: Enforced static data types across all text, integer, currency, and date/time fields.

Conditional Normalization: Built multi-field mapping logic to convert disparate payment intervals into a standard annualized timeframe.

Relational Denormalization: Leveraged Merge Queries using a Left Outer Join on customer_id to build a reliable analytics model.

🔍 Exploratory Data Analysis (EDA) & SQL Queries
Before finalizing the visualization layers, target SQL scripts were executed to validate core dataset patterns and identify outlier behaviors:

SQL
-- 1. Identify Top Performing Insurance Policies by Volume and Revenue
SELECT 
    p.policy_name, 
    COUNT(f.policy_id) AS total_policies_issued,
    SUM(f.premium_amount) AS raw_premium_collections
FROM fact_policy_lifecycle f
JOIN dim_policy_name p ON f.policy_name_code = p.policy_name_code
GROUP BY p.policy_name
ORDER BY raw_premium_collections DESC;

-- 2. Track Claims Settlement Efficiency & Risk Exposures
SELECT 
    c.state,
    COUNT(CASE WHEN f.claim_status = 'Pending' THEN 1 END) AS active_pending_claims,
    SUM(f.claim_amount) AS total_claimed_liability,
    AVG(f.customer_current_age) AS average_claimant_age
FROM fact_policy_lifecycle f
JOIN dim_customer_detail c ON f.customer_id = c.customer_id
GROUP BY c.state
ORDER BY total_claimed_liability DESC;
📐 Data Modeling & DAX Measures
The project transitions from a default single-direction layout into a high-performance bi-directional cross-filtering design to ensure agent codes can accurately filter across customer records and fact dimensions synchronously.

🧪 Core DAX Calculations
Code snippet
// 1. Standardized Annual Premium Calculation
Annual Premium = 
SWITCH(
    TRUE(),
    fact_policy_lifecycle[payment_frequency] = "Monthly", fact_policy_lifecycle[premium_amount] * 12,
    fact_policy_lifecycle[payment_frequency] = "Quarterly", fact_policy_lifecycle[premium_amount] * 4,
    fact_policy_lifecycle[payment_frequency] = "Half-Yearly", fact_policy_lifecycle[premium_amount] * 2,
    fact_policy_lifecycle[payment_frequency] = "Yearly", fact_policy_lifecycle[premium_amount] * 1,
    0
)

// 2. Cumulative Premium Revenue Collected
Total Premium Paid = 
SUMX(
    fact_policy_lifecycle,
    fact_policy_lifecycle[premium_amount] * fact_policy_lifecycle[elapsed_tenure_periods]
)

// 3. Dynamic Row-Level Security Filter Expression
Zonal_Manager_Security = 
VAR _LoggedUser = LOWER(USERPRINCIPALNAME())
VAR _UserSecurityLevel = LOOKUPVALUE(
    dm_zonal_manager[security_level],
    dm_zonal_manager[manager_email],
    _LoggedUser
)
RETURN
IF(
    _UserSecurityLevel = "Owner",
    TRUE(),
    dm_zonal_manager[manager_email] = _LoggedUser
)
🎨 Power BI Dashboard Architecture
The visual layouts are organized into a clean 3-page configuration tailored for clear decision-making:

🎛️ Page 1: Executive KPI Dashboard
KPI Cards: Displaying Total Premium Volume, Net Financial Revenue, Active Policies, and Claims Settlement Rates.

Main Line Chart: Tracking Year-over-Year Premium Growth Trends (2015-2025).

Geospatial Map: Highlighting geographical sales distributions and regional performance across Indian states.

👥 Page 2: Demographic Analysis & Risk Profiling
Donut & Bar Charts: Showing policy type adoption broken down by customer gender, occupation, and smoker status.

Age Segmentation Tree: Visualizing correlation trends between age-at-entry and policy retention rates.

🔒 Page 3: Operational Control & Claims Processing
Claim Funnel: Breaking down processing times from initial claim submission to final payout.

Loan Eligibility Matrix: Detailing active policy values against available loan limits.

💡 Business Insights & Key Findings
The Age Sweet Spot: Customers who buy policies between ages 20 and 35 have the highest customer lifetime value (LTV), thanks to lower entry premiums and longer policy retention times.

Operational Risk Drivers: Premium calculations for self-identified smokers who did not require a medical check showed higher early-tenure claim rates.

Regional Imbalances: Over 60% of premium collections were concentrated in the top 4 states, indicating significant opportunities for growth in underrepresented areas.

🚀 Recommendations & Strategic Solutions
Refine Underwriting Rules: Make medical exams mandatory for high-risk profiles (such as smokers) regardless of their entry age to better manage risk.

Targeted Marketing Campaigns: Roll out regional sales plans to improve customer sign-ups in underperforming states.

Modernize the Claims Process: Implement automated verification loops for pending claims to reduce turnaround times and improve customer satisfaction.

📂 Repository Structure
Code snippet
├── data/
│   ├── true_secure_customer_details.csv
│   └── true_secure_policy_lifecycle.xlsx
├── sql/
│   └── analytical_explorations.sql
├── dax/
│   ├── calculated_columns.dax
│   └── measures.dax
├── visuals/
│   ├── dashboard_executive_view.png
│   └── dashboard_demographics.png
├── True_Secure_Insurance_Analytics.pbix
└── README.md
⚙️ How to Run and View the Project
Clone the Repository:



Launch the Power BI Desktop App: Open True_Secure_Insurance_Analytics.pbix in Microsoft Power BI Desktop.

Update the Data Paths: If prompted, update the local file paths in Power Query to point to your repository data folder and hit Refresh.

🔐 Row-Level Security (RLS) Verification
To test and verify the row-level security setup within Power BI Service:

Navigate to your workspace workspace, click on the Dataset Security settings.

Use the "Test as Role" feature to simulate access permissions.

Enter a regional manager's email address to verify that they can only see the rows and financial data for their assigned area.

📈 Future Improvements
Advanced Predictive Analytics: Integrate Azure Machine Learning scripts directly into the Power BI pipeline to predict future customer churn.

Live Stream Integration: Move from daily automated refreshes to real-time streaming data ingestion using Apache Kafka.

Enriched Demographic Insights: Bring in external macroeconomic data to analyze how regional economic changes affect overall policy renewal rates.
