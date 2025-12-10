# Walmart Sales Data Analysis | End-to-End Data Engineering Project

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?style=for-the-badge&logo=pandas)
![SQL](https://img.shields.io/badge/SQL-Advanced%20Querying-orange?style=for-the-badge&logo=mysql)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-336791?style=for-the-badge&logo=postgresql)
![Kaggle](https://img.shields.io/badge/Kaggle-API%20Integration-20BEFF?style=for-the-badge&logo=kaggle)

## Table of Contents

- [Project Overview](#-project-overview)
- [Architecture & Workflow](#-architecture--workflow)
- [Tech Stack](#-tech-stack)
- [Data Cleaning & Transformation](#-data-cleaning--transformation)
- [Setup & Installation](#-setup--installation)
- [Key Business Questions Solved](#-key-business-questions-solved)
- [Project Structure](#-project-structure)
- [Conclusion](#-conclusion)

---

## Project Overview

This project demonstrates an **end-to-end data analysis pipeline** designed to extract, clean, and analyze retail sales data from Walmart. The project simulates a real-world data engineering workflow where data is fetched from an external API (Kaggle), processed using Python (Pandas), loaded into a relational database (SQL), and finally analyzed to derive actionable business insights.

The goal is to answer critical business questions regarding branch performance, product trends, and customer behavior using advanced SQL techniques.

---

## Architecture & Workflow

1. **Data Extraction**: Utilize the Kaggle API to programmatically download the dataset.
2. **Data Cleaning (Python/Pandas)**:
   - Remove duplicates and handle missing values.
   - Fix data types (converting currency strings to numeric, date parsing).
   - Feature Engineering (creating `total_amount`, standardizing branch names).
3. **Database Loading**:
   - Establish a connection to MySQL/PostgreSQL using `SQLAlchemy`.
   - Load the processed Pandas DataFrame into the database.
4. **Data Analysis (SQL)**: Write complex SQL queries to solve business problems.

---

## Tech Stack

| Category               | Tools / Libraries             |
| :--------------------- | :---------------------------- |
| **Language**           | Python 3.x                    |
| **Data Manipulation**  | Pandas, NumPy                 |
| **Database Connector** | SQLAlchemy, PyMySQL, Psycopg2 |
| **Database**           | MySQL, PostgreSQL             |
| **API**                | Kaggle API                    |
| **IDE**                | VS Code / Jupyter Notebook    |

---

## Data Cleaning & Transformation

Before loading data into SQL, significant pre-processing was performed in Python:

- **Currency Cleaning:** Removed `$` signs from the `unit_price` column to allow for numerical calculations.
- **Feature Engineering:**
  - Added a `total_amount` column (`unit_price * quantity`).
  - Extracted temporal features to analyze sales shifts (Morning, Afternoon, Evening).
- **De-duplication:** Identified and removed duplicate records to ensure data integrity.
- **Normalization:** Standardized column names to snake_case for SQL compatibility.

---

## Setup & Installation

Follow these steps to set up the project environment.

### 1. Prerequisites

- Python 3.8 or higher
- pip (Python package manager)
- Virtual environment support

### 2. Navigate to Project Directory

```bash
cd C:\Users\prajw\Downloads\Walmart-Project-py-sql
```

### 3. Create Virtual Environment

```bash
python -m venv my_env
```

### 4. Activate Virtual Environment

**Windows (Command Prompt):**

```bash
my_env\Scripts\activate.bat
```

**Windows (PowerShell):**

```bash
my_env\Scripts\Activate.ps1
```

**Linux/Mac:**

```bash
source my_env/bin/activate
```

### 5. Install Dependencies

```bash
pip install --upgrade pip
pip install pandas numpy matplotlib seaborn jupyter sqlalchemy psycopg2-binary
```

### 6. Launch Jupyter Notebook

```bash
jupyter notebook
```

### 7. Open and Run Analysis

- Navigate to `prject.ipynb` in the Jupyter interface
- Run cells sequentially to perform analysis
- Refer to `SQL/Walmart Business Problems.pdf` for business questions

### 8. Deactivate Environment

```bash
deactivate
```

### For Detailed Instructions

See `INSTALLATION.txt` for comprehensive step-by-step guide with troubleshooting.

---

## Key Business Questions Solved

The following insights were derived using SQL queries (located in `analysis_queries.sql`).

### 1. Identify the busiest day for each branch

**Question:** Which day of the week has the highest number of transactions for each branch?

```sql
SELECT * FROM
(
	SELECT
		branch,
		TO_CHAR(TO_DATE(date, 'DD/MM/YY'), 'Day') as day_name,
		COUNT(*) as no_transactions,
		RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) as rank
	FROM walmart
	GROUP BY 1, 2
) as ranked_days
WHERE rank = 1;
```

### 2. Calculate total quantity sold per payment method

**Question:** What is the total quantity of items sold for each payment method?

```sql
SELECT
	payment_method,
	SUM(quantity) as no_qty_sold
FROM walmart
GROUP BY payment_method;
```

### 3. Determine average, minimum, and maximum ratings for each category by city

**Question:** Analyze the product ratings for each category in every city.

```sql
SELECT
	city,
	category,
	MIN(rating) as min_rating,
	MAX(rating) as max_rating,
	AVG(rating) as avg_rating
FROM walmart
GROUP BY 1, 2;
```

### 4. Analyze profit per category

**Question:** Which product category generates the highest profit?

```sql
SELECT
	category,
	SUM(total) as total_revenue,
	SUM(total * profit_margin) as profit
FROM walmart
GROUP BY category
ORDER BY profit DESC;
```

### 5. Identify the highest-rated category in each branch

**Question:** Which category has the highest average rating in each branch?

```sql
SELECT * FROM
(
	SELECT
		branch,
		category,
		AVG(rating) as avg_rating,
		RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) as rank
	FROM walmart
	GROUP BY 1, 2
) as ranked_categories
WHERE rank = 1;
```

### 6. Determine the most common payment method for each branch

**Question:** What is the most frequently used payment method at each branch?

```sql
SELECT * FROM
(
	SELECT
		branch,
		payment_method,
		COUNT(*) as total_trans,
		RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) as rank
	FROM walmart
	GROUP BY 1, 2
) as ranked_payment
WHERE rank = 1;
```

### 7. Analyze sales shifts (Morning/Afternoon/Evening)

**Question:** Categorize sales into time shifts and count invoices per shift.

```sql
SELECT
	branch,
	CASE
		WHEN EXTRACT(HOUR FROM time::time) < 12 THEN 'Morning'
		WHEN EXTRACT(HOUR FROM time::time) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END as shift,
	COUNT(*)
FROM walmart
GROUP BY 1, 2
ORDER BY 1, 3 DESC;
```

### 8. Find different payment methods and their transaction volume

**Question:** List all payment methods, the number of transactions, and the total quantity sold.

```sql
SELECT
	payment_method,
	COUNT(*) as no_payments,
	SUM(quantity) as no_qty_sold
FROM walmart
GROUP BY payment_method;
```

### 9. Identify the 5 branches with the highest revenue decrease ratio

**Question:** Which branches experienced the largest percentage decline in revenue from 2022 to 2023?

```sql
WITH revenue_2022 AS (
	SELECT
		branch,
		SUM(total) as revenue
	FROM walmart
	WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD/MM/YY')) = 2022
	GROUP BY 1
),
revenue_2023 AS (
	SELECT
		branch,
		SUM(total) as revenue
	FROM walmart
	WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD/MM/YY')) = 2023
	GROUP BY 1
)
SELECT
	ls.branch,
	ls.revenue as last_year_revenue,
	cs.revenue as cr_year_revenue,
	ROUND(
		(ls.revenue - cs.revenue)::numeric /
		ls.revenue::numeric * 100,
		2
	) as rev_dec_ratio
FROM revenue_2022 as ls
JOIN revenue_2023 as cs
ON ls.branch = cs.branch
WHERE
	ls.revenue > cs.revenue
ORDER BY 4 DESC
LIMIT 5;
```

---

## Project Structure

```text
Walmart-Project-py-sql/
├── my_env/                              # Python virtual environment
│   ├── Scripts/                         # Python executables
│   ├── Lib/                             # Python packages
│   └── pyvenv.cfg                       # Environment configuration
│
├── walmart-dataset/                     # Data folder
│   └── Walmart.csv                      # Main dataset
│
├── SQL/                                 # SQL files and documentation
│   ├── Walmart Business Problems.pdf    # Business problems documentation
│   └── walmart-py-sqlsql.sql            # SQL queries for analysis
│
├── INSTALLATION.txt                     # Setup & installation guide (plain text)
├── Walmart.md                           # This README file
├── prject.ipynb                         # Jupyter notebook for analysis
└── walmart-10k-sales-datasets.zip       # Backup dataset (compressed)
```

### File Descriptions

**prject.ipynb**

- Jupyter notebook containing Python data analysis
- Data cleaning, exploration, and visualization
- Integration with SQL queries

**SQL/walmart-py-sqlsql.sql**

- Contains all SQL queries for business analysis
- Database schema and table creation
- Advanced queries for insights

**SQL/Walmart Business Problems.pdf**

- Documentation of business problems
- Question descriptions and expected outputs
- Analysis requirements

**walmart-dataset/Walmart.csv**

- Main data source (raw data)
- Contains transaction records
- Used for loading into database

**INSTALLATION.txt**

- Comprehensive setup guide
- Step-by-step installation instructions
- Troubleshooting section
- Plain text format for easy reference

**my_env/**

- Isolated Python virtual environment
- Contains all project dependencies
- Python packages: pandas, numpy, jupyter, sqlalchemy, etc.

---
