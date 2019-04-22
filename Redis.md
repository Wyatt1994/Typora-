Redis主要使用c写的，也用到了一些jar包。因此需要make make install命令

#### Redis底层数据结构实现原理

https://www.cnblogs.com/ysocean/p/9102811.html

1.字符串类型：int保存long能表示多的整数，embstr保存短字符串，raw编码保存长字符串。

2.list类型：底层就是**双向链表**linkedlist或ziplist（压缩列表，即元素个数较少，每个元素长度较小时）

3.hash类型：ziplist(即按照key-value-key-value---顺序存储)或者hashtable编码（底层实现是c语言里面的字段dict）

4.set类型：intset（存放整数结合）或hashtable(即其他类型)

5.有序集合zset:ziplist或者skiplist(跳表，底层是**字典dict（也就是key value格式数据结构）**+skiplist实现)这里使用**两种数据结构组合**起来，原因是假如我们单独使用 字典，虽然能以 O(1) 的时间复杂度查找成员的分值，但是因为字典是以**无序**的方式来保存集合元素，所以每次进行范围操作的时候都要进行排序；假如我们单独使用跳跃表来实现，虽然能执行范围操作，但是**查找操作**有 O(1)的复杂度变为了O(logN)。因此Redis使用了两种数据结构来共同实现有序集合。

跳表：skiplist,相当于给有序链表建立了多层索引，每个节点包含**两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素。**

![clip_image003](http://s14.sinaimg.cn/middle/72995dcc07ad698ab8d8d&690)

例如查找元素117，首先从顶层链表开始，依次向下跳跃

![clip_image007](http://s9.sinaimg.cn/middle/72995dcc4cc61f6bff808&690)

#### Redis部署方式：

redis的部署方式有**单节点部署、哨兵方式部署、集群方式**部署3种方式



#### Redis的rehash过程

RedisServer结构体中（Redis使用C语言完成）有一个**Dictionary数组，默认容量大小是16，即有16个数据库**，	**每个 Dictionary结构体中有三个Dict结构体指针**，Dict其实就是一个KV结构，**类似于HashMap**，其中两个作为数据存储的库，一个作为当前使用，一个作为备用rehash；另外一个Dict存储键的过期时间。Redis的rehash过程其实就是将上述的16个Dictionary进行rehash，Redis内部支持一个周期循环运行的ctro函数，这个函数定期检查上述的16个Dictionary是否需要rehash，如果有需要，ctro会完成Dictionary的rehash过程，周期结束后并且记录当前循环到Dictionary数组中的哪一个角标，方便再次循环时从上一次循环停止的地方开始。

其中还有一点，Redis的rehash是延迟性的，每次put会将新元素加入到新的表中，**每次查询时，首先会在新的表中查找，如果没有，在旧的表中查找，查找到的话将这个元素插入到新的表中（这个过程也会涉及到检查元素是否过期的问题）**

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170725164319219?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### rehash步骤

（1）为字典的ht[1]哈希表分配空间

若是扩展操作，那么ht[1]的大小为>=ht[0].used*2的2^n
若是收缩操作，那么ht[1]的大小为>=ht[0].used的2^n
（2）将保存在ht[0]中的所有键值对rehash到ht[1]中，rehash指重新计算键的哈希值和索引值，然后将键值对放置到ht[1]哈希表的指定位置上。

（3）当ht[0]的所有键值对都迁移到了ht[1]之后（ht[0]变为空表），释放ht[0]，将ht[1]设置为ht[0],新建空白的哈希表ht[1]，以备下次rehash使用。

#### 渐进式rehash

整个rehash过程并不是一步完成的，而是分多次、渐进式的完成。如果哈希表中保存着**数量巨大的键值对**时，若一次进行rehash，很有可能会导致服务器宕机。

1.为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表
2.维持索引计数器变量rehashidx，并将它的值设置为0，表示rehash开始
3.每次对字典执行增删改查时，将ht[0]的rehashidx索引上的所有键值对rehash到ht[1]，将rehashidx值+1。

4.当ht[0]的所有键值对都被rehash到ht[1]中，程序将rehashidx的值设置为-1，表示rehash操作完成