# PROJECT DATA MODELING WITH POSTGRES

---
## Intro
This project is used to create a Postgres relational database to store app activities and song metadata. It also contains an ETL pipeline script to extract data from app logs and song metadata files with `JSON` format and load them into the database.

The database will help Sparkify query for data easier by using simple SQL queries instead of doing text parsing from JSON files, and thus, make their analytic tasks easier.

Since Sparkify wants to do analysis on song play data, which are data recorded when a user play a song, the design of the database will focus on this. The schema of the database is described below.

## Database schema
The database is modelled to to need of Sparkify, which is song play analysis. Because Sparkify only wants to do queries and analysis related to song play, a star schema database is used to queries faster and simpler comparing to a normalized schema.

The table `songplays` is the **fact table** containing the attributes `songplay_id` **(Primary Key)**, `star_time`, `user_id`, `level`, `song_id`, `artist_id`, `session_id`, `location` and `user_agent`.

There are **4 dimension tables** in the database:
- `users` contains information of the users. Its attributes are `user_id` **PK**, `first_name`, `last_name`, `gender` and `level`
- `songs` contains information of the songs. Its attributes are `song_id` **PK**, `title`, `artist_id`, `year` and `duration`
- `artists` contains information of the artists. Its attributes are `artist_id` **PK**, `name`, `location`, `latitude` and `longitude`.
- `time` contains information of the time which a song is played on the app. Its attributes are `start_time` **PK**, `hour`, `day`, `week`, `month`, `year` and `weekday`.

## ETL Pipeline
The ETL pipeline need to extract data from 2 types of sources: song metadata files and user logs from the streaming app. Both types are `JSON` files, therefore `pandas` package can be used to read these files and extract values. 

A song metadata file contains information on a single song and its artist. Therefore it can be used to fill the `songs` and `artitsts` tables.

A user log file contains information on the user, how they access the app, the time which the user plays a song and the song's title as well as its artist. Because Sparkify only interests on song play data, the extracted data are first filtered for when the users access the page `NextSong`. Next, they are used to fill the `users` and `time` tables. For the `songplays` table, value for most of its attributes can be found directly from the data. However, since it is the fact table, `song_id` and `artist_id` are used instead of `title` and `artist_name`. To get these information, the `songs` and `artists` tables are joined on the `artist_id`, then the `song_id` and `artist_id` can be searched for using the `title` and `artist_name`.  

## Sample queries
- Top 5 most listened to artists:
> SELECT name, COUNT(songplays.artist_id) as times 
> FROM songplays JOIN artists ON songplays.artist_id = artists.artist_id 
> GROUP BY name ORDER BY times DESC LIMIT 5;

## How to run
1. **Always** run the `create_tables.py` script first to create the Sparkify database and the tables.
2. Run the `etl.py` script to extract data from the logs and insert it into the tables.

## Note
I see that most of the songs in the user logs are not mentioned in the song metadata files. Therefore, most of the `song_id` and `artist_id` in the `songplays` table are `None`, except for 1 row.
