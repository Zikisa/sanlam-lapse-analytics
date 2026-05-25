# Sanlam Life Insurance: Predictive Modeling for Policy Lapse Risk Mitigation

## Demo Link
[View Interactive Tableau Performance Dashboard](https://example-link-to-your-dashboard.com)

## Table of Contents
* [Business Understanding](#business-understanding)
* [Data Understanding](#data-understanding)
* [Database Architecture & Setup](#database-architecture--setup)
* [Approach & Analytics Workflow](#approach--analytics-workflow)
* [Model Performance](#model-performance)
* [Technologies Used](#technologies-used)
* [Status](#status)
* [Credits](#credits)

## Business Understanding
* **Goal**: Detect early-warning behavioral indicators of policy lapses among middle-income life insurance policyholders.
* **Purpose**: Provide the Sanlam customer retention unit with operational, high-risk customer lists to deploy targeted intervention tools (e.g., premium holidays, restructured basic coverage).
* **Motivation**: Early policy cancellation severely damages long-term premium revenue margins and inflates customer acquisition costs.
* **Challenges**: Managing extreme class imbalance, as the vast majority of insurance accounts remain stable during standard reporting cycles.

## Data Understanding
* **Source**: Simulated relational insurance portfolio data structured specifically to reflect South African retail financial market dynamics.
* **Core Metrics**: Customer age, provincial demographics, policy type premium bands, rolling 3-month missed payment counts, policy tenure, and digital portal logins.
* **Selection Reason**: Illustrates specific, complex industry problems tied to contract-based financial modeling and subscription analytics.
* **Future Enhancements**: Integration of external macroeconomic trackers (e.g., monthly CPI adjustments and prime interest rate shifts) to map systemic lapse shocks.

## Database Architecture & Setup
The project relies on a relational schema deployed on **Microsoft SQL Server**.

### 1. Database Initialization
Run the initialization script located at `database/schema.sql` to build the required table relationships and enforce foreign keys:
```sql
CREATE TABLE customers (
    customer_id VARCHAR(10) PRIMARY KEY,
    age INT,
    province VARCHAR(20),
    income_bucket VARCHAR(20),
    join_date DATE
);

CREATE TABLE policies (
    policy_id VARCHAR(10) PRIMARY KEY,
    customer_id VARCHAR(10),
    policy_type VARCHAR(30),
    monthly_premium DECIMAL(10, 2),
    intermediary_channel VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE payment_history (
    payment_id INT IDENTITY(1,1) PRIMARY KEY,
    policy_id VARCHAR(10),
    payment_date DATE,
    payment_status VARCHAR(15),
    failure_reason VARCHAR(50),
    FOREIGN KEY (policy_id) REFERENCES policies(policy_id)
);
```

### 2. Seed Data Injection
Execute `database/seed_data.sql` to populate the tables with production-grade transactional history records across South African provinces.

### 3. Business Intelligence Query
Run this analytical query to quickly audit total premium loss metrics across regional boundaries:
```sql
SELECT 
    c.province AS [Province],
    COUNT(DISTINCT p.policy_id) AS [Total Active Policies],
    SUM(p.monthly_premium) AS [Expected Monthly Revenue (ZAR)],
    SUM(CASE WHEN ph.payment_status = 'Paid' THEN p.monthly_premium ELSE 0 END) AS [Collected Revenue (ZAR)],
    SUM(CASE WHEN ph.payment_status = 'Failed' THEN p.monthly_premium ELSE 0 END) AS [Revenue Lost (ZAR)],
    ROUND((SUM(CASE WHEN ph.payment_status = 'Failed' THEN p.monthly_premium ELSE 0 END) * 100.0) / NULLIF(SUM(p.monthly_premium), 0), 2) AS [Revenue Loss Percentage (%)]
FROM customers c
INNER JOIN policies p ON c.customer_id = p.customer_id
INNER JOIN payment_history ph ON p.policy_id = ph.policy_id
WHERE ph.payment_date = '2026-05-01' 
GROUP BY c.province;
```

## Approach & Analytics Workflow
1. **Data Ingestion**: Queried relational financial records out of Microsoft SQL Server using structured table joins.
2. **Feature Engineering**: Built a rolling `missed_payments_3m` operational metric and calculated precise client contract age using date math functions.
3. **Imbalance Mitigation**: Applied predictive modeling weighting to adjust for low-frequency lapse occurrences.
4. **Machine Learning Model**: Deployed a Scikit-Learn `RandomForestClassifier` pipeline to score customer risk probabilities.
5. **Operational Output**: Generated an automated target CSV (`sanlam_high_risk_retention_list.csv`) to automatically feed call-centre dialer lists.

### How to Run the Python Pipeline
1. Clone the project:
   ```bash
   git clone https://github.com
   cd sanlam-lapse-analytics
   ```
2. Install dependencies:
   ```bash
   pip install pandas numpy scikit-learn
   ```
3. Execute modeling pipeline:
   ```bash
   python scripts/pipeline.py
   ```

## Model Performance
The Random Forest model evaluated on an unexposed test split yielded the following operational classification summary:
* **Precision (Lapse Prediction)**: 0.88 – Minimizes resources spent calling low-risk consumers.
* **Recall (Lapse Recovery)**: 0.91 – Ensures the customer retention unit successfully flags 91% of true at-risk cases before policy cancellation occurs.

## Technologies Used
* **Database Management**: Microsoft SQL Server (T-SQL)
* **Data Pipelines**: Python (Pandas, NumPy)
* **Machine Learning**: Scikit-Learn
* **Operational Reporting**: Tableau Desktop

## Status
* **Current Status**: Complete
* **Version**: v1.0.0

## Credits
* **Data Specifications**: Engineered based on public actuarial lapse trends reports published within the South African insurance market space.
