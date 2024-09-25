# Spotify SQL Project
![](https://github.com/mina407/Spotify/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using SQL. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

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

## Project Steps

#### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:

* Artist: The performer of the track.
* `Track`: The name of the song.
* `Album`: The album to which the track belongs.
* `Album_type`: The type of album (e.g., single or album).
* Various metrics such as danceability, energy, loudness, tempo, and more.
#### 2. Querying the Data

### EDA
```sql
-- Number of Rows 
select count(*) from spotify ;

-- How many artist ?
select count(distinct artist) from spotify ;

-- How many album ?
select count(distinct album) from spotify;

-- Kind Of Album
select distinct album_type from spotify ;

-- Min and Max Duration 
select * 
from (select max(duration_min) from spotify) as max_duration 
cross join(select min(duration_min) from spotify) as min_duration ;

-- findings 
-- min value of duration is 0 let's check those songs
select * from spotify
where duration_min = 0 ;
-- There are tow song have 0 duration we need to delete those songs
DELETE from spotify
where duration_min = 0;

-- How many Channel ? 
select distinct channel from spotify ;

-- most_played_on apps
select distinct most_played_on from spotify ;
```
### Data Analysis

* Retrieve the names of all tracks that have more than 1 billion streams.
```sql
select * from spotify 
where stream > 1000000000 ; 
```
*  List all albums along with their respective artists.
```sql
select distinct album , artist 
from spotify 
order by 1;
```

* Get the total number of comments for tracks where licensed = TRUE.
```sql

select distinct licensed from spotify ;

select sum(comments) as total_number
from spotify 
where licensed = 'true' ; 
```

* Find all tracks that belong to the album type single.
```sql

select track from spotify 
where album_type ilike 'single'
```
* Count the total number of tracks by each artist.
```sql
select artist , count(track) as number_tracks
from spotify 
group by artist
order by 2 desc;
```

* Calculate the average danceability of tracks in each album.
```sql
select 
	album ,
	avg(danceability) as avg_danceability
from spotify
group by 1
order by 2 desc;
```
* Find the top 5 tracks with the highest energy values.
```sql
select  track , 
	max(energy)
from spotify
group by track
order by 2 desc
limit 5 ;
```

* List all tracks along with their views and likes where official_video = TRUE.
```sql
select 
	track ,
	sum(views) as total_views ,
	sum(likes) as total_likes
from spotify
where official_video = 'true'
group by track 
order by total_views desc;
```

* For each album, calculate the total views of all associated tracks.
```sql
select 
	album ,
	track ,
	sum(views) as total_views
from spotify 
group by 1,2
order by total_views desc;
```
* Retrieve the track names that have been streamed on Spotify more than YouTube.
```sql
select * from
			(select 
				track ,
				COALESCE(sum(case when most_played_on = 'Youtube' then stream end) , 0) as streamed_on_youtube ,
				coalesce(sum(case when most_played_on = 'Spotify' then stream end), 0) as streamed_on_spotify
			from spotify
			GROUP by 1 ) as temp_
where streamed_on_spotify > streamed_on_youtube
and streamed_on_youtube <> 0
```
* Find the top 3 most-viewed tracks for each artist using window functions.
```sql
select * from 
		(select artist , 
				track ,
				sum(views) as total_views ,
				dense_rank() over(partition by artist order by sum(views) desc) as rnk
		from spotify 
		group by artist , track 
		) as temp_
where rnk <=3 ; 
```

* Write a query to find tracks where the liveness score is above the average.
```sql
select track ,
		artist ,
		liveness
from spotify
where liveness > (select avg(liveness) from spotify);
```
* Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.
```sql
with cte
as
	(select album ,
		max(energy) as highest ,
		min(energy) as lowest
	from spotify
	GROUP by album  
	)
select * ,
	(highest - lowest) as difference
from cte
order by difference desc; 
```

---
## Query Optimization Technique

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
	-  We began by analyzing the performance of a query using the `EXPLAIN` function.
 		- Execution time (E.T.): **10.915 ms**
   		- Planning time (P.T.): **0.72 ms**
     - - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/mina407/Spotify/blob/main/befor%20optimization.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.087 ms**
        - Planning time (P.T.): **0.144 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/mina407/Spotify/blob/main/after%20optimization.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/mina407/Spotify/blob/main/1.png)
      ![Performance Graph]()
      ![Performance Graph]()

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---


