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

Reference: 
- [彻底搞懂 MySQL 事务的隔离级别](https://developer.aliyun.com/article/743691)
- [中文DDIA 解读](https://raw.githubusercontent.com/Vonng/ddia/master/ch7.md)

Four Possible Concurrent Read Write Problems: 
- Dirty Reads
- Dirty Writes
- Unrepeatable Reads (Read Skew)
- Phathom (Write Skew)

#### 0. Read1 Read2 Read2 Read1 All Read  => No Problem without any Isolation Implemented => No solution needed 

#### 1. Write1 Read2 Read2 Write1 => Problem: Dirty Reads 脏读 => Solution: Read Committed 读已提交, read old value when uncommited
![](img/fig7-2.png)
在[图7-2](img/fig7-2.png)中，用户2 遇到异常情况：邮件列表里显示有未读消息，但计数器显示为零未读消息，因为计数器增长还没有发生。隔离性可以避免这个问题：通过确保用户2 要么同时看到新邮件和增长后的计数器，要么都看不到。反正不会看到执行到一半的中间结果。
这种情况我们称之为脏读。

解决方案为：Read Committed 一句话概括：Read Old Value instead Uncommited New Value
这意味着事务的任何写入操作只有在该事务提交时才能被其他人看到（然后所有的写入操作都会立即变得可见）

#### 2. Write1 Write2 Write2 Write1 => Problem: Dirty Writes 脏写 => Solution: Read Committed 读已提交, add row lock
![](img/fig7-5.png)

**图7-5 如果存在脏写，来自不同事务的冲突写入可能会混淆在一起**

最常见的情况是，数据库通过使用**行锁（row-level lock）** 来防止脏写：当事务想要修改特定对象（行或文档）时，它必须首先获得该对象的锁。然后必须持有该锁直到事务被提交或中止。一次只有一个事务可持有任何给定对象的锁；如果另一个事务要写入同一个对象，则必须等到第一个事务提交或中止后，才能获取该锁并继续。这种锁定是读已提交模式（或更强的隔离级别）的数据库自动完成的。

如何防止脏读？一种选择是使用相同的锁，并要求任何想要读取对象的事务来简单地获取该锁，然后在读取之后立即再次释放该锁。这能确保不会读取进行时，对象不会在脏的状态，有未提交的值（因为在那段时间锁会被写入该对象的事务持有）。

但是要求读锁的办法在实践中效果并不好。因为一个长时间运行的写入事务会迫使许多只读事务等到这个慢写入事务完成。这会损失只读事务的响应时间，并且不利于可操作性：因为等待锁，应用某个部分的迟缓可能由于连锁效应，导致其他部分出现问题。

出于这个原因，大多数数据库使用的**Read Committed, read old value when uncommited**方式防止脏读：对于写入的每个对象，数据库都会记住旧的已提交值，和由当前持有写入锁的事务设置的新值。当事务正在进行时，任何其他读取对象的事务都会拿到旧值。 只有当新值提交后，事务才会切换到读取新值。

#### 3. Read1 Write2 Write2 Read1 => Problem: Unrepeatable Read (Read Skew) 不可重复读 (读取偏差)=> Solution: Snapshot Isolation 可重复读 Read the corresponding snapshot at specific time
![](img/fig7-6.png)
**图7-6 读取偏差：Alice观察数据库处于不一致的状态**

爱丽丝在银行有1000美元的储蓄，分为两个账户，每个500美元。现在一笔事务从她的一个账户中转移了100美元到另一个账户。如果她在事务处理的同时查看其账户余额列表，不幸地在转账事务完成前看到收款账户余额（余额为500美元），而在转账完成后看到另一个转出账户（已经转出100美元，余额400美元）。对爱丽丝来说，现在她的账户似乎只有900美元——看起来100美元已经消失了。

这种异常被称为**不可重复读（nonrepeatable read）**或**读取偏差（read skew）**：如果Alice在事务结束时再次读取账户1的余额，她将看到与她之前的查询中看到的不同的值（600美元）。在读已提交的隔离条件下，**不可重复读**被认为是可接受的：Alice看到的帐户余额时确实在阅读时已经提交了。

> 不幸的是，术语**偏差（skew）** 这个词是过载的：以前使用它是因为热点的不平衡工作量，而这里偏差意味着异常的时机

**快照隔离（snapshot isolation）**是这个问题最常见的解决方案。想法是，每个事务都从数据库的**一致快照（consistent snapshot）** 中读取——也就是说，事务可以看到事务开始时在数据库中提交的所有数据。即使这些数据随后被另一个事务更改，每个事务也只能看到该特定时间点的旧数据。

#### 4. Read1 Read2 Write1 Read2 => Problem: Phantoms (Write Skew) 幻读 (写入偏差)=> Solution: Serialization 序列化 All Transaction no matter Read or Write need to be done one by one

![](img/fig7-8.png)

**图7-8 写入偏差导致应用程序错误的示例**

在两个事务中，应用首先检查是否有两个或以上的医生正在值班；如果是的话，它就假定一名医生可以安全地休班。由于数据库使用快照隔离，两次检查都返回 2 ，所以两个事务都进入下一个阶段。Alice更新自己的记录休班了，而Bob也做了一样的事情。两个事务都成功提交了，现在没有医生值班了。违反了至少有一名医生在值班的要求。

前大多数提供可序列化的数据库都使用了三种技术之一，本章的剩余部分将会介绍这些技术。

- 字面意义上地串行顺序执行事务（参见“[真的串行执行](#真的串行执行)”）
- **两相锁定（2PL, two-phase locking）**，几十年来唯一可行的选择。（参见“[两相锁定（2PL）](#两阶段锁定（2PL）)”）
- 乐观并发控制技术，例如**可序列化的快照隔离（serializable snapshot isolation）**（参阅“[可序列化的快照隔离（SSI）](#序列化快照隔离（SSI）)”
