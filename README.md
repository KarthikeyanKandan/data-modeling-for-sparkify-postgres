# Data Modeling with Postgres for music streaming app 'Sparkify'

## Objective: To create a Postgres database with tables designed to optimize queries on song play analysis

### Introduction

A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analytics team is particularly interested in understanding what songs users are listening to. Currently, they don't have an easy way to query their data, which resides in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

As a data engineer, I've created a Postgres database with tables designed to optimize queries on song play analysis. My role is to create a database schema and ETL pipeline for this analysis, test the database I created and ETL pipeline by running queries given to me by the analytics team from Sparkify and compare my results with their expected results.

### Files

1. [`test.ipynb`](test.ipynb) : Displays the first few rows of each table to check the database.
2. [`create_tables.py`](create_tables.py) : Drops and creates your tables. You run this file to reset your tables before each time you run your ETL scripts.
3. [`etl.ipynb`](etl.ipynb) : Reads and processes a single file from song_data and log_data and loads the data into your tables. This notebook contains detailed instructions on the ETL process for each of the tables.
4. [`etl.py`](etl.py) : Rreads and processes files from song_data and log_data and loads them into your tables.
5. [`sql_queries.py`](sql_queries.py) : Ccontains all the SQL queries, and is imported into the last three files above.

### The Schema for the Song Play Analysis

Using the song and log datasets, I've created a star schema optimized for queries on song play analysis. This includes the following tables.

#### Fact Table

1. `songplays` - records in log data associated with song plays i.e. records with page NextSong
     - songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent

#### Dimension Tables

1. `users` - users in the app
    - user_id, first_name, last_name, gender, level
2. `songs` - songs in the music database
    - song_id, title, artist_id, year, duration
3. `artists` - artists in the music database
    - rtist_id, name, location, latitude, longitude
4. `time` - timestamps of records in songplays broken down into the specific unit
    - `start_time`, hour, day, week, month, year, weekday

### Break down of steps followed

1. Wrote DROP, CREATE and INSERT query statements in sql_queries.py

2. Run in console
 ```
python create_tables.py
```

3. Used test.ipynb Jupyter Notebook to interactively verify that all tables were created correctly.

4. Followed the instructions and completed etl.ipynb Notebook to create the blueprint of the pipeline to process and insert all data into the tables.

5. Once verified that base steps were correct by checking with test.ipynb, filled in etl.py program.

6. Run etl in console, and verify results:
 ```
python etl.py
```

### Use
Remember to run [`create_tables.py`](create_tables.py) before running [`etl.py`](etl.py) to reset your tables. 
Run test.ipynb to confirm your records were successfully inserted into each table.

NOTE: You will not be able to run [`test.ipynb`](test.ipynb), [`etl.ipynb`](etl.ipynb), or [`etl.py`](etl.py) until you have run create_tables.py at least once to create the sparkifydb database, which these other files connect to.

## ETL pipeline

Prerequisites: 
- Database and tables created

1. On the etl.py we start our program by connecting to the sparkify database, and begin by processing all songs related data.

2. We walk through the tree files under /data/song_data, and for each json file encountered we send the file to a function called process_song_file.

3. Here we load the file as a dataframe using a pandas function called read_json().

4. For each row in the dataframe we select the fields we are interested in:
    
    ```
    song_data = [song_id, title, artist_id, year, duration]
    ```
    ```
     artist_data = [artist_id, artist_name, artist_location, artist_longitude, artist_latitude]
    ```
5. Finally, we insert this data into their respective databases.

6. Once all files from song_data are read and processed, we move on processing log_data.

7. We repeat step 2, but this time we send our files under /data/log_data to function process_log_file.

8. We load our data as a dataframe same way as done in songs data. 

9. We filter rows where page = 'NextSong'

10. We convert ts column where we have our start_time as timestamp in millisencs to datetime format. We obtain the parameters we need from this date (day, hour, week, etc), and insert everything into our time dimentional table.

11. Next we load user data into our user table

12. Finally we lookup song and artist id from their tables by song name, artist name and song duration that we have on our song play data. The query used is the following:
    ```
    song_select = ("""SELECT s.song_id, a.artist_id 
                FROM songs s 
                JOIN artists a 
                ON a.artist_id = s.artist_id 
                WHERE s.title = %s 
                AND a.name = %s 
                AND s.duration = %s
               """)
    ```

13. The last step is inserting everything we need into our songplay fact table.
