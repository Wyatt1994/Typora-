## **索引的本质**

**为什么不用哈希表？**

无法进行范围查找



## **磁盘I/O的本质**

使用索引查找时，没进行一次节点比较就会发生一次磁盘I/O

操作系统一般是按逻辑地址**页**进行读取磁盘，一般为4K（**Inndb一个索引节点一般是16K**）。因为是以页为基本单元，因此取数据最好为**页的倍数**。

B树和B+树相对于**AVL树(平衡二叉查找树)**会减少磁盘I/O的次数，因为每个节点有**多个元素**

**为什么非叶子结点不存放数据？**

减少树的高度，提高查找效率

聚集索引**默认为主键索引**，若没有**主键索引**，则会默认寻找一个唯一索引，若没有，则会自动创建一个默认的主键索引。

MyIsam**查询**操作比InnoDB块，但**插入和更新**慢

联合索引匹配原则：

emp_no+titile+from_data

select ... emp_no<'3' and title='S'**能用上索引**,因为索引能用于**仅一个范围查询，前一个可以用到，但后面则不能用到（联想联合索引图就能理解）**

select... titile='S' and emp_no<'3'**也能用上索引**，因为有索引优化器，优化为：select... **emp_no='3' and title='S'**

### 覆盖索引：

就是联合索引时即命中**索引列中的数据字段**，而不用去聚集索引查找完整字段，即无须**回表**。

**B+树：**

![1543494542248](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543494542248.png)

![1543494607335](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543494607335.png)

![1543494670986](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543494670986.png)

![1543494684211](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543494684211.png)

![1543494728311](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543494728311.png)

![1543494906753](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543494906753.png)

![1543495378315](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543495378315.png)

![1543495558812](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543495558812.png)

![1543495570500](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543495570500.png)

![1543495643987](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543495643987.png)

### 联合索引

![1543496503322](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543496503322.png)

![1543496999866](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543496999866.png)

![1543497975578](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543497975578.png)

### 建立索引的原则

![1543498401506](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543498401506.png)

### 大批量更新技巧

下图用到了默认的主键索引，未使用其他所有，将其改为了批量更新

例如更新1-100000

分为1-100 101-200。。。

![1543498842520](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543498842520.png)