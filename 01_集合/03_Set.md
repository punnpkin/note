# Set

Set继承自`Collection`接口，用于存储无序元素，值不能重复。自定义类要使用Set的时候，需要重写`hashcode`方法和`equals`方法。

## 1. HashSet

HashSet是基于HashMap实现的：

**成员变量：**

```java
private transient HashMap<E, Object> map;
private static final Object PRESENT = new Object();
```

**构造函数：**

```java
public HashSet() {
	this.map = new HashMap();
}
```

**Add方法**

```java
public boolean add(E e) {
    return this.map.put(e, PRESENT) == null;
}
```

**Contains方法：**

```java
public boolean contains(Object o) {
    return this.map.containsKey(o);
}
```

**Remove方法**

```java
public boolean remove(Object o) {
    return this.map.remove(o) == PRESENT;
}
```

## 2. TreeSet

```java
TreeSet<E> extends AbstractSet<E> implements NavigableSet<E>
```

继承自`AbstractSet`，实现了`NavigableSet`，实现了元素**有序**。

**成员变量：**

```java
private transient NavigableMap<E, Object> m;
private static final Object PRESENT = new Object();
```

**构造函数：**

```java
public TreeSet() {
    this((NavigableMap)(new TreeMap()));
} // 和TreeMap有关
```

**Add方法**

```java
public boolean add(E e) {
    return this.m.put(e, PRESENT) == null;
}
```

**Contains方法：**

```java
public boolean contains(Object o) {
    return this.map.containsKey(o);
}
```

**Remove方法**

```java
public boolean remove(Object o) {
    return this.m.remove(o) == PRESENT;
}
```

