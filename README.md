# Data Modeling with Cassandra
## Introduction
A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. Their analytics team is particularly interested in understanding what songs users are listening to. Currently, there is no easy way to query the data to generate the results, since the data reside in a directory of CSV files on user activity on the app.

## Project Description

The goal of this project is to create an `Apache Cassandra` (NoSQL Database) which can create queries on song play data analysis.

Data modelling is to be done based on the raw data available in CSV format.  

ETL pipelines that transfers data from CSV files to Cassandra database are to be developed using python.

## Database Modelling

As we are using Apache Cassandra as our database, we need to think about our queries well in advance as joins are not allowed. One table per query is a great strategy. 

In this project we have **three queries** to answer. So, we will create three different tables with appropriate primary key, partition keys and clustering columns.



![Final Data](images/image_event_datafile_new.jpg)



1st **Song plays by session**

Query:
> Give me the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4

Required fields: artist, song, length, sessionid, itemInSession

Partition Key: should be `sessionId`, because this is the primary way our data is being filtered in the query. We are also filtering on `itemInSession`, but that field is less specific and would result in a less logical distribution across nodes.

Primary Key: `sessionId` is not sufficient for a PK, because it is not unique. However, adding `itemInSession` as a clustering column with give us uniqueness.

2nd **Song plays by User's session**

Query:
> Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182

Required fields: artist, song, itemInSession, firstName, lastName, userId, sessionId

Partition Key: should be `userId` in this case, because this is the primary way our data is being filtered in the query. We are also filtering on `sessionId`, but again that field is less specific and would result in a less logical distribution across nodes.

Primary Key: In this table `userId` would not be unique, and adding `sessionId` would still not guarantee uniqueness. However, the query also asks for sorting by `itemInSession`, which means we need it as a clustering column anyway. So a PK that is both unique and sorts the way we would like is `(userId, sessionId, itemInSession)`.

3rd **Users by Song listens**

Query:
> Give me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'

Required fields: firstName, lastName, song, userId

Partition Key: we're filtering by `song` in this case, so we'll use that as our partition key

Primary Key: It's not necessarily required given our query, but I've chosen to include the `userId` field so that we can add it to our PK. I've done this for two reasons, the first being that we need a unique PK and `(song)` or `(song, firstName)` don't suffice. Another reason would be to give us some sorting by `userId`, which feels like a logical way to order. If instead we wanted to sort alphabetically we could do `(song, firstName, userId)`.
