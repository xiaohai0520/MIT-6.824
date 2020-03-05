# MIT-6.824

## 1. MapReduce
### &emsp; 1. Material 
&emsp; &emsp; [Video](https://www.youtube.com/watch?v=cQP8WApzIQQ) &emsp; [Handout](https://pdos.csail.mit.edu/6.824/notes/l01.txt) &emsp;[Reference](http://airekans.github.io/cloud-computing/2014/01/25/mapreduce-intro)
### &emsp; 2. Summary
&emsp; &emsp; MapReduce是一种分布式计算模型，用于解决大于1TB数据量的大数据计算处理。著名的开源项目Hadoop和Spark在计算方面都实现的是MapReduce模型。 
  
&emsp; &emsp; MapReduce的模型设计很容易进行水平横向扩展以加强系统的能力，基本分为两种任务：map和reduce，通过map任务完成程序逻辑的并发，通过reduce任务完成并发结果的归约和收集，使用这个框架的开发者的任务就是把自己的业务逻辑先分为这两种任务，然后丢给MapReduce模型去运行。设计上，执行这两种任务的worker可以运行在普通的PC机器上，不需要使用太多资源。当系统整体能力不足时，通过增加worker即可解决。  
  
&emsp; &emsp; 如何处理较慢的网络？减少网络带宽资源的浪费，都尽量让输入数据保存在构成集群机器的本地硬盘上，并通过使用分布式文件系统GFS进行本地磁盘的管理。尝试分配map任务到尽量靠近这个任务的输入数据库的机器上执行，这样从GFS读时大部分还是在本地磁盘读出来。中间数据传输（map到reduce）经过网络一次，但是分多个key并行执行。  
  
&emsp; &emsp; 如何做负载均衡？某个task运行时间比较其他N-1个都长，大家都必须等其结束那就尴尬了，保证task比worker数量要多，做的快的worker可以继续先执行其他task，减少等待。  
  
&emsp; &emsp; 如何做容错性？重新执行那些失败的MR任务即可，因此需要保证MR任务本身是幂等且无状态的。worker失效如何处理？将失败任务调配到其他worker重新执行，保证最后输出到GFS上的中间结果过程是原子性操作即可。  
   
&emsp; &emsp; Master失效如何处理？因为master是单点，只能人工干预，系统干脆直接终止，让用户重启重新执行这个计算。  
### &emsp; 3. Lab
