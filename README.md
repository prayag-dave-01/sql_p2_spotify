# Spotify Advanced SQL Project and Query Optimization P-2

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Project Overview

**Project Title**: Retail Sales Analysis  
**Level**: Intermediate
**Database**: `p2_spotify_db`

This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to showcase my SQL skills and generate valuable insights from the dataset.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p2_spotify_db`.
- **Table Creation**: A table named `spotify` is created to store the sales data. The table structure includes columns for Artist, Track, Album, Album_type, Danceability, Energy, Loudness, Speechiness, Acousticness, Instrumentalness, Liveness, Valence, Tempo, Duration_min, Title, Channel, Views, Likes, Comment, Licensed, official_video, Stream, Energy, Liveness,and most_playedon

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```

### 2. Data Exploration & Cleaning
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

```sql
SELECT COUNT(*) FROM spotify;

SELECT COUNT(DISTINCT artist) FROM spotify;

SELECT DISTINCT album_type FROM spotify;

SELECT MAX(duration_min) FROM spotify;

SELECT MIN(duration_min) FROM spotify;

SELECT * FROM spotify
WHERE duration_min = 0;

DELETE FROM spotify
WHERE duration_min = 0;
SELECT * FROM spotify
WHERE duration_min = 0;

SELECT DISTINCT channel FROM spotify;

SELECT DISTINCT most_played_on FROM spotify;
```

### 3. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into **easy**, **medium**, and **advanced** levels to help progressively develop SQL proficiency.

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.
  
### 4. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

### Easy Level
1. Retrieve the names of all tracks that have more than 1 billion streams.
2. List all albums along with their respective artists.
3. Get the total number of comments for tracks where `licensed = TRUE`.
4. Find all tracks that belong to the album type `single`.
5. Count the total number of tracks by each artist.

---

1. **Retrieve the names of all tracks that have more than 1 billion streams.**:
```sql
SELECT track
FROM spotify
WHERE stream > 1000000000
```

2. **List all albums along with their respective artists.**:
```sql
SELECT 
     DISTINCT album, 
	 artist
FROM spotify
ORDER BY 1
```

3. **Get the total number of comments for tracks where licensed = TRUE.**:
```sql
SELECT 
    SUM(comments) total_comments
FROM spotify
WHERE licensed = 'true'
```

4. **Find all tracks that belong to the album type single.**:
```sql
SELECT track
FROM spotify
WHERE album_type = 'single'
```

5. **Count the total number of tracks by each artist from highest to lowest.**:
```sql
SELECT 
     artist, 
	 COUNT(*) as total_no_tracks
FROM spotify
GROUP BY 1
ORDER BY 2
```

### Medium Level
1. Calculate the average danceability of tracks in each album.
2. Find the top 5 tracks with the highest energy values.
3. List all tracks along with their views and likes where `official_video = TRUE`.
4. For each album, calculate the total views of all associated tracks.
5. Retrieve the track names that have been streamed on Spotify more than YouTube.

---

1. **Calculate the average danceability of tracks in each album.**:
```sql
SELECT 
     album,
     track,
	 AVG(danceability) as average_danceability
FROM spotify
GROUP BY album, track
```

2. **Find the top 5 tracks with the highest energy values.**:
```sql
SELECT 
    track, 
	energy
FROM spotify 
ORDER BY energy DESC
LIMIT 5
```

3. **List all tracks along with their total views and total likes where official_video = TRUE (List views in descending order).**:
```sql
SELECT 
    track,
	SUM(views) as total_views ,
	SUM(likes) as total_likes
FROM spotify
WHERE official_video = 'true'
GROUP BY 1
ORDER BY 2 DESC
```

4. **For each album, calculate the total views of all associated tracks and list them in desceding order of views.**:
```sql
SELECT
    album,
	track,
    SUM(views) as total_views
FROM spotify
GROUP BY 1, 2
ORDER by 3 DESC
```

5. **Find tracks where the energy-to-liveness ratio is greater than 1.2 and and display top 20.**:
```sql
SELECT track,
       energy / liveness as ratio
FROM spotify
WHERE energy / liveness > 1.2
ORDER BY 2 DESC
LIMIT 20
```

### Advanced Level
1. Find the top 3 most-viewed tracks for each artist using window functions.
2. Write a query to find tracks where the liveness score is above the average.
3. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.
4. Find tracks where the energy-to-liveness ratio is greater than 1.2.
5. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.

---

1. **Find the top 3 most-viewed tracks for each artist using window functions and CTE.**:
```sql
WITH ranking_artist
As
(
  SELECT 
	   artist,
	   track,
	   SUM(views) as total_views,
	   DENSE_RANK() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) as rank
  FROM spotify
  GROUP BY 1, 2
  ORDER BY 1
)
  SELECT * FROM ranking_artist
     WHERE rank <= 3
```

2. **Write a query to find tracks where the liveness score is above the average.**:
```sql
SELECT track
FROM spotify
WHERE 
   liveness > (SELECT AVG(liveness) FROM spotify)
```

3. **Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album. Fetch energy difference in descending order - skip first 5 records and get top 10 further.**:
```sql
WITH CTE
As
(
 SELECT 
     album,
	 MAX(energy) as highest_energy,
	 MIN(energy) as lowest_energy
 FROM spotify
 GROUP BY 1
)
 SELECT
     album,
	 highest_energy - lowest_energy as energy_difference
 FROM CTE
 ORDER BY 2 DESC
 LIMIT 10 OFFSET 5
```

4. **Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions. Exclude tracks with 0 likes.**:
```sql
SELECT
    track,
	views,
	likes,
	SUM(likes) OVER (ORDER BY views) as cumulative_likes
FROM spotify
WHERE likes > 0
```

5. **Retrieve the track names that have been streamed on Spotify more than YouTube.**:
```sql
SELECT * FROM
(
SELECT 
    track,
	COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' then stream END),0) as streamed_on_youtube,
	COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' then stream END), 0) as streamed_on_spotify
FROM spotify
GROUP BY 1
) as t1
  WHERE 
     streamed_on_spotify > streamed_on_youtube
	 AND
	 streamed_on_youtube <> 0
```   

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Subqueries, Window Functions
- **Tools**: pgAdmin 4 (used) (or any SQL editor), PostgreSQL (via Homebrew, Docker, or direct installation)

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding platform usage, customer choice of songs, best performing artists, comparison with other music platforms.

## Author - Prayag Dave

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/prayag-dave-56b3681b3)
- **Mail**: prayagdavework@gmail.com

Thank you for your support, and I look forward to connecting with you!

## License
This project is licensed under the MIT License.
