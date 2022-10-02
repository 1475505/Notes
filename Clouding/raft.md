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

# Raft Summary Card
![](http://img.070077.xyz//20220920102951.png)
