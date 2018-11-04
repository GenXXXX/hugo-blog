---
title: System Design 学习上手
date: 2017-12-28T00:14:19-07:00
showDate: true
tags: ["系统设计", "System Design", "LeetCode"]
---



总有一些犀利的公司试图用System Design来考察new grad的即战力，还是得准备一个。先以设计Twitter为例：

<!--more-->

## 1. Requirements classification

问清楚需求再开始设计，常识。问题例如：

- Will users of our service be able to post tweets and follow other people?
- Should we also design to create and display user’s timeline?
- Will tweets contain photos and videos?
- Will users be able to search tweets?
- Do we need to display hot trending topics?
- Would there be any push notification for new (or important) tweets?

## 2. System interface definition

设计API，抽象软件功能。例如：

```java
postTweet(user_id, tweet_data, tweet_location, user_location, timestamp, …)

generateTimeline(user_id, current_time, user_location, …)
```

## 3. Back-of-the-envelope estimation(粗略估算)

估算scale of the system, 思考scaling, load balancing, caching, partitioning, etc

- What scale is expected from the system (e.g., number of new tweets, number of tweet views, how many timeline generations per sec., etc.)?
- How much storage would we need? We’ll have different numbers if users can have photos and videos in their tweets.

这一部分的估算详见[此贴](http://www.1point3acres.com/bbs/thread-208829-1-1.html)

## 4. Defining data model 

For Twitter, e.g.

**User**: UserID, Name, Email, DoB, CreationData, LastLogin, etc.

**Tweet**: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc.

**UserFollowos**: UserdID1, UserID2

**FavoriteTweets**: UserID, TweetID, TimeStamp

## 5. High-level design

画block diagram设计图。

> For Twitter, at a high level, we would need multiple application servers to serve all the read/write requests with load balancers in front of them for traffic distributions. If we’re assuming that we’ll have a lot more read traffic (as compared to write), we can decide to have separate servers for handling these scenarios. On the backend, we need an efficient database that can store all the tweets and can support a huge number of reads. We would also need a distributed file storage system to store photos and videos.

## 6. Detailed design

- Since we’ll be storing a huge amount of data, how should we partition our data to distribute it to multiple databases? Should we try to store all the data of a user on the same database? What issue can it cause?
- How would we handle hot users, who tweet a lot or follow a lot of people?
- How much and at which layer should we introduce cache to speed things up?

## 7. Bottlenecks

- Is there any single point of failure in our system? What are we doing to mitigate it?
- Do we’ve enough replicas of the data, so that if we lose a few servers, we can still serve our users?

---

## CAP Theorem

[A plain english introduction to CAP Theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem/)

- Consistency: 所有节点获取最新数据的能力
- Availability: 每次请求都能获得响应的能力（不保证最新）
- Partitioning: 分区容错性，在C和A之间做抉择


---

## Key Concepts

**1. Horizontal vs Vertical Scaling**

- 纵向扩展 means increase the resources of a specific node, e.g. add additional memory to a server
- 横向扩展 means increase the number of nodes, e.g. add additional servers thus decreasing the load on any one server

**2. Load Balancer**（负载均衡器）

通常在一个scalable website 的前端会放置一个load balancer, this allows a system to distribute the load evenly so that one server doesn't crash and take down the whole system

**3. Denormalization and NoSQL**

Denormalization means adding redundant information into a database to speed up reads

**4. Database Partitioning (Sharding)**

Sharding means split the data across multiple machines while 你仍然知道哪些data在哪些机器上

- Vertical Partitioning, partitioning by feature
- Hash-based partitioning, using some part of the data (like an ID) to partition it. 比如有N个服务器，讲数据放在mod(key, n)号服务器上
- Directory-based partitioning, maintaining a look-up table for where the data can be found

**5. Caching**

缓存，类比LRU cache

**6. Asynchronous Processing (异步处理)**

耗时较长的slow operation最好be done asynchronously, 否则用户将会被阻塞过长时间。有时可以通过pre-process来加速。

**7. Networking metrics**

- Bandwidth: the maximum amount of data that can be transferred in a unit of time
- Throughput: the actual amount of data that is transferred
- Latency: This is how long it takes data to go from one end to the other

**8. Read-heavy vs. Write-heavy**

Write-heavy: queuing up the writes (but think about the potential failure)

Read-heavy: cache大法好

---

## EXAMPLE

设计Facebook, 实现find shortest path between two people 的功能

Step1: 不考虑数据规模，利用bidirectional-BFS找两人之间最短路径

Step 2: Scale up. 实现以下接口，

```java
int machine_index = getMachineIDForUser(personID);

//go to machine # machine_index

Person friend = getPersonWithID(person_id);
```

首先找server中存储personID的机器ID，再从机器中找到对应的人

Step 3: Optimization. 

- Reduce machine jumps: If 5 of my friends live on the same machine, I should look them up all at once
- Smart division of people and machines. While dividing people across machines, try to divide them by country, city, state, and so on
- ...



