# 📈 Yelp Reviews - End-to-End Sentiment and Data Analytics Project

![Architecture](./Screenshot%202025-05-15%20135011.png)

## 📌 Project Overview

This is an end-to-end data analytics project analyzing **Yelp's open review dataset** (7M+ reviews in 5GB JSON). The project covers:

- Efficient file preprocessing using **Python**
- Data ingestion into **Amazon S3** and **Snowflake**
- Flattening nested JSON into tables
- **Sentiment Analysis** using UDFs
- **SQL-based data exploration** to derive business insights

---

## ⚙️ Tools & Technologies

- **Python** (File splitting, preprocessing)
- **Amazon S3** (Cloud storage)
- **Snowflake** (Data warehousing)
- **SQL** (Data querying & transformations)
- **UDFs** (User-defined functions for sentiment analysis)

---

## 🔄 Project Workflow

1. **JSON Preprocessing**  
   Split a 5GB `yelp_academic_dataset_review.json` file into smaller chunks using Python for efficient upload and processing.  
   📁 Refer to: [`split_files.py`](./split_files.py)

2. **Upload to S3**  
   Upload both business and review data to Amazon S3.

3. **Load into Snowflake**  
   Ingest S3 data into Snowflake and flatten nested JSON into structured tables.

4. **Apply Sentiment Analysis**  
   Use Snowflake UDF to classify each review as Positive, Neutral, or Negative.  
   📁 Refer to: [`UDF and tables.sql`](./UDF and tables.sql)

5. **Data Exploration & Insights**  
   Perform SQL queries to answer key business questions from the Yelp dataset.

---

## 📊 Top 10 Insights (via SQL)

### 1. Number of businesses in each category
```sql
with cte as (
  select business_id, trim(A.value) as category
  from tbl_yelp_businesses, lateral split_to_table(categories,',') A
)
select category, count(*) as no_of_business
from cte
group by 1
order by 2 desc
```
