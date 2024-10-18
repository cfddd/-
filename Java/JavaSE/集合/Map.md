## HashMap 
1. 数组加链表
2. 扩容：2的幂次方扩容
3. hash冲突：拉链法优化，节点上链表加红黑树
## HashSet
`TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。
## HashTable
- 线程安全
- Hashtable 不允许有 null 键和 null 值
## ConcurrentHashMap 
 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。

只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，就不会影响其他 Node 的读写，效率大幅提升。