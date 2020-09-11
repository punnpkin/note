# Queue

```java
public interface Queue<E> extends Collection<E> {
    // 有返回的添加，true或异常
    boolean add(E var1);
	// 无返回
    boolean offer(E var1);
	// 移除
    E remove();
	// 移除，为空时返回null
    E poll();
	// 队头
    E element();
	// 对头，为空时返回null
    E peek();
}
```

# Deque

```java
public interface Deque<E> extends Queue<E>{
    
}
```

|Queue Method |	Equivalent Deque Method|
| ---- | ---- |
|add(e) | addLast(e)|
|offer(e) |	offerLast(e)|
|remove() |	removeFirst()|
|poll() | pollFirst()|
|element() | getFirst()|
|peek() | peekFirst()|

Deque包含了栈的方法：

```java
void push(E var1);

E pop();
```

# ArrayDeque

```java
class ArrayDeque<E> extends AbstractCollection<E> implements Deque<E>, Cloneable, Serializable
```

基于数组的实现

# PriorityQueue

```java
class PriorityQueue<E> extends AbstractQueue<E> implements Serializable
```