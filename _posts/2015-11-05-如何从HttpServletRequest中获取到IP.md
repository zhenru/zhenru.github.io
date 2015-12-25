---
layout: post
title: 数据库事务相关知识简单介绍
---

- 事务是什么
引用百度百科的描述如下：
> 事务（Transaction）是访问并可能更新数据库中的各种数据项的一个程序执行单元（unit）。很多时候事务通常是由数据库操作语言或者是其他的一些高级语言书写的用户程序（java 或者 SQL）引起的，使用例如`begin transaction`和 `end transaction`组成的。事务的执行就是在`begin transaction()`和`end transaction()`之间的全体操作。

  事务是数据库中最为重要的一个特性，如果多个数据库的操作在`begin transaction()`和`end transaction()`之间的操作就是一个事务，在sql中，单条sql自动开启一个事务，

-  事务的特点
 事务的特点简答概括起来就*`ACID`*特性,*`ACID`*代表了是个单词的书写*`Atomicity`*,*`Consistency`*,*`Isolation`*,*`Durability`*.
 1. Atomicity
 > 原子性就是对于处于一个事务中的所有的操作要么执行，要么不执行，是一个执行的基本的单元。如果在事务的执行过程中发生了中断当前事务的情况，当前的事务之前的操作都会被回滚到执行前的状态。
 2. Consistency
 > 持久性就是事务的操作，不会破坏系统中数据的正确性，就是在执行的过程中不会出现数据错误。例如，在银行的各个账户中，当各个账户之间进行转帐的过程中，各个账户中的总额是不会变化的。
- 事务完成那些功能
- 事务是如何实现的。




参考文献
1. [百度百科-事务](https://www.tapd.cn/20006991/prong/iterations)
