# Netflix Movies & TV Shows — SQL Data Analysis Project

This project analyzes over 8,000 Netflix titles using SQL to uncover content patterns, audience trends, and global production behaviors. It demonstrates end-to-end analytical skills including data cleaning, querying, transformation, segmentation, and insight generation.

---

## Dataset

Source: Kaggle  
File included: netflix_titles.csv

Fields include:
- show_id  
- type (Movie or TV Show)  
- title  
- director  
- cast  
- country  
- date_added  
- release_year  
- rating  
- duration  
- listed_in  
- description  

---

## Tech Stack

- SQL (PostgreSQL / MySQL)
- Data cleaning and preprocessing
- Window functions and aggregations
- Business insight extraction

---

## How to Run This Project

1. Create the table:

```sql
CREATE TABLE netflixdb
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
2.Import netflix_titles.csv into netflixdb.

3.Run the SQL queries below.

## Business Problems and Solutions

### 1. Compare the quantity of films to television series.

```sql
SELECT 
    type,
    COUNT(*)
FROM netflixdb
GROUP BY 1;
```

**Objective:** Identifying Netflix's content type distribution.

### 2. Find the most popular movie and television program ratings.

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflixdb
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

**Objective:** Determine the most frequently used rating for each type of content.

### 3. List every film that was released in a given year, such as 2020.

```sql
SELECT * 
FROM netflixdb
WHERE release_year = 2020;
```

**Objective:** Get every film that was released in a given year.

### 4. Discover the five countries with the most Netflix content.


```sql
SELECT * FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflixdb
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:** Identify the top 5 countries with highest content .

### 5. Identify the longest movie.

```sql
SELECT 
    *
FROM netflixdb
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Objective:** Find the movie with the longest duration.

### 6. Learn what has been added in the past five years.


```sql
SELECT *
FROM netflixdb
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find all movies/tv shows by director 'Rajiv Chilaka'

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflixdb
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List all tv shows with more than 5 seasons

```sql
SELECT *
FROM netflixdb
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Objective:** Identify tv shows with more than 5 seasons.

### 9. Determine the amount of content items in each genre.



```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflixdb
GROUP BY 1;
```

**Objective:** Count the number of items in each genre.

### 10. Find out how much content is released on Netflix in India each year and on average.
return top 5 years with highest avg content release!

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflixdb
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List all movies that are documentaries

```sql
SELECT * 
FROM netflixdb
WHERE listed_in LIKE '%Documentaries';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find all content without a director

```sql
SELECT * 
FROM netflixdb
WHERE director IS NULL;
```

**Objective:** List content that does not have a director.

### 13. Determine in how many movies 'Salman Khan' has been an actor in the last ten years.

```sql
SELECT * 
FROM netflixdb
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflixdb
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian movies.

### 15. Categorize content based on the presence of 'Kill' and 'Violence' keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflixdb
) AS categorized_content
GROUP BY category;
```

**Objective:** Categorize content as 'A' if it contains 'kill' or 'violence' and 'U/A' otherwise. Count the number of items in each category.

##  Key Business Insights:

. Content Dominance – Movies dominate Netflix’s library, but TV shows are growing.

. Rating & Genre Trends – Netflix content skews towards specific genres and age ratings.

. Global Expansion – The USA, India, and the UK lead in content availability.

. Indian Content Growth – Massive growth in Indian content releases in recent years.

. Actor & Director Influence – Certain actors and directors appear significantly more.

. Content Categorization –  Filtering can help remove violent content for safer viewing.



