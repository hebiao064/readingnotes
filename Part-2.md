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
- Read1 Read2 Read2 Read1 All Read  => No Problem without any Isonation => No solution needed

- Write1 Read2 Read2 Write1 => Problem: Dirty Reads 脏读 => Solution: Read Committed
![](img/fig7-2.png)
在[图7-2](img/fig7-2.png)中，用户2 遇到异常情况：邮件列表里显示有未读消息，但计数器显示为零未读消息，因为计数器增长还没有发生。隔离性可以避免这个问题：通过确保用户2 要么同时看到新邮件和增长后的计数器，要么都看不到。反正不会看到执行到一半的中间结果。
这种情况我们称之为脏读。

解决方案为：Read Committed 一句话概括：Read Old Value instead Uncommited New Value
这意味着事务的任何写入操作只有在该事务提交时才能被其他人看到（然后所有的写入操作都会立即变得可见）

- Write1 Write2 Write2 Write1 => Problem: Dirty Writes 脏写 => Solution: Read Committed, add row lock
![](img/fig7-5.png)

**图7-5 如果存在脏写，来自不同事务的冲突写入可能会混淆在一起**

最常见的情况是，数据库通过使用**行锁（row-level lock）** 来防止脏写：当事务想要修改特定对象（行或文档）时，它必须首先获得该对象的锁。然后必须持有该锁直到事务被提交或中止。一次只有一个事务可持有任何给定对象的锁；如果另一个事务要写入同一个对象，则必须等到第一个事务提交或中止后，才能获取该锁并继续。这种锁定是读已提交模式（或更强的隔离级别）的数据库自动完成的。

如何防止脏读？一种选择是使用相同的锁，并要求任何想要读取对象的事务来简单地获取该锁，然后在读取之后立即再次释放该锁。这能确保不会读取进行时，对象不会在脏的状态，有未提交的值（因为在那段时间锁会被写入该对象的事务持有）。

但是要求读锁的办法在实践中效果并不好。因为一个长时间运行的写入事务会迫使许多只读事务等到这个慢写入事务完成。这会损失只读事务的响应时间，并且不利于可操作性：因为等待锁，应用某个部分的迟缓可能由于连锁效应，导致其他部分出现问题。

出于这个原因，大多数数据库[^vi]使用[图7-4]()的方式防止脏读：对于写入的每个对象，数据库都会记住旧的已提交值，和由当前持有写入锁的事务设置的新值。当事务正在进行时，任何其他读取对象的事务都会拿到旧值。 只有当新值提交后，事务才会切换到读取新值。


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
