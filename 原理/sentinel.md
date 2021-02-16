# Sentinel哨兵

#### 作用
多个Sentinel实例组成Sentinel系统，监视Redis服务器，负责Fail-Over操作。  
例如: 

Redis系统中有master, slave1, slave2, slave3四台机器，当master下线后，Sentinel机器会监测到，当下线时长超过指定值后，Sentinel系统会执行Fail-Over操作。  

Sentinel系统会选出一个slave机器，将其升级为master。然后向其他slave机器发送复制指令，使它们成为新的master的slave机器。同时Sentinel系统还会监视原来的master机器，如果它上线，则将其也设置为新的master的slave机器。

#### 原理

1. 获取master机器信息和slave机器信息  
    a. 向master机器发送INFO命令  
    Sentinel默认每10秒会向监视的master机器发送INFO命令，根据回复来获取master机器的当前信息。回复的内容包括master机器自身的信息(如run id，服务器角色)和master下的slave机器信息。
    
    INFO命令的返回内容部分如下:   
    ```text
   # Server        
   ...
   run_id:7b11c5692...
   ...
   
   # Replication
   role:master
   ...
   slave0:ip=127.0.0.1,port=11111,state=online,offset-43,lag=0
   slave1:ip=127.0.0.1,port=22222,state=online,offset-43,lag=0
   slave2:ip=127.0.0.1,port=33333,state=online,offset-43,lag=0
   ...
   
   # Other sections
    ```
   b. 向slave机器发送INFO命令  
   Sentinel会对slave机器创建命令连接和订阅连接，在创建命令连接后，Sentinel默认每10秒会向slave机器发送INFO命令，命令回复内容部分如下:  
   ```text
   ...
   run_id:32be982dee...
   ...
   # Replication
   role:slave
   master_host:127.0.0.1
   master_port:6379
   master_link_status:up
   slave_repl_offset:11887
   slave_priority:100
   
   # Other sections
    ```
2. 向master和slave发送信息
    Sentinel默认每隔2秒向所有监视的master和slave机器发送命令: ```PUBLISH __sentinel__:hello "<s_ip>,<s_port>,<s_runid>,<s_epoch>,<m_name>,<m_ip>,<m_port>,<m_epoch>" ```
    即向服务器的__sentinel__:hello频道发送了一条信息
    
3. 主观下线与客观下线
4. 选举Leader与Fail-Over

    a. 当master被判断为客观下线时，Sentinel系统内部会通过**Raft协议**选举出leader。
    
    b. Leader负责进行Fail-Over操作，选择slave升级为master。  
        选择slave是通过如下算法:  
        i. 先排除不符合要求的slave，接下来从符合要求的slave选出优先级最高的  
        ii. 先通过slave优先级(priority)排序  
        iii. 再通过replication offset排序，offset越大的说明和master越同步，优先级越高  
        iv. 再选择run id较小的slave  
        