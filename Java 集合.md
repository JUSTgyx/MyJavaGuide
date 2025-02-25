## 1. 常见集合有哪些

### 1. 单列集合：以 Collection 为核心接口

1. List，存储有序且可重复的元素，比如 ArrayList 基于动态数组，访问快但插入删除慢；LinkedList 基于双向链表，插入删除快但访问慢；Vector 是线程安全的老版本实现
2. Set，用于存储无序且不可重复的元素，比如 HashSet 基于哈希表，查找快但无序；LinkedHashSet 保留插入顺序；TreeSet 基于红黑树，按顺序存储。
3. Queue，用于队列操作，比如 PriorityQueue 基于堆实现按优先级处理，ArrayDeque 是双端队列支持栈和队列操作。

### 2. 双列集合，以 Map 为核心接口

1. HashMap 基于哈希表，键无需且查找快；
2. LinkedHashMap 保留插入顺序；
3. TreeMap 基于红黑树，按键的自然顺序排序；
4. HashTable 是线程安全的老版本实现。

## 2. ArrayList 和 LinkedList 的区别

### 1. 存储空间

- ArrayList：基于数组，物理内存上连续
- LinkedList：底层基于双向链表，逻辑上连续，物理上可以不连续

### 2. 随机访问效率

- ArrayList：支持索引 O1 随机访问
- LinkedList：访问需遍历，复杂度为 On

### 3. 头插效率

- ArrayList：头插需要移动元素来腾出空间，On
- LinkedList：秩序移动指针，O1

### 4. 扩容效率

- ArrayList：空间不足需要新建一个大数组，复制旧数据，影响性能
- LinkedList：随便扩容

### 5. 应用场景

- ArrayList：适合元素高效存储、频繁访问
- LinkedList：适合频繁任意位置插入和删除



## 3. HashMap 的原理

### 存储时

1. 首先通过哈希函数计算出键的 HashCode
2. 用 HashCode 来决定键值存放的位置
3. 如果对应位置的“桶”是空的，直接放进去
4. 如果有其他值（哈希冲突），HashMap 使用链表或者红黑树来存储多个键值对

### 查找

1. 计算 HashCode 并找到对应“桶”
2. 如果只有一个值，返回
3. 多个值（链表、红黑树），逐一对比，找到匹配的
4. 返回

## 4. HashMap 如何扩容

1. 阈值：HashMap 在元素数量超过阈值时，会触发扩容机制。

   如：我们创建了一个大小为 16 的 HashMap，默认阈值为 16 * 0.75 = 12，元素数量超过 12，就会触发扩容

2. 扩容时，HashMap 会先新建一个数组，新数组大小是老数组的两倍，然后将旧数组中元素重新哈希，映射并搬运至新的数组中

3. JDK 1.8 之后对扩容进行了改进，通过位运算判断新下标的位置，而不需要每个节点都重新计算哈希值，提高了效率

   例：旧数组长度为 16（二进制 010000），新数组长度为 32（100000）

   重点在于 key 的哈希值（32）的从右往左第五位（最高位，如果是 32、64 则是第六位）是否为 1，如果是 1 说明需要搬迁至新位置，且新位置的下标就是原下标 + 16（原数组大小），如果是 0 说明吃不到新数组长度的高位，那就还在原位置，不需要搬迁。

## 5. JDK 1.7和1.8中HashMap的区别

### 1.数据结构的变化：

- 在 JDK 1.7 及之前，HashMap 的数据结构是“数组+链表”。当发生哈希冲突时，多个键值对以链表形式存储在同一个“桶”中。 
- 在 JDK 1.8 中，引入了红黑树优化。当链表长度超过 8 时，会自动转换为红黑树，这将时间复杂度从 O(n) 降低到 O(logN)，大幅提高查找效率。

### 2.数据重哈希的优化：

- JDK 1.7：直接使用 hashCode 的低位进行哈希运算，这样当 table 长度较小时，高位没有参与计算，导致哈希分布不均。
- JDK 1.8：在重新计算哈希值时，引入高 16 位和低 16 位的异或运算，让高位也参与运算，从而提高了哈希分布的均匀性，减少了哈希冲突。

### 3.扩容机制的改进：

- JDK 1.7：在扩容时使用“头插法”插入新元素。这种方式会导致链表顺序反转，在多线程环境下，可能引发环形链表问题，导致程序死循环。
- JDK 1.8：改为“尾插法”插入，保证链表顺序一致，避免了扩容中的死循环问题。此外，尾插法与红黑树的结合，也提高了整体的扩容效率。

### 4.重新计算存储位置的方式：

- JDK 1.7：扩容时需要为每个元素重新计算哈希值，直接使用 hash & (table.length - 1)。
- JDK 1.8：利用位运算的特性，判断元素是否需要迁移，如果 hash & oldCapacity == 0，元素保持在原位置；否则，迁移到原位置 + oldCapacity。这种方法避免了重新计算哈希值，提升了扩容效率。

### 5.扩容触发机制优化：

- JDK 1.7：扩容是“先扩容后插入”，无论是否发生哈希冲突，都会执行扩容操作，可能导致无效扩容。
- JDK 1.8：扩容变为“先插入再扩容”，只有当插入后超过阈值或发生哈希冲突时，才会触发扩容，减少了不必要的操作。