📊 Bank Marketing SQL Project

📌 Overview

This project explores a bank marketing dataset using SQL. The goal is to analyze customer demographics, financial behaviors, and campaign outcomes to support better marketing strategies, improve loan targeting, and optimize customer segmentation.

🎯 Objectives

- Analyze customer demographics (age, marital status, education, job type).
- Identify financial patterns (balances, loans, defaults).
- Segment customers by age, balance, and education.
- Evaluate marketing campaign effectiveness.
- Provide business insights and solutions for banks.

🗂️ Database Schema
```sql
DROP TABLE IF EXISTS bank;

CREATE TABLE bank (
    age INT,
    job VARCHAR(50),
    marital VARCHAR(20),
    education VARCHAR(30),
    default VARCHAR(10),
    balance INT,
    housing VARCHAR(10),
    loan VARCHAR(10),
    contact VARCHAR(20),
    day INT,
    month VARCHAR(10),
    duration INT,
    campaign INT,
    pdays INT,
    previous INT,
    poutcome VARCHAR(20),
    y VARCHAR(10)
);
```

💡 Business Problems & Solutions
1️⃣ How many customers are married and have a tertiary education?
```sql
SELECT 
    COUNT(*) AS married_tertiary_count
FROM bank 
WHERE marital = 'married' AND education = 'tertiary';
```

2️⃣ What is the average age of customers for each job category?
```sql
SELECT
    job,
    ROUND(AVG(age), 0) AS avg_age 
FROM bank 
GROUP BY job
ORDER BY avg_age DESC;
```

3️⃣ Total & average balance for customers with/without housing loans
```sql
SELECT
    loan,
    SUM(balance) AS total_balance,
    AVG(balance) AS avg_balance
FROM bank
GROUP BY loan;
```

4️⃣ Rank customers by balance within each marital group
```sql
SELECT 
    age,
    job,
    marital,
    balance,
    RANK() OVER (PARTITION BY marital ORDER BY balance DESC) as balance_rank
FROM bank
ORDER BY marital, balance DESC;
```

5️⃣ Managers/Entrepreneurs with no default & balance > 1000
```sql
SELECT * 
FROM bank
WHERE job IN ('management', 'entrepreneur')
    AND `default` = 'no'
    AND balance > 1000;
```

6️⃣ Customers with balance above average of their education group
```sql
WITH education_balance AS (
    SELECT 
        age,
        job,
        marital,
        education,
        balance,
        AVG(balance) OVER (PARTITION BY education) AS avg_education_balance
    FROM bank
)
SELECT *
FROM education_balance 
WHERE balance > avg_education_balance
ORDER BY education, balance DESC;
```

7️⃣ Age categories by job type
```sql
SELECT 
    job,
    CASE 
        WHEN age < 30 THEN 'Young'
        WHEN age BETWEEN 30 AND 59 THEN 'Middle'
        ELSE 'Senior'
    END AS age_category,
    COUNT(*) AS customers_count,
    AVG(balance) AS avg_balance
FROM bank
GROUP BY 
    job,
    CASE 
        WHEN age < 30 THEN 'Young'
        WHEN age BETWEEN 30 AND 59 THEN 'Middle'
        ELSE 'Senior'
    END 
ORDER BY job, age_category;
```

8️⃣ Average campaign duration by previous outcome
```sql
SELECT 
    COALESCE(poutcome, 'No Data') AS outcome,
    ROUND(AVG(duration), 2) AS avg_duration_seconds,
    ROUND(AVG(duration) / 60, 2) AS avg_duration_minutes,
    COUNT(*) AS customer_count,
    MIN(duration) AS shortest_call,
    MAX(duration) AS longest_call
FROM bank
GROUP BY poutcome
ORDER BY avg_duration_seconds DESC;
```

9️⃣ Customers with both housing & personal loans
```sql
SELECT 
    job,
    SUM(CASE WHEN housing = 'yes' AND loan = 'yes' THEN 1 ELSE 0 END) AS both_loans_count,
    ROUND(
        SUM(CASE WHEN housing = 'yes' AND loan = 'yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2
    ) AS pct_with_both_loans
FROM bank 
GROUP BY job
ORDER BY pct_with_both_loans DESC;
```

🔟 Job-level stats: total, avg age, median, defaults
```sql
WITH job_stats AS (
    SELECT 
        job,
        age,
        balance,
        `default`,
        PERCENT_RANK() OVER (PARTITION BY job ORDER BY balance) AS percentile
    FROM bank
)
SELECT 
    job,
    COUNT(*) AS total_count,
    ROUND(AVG(age), 1) AS avg_age,
    ROUND(AVG(CASE WHEN percentile BETWEEN 0.4 AND 0.6 THEN balance END), 2) AS median_balance,
    ROUND(SUM(CASE WHEN `default` = 'yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS pct_with_default
FROM job_stats
GROUP BY job
ORDER BY total_count DESC;
```

🔍 Key Findings

- Married + tertiary education customers form a valuable premium segment.
- Managers/entrepreneurs tend to have higher balances and lower default risks.
- Customers with housing + personal loans show higher financial dependency.
- Middle-aged group (30–59) is the largest customer base.
- Campaign call duration does not always correlate with positive outcomes.

📈 Conclusion

- This SQL project provides insights into customer behavior, financial risk, and marketing campaign outcomes.
By applying these findings, banks can:
- Improve targeting of marketing campaigns.
- Identify high-value and high-risk customers.
- Personalize loan and savings products.
- Optimize campaign strategies for better conversion rates.

👤 Author

Van Huu Hien Nguyen - Business Analytics Student This project is part of my SQL portfolio, showcasing skills in data cleaning, analysis, and business insights generation. Feel free to reach out for feedback or collaboration!
