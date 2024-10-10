# Netflix Movies and TV Shows: Data Analysis using SQL

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset
- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema
```
CREATE TABLE Netflix
 ( 
  show_id			varchar(10),
  type				varchar(20),
  title				varchar(150),
  director			varchar(300),
  casts				varchar(1000),
  country			varchar(200),			
  date_added		varchar(50),
  release_year	    int,
  rating			varchar(20),
  duration			varchar(20),
  listed_in			varchar(150),
  description		varchar(300)
 );
 ```

-- If not loading automatically(by clicking), do this manually
	COPY Netflix FROM 'S:\Downloads\netflix movie and tvshows.csv' DELIMITER ',' CSV HEADER;
	
```SELECT * FROM Netflix;```

```SELECT count(*) as total_count
FROM Netflix```




## BUSINESS PROBLEMS AND SOLUTIONS :-


### 1.Count the number of Movies vs TV Shows.
```SELECT type, count(*) as Total
FROM Netflix
GROUP BY type;```

### .Find the most common rating for movies and TV shows.

'''SELECT rating, count(*) as total_count
FROM Netflix
GROUP BY rating
ORDER BY total_count DESC
Limit 1;
'''

### 3. List all movies released in a specific year (eg. 2020)

'''SELECT *
FROM Netflix
WHERE 
	type= 'Movie' 
	AND 
	release_year=2020;
'''

### 4. Find the top 5 countries with the most content on Netflix
	
'''SELECT country, count(*) as total_content
FROM Netflix
GROUP BY country
ORDER BY total_content DESC
LIMIT 5;
'''
- It would work BUT some of the content is made by multiple countries, which sql is identifying whole as one. 

- So,
'''SELECT 
	unnest( string_to_array (country,',') ) as new_country,
	-- string_to_array (string,"delimeter")
	--^^Here we convert country_string to array and then unnest the array. Volla! we get individual countries^^--
	count(*) as total_content
FROM Netflix
GROUP BY new_country
ORDER BY total_content DESC
Limit 5;
'''

### 5. Identify the longest movie.

'''SELECT title,duration 
FROM Netflix
WHERE type='Movie' AND duration=(SELECT MAX(duration) FROM Netflix);
'''
- It would work BUT duration is giving in the string format

- So,
'''SELECT title, 
	substring(duration, 1 , position('m' in duration)-1 ) ::int duration
	--substring(string, starting position, ending position)
	--^^We extract string_number part from Given String and typecast into a integer^^--
FROM Netflix
WHERE type='Movie'
ORDER BY duration 
LIMIT 1;
'''

### 6. Find the content added in last 5 years.

''' SELECT title, date_added
FROM Netflix
WHERE
	 To_Date(date_added,'Month,DD,YYYY') >= current_date-interval '5 years'
  '''
- To_Date: Coverts string_data to Date Format
- INTERVAL '5 years': Specifies a time interval of 5 years.
- CURRENT_DATE - INTERVAL '5 years': Subtracts 5 years from the current date.


### 7.Find all the Movies/TV Shows by director 'Rajiv Chilaka'

'''SELECT title
FROM Netflix
WHERE director ilike '%Rajiv Chilaka%';
'''
- search the name case insensitively
- %: because their are the cells with multiple directors


### 8. List all TV shows with more than 5 seasons

 '''SELECT title, duration 
 FROM Netflix
 WHERE type= 'TV Show' 
 	   AND
	   substring(duration, 1 , position('S' in duration)-1 ) ::int >5;
'''
- OR

'''SELECT title,duration
FROM Netflix
WHERE type='TV Show'
	  AND
	  split_part(duration,' ',1):: int >5;
'''

### 9. Count the number of content items in each genre

'''SELECT
		unnest(string_to_array(listed_in,',')) as genre, 
		count(*) as total_content
FROM Netflix
GROUP BY genre;
'''

### 10. Find each year and the average numbers of content release by India on Netflix. Return top 5 year with highest avg content release.

'''SELECT 
	--right(date_added,4) as release_on_netflix,
	EXTRACT(year from  to_date(date_added, 'Month, DD, YYYY') ) as release_on_netflix,
	count(*) as total_content,
	round( count(*)::numeric / (SELECT count(*) FROM Netflix WHERE country ilike '%India%')::numeric *100 ) as avg_content  
FROM Netflix
WHERE country ilike '%India%'
GROUP BY release_on_netflix
ORDER BY avg_content desc
LIMIT 5;
'''

### 11.List all movies that are documentries.

'''SELECT title, listed_in
FROM Netflix
WHERE 
     type='Movie'
	 AND
	 listed_in ilike '%Documentaries%'
'''	 
	 
### 12. Find all content without a director

'''SELECT title
FROM Netflix
WHERE director is Null
'''

### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years

'''SELECT title,casts,release_year
FROM Netflix
WHERE type='Movie'
	  AND
	  casts ilike '%Salman Khan%'
	  AND
      release_year > EXTRACT(year from current_date)-10
'''  

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India

'''SELECT 
	unnest(string_to_array (casts,',') ) as actor,
	count(*) as total_movies
FROM Netflix
WHERE country ilike '%India%'
	  AND
	  type='Movie'
GROUP BY actor
order by total_movies desc
LIMIT 10;
'''

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content 
containing theses keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.

'''WITH new_table
AS
(
  SELECT title,description,

  CASE
      WHEN description ilike '%kill%' or description ilike '%violence%'  
	  THEN 'Bad Content'
	  ELSE 'Good Content'
  END category

  FROM NetfliX
)

SELECT category, count(*) as total_content
FROM new_table
GROUP BY category;
'''

## Findings and Conclusion
Content Distribution: The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
Common Ratings: Insights into the most common ratings provide an understanding of the content's target audience.
Geographical Insights: The top countries and the average content releases by India highlight regional content distribution.
Content Categorization: Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.
This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
