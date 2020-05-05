
---
[UDACITY DATA ENGINEERING NANODEGREE](https://classroom.udacity.com/nanodegrees)

Project: Data Modeling with Cassandra
---

# **Introduction**

A startup called **Sparkify** wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analysis team is particularly interested in understanding what songs users are listening to. Currently, there is no easy way to query the data to generate the results, since the data reside in a directory of CSV files on user activity on the app.

They'd like a data engineer to create an **Apache Cassandra database** which can create queries on song play data to answer the questions, and wish to bring you on the project. Your role is to create a database for this analysis. You'll be able to test your database by running queries given to you by the analytics team from Sparkify to create the results.

# **Project Overview**
In this project, you'll apply what you've learned on data modeling with Apache Cassandra and complete an ETL pipeline using Python. To complete the project, you will need to model your data by creating tables in Apache Cassandra to run queries. You are provided with part of the ETL pipeline that transfers data from a set of CSV files within a directory to create a streamlined CSV file to model and insert data into Apache Cassandra tables.

We have provided you with a project template that takes care of all the imports and provides a structure for ETL pipeline you'd need to process this data.

# **Project Description**

## Part I. Build ETL Pipeline

After implementing the logic in section Part I of the notebook template to iterate through each event file in event_data, a new CSV file in Python is created : **event_datafile_new.csv** representing a small subsetof the denormalized event dataset.

### **Sparkify Denormalized event dataset** 
Wa are now you are ready to work with the CSV file titled <font color=red>event_datafile_new.csv</font>, located within the Workspace directory.  
The event_datafile_new.csv contains the following columns: 
- artist [0]
- firstName of user [1]
- gender of user [2]
- item number in session [3]
- last name of user [4]
- length of the song [5]
- level (paid or free song) [6]
- location of the user [7]
- sessionId [8]
- song title [9]
- userId [10]

The image below is a screenshot of what the denormalized data should appear like in the <font color=red>**event_datafile_new.csv**</font> after the code above is run:  

**Sparkify denormalized events logs data :**
![alt text](./images/image_event_datafile_new.jpg "Sparkify denormalized events logs data")

## Part II. Modeling your NoSQL database on Apache Cassandra database  
1. Design tables to answer the queries outlined in the project template
2. Write Apache Cassandra CREATE KEYSPACE and SET KEYSPACE statements
3. Develop your CREATE statement for each of the tables to address each question
4. Load the data with INSERT statement for each of the tables
5. Include IF NOT EXISTS clauses in your CREATE statements to create tables only if the tables do not already exist. We recommend you also include DROP TABLE statement for each table, this way you can run drop and create tables whenever you want to reset your database and test your ETL pipeline
6. Test by running the proper select statements with the correct WHERE clause

# **Data Modeling with Apache Cassandra** 

## **Business questions from Sparkify Data Analytics team**
In this project, Sparkify Data Analytics team wants to answser 3 questions from the event data :

1. Query 1 : Give me the artist, song title and song's length in the music app history that was heard during  sessionId = 338, and itemInSession  = 4
2. Query 2 : Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182
3. Query 3 : Give  me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'

## **Data modeling - Cassandra Tables design**

A first table **session_details** will be created to answer Query 1 :
``` sql
CREATE TABLE IF NOT EXISTS session_details (session_id int, item_in_session int, artist_name text, song_title text, song_length float, PRIMARY KEY (session_id, item_in_session))
```

A second table **user_activity_in_session** will be created to answer Query 2 :
``` sql
CREATE TABLE IF NOT EXISTS user_activity_in_session (user_id int, session_id int, item_in_session int, artist_name text, song_title text, user_firstname text, user_lastname text, PRIMARY KEY (user_id, session_id, item_in_session))
```

A third table **songs_listeners** will be created to answer Query 3 :
``` sql
CREATE TABLE IF NOT EXISTS songs_listeners (user_id int, song_title text, user_firstname text, user_lastname text, PRIMARY KEY (song_title, user_id))
```

## **Insert data to Cassandra table**
Denormalized event data are loaded to a pandas dataframe.  
The pandas dataframe is then parsed to insert data to the Cassandra tables using using **PreparedStatements** to accelerate the batch data loading.  
Here are the CQL query to insert data for the Cassandra tables :

Insert data to table **session_details**
``` sql
INSERT INTO session_details (session_id, item_in_session, artist_name, song_title, song_length) VALUES (?, ?, ?, ?, ?)
```

Insert data to table **user_activity_in_session**
``` sql
INSERT INTO user_activity_in_session (user_id, session_id, item_in_session, artist_name, song_title, user_firstname, user_lastname) VALUES (?, ?, ?, ?, ?, ?, ?)
```

Insert data to table **songs_listeners**
``` sql
INSERT INTO songs_listeners (user_id, song_title, user_firstname, user_lastname) VALUES ( ?, ?, ?, ?)
```

## **Testing Data Model for Cassandra tables**

Answer to Query 1 : Give me the artist, song title and song's length in the music app history that was heard during  sessionId = 338, and itemInSession  = 4
``` sql
SELECT artist_name, song_title, song_length \
FROM session_details \
WHERE session_id = %s \
AND item_in_session = %s
```

Answer to Query 2 : Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182
``` sql
SELECT item_in_session, artist_name, song_title, user_firstname, user_lastname \
FROM user_activity_in_session \
WHERE user_id = %s \
AND session_id = %s
```

Answer to Query 3 : Give  me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'
``` sql
SELECT user_id, user_firstname, user_lastname \
FROM songs_listeners \
WHERE song_title = %s
```

# **Project Files**

- **docker-compose.yml** configuration file to run a local Cassandra cluster using Docker

- **requirements.txt** PIP install requirements to install necessary python packages

- **Project_1B_Data_Model_NoSQL_Cassandra.ipynb** Jupyter notebook to process ETL pipeline, create, insert data and query Cassandra tables


# **Running code**

Create a conda python environment
```python
conda create -n udacity python=3.7
```
Active conda environment and install python package environment
```python
conda activate udacity
pip install -r requirements.txt
```
Before running code in Jupyter notebook, run a local Cassandra cluster using Docker (Linux, MacOS or Windows using WSL)
```docker
docker-compose up -d
```
Cassndra NoSQL database will run on **localhost:9042**  

See docker-compose.yml for details. More on using docker for Cassandra can be found : [Building a Python Data Pipeline to Apache Cassandra on a Docker Container](https://medium.com/swlh/building-a-python-data-pipeline-to-apache-cassandra-on-a-docker-container-fc757fbfafdd) 