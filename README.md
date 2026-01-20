# SQL for Data Analytics Project README

## Introduction

This project dives into the data job market with a focus on **Data Analyst** roles. It explores the top-paying jobs, in-demand skills, and where high demand meets high salary to find the most optimal skills for data professionals.

The complete set of SQL scripts can be found in the [project_sql folder](sql_load/project_sql)

## Background

Driven by a desire to navigate the data science industry more effectively, this project answers five core questions:

1. What are the highest-paying data analyst jobs?
2. What skills are required for these top-paying roles?
3. What are the most in-demand skills for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn (high demand + high pay)?

## Tools I Used

* **SQL:** The backbone of the analysis, allowing for data extraction and insight generation.
* **PostgreSQL:** The chosen database management system for handling job posting data.
* **Visual Studio Code:** The primary IDE for database management and executing SQL queries.
* **Git & GitHub:** Used for version control and sharing the project analysis.

## The Analysis

Each query in this project was designed to investigate specific aspects of the data job market:

### 1. Top Paying Data Analyst Jobs

Identified the top 10 highest-paying Data Analyst roles that are available remotely.

```
SELECT
    job_id,
    job_title_short,
    job_schedule_type,
    job_location,
    salary_year_avg,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN
    company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_location = 'Anywhere' AND
    job_title_short = 'Data Analyst' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

Here's the breakdown of the top data analyst jobs in 2023:
- **Wide Salary Range:** Top 10 paying data analyst roles span from $184,000 to $650,000, indicating significant salary potential in the field.
- **Diverse Employers:** Companies like
SmartAsset, Meta, andgAT&T are among those offering high salaries, showing a broad interest across different industries.
- **Job Title Variety:** There's a high diversity in job titles, from Data Analyst to Director of Analytics, reflecting varied roles and specializations within data analytics.

<img width="817" height="455" alt="Top Paying Data Jobs" src="https://github.com/user-attachments/assets/81beb263-613e-46ff-a125-47dae5f6be38" />


### 2. Skills for Top Paying Jobs

Determined what specific skills are required for the highest-paying roles identified in the previous step.

* **Key Insight:** **SQL** and **Python** appeared as common requirements across almost every top-paying role.
```
WITH top_paying_skills AS(
    SELECT
        job_id,
        job_title_short,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN
        company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_location = 'Anywhere' AND
        job_title_short = 'Data Analyst' AND
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT
    top_paying_skills.*,
    skills
FROM top_paying_skills
INNER JOIN skills_job_dim ON top_paying_skills.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY salary_year_avg DESC;
```

<img width="531" height="510" alt="Screenshot 2026-01-20 at 3 56 00 PM" src="https://github.com/user-attachments/assets/d800bc05-1764-494b-b77a-5a4d3280ed12" />

### 3. Most In-Demand Skills

Identified which skills are most frequently mentioned in all Data Analyst job postings.

```
SELECT
    skills,
    COUNT(skills_job_dim.skill_id) AS demand_count
FROM
    job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 10;
```


* **Key Insight:** **SQL, Excel, Python, Tableau, and Power BI** are the top 5 most in-demand skills.
<img width="277" height="296" alt="Screenshot 2026-01-20 at 3 58 10 PM" src="https://github.com/user-attachments/assets/7c062b60-f18a-446d-af78-c267436d2e9f" />

### 4. Skills Based on Salary

Analyzed the average salary associated with different skills.
```
SELECT
    skills,
    ROUND(AVG(salary_year_avg) )AS avg_salary
FROM
    job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```

* **Key Insight:** High-paying skills for data analysts often involve **Big Data, Machine Learning, and Cloud Computing** (e.g., PySpark, Pandas, AWS).
<img width="256" height="500" alt="Screenshot 2026-01-20 at 4 00 49 PM" src="https://github.com/user-attachments/assets/775a2339-a7e3-4462-8357-2cbdead09fe3" />


### 5. Most Optimal Skills

Combined demand and salary data to find skills that are both high in demand and offer high pay.

```
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.skill_id) AS demand_count
    FROM
        job_postings_fact
        INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
        INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_dim.skill_id
), average_salary AS (
    SELECT
        skills_dim.skill_id,
        ROUND(AVG(salary_year_avg),0) AS avg_salary
    FROM
        job_postings_fact
        INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
        INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_dim.skill_id
)
SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN
    average_salary
    ON skills_demand.skill_id = average_salary.skill_id
ORDER BY demand_count DESC;
```


* **Key Insight:** When filtered for a demand count greater than 10, **Cloud-based databases (Snowflake, Azure)** and programming languages like **Python** represent the most optimal career investments.

<img width="600" height="500" alt="Screenshot 2026-01-20 at 4 02 07 PM" src="https://github.com/user-attachments/assets/9289e4c6-003f-49cc-b5fd-01b0239a5d1e" />

## What I Learned

* Advanced SQL techniques including **CTEs, Subqueries, and Unions**.
* Data cleaning and type casting for better data integrity.
* The importance of merging technical skills (SQL) with strategic business questions.

## Conclusions

The data clearly shows that while foundational tools like **SQL and Excel** are necessary for entry (high demand), specialized knowledge in **Cloud tools and Big Data** is the path to the highest salaries in the industry.
