# Designing Data-Intensive Applications 

## Resource
[English Version](https://github.com/sjtuhjh/appdocs/blob/master/BooksAndPapers/GeneralPapers/Designing%20Data-Intensive%20Applications.pdf)
[Chinese Version](https://github.com/Vonng/ddia/blob/master/ch3.md)

## Table of Contents
### Part II. Foundations of Data Systems
* [Chapter 5: Replication](#chapter-5)
* [Chapter 6: Partitioning](#chapter-6)
* [Chapter 7: Transactions](#chapter-7)
* [Chapter 8: The Trouble with Distributed Systems](#chapter-8)
* [Chapter 9: Consistency and Consensus](#chapter-9)



## Contents
### <a name="chapter-7"></a> Chapter 7: Transactions

Reference: [彻底搞懂 MySQL 事务的隔离级别](https://developer.aliyun.com/article/743691)

Five Possible Concurrent Read Write Possibility
- RRRR All Read  => No Problem without any Isonation

- Write1 Read2 Read2 Write1 => Dirty Reads




- **Data-Intensive** vs Compute-Intensive
- Functional(what it should do) vs Non-functional Requirements(Reliability, Scalability and Maintainability)
- **Reliability**
  - Hardware Faults
  - Software Faults
  - Human Faults
    - Design systems in a way that minimizes opportunities for error, for examples: abstractions, APIs and admin interfaces
    - Decouple: Provide Sandbox enviroments where people can explore and experiment
    - Test thoroughly
    - Quick and fast recovery, rollback
    - Telemetry: detailed and clear monitoring
    
- **Scalability**
  - Describing Load: Twitter's pull and push model, Fan-out
    - **Pull Model**: Posting a tweet simply inserts the new tweet into a global collection of tweets, when user requests home timeline, 
    find all tweets from their following
    ```sql
      SELECT tweets.*, users.* 
      FROM tweets 
      JOIN users   ON tweets.sender_id = users.id 
      JOIN follows ON follows.followee_id = users.id 
      WHERE follows.follower_id = current_user
    ```
    - **Push Model**: Maintain a cache for each user's home timeline like a mailbox, when a user post a tweet, it inserts his/her
    tweet to his follower's mailbox
    - Initially they use Pull Model(system cannot keep up with the load of home timeline queries), then Push Model(write is 
    100x less so looks good), but it doesn't work very well for celebrities, ex. Lady Gaga. Then they combine both approach
    as solution.
  - Describing Performance
    - Latency and Response time
    - p95, p99, p999's meaning
  - Approaches for increasing load
    - Scaling up(Vertical Scaling): More powerful machine
    - Scaling out(Horizontal Scaling): More machines
    
- **Maintainability** 

### Questions
1. Why abstraction helps minimize chance of errors?
