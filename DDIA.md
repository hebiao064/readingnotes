# Designing Data-Intensive Applications 

## Resource
[English Version](https://github.com/sjtuhjh/appdocs/blob/master/BooksAndPapers/GeneralPapers/Designing%20Data-Intensive%20Applications.pdf)

[Chinese Version](https://github.com/Vonng/ddia/blob/master/ch3.md)

## Table of Contents
### Part I. Foundations of Data Systems
* [Chapter 1: Reliable, Scalable, and Maintainable Applications](#chapter-1)
* [Chapter 2: Data Models and Query Languages](#chapter-2)
* [Chapter 3: Storage and Retrieval](#chapter-3)




### <a name="chapter-1"></a>Chapter 1: Reliable, Scalable, and Maintainable Applications

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

### <a name="chapter-2"></a>Chapter 2: Data Models and Query Languages
- **Relational vs Document Model**
  - Relational model can handle many-to-many relationships well
  - Document Model was separated to Document Database and Graph Database: they don't enforce schema when store, but has limitation
  on data localty for queries
    - **Document Database**: Data comes in self-contained documents and less relationship
    - **Graph Database**: Heavy relationship, anything could be related
- Query languages
  - Declarative: like c#, java, c++
  - Imperative: like sql, css-selector
    - Just specify the pattern of the data you want, doesn't care how to achieve this

### Questions
1. Why Graph database is not relational database

### <a name="chapter-3"></a>Chapter 3: Storage and Retrieval






