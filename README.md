# Introduction to NoSQL Databases (Peer Graded Assignment)
## Course 10 from 13 (Part of IBM Data Engineering Professional Certificate by Coursera)

<p>Having hard time on up and running IBM Skill Network Lab, I decided doing the assignment fully using WSL Ubuntu 20.04 on my local Windows 10 Pro x64. There are some preparation step to do it:</p>
<ol>
  <li>Install npm</li>
  <li>Install openjdk 8 jre</li>
  <li>Using npm, install couchimport</li>
  <li>Get mongodb-database-tools for ubuntu</li>
  <li>Install mongodb (make sure you can access mongodb cli)</li>
  <li>Install cassandra (make sure you can access the cqlsh)</li>
<li>Set up the IBM Cloudant connection (use credential from IBM Cloudant instance), using curl - make sure you can access your Cloudant DB from local ubuntu terminal</li>
</ol> 

<p>Here is my complete syntax on each step, for the whole assignment (screenshoot provided):</p>
<code>
== 2 create index on table movies
curl -X POST $CLOUDANTURL/movies/_index \
-H"Content-Type: application/json" \
-d'{
    "index": {
        "fields": ["Director"]
    }
}'

== 3 create query on specific director
curl -X POST $CLOUDANTURL/movies/_find \
-H"Content-Type: application/json" \
-d'{ 
    "selector":
        {
            "Director":"Richard Gage"
        }
    }'
    
== 4 create index title
curl -X POST $CLOUDANTURL/movies/_index \
-H"Content-Type: application/json" \
-d'{
    "index": {
        "fields": ["Title"]
    }
}'

==5 query to list only the “year” and “Director” keys for the ‘Top Dog’
curl -X POST $CLOUDANTURL/movies/_find \
-H"Content-Type: application/json" \
-d'{ "selector":
        {
            "title":"Top Dog"    
        },
	"fields":["title","year","Director"]
    }'


==6 export from cloudant to json local
couchexport --url $CLOUDANTURL --db movies --type jsonl > movies.json

==7 import json to mongodb
mongoimport --authenticationDatabase admin --db entertainment --collection movies --file movies.json

==8 query to find the year in which most number of movies were released
db.movies.aggregate([
    {"$group" : {_id:"$year", count:{$sum:1}}},
    {$sort:{count:-1}},
    {"$limit":1}
])

db.movies.aggregate([
    {"$group" : {_id:"$year", count:{$sum:1}}},
    {$sort:{count:-1}}
])

==9 query to find the count of movies released after the year 1999
db.movies.aggregate([
    {"$group" : {_id:"$year", count:{$sum:1}}},
	{$match: {_id: {"$gt":1999}}},
    {$sort:{_id:1}}
])

==10 query to find out the average votes for movies released in 2007
db.movies.aggregate([
  {$match: {year: 2007}},
  {$group: {_id: "$title",averageVotes: {$avg: "$Votes"}}},
  {$sort:{avg_vote:-1}}
]);

==11 Export the fields _id, “title”, “year”, “rating” and “director” from the ‘movies’ collection into a file named partial_data.csv
mongoexport --authenticationDatabase admin --db entertainment --collection movies --out partial_data.csv --type=csv --fields _id,title,year,rating,Director

==12 Import partial_data.csv into cassandra server into a 
==keyspace named entertainment and table named movies
CREATE KEYSPACE entertainment WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 3};
use entertainment; 
CREATE TABLE movies(id int PRIMARY KEY,title text,year int,rating int,Director text);

==13 Write a cql query to count the number of rows in the movies table
select count(*) from entertainment.movies;

==14 Create an index for the column rating in the movies table using cql
create index rating_index on movies(rating);

==15 Write a cql query to count the number of in the movies that are rated 'G'
select count(*) from entertainment.movies where rating='G';

</code>
