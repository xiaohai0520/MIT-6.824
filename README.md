# MIT-6.824

## 1. MapReduce
### 1. Material 
[Video](https://www.youtube.com/watch?v=cQP8WApzIQQ) &emsp; [Handout](https://pdos.csail.mit.edu/6.824/notes/l01.txt) &emsp;[Reference](http://airekans.github.io/cloud-computing/2014/01/25/mapreduce-intro)
### 2. Summary
MapReduce是一种分布式计算模型，用于解决大于1TB数据量的大数据计算处理。著名的开源项目Hadoop和Spark在计算方面都实现的是MapReduce模型。 
  
MapReduce的模型设计很容易进行水平横向扩展以加强系统的能力，基本分为两种任务：map和reduce，通过map任务完成程序逻辑的并发，通过reduce任务完成并发结果的归约和收集，使用这个框架的开发者的任务就是把自己的业务逻辑先分为这两种任务，然后丢给MapReduce模型去运行。设计上，执行这两种任务的worker可以运行在普通的PC机器上，不需要使用太多资源。当系统整体能力不足时，通过增加worker即可解决。  
  
如何处理较慢的网络？减少网络带宽资源的浪费，都尽量让输入数据保存在构成集群机器的本地硬盘上，并通过使用分布式文件系统GFS进行本地磁盘的管理。尝试分配map任务到尽量靠近这个任务的输入数据库的机器上执行，这样从GFS读时大部分还是在本地磁盘读出来。中间数据传输（map到reduce）经过网络一次，但是分多个key并行执行。  

如何做负载均衡？某个task运行时间比较其他N-1个都长，大家都必须等其结束那就尴尬了，保证task比worker数量要多，做的快的worker可以继续先执行其他task，减少等待。  
  
如何做容错性？重新执行那些失败的MR任务即可，因此需要保证MR任务本身是幂等且无状态的。worker失效如何处理？将失败任务调配到其他worker重新执行，保证最后输出到GFS上的中间结果过程是原子性操作即可。  
   
Master失效如何处理？因为master是单点，只能人工干预，系统干脆直接终止，让用户重启重新执行这个计算。  

### 3. Lab
用Golang建立了一个MapReduce库方法，同时学习分布式系统怎么进行容错。  

在第一部分编写简单的MapReduce程序（Mapper和Reducer）。在第二部分编写master，master往workers分配任务，处理workers的错误。  

注意，MapReduce可以不需要很多机器，在同一台机器上，MapReduce将工作分配到一系列的工作线程（Golang中用goroutine实现）上，这些线程担任Mapper或Reducer，Master则是一个特殊的线程，它调度Mapper和Reducer，为它们分配工作。 

## 1. Raft
### 1. Material 
[Video](https://www.youtube.com/watch?v=64Zp3tzNbpE) &emsp; [Handout](https://pdos.csail.mit.edu/6.824/notes/l01.txt) &emsp;[Reference](https://github.com/maemual/raft-zh_cn)
### 2. Summary
不同于Paxos算法直接从分布式一致性问题出发推导出来，Raft算法则是从多副本状态机的角度提出，用于管理多副本状态机的日志复制。Raft实现了和Paxos相同的功能，它将一致性分解为多个子问题：Leader选举（Leader election）、日志同步（Log replication）、安全性（Safety）、日志压缩（Log compaction）、成员变更（Membership change）等。同时，Raft算法使用了更强的假设来减少了需要考虑的状态，使之变的易于理解和实现。

Raft将系统中的角色分为领导者（Leader）、跟从者（Follower）和候选人（Candidate）：

Leader：接受客户端请求，并向Follower同步请求日志，当日志同步到大多数节点上后告诉Follower提交日志。
Follower：接受并持久化Leader同步的日志，在Leader告之日志可以提交之后，提交日志。
Candidate：Leader选举过程中的临时角色。

### 3. Lab

