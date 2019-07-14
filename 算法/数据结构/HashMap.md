# HashMap

HashMap可以接受null键值和值；

HashMap是非synchronized，即线程不安全的；

HashMap很快；

HashMap储存的是键值对。

**名词：**

1. `loadFactor`：加载因子。默认值DEFAULT_LOAD_FACTOR = 0.75f； 
2. `capacity`：容量；
3. `threshold`：阈值=capacity*loadFactor。当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍（capacity*2）；
4. `size`：HashMap的大小，它是HashMap保存的键值对的数量。

- [数据结构](#数据结构)
  - [数组](#数组)
  - [单链表](#单链表)
  - [HashMap](#HashMap)
    - [hash](#hash)
    - [put()](#put())
    - [get()](#get())
    - [remove()](#remove())
    - [序列化和反序列化](#序列化和反序列化)

## 数据结构

### 数组

- 优点：连续的内存，通过下标可以快速寻址
- 缺点：插入节点困难，需要把指定位置之后的数据往后移动再插入

### 单链表

- Head：头结点
- Tail：尾节点
- Val：节点值，Next：指向下一个节点
- 优点：插入删除简单，直接改变节点指向
- 缺点：查询效率低，需要遍历

### HashMap

关系运算：pos = key % size

- pos：下标

- key：值

- size：数组的大小

- 这样容易发生碰撞冲突，即pos值相同的元素发生碰撞，可以使用单链表解决冲突，如果发生碰撞，则通过链表把碰撞冲突的元素组织起来，但是这样的查询效率很低，如果链表的深度很大，就会查询很慢

- Jdk1.8之后，如果链表深度超过某个限制(默认为8)，则用红黑树(平衡二叉树)代替链表，提高速度

- HashMap源码

  ```java
  public HashMap() {
      this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
  }
  
  // 默认加载因子
  static final float DEFAULT_LOAD_FACTOR = 0.75f;
  ```
  
- 扩容

  - 扩容的条件：实际节点数大于等于容量的3/4；
  - 扩容后数据排布：要么是原下标位置；要么是原下标+原容量位置

- 为什么数组分配的初始容量都是2的倍数

  - 可以提高运算速度
  - 增加散列度，降低冲突
  - 减少内存碎片

#### hash

1.  对key的hashCode做hash操作（高16bit不变，低16bit和高16bit做了一个异或）； 
2.  h & (length-1);   //通过位操作得到下标index。

#### put()

1. 对key的hashCode做hash操作，然后再计算在bucket中的index（1.5 HashMap的哈希函数）； 
2. 如果没碰撞直接放到bucket里； 
3. 如果碰撞了，以链表的形式存在buckets后； 
4. 如果节点已经存在就替换old value(保证key的唯一性) 
5. 如果bucket满了(超过阈值，阈值=loadfactor*current capacity，load factor默认0.75)，就要resize。

#### get()

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

通过对key的hashCode()进行hashing，并计算下标`( n-1 & hash)`，从而获得buckets的位置。

如果产生碰撞，则利用`key.equals()`方法去链表中查找对应的节点。

#### remove()

先判断是否为红黑树，是的话，就按照红黑树删除元素，否则按照单链表改变指针指向

#### 序列化和反序列化

##### 序列化

- 调用writeObject；

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws IOException {
    int buckets = capacity();
    // Write out the threshold, loadfactor, and any hidden stuff
    s.defaultWriteObject();
    s.writeInt(buckets);
    s.writeInt(size);
    internalWriteEntries(s);
}
```

- 先调用capacity()，不为空就返回table.length；

```java
final int capacity() {
    return (table != null) ? table.length :
    (threshold > 0) ? threshold :
    DEFAULT_INITIAL_CAPACITY;
}
```

- buckets：数组的容量；

- size：键值对的数量
- 序列化：只存储了数组的容量，实际节点的数量

##### 反序列化

- reinitialize()；初始化

```java
void reinitialize() {
    table = null;
    entrySet = null;
    keySet = null;
    values = null;
    modCount = 0;
    threshold = 0;
    size = 0;
}
```

- s.readInt()；先读入数组的大小
- int mappings = s.readInt()；再读入键值对的数量
- 判断mapping 是否大于0，小于0报错，大于0就计算threshold
- 反序列化：写入的是初始数组大小，原来容器中节点的数量，然后再写入各个元素的值

#### 如何使HashMap线程安全

调用工具类Collections.synchronizedMap(map);

```java
HashMap map =new HashMap();
map.put("测试","使map变成有序Map");
Map map1 = Collections.synchronizedMap(map);
```

### ConcurrentHashMap

ConcurrentHashMap内部分为很多段，每个段其实就是一个小的HashTable,它们有自己的锁。

只要多个修改操作发生在不同的段上，它们就可以并发进行。

把一个整体分成了16个段,也就是最高支持16个线程的并发修改操作。

