# Raft协议

#### 角色
leader: 领导者，可以提出提案  
follower: 跟随者，可以接受请求  
candidate: 成为领导的候选人，如果follower一段时间接受不到leader的消息，会成为candidate尝试让自己成为leader

#### 流程
1. 选出leader
2. leader不挂掉，就一直负责处理消息，并将日志同步给follower，无需每次都选举leader

#### 例子
[动画演示](http://thesecretlivesofdata.com/raft/)
1. 正常(有leader)情况下数据同步  
    a. leader接受消息
    b. leader发送消息给follower
    c. follower接受leader的消息  
    d. leader收到follower的回复，发现认可该提案的节点占大多数，则commit该提案
    e. leader回复client，并告知follower
    
    同时，follower一直在倒计时，leader则定期给follower发送心跳消息，告知follower自己仍然存活。当follower接收到leader的心跳消息，则重置倒计时定时器。
    如果follower超时没有接收到来自leader的消息，则认为leader已无效，则将自己变更为candidate去竞争leader。
    
2. leader选举  
    a. follower超时未接收到来自leader的消息(或是第一次选举leader)  
    b. follower认为leader已无效(或是第一次选举leader)，变更自身为candidate(注意可能有**一个或多个**follower变为candidate)  
    c. candidate向其他节点发送消息，同时开启倒计时  
    d. 其他节点(candidate或follower)收到消息，根据策略[^1]决定是否认可新leader，发送回复  
    e. candidate收到回复，新leader选举出来，leader定期发送心跳消息证明自己仍然存活  
    f. 若candidate超时未收到其他节点的回复，则重新开启一轮投票  
    
3. 网络分区与恢复
    a. 假设有五个节点a b c d e，a是leader，这时候的多数节点为3  
    b. 网络分区后，a b可以联通，c d e三者可以联通，两方无法联通，这时a仍然是leader  
    c. client向a发送消息，a只能收到来自b的回复，只有2个节点认可提案，无法达成一致  
    d. 同时，c超时未收到a的消息，变更为candidate，向其他节点发送提案  
    e. 节点d e同意c成为leader  
    f. 节点c d e可以正常运行，term增大  
    g. 网络分区恢复后，由于c的term比a大，a b认可c为新leader，从c那里同步新的数据  
    
[^1]: Leader Election策略，如最简单的，先到先得，投票给自己第一个收到的，或者投票给数据比自己新的，等。