# Spotify Analysis Project with Query Optimisation

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves various steps starting from basic cleaning of data using **excel** and then analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, uncovering valuable insights based on different problem statements by performing SQL queries and optimizing query performance. The primary goals of the project is to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
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
## Project Steps

### 1. EDA
Before diving into SQL, it’s important to understand the dataset thoroughly by knowing different attributes
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.
Various queries involved are:-
```sql
select count(*) from spotify;
select count(distinct artist) from spotify;
select count(distinct album) from spotify;
select distinct album_type from spotify;
select max(duration_min) from spotify;
select min(duration_min) from spotify;
select count(distinct channel) from spotify;
select distinct most_played_on from spotify;
```

### 2. Removing inconsistencies within the data
After getting familiar ith the data, removing inconsistencies which can hinder the calculations
```sql
delete from spotify
where duration_min=0;
```

#### Problem statement set-1
- Retrieve the names of all tracks that have more than 1 billion streams.
- List all albums along with their respective artists.
- Get the total number of comments for tracks where licensed = TRUE.
- Find all tracks that belong to the album type single.
- Count the total number of tracks by each artist.

```sql
-- P1
select track from spotify 
where stream>1000000000;
-- P2
select distinct album,artist from spotify
order by 1;
-- P3
select sum(comments) as total_comments 
from spotify
where licensed='true';
-- P4
select track from spotify 
where album_type= 'single';
-- P5
select artist,
count(track) as total_songs_by_artist
from spotify
group by 1
order by 2;
```
    
####  Problem statement set-2
- Calculate the average danceability of tracks in each album.
- Find the top 5 tracks with the highest energy values.
- List all tracks along with their views and likes where official_video = TRUE.
- For each album, calculate the total views of all associated tracks.
- Retrieve the track names that have been streamed on Spotify more than YouTube.

```sql
-- P1
select album, 
avg(danceability) as avg_danceability
from spotify
group by 1
order by 2 desc;
-- P2
select track,avg(energy) 
from spotify 
group by 1
order by 2 desc
limit 5;
-- P3
select track,
sum(views) as total_views, 
sum(likes) as total_likes
 from spotify
where official_video= 'true'
group by 1;
-- P4
select album,track, sum(views) from spotify
group by 1,2
order by 3 desc;
-- P5
with my_cte as (
select track,
coalesce(sum(case when most_played_on='Spotify' then stream end),0) as counts_on_spotify,
coalesce(sum(case when most_played_on='Youtube' then stream end),0) as counts_on_youtube
from spotify
group by 1
)
select track from my_cte
where counts_on_spotify>counts_on_youtube
and
counts_on_youtube!=0;
```
  
#### Problem statement set-3
- Find the top 3 most-viewed tracks for each artist.
- Write a query to find tracks where the liveness score is above the average.
- Calculate the difference between the highest and lowest energy values for tracks in each album.

```sql
-- P1
with my_cte1 as (
select distinct artist,
track,
sum(views) over(partition by artist,track) as final_views from spotify
),
my_cte2 as (
select artist,
track,
dense_rank() over(partition by artist order by final_views desc) as dr
from my_cte1
)
select artist,track from my_cte2
where dr in(1,2,3);
-- P2
select track,liveness from spotify 
where liveness > (select avg(liveness) from spotify);
--P3
with my_cte as (
select distinct album,track,energy,
max(energy) over(partition by album) as max_level,
min(energy) over(partition by album) as min_level
from spotify
)
select distinct album,
max_level-min_level as energy_difference
from my_cte
order by 2 desc;
```

###  Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---



Here’s an updated section for the **Spotify Analysis Project with Query Optimization** focusing on the query optimization performed. 
---

## Query Optimization Technique 

To improve query performance, carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - Began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, ran the same query again and observed significant improvements in performance:
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
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools**: pgAdmin 4 


---


