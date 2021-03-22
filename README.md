# Building a Recommender with Apache Mahout on Amazon Elastic MapReduce (EMR)

The first part includes the workflows for building a simple movie recommender in order to demonstrate how to build an analytic job with Mahout on EMR.
Second part includes, running a simple web service to provide results to client application.

source: [AWS Big Data](https://aws.amazon.com/blogs/big-data/building-a-recommender-with-apache-mahout-on-amazon-elastic-mapreduce-emr/)   

## Part 1 - Building a Recommender

Recommenders are the systems that take behavior-based inputs, and identify which items users are likely to prefer. 

Data set: [MovieLens data set](http://grouplens.org/datasets/movielens/)

1. Start up an EMR cluster with the following settings.
![Setting_Cluster](https://user-images.githubusercontent.com/80620663/111989951-6db5ea00-8b38-11eb-983f-081bfd466df4.PNG)
2. Connecting to cluster via SSH.

3. Get the MovieLens data
> `wget http://files.grouplens.org/datasets/movielens/ml-1m.zip`  
> `unzip ml-1m.zip`
> 
> Convert ratings.dat, trade “::” for “,”, and take only the first three columns:

> `cat ml-1m/ratings.dat | sed 's/::/,/g' | cut -f1-3 -d, > ratings.csv`
> 
> Put ratings file into HDFS:

> `hdfs dfs -put ratings.csv /user/hadoop/ratings.csv`
> 
4. Run the recommender job:
> `mahout recommenditembased --input /user/hadoop/ratings.csv --output recommendations --numRecommendations 10 --outputPathForSimilarityMatrix similarity-matrix --similarityClassname SIMILARITY_COSINE`
5. Look for the results in the part-files containing the recommendations:
> `hdfs dfs -ls /user/hadoop/recommendations`  
> `hdfs dfs -cat /user/hadoop/recommendations/part-r-00000 | head`  
> 
> ![map-reduce-job-output](./screens/map-reduce-job-output.png)

## Part 2 - Building a web Service

1. Get Twisted, and Klein and Redis modules for Python.
> `sudo yum install python-pip`  
> `sudo pip install twisted`  
> `sudo pip install klein`
> `sudo pip install redis`
2. Checking the downloaded version
> `twistd -version`  
3. Install Redis and start up the server.
> `wget http://download.redis.io/releases/redis-2.8.7.tar.gz`  
> `tar xzf redis-2.8.7.tar.gz`  
> `cd redis-2.8.7`  
> `make`  
> `./src/redis-server &`  
4. Build a web service that pulls the recommendations into Redis and responds to queries.  
> `nano server.py` 
> 
> nano command to save

> ctrl+o 
>
> nano command to exit

> ctrl+x
5. Start the web service.
> `twistd -noy server.py &`
6. Test the web service with user id 28”:
> `curl localhost:4200/28`  
> 
> ![recommender-service-api](./screens/recommender-service-api.png)
