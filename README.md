# Netflix movies and Tv Shows Data Analysis using SQL
![Netflix Logo](https://github.com/SahanaGolbhavi/sql_Project-on-Netflix-Dataset/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objective
Analyze the distribution of content types (movies vs TV shows).
Identify the most common ratings for movies and TV shows.
List and analyze content based on release years, countries, and durations.
Explore and categorize content based on specific criteria and keywords.

## DataSet
The data for this project is sourced from the Kaggle dataset:
Dataset Link:(https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

##Schema
```sql
CREATE TABLE IF NOT EXISTS Netflix_Tab(
    show_id	varchar(10),
    type varchar(10),
    title varchar(150),
    director varchar(210),
    cast varchar(1000),	
    country	varchar(150),
    date_added	varchar(50),
    release_year INT,
    rating varchar(10),
    duration varchar(15),
    listed_in varchar(150),
    description varchar(250));
```


### 1 Count the number of Movies and TV shows

sql```
SELECT 
	type,
    count(*) as total_content 
FROM Netflix_TAB
GROUP BY type;
```


### 2  Find the most common rating for movies and TV shows

sql```
SELECT 
	type,
    rating
FROM
(
SELECT
	type ,
    rating,
    COUNT(*),
    RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
FROM Netflix_Tab 
GROUP BY 1, 2
) as t1
WHERE ranking = 1;
```


### 3 List all movies released in a specific year

sql```
SELECT * FROM Netflix_Tab 
WHERE
	type = 'Movie'
    AND
 release_year = 2020;
 ```


 ### 4 Find the top 5 countries with the most content on Netflix
 
sql```
SELECT 
    TRIM(value) AS new_country, 
    COUNT(show_id) AS total_content
FROM NETFLIX_TAB, 
LATERAL FLATTEN(input => SPLIT(country, ','))
GROUP BY new_country
ORDER BY total_content DESC
LIMIT 5;
```


### 5 Identify the longest movie

sql```
SELECT * FROM Netflix_Tab 
WHERE
	type = 'Movie'
	AND
duration = (SELECT MAX(duration) FROM Netflix_Tab);
```


### 6 Find content added in last 5 years

sql```
SELECT *  
FROM Netflix_Tab  
WHERE date_added_clean >= CURRENT_DATE - INTERVAL '5 years';
```
    
### 7 Find all the movies / Tv shows by director 'Toshiya Shinohara'

sql ```
SELECT *
FROM netflix_tab 
WHERE director LIKE '%Toshiya Shinohara%';
```

### 8 List all TV shows with more than 5 seasons

sql```
SELECT * FROM netflix_tab 
WHERE 
	type = 'Tv Show'
	AND
duration > '5 sessons';
```

### 9 Count the number of content items in each genre

sql```
SELECT 
    TRIM(value) AS genre,
    COUNT(show_id) AS total_content
FROM netflix_tab,
    LATERAL FLATTEN(input => SPLIT(listed_in, ','))
GROUP BY genre;
```


### 10 Find each year and the 

sql```
SELECT 
    EXTRACT(YEAR FROM date_added_clean) AS year,
    COUNT(*) AS yearly_content,
    ROUND(
        COUNT(*)::NUMERIC / 
        NULLIF((SELECT COUNT(*) FROM netflix_tab WHERE country = 'India'), 0) * 100, 2
    ) AS avg_content_per_year
FROM netflix_tab
WHERE country = 'India'
AND date_added_clean IS NOT NULL
GROUP BY year
ORDER BY year;
```

### 11 List all movies that are documentaries

sql```
SELECT * FROM netflix_tab
WHERE
	listed_in LIKE '%Documentaries%' ;
```

### 12 Find all the content  without director

sql```
SELECT * FROM netflix_tab
WHERE
	director IS NULL ; 
```


### 13 Find how many movies actor 'Salman Khan' appeared in last 10 years

```sql
SELECT * FROM netflix_tab
WHERE
	cast LIKE '%Salman khan%'
    AND
    release_year > EXTRACT(YEAR FROM current_date) - 10; 
```

### 14 Find the top 10 actors who have appeared in the highest number of movies produced in India

```sql
SELECT 
    TRIM(actor.value) AS actor,  
    COUNT(*) AS total_content
FROM netflix_tab, 
    LATERAL FLATTEN(INPUT => SPLIT(cast, ',')) AS actor  -- Corrected syntax
WHERE country ILIKE '%India%'
GROUP BY actor
ORDER BY total_content DESC
LIMIT 10;
```


###15 Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label the content contaning these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category

```sql
WITH new_table
AS
(
SELECT *,
    CASE
        WHEN description ILIKE '%kills%' 
            OR description ILIKE '%violence%' 
        THEN 'Bad_Content'
        ELSE 'Safe_Content'  
    END AS category
FROM netflix_tab
)
SELECT 
    category,
    COUNT(*) as total_content
FROM new_table
GROUP BY 1;
```



