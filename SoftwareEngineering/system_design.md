# Scalability
## Load Balancer
- RR（Round Robin）：DNS return IP one by one, i.e. S1, S2, S3, S1, S2...
- Based on the load of servers (i.e. with weights)
- Dynamic balancing: least connections/server CPU...

That is the **sharding**. Servers may partioning（分流）and replicating（备份）

Need to have a RAID server to store sessions? Or, we can store the address of the server in the cookie but private IP is visiable to the whole world! --> Store *a random number* and let the load balancer remember which number belongs to which server.

> Server-side consistency:
> N(replica nodes) < W(receipt of update required before change) + R(access replicas in read contact) is strong!


## Database Replication
> Master-Slave：R in slaves and RW in master
> Master-Master：all nodes RW and sync
> Tree-Replication：combine masters and slaves as tree
> Buddy：B replice to C and is A's backup.

for easy to migrate, You can stay with MySQL, and use it like a NoSQL database, or you can switch to a better and easier to scale NoSQL database like MongoDB. You will need to introduce a cache.

## Cache Pattern
We recommend **Cached Objects**，Assemble the data in the database as the complete instance of a class as cache.

And so makes async-processing available. 

## Trade-offs
-   **性能**与**可扩展性**
-   **延迟**与**吞吐量**
-   **可用性**与**一致性**
![](http://img.070077.xyz/20221217033423.png)

## Concurrency
- Share-state: use lock to prevent data race
- Message-Passing: async
  - Publish/Subscribe
  - Point2Point
  - Store-Forward
    ![](http://img.070077.xyz/20221217031337.png)
  - Request-Reply
- DataFlow: Data blocked until ready
- Transactional Memory：idempotent scope required.