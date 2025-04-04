# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Logo](NetflixLogo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

##  Objectives
<ul>
  <li>Analyze the distribution of content types (movies vs TV shows).</li>
  <li>Identify the most common ratings for movies and TV shows.</li>
  <li>List and analyze content based on release years, countries, and durations.</li>
  <li>xplore and categorize content based on specific criteria and keywords.</li>
</ul>

## Dataset 
The data for this project is sourced from the Kaggle dataset:
<ul>
  <li>Dataset Link: <a href="https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download">Netflix DataSet</a></li>
</ul>

## Schema 
```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows
```sql
SELECT 
		type,
		COUNT(*) as total_content
	FROM netflix 
	GROUP BY type
```
**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows
```sql
SELECT 
		s.type,
		s.rating as highest_ranking
	FROM (
		SELECT 
			type,
			rating,
			COUNT(*) ,
			RANK() OVER(PARTITION by type ORDER BY COUNT(*) desc ) as ranking
		FROM netflix
		GROUP BY 1,2
		) as s
	WHERE s.ranking = 1;
```
**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT * 
	FROM
	(
		SELECT 
		-- country,
			TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) as country,
			COUNT(*) as total_content
		FROM netflix
		GROUP BY 1
	)as t1
	WHERE country IS NOT NULL
	ORDER BY total_content DESC
	LIMIT 5
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
		
	FROM netflix
	WHERE 	
		type = 'TV Show' 
		AND 
		COALESCE(SPLIT_PART(duration, ' ', 1) :: numeric,0) > 5
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT genre,COUNT(*)
	FROM (
		SELECT 
			TRIM(UNNEST(STRING_TO_ARRAY(listed_in, ','))) as genre
		FROM netflix 
	) 
	GROUP BY 1
	ORDER BY 2 desc
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. Return top 5 year with highest avg content release!

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:** Calculate and rank years by the average number of content releases by India.




