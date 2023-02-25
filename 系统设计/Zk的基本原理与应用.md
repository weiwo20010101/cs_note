# Zk的基本原理与应用

![image-20230225130851898](/Users/bytedance/Library/Application Support/typora-user-images/image-20230225130851898.png)1

1；分布式锁；redis/zk

2；动态配置；

3；分布式协调；状态监控；提供高可用；协调各个节点之间的工作，以达到协同工作

4；leader/master选举；zk实现了leader选择算法，可以帮助选出leader

5； zk可以保持多个节点之间的数据一致性，以主从复制的架构，通过一致性算法zab（原子广播和崩溃恢复）保证各个节点之间的数据一致性。

