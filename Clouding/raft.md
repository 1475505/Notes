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

在这里配置了Pretty Debug，不得不说，工具还是很强大的。

# Lab2b

- 到这里，遵循[Locking Advice](https://pdos.csail.mit.edu/6.824/labs/raft-locking.txt) 就很重要了。
>The locking rules a programmer chooses (e.g. "a goroutine must hold rf.mu when using rf.currentTerm or rf.state") are often called a "locking protocol".

->设定2：这里对于日志`Append`操作的发起，是结合心跳操作进行的。我想到了两个问题：1. 未处理完；2. 心跳太慢。这两个按照Figure 2或许有保护机制。
- 遇到过等锁死循环(i.e. dead loop)。
- 现在的心跳处理等锁太久，又出现多主节点的情况了。TODO
- `applyCh`还不知道怎么使用，会一直`block`之。