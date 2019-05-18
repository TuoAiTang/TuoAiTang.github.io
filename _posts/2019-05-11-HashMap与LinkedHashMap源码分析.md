---
layout:       post
title:        "HashMap/LinkedHashMap源码分析"
subtitle:     "看看JDK源码和自己手写的HashMap到底有什么区别"
date:         2019-5-11 18：00
catalog:      true
tags:
    - HashMap
    - JDK源码
---

## HashMap特点

### HashMap冲突时先拉出一个链表，当链表节点超过TREEIFY_THRESHOLD, 自动进行TREEIFY将链表转换成红黑树,将Node转换成TreeNode

### 奇妙的内部类继承关系
- HashMap.TreeNode 继承 LinkedHashMap.Entry
- LinkedHashMap 继承 HashMap
- LinkedHashMap.Entry 继承自HashMap.Node

```java
class testJ{	
	public static void main(String[] args) {
		HashMap hm = new HashMap();
	}
}

class HashMap{
	class TreeNode extends LinkedHashMap.Entry{

	}
	static class Node{

	}
}

class LinkedHashMap extends HashMap{
	static class Entry extends HashMap.Node{

	}
}
```

> [Finished in 1.9s]


### HashMap的表的大小一定是2^n

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### HashMap取址的方式
`Node<K, V> p = table[(n - 1) & hash(key)]`
因为n永远是2 ^ x, 所以 `n - 1 = 2 ^ x - 1`, 那么反映在二进制位上就是n - 1 的低位全为1, 高位全为0。

### HashMap中的hash()

```java
/**
 * Computes key.hashCode() and spreads (XORs) higher bits of hash
 * to lower.  Because the table uses power-of-two masking, sets of
 * hashes that vary only in bits above the current mask will
 * always collide. (Among known examples are sets of Float keys
 * holding consecutive whole numbers in small tables.)  So we
 * apply a transform that spreads the impact of higher bits
 * downward. There is a tradeoff between speed, utility, and
 * quality of bit-spreading. Because many common sets of hashes
 * are already reasonably distributed (so don't benefit from
 * spreading), and because we use trees to handle large sets of
 * collisions in bins, we just XOR some shifted bits in the
 * cheapest possible way to reduce systematic lossage, as well as
 * to incorporate impact of the highest bits that would otherwise
 * never be used in index calculations because of table bounds.
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
   
将键对象自身的hashcode进行了一个位操作，应用这个变换，可以将高位的影响传递到hashcode中。有效的避免冲突，但有些时候对象的hashcode已经是分布良好的，那么，这样的对象不会从这个变换中获益。该变换比较适用于比较小的table，因为这样的table高位全为0。

### HashMap判断是否包含一个对象之getNode(hash(key), key)方法
```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
先取址第一个节点，总是判断第一个节点是不是和对象key相同(`1.hashcode 2. 是否为同一个对象的引用或者equals`)
第一个节点不是要找的对象时，分两种情况。
1. 若这个节点已经是一个TreeNode，那么调用TreeNode的getTreeNode(hash, key)方法在树中进行二分查找。
2. 若这个节点仍然是Node(普通的链表节点)，那么以线性时间执行顺序的遍历。

### 再散列(rehash/resize)

再散列的场景： 当put操作时发现表的size已经达到`table.length * loadfactor`
再散列的操作： 
1. 建立一个新的Node<K, V> [] newTab 
2. 顺序遍历原来的oldTab，将每个节点重新计算hash，因为表的大小是2^n， 所以hash要么和之前的保持一致，要么是之前的两倍
3. 如果当前的节点没有拉链，那么直接插入。如果当前节点还有后续元素，同样分两种情况(tree/linkedlist)。
4. 如果是链表形式的Node，那么用`e.hash & oldCap`即可判断当前的节点是否可以从原来的链表中分离出来，插入到newTab中。

```java
Node<K,V> loHead = null, loTail = null;
Node<K,V> hiHead = null, hiTail = null;
Node<K,V> next;
do {
    next = e.next;
    if ((e.hash & oldCap) == 0) {
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    }
    else {
        if (hiTail == null)
            hiHead = e;
        else
            hiTail.next = e;
        hiTail = e;
    }
} while ((e = next) != null);
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

### HashMap键值对允许空值的键，也允许空值的值

因此用`map.get(key) == null`并不能总是正确的等价与`containsKey(key)`。
同时，hashtable既不允许null键，也不允许null值。会在运行时报出异常。
鉴于Hashtable是早于HashMap出现的，我认为这一点限制是完全可以像HashMap那样进行改进的。
这也许算是Hashtable的一个缺陷吧，好在Hashtable在并发上因为读写方法都加锁，导致并发性能也不理想的原因，也逐渐不被使用了。

Hashtable部分源码：
```java
public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        /* 
        这一行就和HashMap不同，导致这一行当key == null时， 因为直接引用key.hashCode()抛出异常
        而HashMap中调用putVal(hash(key), key, value, false, true)
        
        static final int hash(Object key) {
            int h;
            return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
        }

        因此不会因为key为空而抛出异常
         */
        int hash = key.hashCode();  

        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```


```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
map.put(null, null);	
System.out.println(map.get(null));	//null
System.out.println(map.containsKey(null));	//true
```

这是因为getNode(hash(key), key)方法只会比较,那个`hash & tab.length - 1`位置的node是否为空。
而不管node.K == null, 还是node.V == null, 都不能说明node == null.

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

也就是说e.value == null时，map中还是有这个键值对的。



### 预留回调函数的机制, 为了继承自HashMap的LinkedHashMap

```java
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

预留这些函数的好处： LinkedHashMap在继承时完全不用重写基本的put, remove等函数，只用重写它要用到的这些回调函数
1. afterNodeAccess()，在replace, compute, merge， put函数中调用
2. afterNodeInsertion(), 在put, merge, compute函数中会调用
3. afterNodeRemoval(), 在remove函数中调用

get方法中没有调用afterNodeAccess()是因为，在LinkedHashMap中重写了get方法。因为要根据accessOrder来判断是否调用。


## LinkedHashMap
### 继承自HashMap, 继承了绝大部分方法

但增加了一个继承自HashMap.Node的Entry类。维护了一个	双向链表。可以实现元素的顺序访问(两种顺序： accessOrder)。顺序访问依赖于map.entrySet().iterator()，该方法会创建一个该Map所维护的那个双向链表的迭代器，从而以LinkedList的顺序访问Entry。

```java
/**
 * The iteration ordering method for this linked hash map: <tt>true</tt>
 * for access-order, <tt>false</tt> for insertion-order.
 * 默认为false
 * true: 按访问顺序
 * false: 按插入顺序
 * @serial
 */
final boolean accessOrder;

/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;

static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```


按访问顺序访问：

```java
Map<Integer, Integer> map = new LinkedHashMap<Integer, Integer>(10, 0.75f, true);
map.put(1, 2);
map.put(3, 4);
map.put(5, 6);
map.get(1);
Iterator<Map.Entry<Integer, Integer>> it = map.entrySet().iterator();
while(it.hasNext()){
	System.out.println(it.next());
}   
```

```c
3=4
5=6
1=2
```



### 重写/实现了HashMap中的回调方法

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    //omitted
}
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
void afterNodeRemoval(Node<K,V> e) { // unlink
    //omitted
}
```

afterNodeInsertion方法执行时需要判断是否需要把最近最少访问的元素(也就是head)删除掉。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

判断条件中evict在put时传递的是true, 第三个条件是一个函数返回值。这个函数默认返回false,那么就是永远不会驱除eldest element。
当我们想要实现LRU时，重写该方法，即可。

```java
@Override
protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
    return size() > CACHE;
}
```

当前的size比规定的CACHE大时，返回true, 那么LinkedHashMap就可以自动的去执行驱除的逻辑了。

### 重写了get方法

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```

当决定访问顺序为true， 即访问顺序时, afterNodeAccess(e)会得到执行，将e这个节点加到双向链表的尾巴上。

### 利用LinkedHashMap快速实现LRU

```java
public static void main(String[] args) {
	int size = 5;

    /**
     * false, 基于插入排序
     * true, 基于访问排序
     */
    Map<String, String> map = new LinkedHashMap<String, String>(size, .75F,
            true) {

        @Override
        protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
            boolean tooBig = size() > size;

            if (tooBig) {
                System.out.println("recently least key=" + eldest.getKey());
            }
            return tooBig;
        }
    };

    map.put("1", "1");
    map.put("2", "2");
    map.put("3", "3");
    map.put("4", "4");
    map.put("5", "5");
    System.out.println(map.toString());

    map.put("6", "6");
    map.get("2");
    map.put("7", "7");
    map.get("4");

    System.out.println(map.toString());
}
```

输出结果：
```c
{1=1, 2=2, 3=3, 4=4, 5=5}
recently least key=1
recently least key=3
{5=5, 6=6, 2=2, 7=7, 4=4}
```





