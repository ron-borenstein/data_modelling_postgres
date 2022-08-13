# Project Intro

Sparkify is a new music streaming app. The goal of this project is to build a new database with tables designed to optimize queries on song play analysis.

## Python Scripts

There are two python scripts that are needed to run for this project. To run them, open a terminal and use the following commands in this order:

1. `python create_tables.py` - creates the database, drops any old tables and creates all tables
2. `python etl.py` - loads and processes the raw data from json files and inserts into the relevant tables

## Files in the repo

1. `sql_queries.py` - all sql queries used to create and drop tables, and to insert the data into the tables
2. `test.ipynb` - a jupyter notebook to be used in order to check that all tables were created with no obvious data integriy issues

#### `data` directory

1. `song_data` - a directory with json files containing data about individual songs. Used to create the dimenstion files for songs & artists
2. `log_data` - raw streaming data stored in json files. Used to create the songplays fact table, and time and users dimenstion tables

## Database schema and ETL pipeline

This is a star schema database. It has one fact table, `songplays`, which represents music streaming events in sequence. It has several dimension tables enriching the data:
1. `songs` - information on each song
2. `artists` - information on each artist
3. `users` -  information on each user
4. `time` - the event timestamp parsed into different time frames

The ETL pipeline first processes the Song data files to create the dimension table for songs and artists. Then it processes the log files, extracts user & time information into the dimension tables and creates the fact table.

As part of the ETL creation of the fact table, the query tries to match artist information by looking at the dimension tables for matches on song title, artist name and song duration in order to have song_id and artist_id as foreign keys in the fact table. 

Currently only one song in the fact table has a song_id and artist_id.


## Example queries

1. Get users' total listening time

```
SELECT 
    users.*, 
    sum(duration) 
from users join songplays on users.user_id = songplays.user_id 
join songs on songs.song_id = songplays.song_id 
group by users.user_id, first_name,last_name,gender,level;
```

2. Get a list of songs played by order with user information, song name, duration, and artist name & location:

```
SELECT 
    songplays.songplay_id, 
    songplays.start_time,
    users.first_name,
    users.last_name,
    users.gender,
    users.level,
    songs.title,
    songs.duration,
    artists.name,
    artists.location 
from songplays 
join users on users.user_id = songplays.user_id 
join songs on songs.song_id = songplays.song_id 
join artists on artists.artist_id = songplays.artist_id 
order by songplays.start_time
```

3. Get number of songs played per week by gender and level

```
SELECT 
    users.level,
    users.gender,
    time.week, 
    count(songplays.songplay_id) as songs_played 
from songplays 
join time on songplays.start_time = time.start_time 
join users on songplays.user_id = users.user_id 
group by users.level, users.gender, time.week
order by users.level, users.gender, time.week
```