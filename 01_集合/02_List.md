# List

## 1. ArrayList

### 1.1 定义

```java
class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable;
```

```java
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] EMPTY_ELEMENTDATA = new Object[0];
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];
transient Object[] elementData;
private int size;
```

### 1.2 构造

```java
public ArrayList();							// 将一个空数组{}赋值给了elementData
public ArrayList(int initialCapacity);		// 根据长度初始化
public ArrayList(Collection<? extends E> c);// 传递任何实现了Collection接口的类
											// 调用集合toArray()方法转为数组内赋值给elementData
```

```java 
this.elementData;		// 初始化的元素 
```

底层是一个Object[]，添加到ArrayList中的数据保存在了`elementData`属性中

### 1.3 常用方法

ArrayList有很多常用方法，`add，addAll，set，get，remove，size，isEmpty`等。

add可能会引发扩容，当数组的大小大于初始容量的时候(比如初始为10，当添加第11个元素的时候)，就会进行扩容，新的容量为旧的容量的1.5倍。





## 2. LinkedList

[LinkedList](https://www.cnblogs.com/yijinqincai/p/10964188.html)

### 2.1 定义

```java
class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, Serializable;
```

```java
transient int size;
transient LinkedList.Node<E> first;
transient LinkedList.Node<E> last;
```

```java
// Node 双向链表
private static class Node<E> {
    //当前节点元素值
    E item;
    //下一个节点
    Node<E> next;
    //上一个节点
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```



### 2.2 构造

``` java
public LinkedList();
public LinkedList(Collection<? extends E> c)
```

### 2.3 方法

#### 添加元素

LinkedList主要提供`addFirst`、`addLast`、`add`、`addAll`等方法来实现元素的添加

```java
public void addFirst(E e) {
    this.linkFirst(e);
}

public void addLast(E e) {
    this.linkLast(e);
}

public boolean add(E e) {
    this.linkLast(e);
    return true;				// 因为是无界的，所以永远是真
}

public boolean addAll(Collection<? extends E> c) {
	return this.addAll(this.size, c);
}
```



```java
// linkFirst
private void linkFirst(E e) {
    //将内部保存的首节点点赋值给f
    final Node<E> f = first;
    //创建新节点，新节点的next节点是当前的首节点
    final Node<E> newNode = new Node<>(null, e, f);
    //把新节点作为新的首节点
    first = newNode;
    //判断是否是第一个添加的元素
    //如果是将新节点赋值给last
    //如果不是把原首节点的prev设置为新节点
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    //更新链表节点个数
    size++;
    //将集合修改次数加1
    modCount++;
}
```



```java
void linkLast(E e) {
    //将内部保存的尾节点赋值给l
    final Node<E> l = last;
    //创建新节点，新节点的prev节点是当前的尾节点
    final Node<E> newNode = new Node<>(l, e, null);
    //把新节点作为新的尾节点
    last = newNode;
    //判断是否是第一个添加的元素
    //如果是将新节点赋值给first
    //如果不是把原首节点的next设置为新节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    //更新链表节点个数
    size++;
    //将集合修改次数加1
    modCount++;
}
```



```java
//获取指定位置的节点
Node<E> node(int index) {
    //如果index小于size的一半，就从首节点开始遍历，一直获取x的下一个节点
    //如果index大于或等于size的一半，就从尾节点开始遍历，一直获取x的上一个节点
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

void linkBefore(E e, Node<E> succ);

```

#### 删除方法

LinkedList提供了`remove`、`removeFirst`、`removeLast`等方法删除元素。

基本就是查找元素，断开链接等。

### 2.4 队列操作

#### FIFO

```java
//获取队列的第一个元素，如果为null会返回null
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
//获取队列的第一个元素，如果为null会抛出异常
public E element() {
    return getFirst();
}
//获取队列的第一个元素，如果为null会返回null
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
//获取队列的第一个元素，如果为null会抛出异常
public E remove() {
    return removeFirst();
}
//将元素添加到队列尾部
public boolean offer(E e) {
    return add(e);
}
```

#### LIFO 栈

```java
//将元素添加到首部
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}
//将元素添加到尾部
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
//获取首部的元素值
public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
//获取尾部的元素值
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
//删除首部，如果为null会返回null
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
//删除尾部，如果为null会返回null
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
//将元素添加到首部
public void push(E e) {
    addFirst(e);
}
//删除首部，如果为null会抛出异常
public E pop() {
    return removeFirst();
}
//删除链表中元素值等于o的第一个节点，其实和remove方法是一样的，因为内部还是调用的remove方法
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

//删除链表中元素值等于o的最后一个节点
public boolean removeLastOccurrence(Object o) {
    //因为LinkedList允许存在null，所以需要进行null判断
    if (o == null) {
        //和remove方法的区别它是从尾节点往前遍历
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                //调用unlink方法删除指定节点
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

> 注意：
>
> LinkedList也算是实现了随机访问，但是效率比较低的