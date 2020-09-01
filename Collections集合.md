# ConcurrentHashMap基于JDK1.8

采用table数组+单项链表+红黑树结构实现;采用数组元素作为锁，来实现对每一行数据进行加锁，减少并发冲突的概率

HashEntry在1.8中统称为Node，当Node的数量超过8个时，会自动从链表转换为TreeNode红黑树，查询时时间复杂度由O(n)变为O(log(n))

使用内置锁synchronized代替重入锁ReentrantLock

- 因为粒度降低了，两种方式性能相差不多，在粗粒度加锁中ReentrantLock可通过Condition来控制各个低粒度的边界，更加灵活。而在低粒度中，Condition没有优势
