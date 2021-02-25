# Redis和Raft对比

Redis在哨兵leader选举，和cluster集群中的主备切换，用到了Raft协议。但Redis的实现目标及实现方式与Raft有所不同。  
暂且引用 [Raft协议与Redis集群中的一致性协议的异同](https://zhuanlan.zhihu.com/p/112651338) 加以说明