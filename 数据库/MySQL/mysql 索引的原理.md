> 1. [(六)MySQL索引原理篇：深入数据库底层揭开索引机制的神秘面纱！ - 掘金 (juejin.cn)](https://juejin.cn/post/7151275584218202143)
> 2. [数据结构之美：为什么 MySQL 选择 B+树做索引？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/604163580)
> 3. [面试官:谈谈你对mysql联合索引的认识? - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/115778804)
> 4. [这篇 MySQL 索引和 B+Tree 讲得太通俗易懂 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/293128007)

在查询没有索引的数据表是会全表扫描，而**全表扫描**时，会发生磁盘`IO`，读到数据后会载入内存，如果不符合条件则继续发生磁盘`IO`读取其他数据。这显然是低效的，对于`IO`和查询时间都不友好。

但是利用程序的**局部性原理**，`MySQL`一次磁盘`IO`不仅仅只会读取一条表数据，而是会读取多条数据。在`InnoDB`引擎中，一次默认会读取`16KB`数据到内存。
## 索引
索引的本质就是选择一种合理的数据结构，在执行效率方面能尽可能地高，在存储空间方面，又不要消耗太多的内存。
## 索引数据结构
### 1. 哈希表
哈希表是一种以 key-value 键值对存储数据的结构

哈希表存在 **hash 碰撞** 的问题：随着数据量的增多，不同 key 经过哈希计算后结果一样。解决办法是使用链表，但是当数据量比较大时，链表的长度还是会比较大，性能开销就在链表查询上。

哈希表是散列存储，因此这种结构适用于只有等值查询的场景，比如 Memcached 及其他一些 NoSQL 引擎。
### 2. 有序数组
有序数组查询效率好，但插入和删除性能差降。因此，有序数组只适合静态存储引擎。
### 3. 搜索树
#### **二叉查找树**
  将我们的二分查找算法，抽象为一个树结构。但是二叉查找树存在一个问题就是**退化为线性查找**。
#### **AVL 平衡树**
  平衡二叉树追求绝对严格的平衡，平衡条件必须满足左右子树高度差不超过 1，该规则在于**频繁的插入、删除等操作的情景性能肯定会出现问题，因此诞生了红黑树**
#### **红黑树**
  红黑树是一种自适应的平衡树，会根据插入的节点数量以及节点信息，自动调整树结构来维持平衡。**但是数据存储还是分散的，不符合程序的局部性原理**

    简洁地解释红黑树的特性： 
    我会开始解释红黑树是一种自平衡的二叉搜索树，然后简要说明根节点和叶节点的颜色约束，以及红色节点约束。

    提供插入、删除和查找操作的时间复杂度： 
    我会明确指出红黑树能够在最坏情况下保持 O(log n) 的时间复杂度，其中 n 是树中节点的数量。这是红黑树的一个重要优势，也是它被广泛应用的原因之一。

    举例说明应用场景： 
    我会提到红黑树在计算机科学中的实际应用，例如在实现高效的动态数据结构中所起的作用。我可能会提及其在标准库中的应用，如C++中的std::map和std::set，以及Java中的TreeMap和TreeSet等。

    展示对红黑树的理解： 
    我会尽量以简洁而准确的语言回答问题，以表明我对红黑树的理解程度，并且愿意根据需要提供更多细节或进一步探讨相关话题。

    回答可能的进阶问题： 
    如果面试官对红黑树有更深入的问题，我会根据自己的理解和经验，尽力回答这些问题，展示自己对数据结构和算法的深入了解。
#### **B-树(Balance Tree)**
  也就是平衡的多路搜索树（注意：这里的“-”只是一个连接符，千万不要读成B减树），它的高度远小于平衡二叉树的高度。在文件系统和数据库系统中的索引结构经常采用 B 树来实现。
  
  对比之前的红黑树更矮，检索数据更快，也能够充分利用局部性原理减少`IO`次数，但**对于大范围查询的区间查找，依旧需要通过多次磁盘`IO`来检索数据**（每个节点上都承载在数据）。
#### **B+树**
特点：

  - **m 叉树只存储索引，并不真正存储数据**
  - 通过链表将叶子节点串联在一起，这样可以方便按区间查找；
  - 一般情况，根节点会被存储在内存中，其他节点存储在磁盘中。

差异点：

- B+树的**数据保存在叶子节点**，并且组成了一个增序的链表，B-树的所有节点都保存了数据；
- B+树叶子节点的数据是有序链表，因此支持 range-query范围查询，而 B-树不支持；
- B+树叶子节点的数据是有序**双向链表**，方便asc升序查询和desc降序查询；

`B+`树当需要做范围查询时，只需要定位第一个节点，就可以根据节点之间的指针获取到对应范围之内的所有节点，**也就是只需要发生一次`IO`。**

## 实现索引
### MyISAM 引擎

MyISAM 采用的非聚簇索引，B+树的非叶子节点存储索引值和指向子节点的指针，**叶子节点上存放的是索引值和数据在磁盘上的物理地址**，所以通过索引定位到数据地址后，需要到磁盘上回表获取数据，索引模型示意图如下：
  
![](https://pic1.zhimg.com/80/v2-45b7b9948fd84664f01bd57a3d3eaf94_720w.webp)

  

所以，在 MyISAM引擎中，B+树的叶子节点不存储数据，而是数据的物理地址，不管是主键索引还是非主键索引，都需要回表获取数据。

也就是说，MyISAM引擎是非聚簇索引根据相关的键对原来的 InnoDB 引擎聚集索引创建的B+树结构的索引，并不存储数据结构。每次使用索引找到对应的键后，都会回到原表存储的位置上再查找。原表也可能不是叶子节点，接下来就需要按照InnoDB 引擎的规则查找了。
### InnoDB 引擎
InnoDB 采用的聚簇索引，B+树的非叶子节点（内部节点）存放的是索引值和指向子节点的指针，叶子节点上存放的是索引值和数据。每一个索引在 InnoDB 里面对应一棵 B+ 树。

**非聚簇索引**，B+树的**非叶子节点存储索引值和指向子节点的指针**，**叶子节点存放的是索引值和聚簇索引值**。因此非聚簇索引需要先遍历非聚簇索引 B+树定位到聚簇索引的值，再到聚簇索引上回表获取数据。

聚簇索引的优点：**可以避免每棵索引树上都存放数据，使得在相同的内存空间下存放的更多的索引节点，减少磁盘 IO。**

聚簇索引示意图如下：
![](https://pic3.zhimg.com/80/v2-e2d7390d3cf77c9b6b13e2ed5fabc7ca_720w.webp)

非聚簇索引示意图如下：

![](https://pic1.zhimg.com/80/v2-0a68bf8128832a6622031d52d121b66c_720w.webp)

**所以，在 InnoDB中，如果是通过主键索引查询的话，可以直接从叶子节点拿到数据。如果是通过非聚簇索引查询，则需要根据非聚簇索引定位到主键索引，然后再从叶子节点拿到数据。**
## 相关概念
> [给我一分钟，让你彻底明白MySQL聚簇索引和非聚簇索引 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/142139541)
### 聚簇索引
在 InnoDB 里，主键索引也被称为聚簇索引（clustered index），主键索引的叶子节点存放的是整行数据，**索引和数据存储在一起**。一个表中聚簇索引只能有一个。
### 非聚簇索引

在 InnoDB 引擎里，非主键索引也被称为二级索引（secondary index），非主键索引的叶子节点内容是主键的值，**索引和数据分开存储**。

在 MyISAM 引擎中，使用的是非聚簇索引。
### 联合索引
**联合索引规则**：
- **多列排序规则**：对于联合索引，索引列的顺序非常重要。在构建 B + 树时，会按照联合索引中列的定义顺序依次对数据进行排序。
- **叶节点存储数据的引用**：叶节点存储了完整的数据记录或者数据记录的指针，这些数据记录是按照联合索引的排序规则有序存储的。

联合索引是指：**将表中多个字段联合组合成一个索引**，比如：index(age, sex)
联合索引在 B+树索引模型示意图如下：

![](https://pic3.zhimg.com/80/v2-52f89d8c6674012778728f06af4c8996_720w.webp)

  

查询分析：

- 首先，从根节点根据组合索引里面的所有字段进行精确匹配查到到 age=30 and sex='男'的记录有两条；
- 然后，获取 id2 和 id3 两个节点中指向子节点的指针，定位到子节点，再定位到叶子节点，从叶子节点中拿到聚簇索引的值 id2 和 id3；
- 最后，到聚簇索引上遍历 id2 和 id3，直到叶子节点上获取目标数据；
### 索引覆盖

在当前索引树上能直接查找所需结果，不需要回表，这就是索引覆盖。
```sql
select id from user where age = 30 and sex = '男'; -- 比如上面的案例
```
id 已经在当前索引的叶子节点，所以不需要到聚簇索引（主键上的信息）上回表，因此这就是一个索引覆盖的场景。

由于覆盖索引可以减少树的搜索次数，显著提升查询性能，所以使用覆盖索引是一个常用的性能优化手段。
### 最左前缀原则
B+树可以利用索引的“最左前缀”来定位记录。最左前缀可以是联合索引的最左 N 个字段，也可以是字符串索引的最左 M 个字符 比如：对于以下查询条件，联合索引 index(a, b, c)都可以匹配索引

- where a = ?
- where a = ? and b = ?
- where a = ? and b = ? and c = ?  
    

但是，对于 where a = ？and c = ? 只有 a条件可以匹配，c 条件无法匹配

> **规则总结：**

当遇到范围查询(>、<、between、like)就会停止匹配。

MySQL存在索引优化器，可以自动重排键以自动匹配上最优的索引
## 总结
聚簇索引和非聚簇索引的根本区别：

- 聚簇索引中，表数据和索引数据是按照相同顺序存储的，非聚簇索引则不是。
- 聚簇索引在一张表中是唯一的，只能有一个，非聚簇索引则可以存在多个。
- 聚簇索引在逻辑+物理上都是连续的，非聚簇索引则仅是逻辑上的连续。
- 聚簇索引中找到了索引键就找到了行数据，但非聚簇索引还需要做一次回表查询。

`InnoDB`-非聚簇索引与`MyISAM`-非聚簇索引的区别：

- `InnoDB`中的非聚簇索引是以聚簇索引的索引键，与具体的行数据建立关联关系的。
- `MyISAM`中的非聚簇索引是以行数据的地址指针，与具体的行数据建立关联关系的。

一般来说，由于`MyISAM`引擎中的索引可以根据指针直接获取数据，不需要做二次回表查询，因此从整体查询效率来看，会比`InnoDB`要快上不少。
