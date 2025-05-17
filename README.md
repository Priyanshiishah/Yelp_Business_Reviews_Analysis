# 📈 Yelp Reviews - End-to-End Sentiment and Data Analytics Project

![Architecture](https://github.com/Priyanshiishah/Yelp_Business_Reviews_Analysis/blob/50e1ab28873575f22566159c3cee3e778603c23f/Workflow%20of%20Project.png).

## 📌 Project Overview

This end-to-end data analytics project analyzes **Yelp's open review dataset** (7 M+ reviews in 5GB JSON). The project covers:

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

### 2. Find the top 10 users who have reviewed the most businesses in the 'Restaurants' category
```sql
select r.user_id, count(distinct r.business_id) 
from tbl_yelp_reviews r
inner join tbl_yelp_businesses b 
on r.business_id=b.business_id
where b.categories ilike '%restaurant%'
group by 1
order by 2 desc
limit 10
```

### 3. Find the most popular categories of businesses (based on the number of reviews)
```sql
with cte as (
select business_id, trim(A.value) as category
from tbl_yelp_businesses
,lateral split_to_table(categories,',') A
)
select category, count(*) as no_of_reviews
from cte
inner join tbl_yelp_reviews r 
on cte.business_id=r.business_id
group by 1
order by 2 desc
```

### 4. Find the top 3 most recent reviews for each business
```sql
with cte as (
select r.*, b.name
, row_number() over(partition by r.business_id order by review_date desc) as rn
from tbl_yelp_reviews r
inner join tbl_yelp_businesses b 
on r.business_id=b.business_id
)
select * from cte
where rn<=3
```

 ### 5. Find the month with the highest number of reviews
```sql
select month(review_date) as review_month, count(*) as no_of_reviews
from tbl_yelp_reviews
group by 1
order by 2 desc
```

### 6. Find the percentage of 5-star reviews for each business
```sql
select b.business_id, b.name, count(*) as total_reviews
, sum(case when r.review_stars=5 then 1 else 0 end) as star5_reviews
, star5_reviews*100/total_reviews as percent_5_star
from tbl_yelp_reviews r
inner join tbl_yelp_businesses b 
on r.business_id=b.business_id
group by 1,2
```


### 7. Find the top 5 most reviewed businesses in each city
```sql
with cte as (
select b.city, b.business_id, b.name, count(*) as total_reviews
from tbl_yelp_reviews r
inner join tbl_yelp_businesses b 
on r.business_id=b.business_id
group by 1,2,3
)
select *
from cte
qualify row_number() over(partition by city order by total_reviews desc) <= 5
```

### 8. Find the average rating of businesses that have at least 100 reviews
```sql
select b.business_id, b.name, count(*) as total_reviews
, avg(review_stars) as avg_rating
from tbl_yelp_reviews r
inner join tbl_yelp_businesses b 
on r.business_id=b.business_id
group by 1,2
having count(*) >=100
```

### 9. List the top 10 users who have written the most reviews and the businesses they reviewed
```sql
with cte as (
select r.user_id, count(*) as total_reviews
from tbl_yelp_reviews r
inner join tbl_yelp_businesses b 
on r.business_id=b.business_id
group by 1
order by 2 desc
limit 10
)
select user_id, business_id
from tbl_yelp_reviews where user_id in (select user_id from cte)
group by 1,2
order by user_id
```

### 10. Find the top 10 businesses with the highest positive sentiment reviews
```sql
select r.business_id,b.name, count(*) as total_reviews
from tbl_yelp_reviews r
inner join tbl_yelp_businesses b 
on r.business_id=b.business_id
where sentiments = 'Positive'
group by 1,2
order by 3 desc
limit 10
```
