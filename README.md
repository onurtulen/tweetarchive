# Tweet Archive

## Searchable tweet archive, powered by Hibernate Search and Spring Boot

This is a personal tweet archive. Its purpose is to store your tweets, retweets, quoted tweets and favorites in a PostgreSQL database and provide a full text search with Hibernate Search.

So fare, the storage of your tweets and retweets is implemented as well as deleting tweets, if the application is configured to track your account.

There's a super simple "interface" to upload an archive generated by Twitter itself, but the search is only available as a REST endpoint. Be aware: No security has been implemented yet, don't run this publicly if you don't want to expose your Twitter history!

## How to build and run

To build this project, you'll need a valid Java 1.8 installation and either a local PostgreSQL database running on localhost:5432 with a schema named `tweetArchive` and user `tweetArchive` with the same password or a Docker installation.

The database can be configured through the means of [Spring Boot](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

### Built the application

```
./mvnw clean verify
```

### Register an application with Twitter and generate access tokens (optional)

*Note:* This is an optional step. If you just want to upload an existing Twitter archive, skip it. If you want the _tweet-archive_ to track your new tweets and deletions, follow the instructions.

Login to Twitter and open [https://apps.twitter.com](https://apps.twitter.com) and hit "Create new app". This will be your tweet archive. Give a reasonable name. You won't need a callback URL. The program won't need write permissions, I'd remove them. Then, goto "Keys and Access Tokens" and note the consumer key and secret. Take this values and run

```
java -jar target/tweetarchive-0.0.1-SNAPSHOT.jar --generate-tokens consumer_key,consumer_secret
```

Follow the instructions. You'll need to open an URL like _https://api.twitter.com/oauth/authorize?oauth_token=someToken_. Do this and copy the PIN you'll get into the shell.

This will generate a properties file containing your apps consumer token and secret as well as an access token and secret for your account.

### Run directly

Right now, you can start the application with

```
java -jar target/tweetarchive-0.0.1-SNAPSHOT.jar
```

if you have a local PostgresSQL database ready.

### Or build a Docker image

If you plan to run this permanently, install [Docker](https://www.docker.com) for your platform and run:

```
./mvnw clean verify docker:build
```

This will create one Docker image based on the official Java image, containing this application and a link to a Docker container running PostgreSQL. 

### Run the Docker image

After the above step, run

```
./mvnw docker:run
```

It will start a PostgresSQL container and this apps container. The database files will be stored inside `./var/db/prod` and the Lucene search index at `./var/index/prod` so that those data won't vanish if you stop and restart the container.


## Use the application

I assume that you used the Docker method. If you configured your credentials, than the application will track your new tweets.

### Upload a Twitter archive

Open [http://localhost:8980/upload](http://localhost:8980/upload) and upload the file you received from Twitter. That take a while depending on the size, but you'll get a notice eventually.

### Search your tweets

Those are only examples. 

All tweets containing the keyword _Java_

```
curl -X "GET" "http://localhost:8980/search?q=java"
```

All tweets containing the keyword _JavaOne_ autumn 2015

```
curl -X "GET" "http://localhost:8980/search?q=JavaOne&from=2015-10-15&to=2015-10-31"
```


All tweets that I send to [Vlad](https://twitter.com/vlad_mihalcea):

```
curl -X "GET" "http://127.0.0.1:8980/extendedSearch?q=reply.to:vlad_mihalcea"
```

The "extendedSearch" endpoint supports all Lucene queries and escapes. All books I read 2015:

```
curl -X "GET" "http://127.0.0.1:8980/extendedSearch?q=%22Gelesen%20%2F%20Read:%22%20AND%20year:2015"
```

All tweets I sent from my iPhone:

```
curl -X "GET" "http://127.0.0.1:8980/extendedSearch?q=source:%22Twitter%20for%20iPhone%22"
```

As you can see, you can do a lot with a simple archive application.