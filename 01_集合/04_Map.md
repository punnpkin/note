# Map

Map下有两个接口：AbstractMap和SortedMap

## 1. HashMap

1. 计算hashcode

2. 高位无符号右移16位以参与**异或**运算（大多数length一般都小于2^16即小于65536）

3. 取模：（length-1）& hash（就是低位全1，保留了hashcode的零散性质）

4. 扩容：数组的长度或扩大到原来的2，4，8倍以上，取余的时候很好算

   原hash新增是1，就索引加一个旧的容量值，新增是0，则不变

HashMap继承自AbstractMap，实现了Map：

```java
HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>
```

### 1.1 成员变量：

```java
transient int size;				//
transient int modCount;			// HashMap被改变的次数
int threshold;					// 门限值
final float loadFactor;			// 装载因子

static class Node<K, V> implements Entry<K, V>

transient HashMap.Node<K, V>[] table;		// 实现数组-链表
transient Set<Entry<K, V>> entrySet;		//
```

`initialCapacity`默认为16，`loadFactory`默认为0.75，容量等于`16*0.75=12`

### 1.2 构造函数：

`HashMap`提供了四个构造函数，用于指定门限值和装载因子：

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0) {
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    } else {
        if (initialCapacity > 1073741824) {
            initialCapacity = 1073741824;
        }

        if (loadFactor > 0.0F && !Float.isNaN(loadFactor)) {
            this.loadFactor = loadFactor;
            this.threshold = tableSizeFor(initialCapacity);
        } else {
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
        }
    }
}
```

### 1.3 Put方法

索引冲突的几种种情况：

1. key值相同：进行值覆盖

2. key值不同，hash相同：链表或者树
3. key值不同，hash不同，取模后相同（索引位置相同）：链表或者树

```java
public V put(K key, V value) {
    return this.putVal(hash(key), key, value, false, true);
}

// hash 计算hash值
static final int hash(Object key) {
    int h;
    // 支持 key = null 的形式，
    // hashcode ^ (h >>> 16)
    // 高位参与运算，大多数length一般都小于2^16即小于65536
    return key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;
}

// putval
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict){
    HashMap.Node[] tab; // Node类实现了一个链表数组
    					// 数组的长度总是 2的n次方
    
    int n;				// n是tab的长度，第一次put或者中途扩容的时候会重新设置
    if ((tab = this.table) == null || (n = tab.length) == 0) {
        n = (tab = this.resize()).length;
    }
    
    Object p;
    int i;
    // 根据hash计算索引位置，若空则插入一个node
    if ((p = tab[i = (n - 1) & hash]) == null) {
        tab[i] = this.newNode(hash, key, value, (HashMap.Node)null);
    } else { //说明待插入位置存在元素
        Object e;
        Object k;
        
        if(((HashMap.Node)p).hash == hash && 
           ((k = ((HashMap.Node)p).key) == key || 
             key != null && key.equals(k))){ 
            
            e = p;
        } else if(p instanceof HashMap.TreeNode){ // p是红黑树
            e = ((HashMap.TreeNode)p).putTreeVal(this, tab, hash, key, value);
        } else {
            int binCount = 0;
            
             while(true) { 
                 // 遍历这个节点，如果 binCount > 7 转为红黑树，
                 // code
                 break;
                 // 否则接在列表后面
                 // code
                 break;
                 
                 ++binCount;
             }
        }
        
        if (e != null) { 
        	// 修改原值~
        }
    }
}
```

**Node内部类：**像是一个链表的形式

```java
static class Node<K, V> implements Entry<K, V> {
    final int hash;
    final K key;
    V value;
    Node<K, V> next;
}
```

### 1.4 get方法：

```java
public V get(Object key) {
    HashMap.Node e;
    // 是根据输入节点的 hash 值和 key 值利用getNode 方法进行查找
    return (e = this.getNode(hash(key), key)) == null ? null : e.value;
}

final HashMap.Node<K, V> getNode(int hash, Object key) {
    HashMap.Node[] tab;
    HashMap.Node first;
    int n;
    // tab非空，长度>0，索引处非空
    if ((tab = this.table) != null && (n = tab.length) > 0 && (first = tab[n - 1 & hash]) != null) {
        Object k;
        // 检查头节点
        if (first.hash == hash && ((k = first.key) == key || key != null && key.equals(k))) {
            return first;
        }

        HashMap.Node e;
        if ((e = first.next) != null) {
            
            // 若定位到的节点是　TreeNode 节点，则在树中进行查找
            if (first instanceof HashMap.TreeNode) {
                return ((HashMap.TreeNode)first).getTreeNode(hash, key);
            }
		
         	// 否则在链表中进行查找
            do {
                if (e.hash == hash && ((k = e.key) == key || key != null && key.equals(k))) {
                    return e;
                }
            } while((e = e.next) != null);
        }
    }

    return null;
}
```

### 1.5 Resize方法

我们在扩容的时候，一般是把长度扩为**原来2倍**，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。（左边为0不移动，左边为1移动）

``` 
0000 0000 0000 0000 0000 0000 0000 1111	     n = 16       
1111 1111 1111 1111 0000 1111 0000 0101                0101		5
1111 1111 1111 1111 0000 1111 0001 0101                0101		5

0000 0000 0000 0000 0000 0000 0001 1111	     n = 32
1111 1111 1111 1111 0000 1111 0000 0101               00101		5
1111 1111 1111 1111 0000 1111 0001 0101               10101		5 + 16
```

因此，我们在扩充HashMap的时候，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”

```java
final HashMap.Node<K, V>[] resize() {
    HashMap.Node<K, V>[] oldTab = this.table; 			// 把旧的表装好
    int oldCap = oldTab == null ? 0 : oldTab.length;	// 计算旧表的容量
    int oldThr = this.threshold;						// 计算旧表门限
    int newThr = 0;
    int newCap;
    if (oldCap > 0) {						// 原表非空的情况
        if (oldCap >= 1073741824) {			// 超出去就不扩了
            this.threshold = 2147483647;
            return oldTab;
        }

        if ((newCap = oldCap << 1) < 1073741824 && oldCap >= 16) {
            newThr = oldThr << 1;
        }
    } else if (oldThr > 0) {				// 原表是空（其它三个构造函数）
        newCap = oldThr;					// 容量设定为门限
    } else {
        newCap = 16;						// 空参构造函数创建的
        newThr = 12;
    }
    
    if (newThr == 0) {						// 设定门限
        float ft = (float)newCap * this.loadFactor;
        newThr = newCap < 1073741824 && ft < 1.07374182E9F ? (int)ft : 2147483647;
    }
    
    this.threshold = newThr;				// 开始创建新表
    HashMap.Node<K, V>[] newTab = new HashMap.Node[newCap];
    this.table = newTab;
    if (oldTab != null) {
        for(int j = 0; j < oldCap; ++j) {
            // rehash开始
            HashMap.Node e;
            if ((e = oldTab[j]) != null) {
                // 原表置空
                oldTab[j] = null;
                if (e.next == null) {
                    // 原节点是单点，用它的hash值，重新计算索引
                    newTab[e.hash & newCap - 1] = e;
                } else if (e instanceof HashMap.TreeNode) {
                    // 红黑树rehash
                    ((HashMap.TreeNode)e).split(this, newTab, j, oldCap);
                } else {
                    // 链表rehash
                    Node<K,V> loHead = null, loTail = null; // 分成两组，索引移位和不移位
                    Node<K,V> hiHead = null, hiTail = null; // 
                    Node<K,V> next;
                    do {
                        next = e.next; 						// 保存下一个节点
                        if ((e.hash & oldCap) == 0) {		// 扩2倍对应的值是0
                            if (loTail == null) {
                                loHead = e;
                            } else {
                                loTail.next = e;
                            }

                            loTail = e;						// loTail = loTail.next
                        } else {
                            if (hiTail == null) {
                                hiHead = e;
                            } else {
                                hiTail.next = e;
                            }

                            hiTail = e;
                        }
                        e = next;							// 遍历链表操作
                    } while(next != null);
                    
                    if (loTail != null) {					// 分别移动到新数组上
                        loTail.next = null;					// 低位直接复制
                        newTab[j] = loHead;
                    }

                    if (hiTail != null) {					// 高位 原索引+原容量
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    
    return newTab;
}
```

## 2. TreeMap

TreeMap是一个通过红黑树实现有序的key-value集合，

TreeMap继承AbstractMap，也即实现了Map，它是一个Map集合，

TreeMap实现了NavigableMap接口，它支持一系列的导航方法，

TreeMap实现了Cloneable接口，它可以被克隆。

映射根据**其键的自然顺序进行排序**，或者根据**创建映射时提供的 Comparator 进行排序**，具体取决于使用的构造方法。

### 2.1 定义

```java
class TreeMap<K, V> extends AbstractMap<K, V> implements NavigableMap<K, V>
```

AbstractMap表明TreeMap为一个Map即支持key-value的集合， 

NavigableMap则意味着它支持一系列的导航方法，具备针对给定搜索目标返回最接近匹配项的导航方法 。

### 2.2 成员变量

```java
private final Comparator<? super K> comparator;			// 比较器
private transient TreeMap.Entry<K, V> root;				// 红黑树根节点
private transient int size = 0;							// 
private transient int modCount = 0;		

private static final boolean RED = false;				// 红色~
private static final boolean BLACK = true;
```

### 2.3 构造方法

#### 默认构造

使用java的默认的比较器比较Key的大小，从而对TreeMap进行排序

```java
public TreeMap() {
    this.comparator = null;
}
```

#### 比较器构造

指定比较器

```java
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
```

#### 带Map构造

会调用putAll()将m中的所有元素添加到TreeMap中

```java
public TreeMap(Map<? extends K, ? extends V> m) {
    this.comparator = null;
    this.putAll(m);
}
```

#### 带SortedMap的构造函数

```java
public TreeMap(SortedMap<K, ? extends V> m) {
    this.comparator = m.comparator();

    try {
        this.buildFromSorted(m.size(), m.entrySet().iterator(), (ObjectInputStream)null, (Object)null);
    } catch (ClassNotFoundException | IOException var3) {
    }

}
```

## 3. ConcurrentHashMap

我们知道HashMap是线程不安全的，在多线程环境下，使用Hashmap进行put操作会引起死循环，导致CPU利用率接近100%，所以**在并发情况下不能使用HashMap**。

**HashTable**

HashTable和HashMap的实现原理几乎一样，差别无非是

- HashTable不允许key和value为null
- HashTable是线程安全的

但是HashTable线程安全的策略实现代价却太大了，简单粗暴，get/put所有相关操作都是synchronized的，这相当于**给整个哈希表加了一把大锁**

**ConcurrentHashMap**

主要就是为了应对hashmap在并发环境下不安全而诞生的，ConcurrentHashMap的设计与实现非常精巧，大量的利用了**volatile，final，CAS等lock-free技术来减少锁竞争**对于性能的影响。

我们都知道Map一般都是**数组+链表结构**（JDK1.8该为数组+红黑树）。

**ConcurrentHashMap避免了对全局加锁改成了局部加锁操作**，这样就极大地提高了并发环境下的操作速度

**内部大量采用CAS操作，这里我简要介绍下CAS。**

CAS是compare and swap的缩写，即我们所说的比较交换。cas是一种基于锁的操作，而且是乐观锁。在java中锁分为乐观锁和悲观锁。悲观锁是将资源锁住，等一个之前获得锁的线程释放锁之后，下一个线程才可以访问。而乐观锁采取了一种宽泛的态度，通过某种方式不加锁来处理资源，比如通过给记录加version来获取数据，性能较悲观锁有很大的提高。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存地址里面的值和A的值是一样的，那么就将内存里面的值更新成B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要自旋，到下次循环才有可能机会执行。



## 4. LinkedHashMap

类定义

```java
public class LinkedHashMap<K, V> extends HashMap<K, V> implements Map<K, V>{}

// 成员变量
transient LinkedHashMap.Entry<K, V> head;
transient LinkedHashMap.Entry<K, V> tail;
final boolean accessOrder;
```

基本数据结构：

```java
static class Entry<K, V> extends Node<K, V> {
    LinkedHashMap.Entry<K, V> before;
    LinkedHashMap.Entry<K, V> after;

    Entry(int hash, K key, V value, Node<K, V> next) {
        super(hash, key, value, next);
    }
}
```

Entry继承自Node：

```java
// Entry 的属性
final int hash;
final K key;
V value;
HashMap.Node<K, V> next;

// next是索引冲突时连接Entry使用的，before、after是维护先候顺序
Entry<K, V> before;
Entry<K, V> after;
```

<img src="04_Map.assets/249993-20161215143120620-1544337380.png" alt="img" style="zoom:50%;" />



构造函数：

```java
// 空参构造函数
public LinkedHashMap() {
    // 调用父类方法来构造一个底层存放的table数组
    super();
    // 定义了存储的顺序是和插入有关还是访问有关
    this.accessOrder = false;
}

// accessOrder
public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

```java
// 在Node移除之后，维护链表
void afterNodeRemoval(Node<K, V> e) {}
// 插入之后
void afterNodeInsertion(boolean evict) {}
// 访问之后，在access为true时，把节点移动到最后
void afterNodeAccess(Node<K, V> e) {}
```

```java
void afterNodeRemoval(Node<K, V> e) {
    LinkedHashMap.Entry<K, V> p = (LinkedHashMap.Entry)e;
    LinkedHashMap.Entry<K, V> b = p.before;
    LinkedHashMap.Entry<K, V> a = p.after;
    p.before = p.after = null;
    if (b == null) {
        this.head = a;
    } else {
        b.after = a;
    }

    if (a == null) {
        this.tail = b;
    } else {
        a.before = b;
    }
}
```

```java
// 把链表的头节点删除掉
/* evict 非构造函数插入为true
 * first 表是不是空的
 * removeEldestEntry 方法的默认实现返回false
 */
void afterNodeInsertion(boolean evict) {
    LinkedHashMap.Entry first;
    if (evict && (first = this.head) != null && this.removeEldestEntry(first)) {
        K key = first.key;
        this.removeNode(hash(key), key, (Object)null, false, true);
    }
}
```

```java
// 
void afterNodeAccess(Node<K, V> e) {
    LinkedHashMap.Entry last;
    if (this.accessOrder && (last = this.tail) != e) {
        // accessOrder为true且当前节点不是tail节点
        LinkedHashMap.Entry<K, V> p = (LinkedHashMap.Entry)e;
        LinkedHashMap.Entry<K, V> b = p.before;
        LinkedHashMap.Entry<K, V> a = p.after;
        p.after = null;
        if (b == null) {
            this.head = a;
        } else {
            b.after = a;
        }

        if (a != null) {
            a.before = b;
        } else {
            last = b;
        }

        if (last == null) {
            this.head = p;
        } else {
            p.before = last;
            last.after = p;
        }

        this.tail = p;
        ++this.modCount;
    }

}
```























