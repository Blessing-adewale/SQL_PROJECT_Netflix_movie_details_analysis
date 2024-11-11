# SQL_PROJECT_Netflix_movie_details_analysis

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS movie_details;
CREATE TABLE movie_details
(
show_id	VARCHAR(15) PRIMARY KEY,
type VARCHAR(15),	
title VARCHAR(150),
director VARCHAR(220),
casts VARCHAR(1000),
country	VARCHAR(200),
date_added VARCHAR(50),
release_year INT,	
rating	VARCHAR(40),
duration VARCHAR(50),
listed_in VARCHAR(100),
description VARCHAR(250)
);
```
## Data Exploration
```sql
SELECT * FROM movie_details;

SELECT COUNT(*) AS total_content 
FROM movie_details;

SELECT DISTINCT type 
FROM movie_details;
```
## Business Problems and Solutions

## 1. Count the number of Movies vs TV Shows
```sql
SELECT  type,
COUNT(*) AS total
FROM movie_details
GROUP BY type;
```
## 2. Find the most common rating for movies and TV shows
```sql
WITH HIGH_RATING AS 
(SELECT  type, 
rating, 
count(rating) as total_ratings,
RANK() OVER(PARTITION BY type ORDER BY count(rating)desc) AS ratings
FROM	movie_details
GROUP BY 1,2
)
SELECT * FROM high_rating
WHERE ratings = 1;
```
### 3. List all movies released in a year 2020
```sql
SELECT * FROM movie_details
WHERE type = 'Movie'
AND release_year = 2020;
```	
## 4. Find the top 5 countries with the most content on Netflix
```sql
SELECT unnest(string_to_array(country,',')) AS country,
	COUNT(*) AS total_contents 
FROM movie_details
GROUP BY country
ORDER BY 2 DESC
LIMIT 5;
```
## 5. Identify the longest movie
```sql
SELECT * FROM movie_details
WHERE type='Movie' AND duration is not null
order by split_part(duration,' ',1)::int desc
```
## 6. Find content added in the last 5 years
```sql
SELECT * FROM movie_details
where TO_DATE(date_added,'Month DD,YYYY')>= CURRENT_DATE - INTERVAL '5 years'
```
## 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
SELECT * FROM movie_details
WHERE director LIKE '%Rajiv Chilaka%';

OR 

SELECT * FROM
	(SELECT *, 
	UNNEST(STRING_TO_ARRAY(director,',')) AS director_name
	FROM movie_details)
WHERE director_name = 'Rajiv Chilaka';
```
## 8. List all TV shows with more than 5 seasons
```sql
SELECT * FROM movie_details
WHERE TYPE = 'TV Show' 
AND SPLIT_PART(duration,' ',1)::INT > 5;
```
## 9. Count the number of content items in each genre
```sql
SELECT  genre,
COUNT(*) AS total_content
FROM
	(SELECT*,
	TRIM(UNNEST(STRING_TO_ARRAY(listed_in,','))) AS genre
	FROM movie_details)
GROUP BY genre;
```
## 10.Find each year and the average numbers of content release in India on netflix. return top 5 year with highest avg content release!
```sql
SELECT  country,
release_year,
COUNT(show_id) AS total_content,
ROUND(COUNT(show_id)::NUMERIC/(SELECT COUNT(show_id) FROM movie_details WHERE country = 'India')::NUMERIC * 100,2) 
AS avg_content
FROM movie_details
WHERE country = 'India'
GROUP BY 1,2
ORDER BY avg_content DESC
LIMIT 5;
```
## 11. List all movies that are documentaries
```sql
SELECT * FROM movie_details
WHERE listed_in ILIKE '%documentaries%'
```
## 12. Find all content without a director
```sql
SELECT * FROM movie_details
WHERE director is null;
```
## 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
SELECT * FROM movie_details
WHERE release_year>EXTRACT(YEAR FROM current_date) - 10
AND casts ILIKE '%Salman Khan%';
```
## 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
SELECT country,
	TRIM(UNNEST(STRING_TO_ARRAY(casts,','))) AS actors,
	COUNT(*) AS no_of_movies
FROM movie_details
WHERE country ILIKE '%India%'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 10;
```		
## 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how manyitems fall into each category.
```sql
SELECT  category, 
COUNT(*)
FROM
	(SELECT *,
	CASE
	WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
	ELSE 'Good'
	END AS category
	FROM movie_details)
GROUP BY category;
```

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

This project is part of my portfolio, showcasing my SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!
