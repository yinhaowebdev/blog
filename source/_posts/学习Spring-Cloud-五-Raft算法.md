title: '学习Spring Cloud(五): Raft算法'
author: Billy Yin
tags: []
categories: []
date: 2020-12-24 17:02:00
---
### 引言
假设 Server 有多个节点，需要用一些策略保证节点数据的一致性。
当前被广泛应用的一致性算法有：
* Raft
* Paxos
* ZAB
* Gossip

本文介绍其中的Raft，使用它的微服务组件有 consul 和 ectd 

### 基础
对于集群中每一个节点，有三种可能的状态
* Leader
* Candidate
* Follower

对于每一次变化，有两个过程：
* Log
* Commit

### 选举
##### 概述 
* 一开始，所有节点都是Follower
* 当一个节点“听不见”Leader的时候，它会变成Candidate
* Candidate向其它节点“拉票”
* 节点收到“拉票”后可以投票
* 当Candidate收到了大多数节点的投票时，它变成Leader

##### 详细
* Raft中关于选举，有两种timeout
* election timeout：Follower变为Candidate的时间（随机 150ms ～ 300ms）
* election timeout之后Follower变为Candidate，开始一个term：投票给自己并给其它节点发送 Request Vote
* 如果收到Request Vote的节点在当前的term中还没有投票，那么会投票给发来Request Vote的Candidate
* 发送Request Vote和投票都会导致election timeout重置
* 当Candidate收到了大多数节点的投票时，它变成Leader
* heartbeat timeout: Leader向其它节点发送Append Entries的时间间隔
* Leader向其它节点发送Append Entries，Follower收到Append Entries之后会有 acknowledge response


* 当Follower超过election timeout没有收到心跳信息，就会变成Candidate，开启下一个term


* 假如两个节点同时变成Candidate，得票数一样，那么没有节点会得到`大多数`“选票”，进入下一个term，重新选举



### Log复制
##### 概述
* 所有客户端发来的针对集群的改变都需要经过Leader
* Leader收到Client发来的改变后，不会马上提交变化，而是先加到log列表中
* Leader将log发往其它节点
* 当多数节点都收到了log，Leader会提交变化，并通知节点提交变化

##### 详情 
* Client发送请求到Leader
* Leader生成log
* Log随每次心跳发送的Append Entries“复制”到Follower
* 收到大多数节点的Acknowledge之后，Leader会提交变化，同时向client发送response
* commit信息随心跳发往Follower


* 遇到分区故障时，raft依然可以保持一致性
* 接收到更高term的Append Entries时，原有的Leader会变为Follower，节点中的未提交logs会被接收到的新Append Entries中的覆盖


参考：http://thesecretlivesofdata.com/raft