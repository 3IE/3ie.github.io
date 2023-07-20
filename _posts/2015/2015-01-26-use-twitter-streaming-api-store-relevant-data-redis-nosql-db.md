---
title: "How to use Twitter Streaming API to Store Relevant Data into a Redis NoSQL DB"
date: "2015-01-26"
categories: 
  - "technical"
tags: 
  - "java-technical"
  - "nosql"
  - "xcode-tips"
---

Today's post is an introduction to [Twitter Streaming API](https://dev.twitter.com/streaming/overview) and Redis in Java, using third-party libraries to use these components easily.

## Project

I'll use IntellijIdea as IDE through this tutorial ; it's also possible to use Eclipse and NetBeans. Start by creating a new Maven project, uncheck _From archetype_ then click _Next_. Provide the informations relative to your projets then click _Create_.

[![Screen Shot 2014-11-19 at 15.53.06](/assets/images/Screen-Shot-2014-11-19-at-15.53.06-300x233.png)](http://nscurious.com/wp-content/uploads/2014/11/Screen-Shot-2014-11-19-at-15.53.06.png)

The important file you'll need is _pom.xml_, which acts as the dependency manager reference for Maven. Indeed, that's were you'll tell your program to integrate new dependencies. We'll use several third-party libraries, for both Twitter client and Redis management. Start by adding the following dependencies:

<!-- https://github.com/twitter/hbc -->
<dependency>
    <groupId>com.twitter</groupId>
    <artifactId>hbc-core</artifactId>
    <version>2.2.0</version>
</dependency>

<dependency>
    <groupId>com.twitter</groupId>
    <artifactId>hbc-twitter4j</artifactId>
    <version>2.2.1-SNAPSHOT</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-nop</artifactId>
    <version>1.7.5</version>
    <scope>compile</scope>
</dependency>

<!-- http://mvnrepository.com/artifact/com.google.code.gson/gson/2.3 -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.3</version>
</dependency>

<!-- https://github.com/xetorthio/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.6.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>

You may also specify a specific build configuration:

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <version>2.7</version>
            <configuration>
                <skip>true</skip>
            </configuration>
        </plugin>
    </plugins>
</build>

Last thing is to setup a run configuration using _Run > Edit Configurations_. Add a new one of type _Application_ ; enter your MainClass and check _Single instance only_ to avoid stopping and starting server manually each time your re-deploy.

[![Screen Shot 2014-11-19 at 16.05.21](/assets/images/Screen-Shot-2014-11-19-at-16.05.21.png)](http://nscurious.com/wp-content/uploads/2014/11/Screen-Shot-2014-11-19-at-16.05.21.png)

You're all set up!

## Twitter Streaming API

Specifications can be found [here](https://dev.twitter.com/streaming/overview) but to put it in a nutshell, in contrary to a standard REST API which serves a response per request, a streaming API needs a connection to be opened, and the API is then sending a stream continuously. You may have to reload the connection sometimes if no data is transmitted during a pre-established time interval, but the general behaviour is the websocket-like behaviour.

To get rid of low-level implementation details, we'll use a third-party library named [Hosebird Client](https://github.com/twitter/hbc). You already imported this dependency in your _pom.xml_ file, so we'll ask Maven to download it automatically. In the right side panel, open the _Maven_ box and under _yourProject > Lifecycle_, sequentially hit _Clean_, _Package_ and _Install_. You can now use the lib in your project without any effort.

Prerequisites: you have create a new application on Twitter dev portal [from here](https://apps.twitter.com) and generate a new access token. Save your app consumer key and secret, and your access token and access token secret. You'll need them very soon.

Here is the code snippet that listen for a new tweets (or retweets) containing the hashtag _#nscurious_:

BlockingQueue<String> queue = new LinkedBlockingQueue<String>(10000);
StatusesFilterEndpoint endpoint = new StatusesFilterEndpoint();
endpoint.trackTerms(Lists.newArrayList("#nscurious"));

Authentication auth = new OAuth1("YourAPIKey", "YouAPISecret", "YourAccessToken", "YourAccessTokenSecret");

Client client = new ClientBuilder()
        .hosts(Constants.STREAM\_HOST)
        .endpoint(endpoint)
        .authentication(auth)
        .processor(new StringDelimitedProcessor(queue))
        .build();

client.connect();

for (int msgRead = 0; msgRead < 1000; msgRead++) {
    //Called once a new tweet is detected
    String msg = queue.take();
    System.out.println(msg);
}

client.stop();

Run this code, try posting a new tweet containing the hashtag _#nscurious_ and have a look at your console log. A massive string will be printed ; in fact, this is the complet Json object of your tweet, with its unique Id, its content, the hashtag and mentions contained and so on. You can have a deeper look at a basic tweet structure on the Twitter dev portal, we'll simply use its unique Id in this tutorial as it's for demonstration purpose only.

## Save Them All (into Redis)

Now you keep track of tweets live, you may want to store them into a database to provide data visualisation later for exemple. This topic does not deal with data analysis and data mining, so we'll only store the full tweet structure so that it's possible to find it back later. Start by downloading Redis from the [official website](http://redis.io/download) and install it on your local machine (or on a remote server). Run Redis, the default port is 6379. Your Redis is now live, and you can use [Redis Desktop Manager](http://redisdesktop.com) as a GUI to manager your Redis databases.

Redis is using a master/slave replication model to offer high performances. To summarise, write operations are done on the master instance (single instance) and data are automatically replicated on the slaves (from 0 to unlimited) - which can be hosted either on the same server or on remote server. Read operations are only performed on the slaves. Many configurations are possible in Redis, you can checkout the documentation on the official website to get more informations. For this tutorial, we'll only use a master instance with no replication.

To access your Redis db from your Java application, we'll use [Jedis](https://github.com/xetorthio/jedis), a wrapper to access your Redis instances easily, and [Gson](https://code.google.com/p/google-gson/), a library written by Google to manipulate Json objects. Jedis provides both a lightweight possible implementation - the one we'll use - or more complex ways to handle Redis clusters. Now switch to your Java application.

We'll first create a Jedis pool, in charge of managing Redis instances, to get an instance of our Redis server.

JedisPool pool = new JedisPool(new JedisPoolConfig(), "localhost", 6379, 90, "password");
Jedis jedis = pool.getResource();

Good practices encourage you to set a password to access the master instance. If you did it in a config file, set the password as the last argument when creating your JedisPool instance.

Now modify your previous implementation to, instead of logging our tweet, converting it to a JsonObject instance using Gson and inserting it into your Redis instance:

Gson gson = new Gson();

for (int msgRead = 0; msgRead < 1000; msgRead++) {
    String msg = queue.take();
    JsonObject o = gson.fromJson(msg, JsonElement.class).getAsJsonObject();
    //We set the Tweet Id as key, and the full tweet object as value
    jedis.set(o.get("id").getAsInt(), msg);
    System.out.println("Tweet with Id " + o.get("id").getAsInt() + " inserted");
}

Run your program and post a tweet containing the hashtag #NSCurious, then open your GUI Redis Manager and see the result. You may see several instances like this:

[![Screen Shot 2014-11-19 at 17.02.24](/assets/images/Screen-Shot-2014-11-19-at-17.02.24.png)](http://nscurious.com/wp-content/uploads/2014/11/Screen-Shot-2014-11-19-at-17.02.24.png)

It's done. This code must not be sent to production cause there's no error handling or whatever that should be done before deploying a product on a production environment. But it's a good start to introduce Redis using a concrete example. For the anecdote, I ran such a program on a trending hashtag (#MTVStars) and I got 504 tweets into my db in less than 2 seconds. So use a rare tweet while developing, and test your server capacity after.

If you have any questions on this tutorial or on advanced Redis usage, please leave a comment on this post.
