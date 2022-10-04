![](http://img.070077.xyz//20220920103034.png)
# PreKnow in Paper
## Raft Basics
- A Raft cluster contains several servers, five is a typical number
- a server - three states: `leader`, `follower`, `candidate`
![](http://img.070077.xyz//20220920102022.png)
- term: Raft ensures that there is at most one leader in a given term
![](http://img.070077.xyz//20220920102324.png)
- Two RPC Types: (*idempotent*)
  - RequestVote RPC
  - AppendEntries RPCs：leader includes the index and term of the log entry to guarantee consistency
- Server之间存在心跳检测
  - election timeout: Follower receives no communication over a period of time -> `elect new leader`
  - See State Trans Graph
- Election restriction
  - the voter denies its vote if its own log is more up-to-date than that of the candidate.
  - log entries only flow in one direction, from leaders to followers, and leaders never overwrite existing entries in their logs.
- 
# Raft Summary Card
![](http://img.070077.xyz//20220920102951.png)

# Lab2a
## 类间关系
- `Raft` 类对应的是每一个节点的信息，不是静态的。
-> 设定1：对于HeartBeat timeout（收）发起选举，此时rf.HeartBeatSentTime（发）视为心跳。根据状态的不同，或许可以为一个字段复用。不过为了少给自己添加不必要的混乱，还是继续用两个吧。
## 坑
`Go`语言的闭包会使对应的外部变量始终为外部变量（？），请看：
```go
for i := 0; i < len(rf.peers); i++ {  
   if i == rf.me {  
      continue  
   }  
   wg.Add(1)  
   go func() {  
      DPrintf("heartbeat %d/%d by leader %d.", i, len(rf.peers), rf.me)  
      appendEntries := &AppendEntries{}  
      heartbeatReply := &HeartbeatReply{}  
      succ := rf.sendHeartbeat(i, appendEntries, heartbeatReply)  
      if !succ { ... }  
   }()  
}
```
这里，闭包中的`i`会在执行时使用外部值，如你可能会遇到输出`heartbeat 3/3 by leader 1.`，这是因为外面的`i`已经完成了循环，使得函数内读取到`i = len(rf.peers)`.

