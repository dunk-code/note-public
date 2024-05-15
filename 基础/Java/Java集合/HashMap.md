# HashMap（1.7）

## 内部结构

![image-20210418215705517](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162756.png)

## 源码分析

### 属性

>table 散列表

```java
transient Entry<K,V>[] table;
```

>当前散列表中所有元素个数

```java
transient int size;
```

>对HashMap的修改次数

```java
transient int modCount;
```

### 构造方法

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    // Find a power of 2 >= initialCapacity
    int capacity = 1;
    while (capacity < initialCapacity)
        capacity <<= 1;
    this.loadFactor = loadFactor;
    threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    useAltHashing = sun.misc.VM.isBooted() &&
            (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    init();
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}

public HashMap(Map<? extends K, ? extends V> m) {
    this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                  DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
    putAllForCreate(m);
}
```

>Entry

```java
static class Entry<K,V> implements Map.Entry<K,V
    final K key;
    V value;
    Entry<K,V> next;
    int hash;
    /**
     * Creates new entry.
     */
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
```



### put方法

```java
public V put(K key, V value) {
    //key为null执行操作
    if (key == null)
        return putForNullKey(value);
    //求插入值的地址（路由寻址）
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    //循环当前桶位的单链表
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        //hash值相等并且key相等，找到相同的节点
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            //改变key值，返回原来的value
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    //没有找到与插入节点相同的节点
    addEntry(hash, key, value, i);
    return null;
}
```

>hash

```java
final int hash(Object k) {
   	//hashSeed哈希种子
    int h = 0;
    if (useAltHashing) {
        if (k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }
        h = hashSeed;
    }
    //h = 0 0 ^ n = n
    h ^= k.hashCode();
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```



>putForNullKey

```java
private V putForNullKey(V value) {
    //key == null 固定放在0的位置
    //循环所有桶位找到key为null的节点修改value返回原来的value
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    //没有找到
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
```

>indexFor

```java
static int indexFor(int h, int length) {
    return h & (length-1);
}
```

>addEntry

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    //元素个数达到扩容阈值，并且当前桶位不为空，进行扩容
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    createEntry(hash, key, value, bucketIndex);
}
```

>createEntry

```java
void createEntry(int hash, K key, V value, int bucketIndex) {
    //e当前桶位的头节点
    Entry<K,V> e = table[bucketIndex];
    //头插法，将新节点的后继节点设置为e
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```



### resize方法

>resize

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    //如果原始容量等于最大容量 扩容阈值等于整形最大值
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    //创建一个新的table容量为newCapacity
    Entry[] newTable = new Entry[newCapacity];
    boolean oldAltHashing = useAltHashing;
    //设置JVM的环境变量->改变哈希种子->改变hash值
    useAltHashing |= sun.misc.VM.isBooted() &&
            (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    boolean rehash = oldAltHashing ^ useAltHashing;
    transfer(newTable, rehash);
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

>transfer

```java
void transfer(Entry[] newTable, boolean rehash) {
    //头插法
    int newCapacity = newTable.length;
    //遍历每个桶位的单链表	
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            //通常都为false
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            //重新路由寻址
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

### get方法

```java
public V get(Object key) {
    //key为空调用方法getForNullKey
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);
    return null == entry ? null : entry.getValue();
}
```

>getForNullKey

```java
private V getForNullKey() {
    //在桶位0进行寻找
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}
```

>getEntry

```java
final Entry<K,V> getEntry(Object key) {
    //计算hash值
    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

### remove方法

```java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}
```

>removeEntryForKey

```java
final Entry<K,V> removeEntryForKey(Object key) {
    //确定桶位
    int hash = (key == null) ? 0 : hash(key);
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;
    //循环遍历桶位
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        //找到当前删除节点
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            //删除的是链表的头及诶单
            if (prev == e)
                table[i] = next;
            else
                prev.next = next;
            //记录删除
            e.recordRemoval(this);
            return e;
        }
        //未找到
        prev = e;
        e = next;
    }
    //返回删除节点
    return e;
}
```

>以下代码抛出ConcurrentModificationException异常，原因：迭代器进行遍历的时候需要判断modCount != expectedModCount，而remove会改变modCount 

```java
public static void main(String[] args) {
    Map<Integer,Object> map = new HashMap<>();
    map.put(1,new Object());
    map.put(2,new Object());
    for (int key : map.keySet()) {
        if (key == 2) {
            map.remove(key);
        }
    }
}
```

```java
public final boolean hasNext() {
    return next != null;
}
final Entry<K,V> nextEntry() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    Entry<K,V> e = next;
    if (e == null)
        throw new NoSuchElementException();
    if ((next = e.next) == null) {
        Entry[] t = table;
        while (index < t.length && (next = t[index++]) == null)
            ;
    }
    current = e;
    return e;
}
```

### modCount的作用

Fail-Fast 机制
我们知道 java.util.HashMap 不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了map，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。这一策略在源码中的实现是通过 modCount 域，modCount 顾名思义就是修改次数，对HashMap 内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器的 expectedModCount。在迭代过程中，判断 modCount 跟 expectedModCount 是否相等，如果不相等就表示已经有其他线程修改了 Map：注意到 modCount 声明为 volatile，保证线程之间修改的可见性。**但在JDK7和JDK8中，已经没有这样声明了！！！！！**



> 请注意，此异常并不总是表示对象已被其他线程同时修改。如果单个线程发出一系列违反对象约定的方法调用，则该对象可能会抛出此异常。 例如，如果线程使用有fail-fast机制的迭代器在集合上迭代时修改了集合，迭代器将抛出此异常。

# concurrentHashMap（1.7）

## 内部结构

ConcurrentHashMap为了提高本身的并发能力，在内部采用了一个叫Segment的结构，一个Segment其实就是一个Hashtable的结构，Segment内部维护了一个链表数组

![jdk1.7conhashmap结构](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162757.png)

## 源码分析

### 属性

>DEFAULT_CONCURRENCY_LEVEL  表示Segment的个数

```java
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

>DEFAULT_CAPACITY  表示HashEntry的个数

```java
private static final int DEFAULT_CAPACITY = 16;
```

>MIN_SEGMENT_TABLE_CAPACITY  每个segment中最少的hashEntry 的个数

```java
static final int MIN_SEGMENT_TABLE_CAPACITY = 2;
```

>MAX_SCAN_RETRIES 获取锁自旋次数

```java
static final int MAX_SCAN_RETRIES =
            Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;
```



### 构造方法

```java
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    int sshift = 0;
    int ssize = 1;
    //initialCapacity表示hashEntry的数量 
    //sssize表示segment的个数
    //sshift表示左移的次数，后面取数组偏移地址的时候需要使用
    //大于concurrencyLevel的最小2的幂次方数
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    //hash值需要右移的位数
    this.segmentShift = 32 - sshift;
    //方便后面计算segment下标
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //每个segment中hashEntry的个数
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    //hashEntry的个数（大于等于c的2的幂次方数）
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;
    // create segments and segments[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

>segment

segment长度确定后就不再改变	2

```java
Segment(float lf, int threshold, HashEntry<K,V>[] tab) {
    this.loadFactor = lf;
    this.threshold = threshold;
    this.table = tab;
}
```

### unsafe

>UnSafe操作数组元素

```java
static {
    int ss, ts;
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class tc = HashEntry[].class;
        Class sc = Segment[].class;
        TBASE = UNSAFE.arrayBaseOffset(tc);
        SBASE = UNSAFE.arrayBaseOffset(sc);
        ts = UNSAFE.arrayIndexScale(tc);
        ss = UNSAFE.arrayIndexScale(sc);
        HASHSEED_OFFSET = UNSAFE.objectFieldOffset(
            ConcurrentHashMap.class.getDeclaredField("hashSeed"));
    } catch (Exception e) {
        throw new Error(e);
    }
    if ((ss & (ss-1)) != 0 || (ts & (ts-1)) != 0)
        throw new Error("data type scale not a power of two");
    SSHIFT = 31 - Integer.numberOfLeadingZeros(ss);
    TSHIFT = 31 - Integer.numberOfLeadingZeros(ts);
}
```

1.unsafe = Util.getUnsafe();//初始化unsafe

2.final int base = unsafe.arrayBaseOffset(long[].class);//获取数组头位置

3.final int scale = unsafe.(long[].class);//获取单个数组大小

4.valueOffset = base + (scale * N);//获取第几个元素的偏移量

**这样你就可以随意的操作数组里元素**

### put方法

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    //segment.length = 16
    //segmentMask = 15 1111
    //hash >>> segmentShift取hash值的高四位
    int j = (hash >>> segmentShift) & segmentMask;
    //内部寻找key使用的是hash的低四位
    //put之前判断当前的segment是否为空，为空需要先创建segment
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         //ss = UNSAFE.arrayIndexScale(sc);//第几个元素
         //SSHIFT = 31 - Integer.numberOfLeadingZeros(ss);
         //ss = 32 
         //31 - Integer.numberOfLeadingZeros(ss) = 31 - 26 = 5
         // j << SSHIFT == j << 5 = j * 32
         // j << SSHIFT) + SBASE = j * 32 + SBASE
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}
```

>numberOfLeadingZeros 返回i二进制从左数第一个不是0的位  前的所有的0  的个数（前导0的个数）

```java
public static int numberOfLeadingZeros(int i) {
    // HD, Figure 5-6
    if (i == 0)
        return 32;
    int n = 1;
    if (i >>> 16 == 0) { n += 16; i <<= 16; }
    if (i >>> 24 == 0) { n +=  8; i <<=  8; }
    if (i >>> 28 == 0) { n +=  4; i <<=  4; }
    if (i >>> 30 == 0) { n +=  2; i <<=  2; }
    n -= i >>> 31;
    return n;
}
```

>ensureSegment 创建Segment并且保证其他线程没有修改的情况下线程安全的创建

通过segment[0]即s0为模板进行创建

```java
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    //防止其他线程完成创建segment，再次判断
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        Segment<K,V> proto = ss[0]; // use segment 0 as prototype
        int cap = proto.table.length;
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        //再次判断	
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
            == null) { // recheck
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            //自旋，创建完毕，进行赋值（保证seg被赋值成功）
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```

### segment put方法

>put 这里的put相当于hashmap的put

区别于HashMap的put：

- 整个过程都加ReentrantLock锁
- 添加节点后没有直接让table[index] 指向node而是使用Unsafe的putOrderedObject线程安全的指向

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    //node当前插入的节点
    //使用ReentrantLock保证线程安全
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        //路由寻址（寻找桶位）
        int index = (tab.length - 1) & hash;
        //当前桶位的第一个节点
        HashEntry<K,V> first = entryAt(tab, index);
        //循环当前桶位的单链表
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
               //如果找到了key值相同的节点，直接覆盖
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    //onlyIfAbsent = false 表示存在相同的key可以修改vlaue值
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {//没有找到
                //当前节点不为空，头插法插入当前链表
                if (node != null)
                    node.setNext(first);
                else
                    //为空创建新的hashEntry（头插法）
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                //如果大于扩容阈值，进行rehash
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                //否则将新的hashEntry链表放入当前segment数组中
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}
```

>entryAt 获取当前segment中的第i个hashEntry对象（直接从内存中获取值）

```java
static final <K,V> HashEntry<K,V> entryAt(HashEntry<K,V>[] tab, int i) {
    return (tab == null) ? null :
        (HashEntry<K,V>) UNSAFE.getObjectVolatile
        (tab, ((long)i << TSHIFT) + TBASE);
}
```

>setEntryAt（直接在内存中进行赋值）

```java
static final <K,V> void setEntryAt(HashEntry<K,V>[] tab, int i,
                                   HashEntry<K,V> e) {
    UNSAFE.putOrderedObject(tab, ((long)i << TSHIFT) + TBASE, e);
}
```

>scanAndLockForPut

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
    //尝试获取锁
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {//每次获取尝试获取锁的过程中遍历当前链表
            if (e == null) {
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        //retries偶数进行
        //重新获取当前hashEntry的头节点
        //如果发生改变重新进行操作
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

>entryForHash  重新获取当前hashEntry的头节点

```java
static final <K,V> HashEntry<K,V> entryForHash(Segment<K,V> seg, int h) {
    HashEntry<K,V>[] tab;
    return (seg == null || (tab = seg.table) == null) ? null :
        (HashEntry<K,V>) UNSAFE.getObjectVolatile
        (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
}
```

总结一下：

在segment进行put操作获取尝试获取锁的过程中做了哪些操作

1. retries小于0：自旋的过程中循环当前hashEntry的所有链表，如果找到与当前key相同的节点为止  retries赋值为0，未找到则创建一个新的node
2. retries不停地自增，直到（CPU的核数大于1的时候为64）1的时候结束自旋，获取锁失败
3. retries为偶数的时候重新获取当前的HashEntry，判断是否被修改，如果被修改retries赋值为-1继续进行自旋

#### rehash

```java
private void rehash(HashEntry<K,V> node) {
    //计算新的hashEntry的参数
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    int newCapacity = oldCapacity << 1;
    threshold = (int)(newCapacity * loadFactor);
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    int sizeMask = newCapacity - 1;
    //循环旧hashEntry
    for (int i = 0; i < oldCapacity ; i++) {
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            int idx = e.hash & sizeMask;
            //当前链表只有一个节点，直接赋值
            if (next == null)   //  Single node on list
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                //分片段转移
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                newTable[lastIdx] = lastRun;
                // Clone remaining nodes
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

### remove方法

>remove

```java
public V remove(Object key) {
    int hash = hash(key);
    Segment<K,V> s = segmentForHash(hash);
    return s == null ? null : s.remove(key, hash, null);
}
```

>remove

```java
final V remove(Object key, int hash, Object value) {
    if (!tryLock())
        scanAndLock(key, hash);
    V oldValue = null;
    try {
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> e = entryAt(tab, index);
        HashEntry<K,V> pred = null;
        while (e != null) {
            K k;
            HashEntry<K,V> next = e.next;
            if ((k = e.key) == key ||
                (e.hash == hash && key.equals(k))) {
                V v = e.value;
                if (value == null || value == v || value.equals(v)) {
                    if (pred == null)
                        setEntryAt(tab, index, next);
                    else
                        pred.setNext(next);
                    ++modCount;
                    --count;
                    oldValue = v;
                }
                break;
            }
            pred = e;
            e = next;
        }
    } finally {
        unlock();
    }
    return oldValue;
```

### size方法

```java
public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```



# concurrentHashMap（1.8）

## 内部结构

## 源码分析

### 属性

>sizeCtl 扩容阈值

```java
private transient volatile int sizeCtl;
```

>BASECOUNT 散列表中的元素个数

```java
private static final long BASECOUNT;
```

>counterCells

```java
private transient volatile CounterCell[] counterCells;
```

### 构造方法

```java
public ConcurrentHashMap() {
}

public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}

public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
    this.sizeCtl = DEFAULT_CAPACITY;
    putAll(m);
}
```

>tableSizeFor的作用：返回一个大于当前的cap的一个数字，这个数字一定是2的次方数

```java
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

将任何一个二进制数 0001 1101 1100 => 0001 1111 1111 将任何一个二进制数第一位1之后的数字全部置1

### put方法

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            //初始化table
            tab = initTable();
        //如果桶位为空
        //自旋的创建一个Node放入当前桶位
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //MOVED = -1;
        //如果发现当前桶位头节点的hash值为-1,表示当前hashMap正在被别的线程扩容
        else if ((fh = f.hash) == MOVED)
            //帮助扩容
            tab = helpTransfer(tab, f);
        else {
            //桶位头节点存在，判断插入
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

#### initTable

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            //别的线程成功的创建了table直接返回
            Thread.yield(); // lost initialization race; just spin
        //先将sc变为-1，如果失败自旋等待，如果成功，创建Node并且返回
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    //0.75n
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```



>这里将红黑树封装成一个TreeBin对象

为什么要这样做？

- 在进行插入操作是，树为了保持平衡会进行旋转，从而导致根节点即桶位上的头节点改变，从而导致锁改变，为了防止平衡操作改变锁，所以封装了一个TreeBin对象

#### treeifyBin

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
```

#### TreeNode

```java
TreeBin(TreeNode<K,V> b) {
    super(TREEBIN, null, null, null);
    this.first = b;
    TreeNode<K,V> r = null;
    for (TreeNode<K,V> x = b, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;
        if (r == null) {
            x.parent = null;
            x.red = false;
            r = x;
        }
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            for (TreeNode<K,V> p = r;;) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);
                    TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    r = balanceInsertion(r, x);
                    break;
                }
            }
        }
    }
    this.root = r;
    assert checkInvariants(root);
}
```

## addCount方法

```java
//新增元素时，也就是在调用 putVal 方法后，为了通用，增加了个 check 入参，用于指定是否可能会出现扩容的情况
//check >= 0 即为可能出现扩容的情况，例如 putVal方法中的调用
private final void addCount(long x, int check){
    ... ...
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        //检查当前集合元素个数 s 是否达到扩容阈值 sizeCtl ，扩容时 sizeCtl 为负数，依旧成立，同时还得满足数组非空且数组长度不能大于允许的数组最大长度这两个条件才能继续
        //这个 while 循环除了判断是否达到阈值从而进行扩容操作之外还有一个作用就是当一条线程完成自己的迁移任务后，如果集合还在扩容，则会继续循环，继续加入扩容大军，申请后面的迁移任务
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null && (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            // sc < 0 说明集合正在扩容当中
            if (sc < 0) {
                //判断扩容是否结束或者并发扩容线程数是否已达最大值，如果是的话直接结束while循环
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 || sc == rs + MAX_RESIZERS || (nt = nextTable) == null || transferIndex <= 0)
                    break;
                //扩容还未结束，并且允许扩容线程加入，此时加入扩容大军中
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            //如果集合还未处于扩容状态中，则进入扩容方法，并首先初始化 nextTab 数组，也就是新数组
            //(rs << RESIZE_STAMP_SHIFT) + 2 为首个扩容线程所设置的特定值，后面扩容时会根据线程是否为这个值来确定是否为最后一个线程
            else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}

```

### helpTransfer

```java
//扩容状态下其他线程对集合进行插入、修改、删除、合并、compute等操作时遇到 ForwardingNode 节点会调用该帮助扩容方法 (ForwardingNode 后面介绍)
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    if (tab != null && (f instanceof ForwardingNode) && (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        int rs = resizeStamp(tab.length);
        //此处的 while 循环是上面 addCount 方法的简版，可以参考上面的注释
        while (nextTab == nextTable && table == tab && (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

### tryPresize

```java
//putAll批量插入或者插入节点后发现链表长度达到8个或以上，但数组长度为64以下时触发的扩容会调用到这个方法
private final void tryPresize(int size) {
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY : tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    //如果不满足条件，也就是 sizeCtl < 0 ，说明有其他线程正在扩容当中，这里也就不需要自己去扩容了，结束该方法
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
        //如果数组初始化则进行初始化，这个选项主要是为批量插入操作方法 putAll 提供的
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            //初始化时将 sizeCtl 设置为 -1 ，保证单线程初始化
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    //初始化完成后 sizeCtl 用于记录当前集合的负载容量值，也就是触发集合扩容的阈值
                    sizeCtl = sc;
                }
            }
        }
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        //插入节点后发现链表长度达到8个或以上，但数组长度为64以下时触发的扩容会进入到下面这个 else if 分支
        else if (tab == table) {
            int rs = resizeStamp(n);
            //下面的内容基本跟上面 addCount 方法的 while 循环内部一致，可以参考上面的注释
            if (sc < 0) {
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 || sc == rs + MAX_RESIZERS || (nt = nextTable) == null || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
        }
    }
}
```

### fullAddCount

>fullAddCount

这个方法有些复杂，但是目的很单纯，就是一定要修改成功。自旋里可以分为三个大分支：

- counterCells数组不为空，则哈希映射对应格子，多次失败后冲突升级触发扩容，依然不成功则重复哈希映射自旋。
- counterCells数组为空，则初始化。
- 没有获取初始化的权利，则cas修改baseCount。
  需要解释两个变量的作用：
- collide，意为是否发生碰撞，即为竞争，cas修改对应位置的格子不成功collide就会被设置为true，意为升级，再哈希循环一次还不成就可能触发扩容（2倍）。CounterCell[]数组长度和cpu的核数有关，若数组长度n>=NCPU不再冲突升级了（collide=false），也不会再触发扩容，而是不断再哈希自旋重试；
- cellsBusy，相当于一把自旋锁，cellsBusy=1获取锁，cellsBusy=0释放锁。在分支中扩容CounterCell、新增格子或CounterCell数组的初始化都会用到cellsBusy。

```java
private final void fullAddCount(long x, boolean wasUncontended) {
    int h; // h 类似于 hash
    if ((h = ThreadLocalRandom.getProbe()) == 0) {
        // h = 0 则初始化重新获取哈希值，并wasUncontended=true意为没有竞争
        ThreadLocalRandom.localInit();      // force initialization
        h = ThreadLocalRandom.getProbe();
        wasUncontended = true;
    }
    // false 为没有发生碰撞，竞争
    // true 的意为升级为严重竞争级别，可能触发扩容
    boolean collide = false;                // True if last slot nonempty
    for (;;) {
        CounterCell[] as; CounterCell a; int n; long v;
        if ((as = counterCells) != null && (n = as.length) > 0) {
            // 1. as不等于null且有CounterCell
            if ((a = as[(n - 1) & h]) == null) {
                // （1）映射找到的CounterCell=null，则新建一个
                if (cellsBusy == 0) {            // Try to attach new Cell
                    CounterCell r = new CounterCell(x); // Optimistic create
                    if (cellsBusy == 0 &&
                        U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                        // cellsBusy 相等于一把乐观锁，到这里说明没有其他线程竞争
                        boolean created = false;
                        try {               // Recheck under lock
                            CounterCell[] rs; int m, j;
                            if ((rs = counterCells) != null &&
                                (m = rs.length) > 0 &&
                                rs[j = (m - 1) & h] == null) {
                                // j位置依然是空的，则r赋值给j位置
                                rs[j] = r;
                                // 设置创建成功
                                created = true;
                            }
                        } finally {
                            // 释放锁
                            cellsBusy = 0;
                        }
                        if (created)
                            // 结束循环
                            break;
                        continue;           // Slot is now non-empty
                    }
                }
                collide = false;
            }
            // （2）映射位置的CounterCell不为空，发生哈希冲突
            else if (!wasUncontended)       // CAS already known to fail
                // （2.1）wasUncontended=false说明有竞争，则不继续向后走去抢了，
                // 走（5）再哈希，再次循环就认为没有竞争wasUncontended=true
                wasUncontended = true;      // Continue after rehash
            // （2.2） wasUncontended=true,认为没有竞争，则尝试cas 给该CounterCell里value+x
            else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
                // 修改成功就退出了
                break;
            // （2.3）修改未成功
            else if (counterCells != as || n >= NCPU)
                // （2.3.1）as地址变了（其他线程扩容了） or n即as数组长度已经大于等于cpu核数了
                // （没有多余的核数给其他线程），就不冲突升级了，走（5）再哈希，再次循环尝试
                collide = false;            // At max size or stale
            // （2.3.2）counterCells = as && n < NCPU
            else if (!collide)
                // collide=false 则升级冲突级别为true，走（5）再哈希，再次循环尝试
                collide = true;
            // （3）已经是严重冲突collide=true
            else if (cellsBusy == 0 &&
                     U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                try {
                    if (counterCells == as) {// Expand table unless stale
                        // counterCells 扩容为 2倍
                        // 这个扩容的触发机制就是映射到的counterCell不为null，且多次尝试cas操作+x失败,
                        // 且当前counterCells地址没有被修改,且数组长度小于NCPU（cpu核数）时触发2倍扩容
                        CounterCell[] rs = new CounterCell[n << 1];
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];
                        counterCells = rs;
                    }
                } finally {
                    cellsBusy = 0;
                }
                collide = false;
                continue;                   // Retry with expanded table
            }
            // （5）重新生成一个伪随机数赋值给h，进行下一次循环判断
            // 再哈希
            h = ThreadLocalRandom.advanceProbe(h);
        }
        // 2.as 为null or as是空的
        else if (cellsBusy == 0 && counterCells == as &&
                 U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
            boolean init = false;
            try {                           // Initialize table
                // 多次判断counterCells == as，未防止as变更
                if (counterCells == as) {
                    // 初始化CounterCell数组，初始容量为2
                    CounterCell[] rs = new CounterCell[2];
                    rs[h & 1] = new CounterCell(x);
                    counterCells = rs;
                    init = true;
                }
            } finally {
                cellsBusy = 0;
            }
            if (init)
                break;
        }
        // 3. 2中修改CELLSBUSY失败没抢到初始化as的锁，则尝试 直接cas修改baseCount + x
        else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
            break;                          // Fall back on using base
    }
}

```

### 总结

-  在调用 addCount 方法增加集合元素计数后发现当前集合元素个数到达扩容阈值时就会触发扩容 。

-  扩容状态下其他线程对集合进行插入、修改、删除、合并、compute 等操作时遇到 ForwardingNode 节点会触发扩容 。

- putAll 批量插入或者插入节点后发现存在链表长度达到 8 个或以上，但数组长度为 64 以下时会触发扩容  。

  > 注意：桶上链表长度达到 8 个或者以上，并且数组长度为 64 以下时只会触发扩容而不会将链表转为红黑树 。

## 扩容

### 为什么是8？

> 红黑树的平均查找长度是log(n)，如果长度为8，平均查找长度为log(8)=3，链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，这才有转换成树的必要；链表长度如果是小于等于6，6/2=3，而log(6)=2.6，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短。

### 源码

```java
//调用该扩容方法的地方有：
//java.util.concurrent.ConcurrentHashMap#addCount        向集合中插入新数据后更新容量计数时发现到达扩容阈值而触发的扩容
//java.util.concurrent.ConcurrentHashMap#helpTransfer    扩容状态下其他线程对集合进行插入、修改、删除、合并、compute 等操作时遇到 ForwardingNode 节点时触发的扩容
//java.util.concurrent.ConcurrentHashMap#tryPresize      putAll批量插入或者插入后发现链表长度达到8个或以上，但数组长度为64以下时触发的扩容
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    //计算每条线程处理的桶个数，每条线程处理的桶数量一样，如果CPU为单核，则使用一条线程处理所有桶
    //每条线程至少处理16个桶，如果计算出来的结果少于16，则一条线程处理16个桶
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // 初始化新数组(原数组长度的2倍)
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        //将 transferIndex 指向最右边的桶，也就是数组索引下标最大的位置
        transferIndex = n;
    }
    int nextn = nextTab.length;
    //新建一个占位对象，该占位对象的 hash 值为 -1 该占位对象存在时表示集合正在扩容状态，key、value、next 属性均为 null ，nextTable 属性指向扩容后的数组
    //该占位对象主要有两个用途：
    //   1、占位作用，用于标识数组该位置的桶已经迁移完毕，处于扩容中的状态。
    //   2、作为一个转发的作用，扩容期间如果遇到查询操作，遇到转发节点，会把该查询操作转发到新的数组上去，不会阻塞查询操作。
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    //该标识用于控制是否继续处理下一个桶，为 true 则表示已经处理完当前桶，可以继续迁移下一个桶的数据
    boolean advance = true;
    //该标识用于控制扩容何时结束，该标识还有一个用途是最后一个扩容线程会负责重新检查一遍数组查看是否有遗漏的桶
    boolean finishing = false; // to ensure sweep before committing nextTab
    //这个循环用于处理一个 stride 长度的任务，i 后面会被赋值为该 stride 内最大的下标，而 bound 后面会被赋值为该 stride 内最小的下标
    //通过循环不断减小 i 的值，从右往左依次迁移桶上面的数据，直到 i 小于 bound 时结束该次长度为 stride 的迁移任务
    //结束这次的任务后会通过外层 addCount、helpTransfer、tryPresize 方法的 while 循环达到继续领取其他任务的效果
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            //每处理完一个hash桶就将 bound 进行减 1 操作
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                //transferIndex <= 0 说明数组的hash桶已被线程分配完毕，没有了待分配的hash桶，将 i 设置为 -1 ，后面的代码根据这个数值退出当前线的扩容操作
                i = -1;
                advance = false;
            }
            //只有首次进入for循环才会进入这个判断里面去，设置 bound 和 i 的值，也就是领取到的迁移任务的数组区间
            else if (U.compareAndSwapInt(this, TRANSFERINDEX, nextIndex, nextBound = (nextIndex > stride ? nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            //扩容结束后做后续工作，将 nextTable 设置为 null，表示扩容已结束，将 table 指向新数组，sizeCtl 设置为扩容阈值
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            //每当一条线程扩容结束就会更新一次 sizeCtl 的值，进行减 1 操作
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                //(sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT 成立，说明该线程不是扩容大军里面的最后一条线程，直接return回到上层while循环
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                //(sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT 说明这条线程是最后一条扩容线程
                //之所以能用这个来判断是否是最后一条线程，因为第一条扩容线程进行了如下操作：
                //    U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2)
                //除了修改结束标识之外，还得设置 i = n; 以便重新检查一遍数组，防止有遗漏未成功迁移的桶
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            //遇到数组上空的位置直接放置一个占位对象，以便查询操作的转发和标识当前处于扩容状态
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            //数组上遇到hash值为MOVED，也就是 -1 的位置，说明该位置已经被其他线程迁移过了，将 advance 设置为 true ，以便继续往下一个桶检查并进行迁移操作
            advance = true; // already processed
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    //该节点为链表结构
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        //遍历整条链表，找出 lastRun 节点
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        //根据 lastRun 节点的高位标识(0 或 1)，首先将 lastRun设置为 ln 或者 hn 链的末尾部分节点，后续的节点使用头插法拼接
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        //使用高位和低位两条链表进行迁移，使用头插法拼接链表
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        //setTabAt方法调用的是 Unsafe 类的 putObjectVolatile 方法
                        //使用 volatile 方式的 putObjectVolatile 方法，能够将数据直接更新回主内存，并使得其他线程工作内存的对应变量失效，达到各线程数据及时同步的效果
                        //使用 volatile 的方式将 ln 链设置到新数组下标为 i 的位置上
                        setTabAt(nextTab, i, ln);
                        //使用 volatile 的方式将 hn 链设置到新数组下标为 i + n(n为原数组长度) 的位置上
                        setTabAt(nextTab, i + n, hn);
                        //迁移完成后使用 volatile 的方式将占位对象设置到该 hash 桶上，该占位对象的用途是标识该hash桶已被处理过，以及查询请求的转发作用
                        setTabAt(tab, i, fwd);
                        //advance 设置为 true 表示当前 hash 桶已处理完，可以继续处理下一个 hash 桶
                        advance = true;
                    }
                    //该节点为红黑树结构
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        //lo 为低位链表头结点，loTail 为低位链表尾结点，hi 和 hiTail 为高位链表头尾结点
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        //同样也是使用高位和低位两条链表进行迁移
                        //使用for循环以链表方式遍历整棵红黑树，使用尾插法拼接 ln 和 hn 链表
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            //这里面形成的是以 TreeNode 为节点的链表
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        //形成中间链表后会先判断是否需要转换为红黑树：
                        //1、如果符合条件则直接将 TreeNode 链表转为红黑树，再设置到新数组中去
                        //2、如果不符合条件则将 TreeNode 转换为普通的 Node 节点，再将该普通链表设置到新数组中去
                        //(hc != 0) ? new TreeBin<K,V>(lo) : t 这行代码的用意在于，如果原来的红黑树没有被拆分成两份，那么迁移后它依旧是红黑树，可以直接使用原来的 TreeBin 对象
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        //setTabAt方法调用的是 Unsafe 类的 putObjectVolatile 方法
                        //使用 volatile 方式的 putObjectVolatile 方法，能够将数据直接更新回主内存，并使得其他线程工作内存的对应变量失效，达到各线程数据及时同步的效果
                        //使用 volatile 的方式将 ln 链设置到新数组下标为 i 的位置上
                        setTabAt(nextTab, i, ln);
                        //使用 volatile 的方式将 hn 链设置到新数组下标为 i + n(n为原数组长度) 的位置上
                        setTabAt(nextTab, i + n, hn);
                        //迁移完成后使用 volatile 的方式将占位对象设置到该 hash 桶上，该占位对象的用途是标识该hash桶已被处理过，以及查询请求的转发作用
                        setTabAt(tab, i, fwd);
                        //advance 设置为 true 表示当前 hash 桶已处理完，可以继续处理下一个 hash 桶
                        advance = true;
                    }
                }
            }
        }
    }
}
```

### 图解

#### CPU核数与迁移任务hash桶数量分配的关系

![cHashMap扩容1](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420163516.png)

#### 单线程下线程的任务分配与迁移操作

![image-20210603215940951](https://gitee.com/zhang-songyao/blog-images/raw/master/20210603215943.png)

#### 多线程如何分配任务？

![20210420162800](https://gitee.com/zhang-songyao/blog-images/raw/master/20210603220036.png)

引入了一个ForwardingNode类，在一个线程发起扩容的时候，就会改变sizeCtl这个值，其含义如下：

```
sizeCtl ：默认为0，用来控制table的初始化和扩容操作，具体应用在后续会体现出来。  
-1 代表table正在初始化  
-N 表示有N-1个线程正在进行扩容操作  
其余情况：  
1、如果table未初始化，表示table需要初始化的大小。  
2、如果table初始化完成，表示table的容量，默认是table大小的0.75倍  
```

扩容时候会判断这个值，如果超过阈值就要扩容，首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素f，初始化一个forwardNode实例fwd，如果f == null，则在table中的i位置放入fwd，否则采用头插法的方式把当前旧table数组的指定任务范围的数据给迁移到新的数组中，然后给旧table原位置赋值fwd。直到遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍 ，扩容完成。在此期间如果其他线程的有读写操作都会判断head节点是否为forwardNode节点，如果是就帮助扩容

#### 普通链表如何迁移？

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210603220123.png)

#### 什么是 lastRun 节点？

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210603220147.png)

#### 红黑树如何迁移？

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210603220157.png)

#### hash桶迁移中以及迁移后如何处理存取请求？

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210603220215.png)

##### 在扩容时读写操作如何进行

- 对于get读操作，如果当前节点有数据，还没迁移完成，此时不影响读，能够正常进行。 如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时get线程会帮助扩容。 
- 对于put/remove写操作，如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时写线程会帮助扩容，如果扩容没有完成，当前链表的头节点会被锁住，所以写线程会被阻塞，直到扩容完成。

#### 多线程迁移任务完成后的操作

![image-20210612100819689](https://gitee.com/zhang-songyao/blog-images/raw/master/20210612100820.png)

### 对于size和迭代器是弱一致性

- volatile修饰的数组引用是强可见的，但是其元素却不一定，所以，这导致size的根据sumCount的方法并不准确。 
- 同理Iteritor的迭代器也一样，并不能准确反映最新的实际情况 

### 扩展问题

1、为什么HashMap的容量会小于数组长度？

答：HashMap是为了通过hash值计算出index，从而最快速的访问 。如果容量大于数组很多的话再加上散列算法不是非常优秀的情况下很容易出现链表过长的情况，虽然现在出现了红黑树，但是速度依旧不如直接定位到某个数组位置直接获取元素的速度快，所以最理想的情况是数组的每个位置放入一个元素，这样定位最快，从而访问也最快，集合容量小于数组长度的原因在于尽量去分散元素的分布，相当于是拉长了分布的范围，尽量减少集中到一起的概率，从而提高访问的速度，同时，负载因子只要小于 1 ，就不存在容量等于数组长度的情况 。

 

2、扩容期间在未迁移到的hash桶插入数据会发生什么？

答：只要插入的位置扩容线程还未迁移到，就可以插入，当迁移到该插入的位置时，就会阻塞等待插入操作完成再继续迁移 。

 

3、正在迁移的hash桶遇到 get 操作会发生什么？

答：在扩容过程期间形成的 hn 和 ln链 是使用的类似于复制引用的方式，也就是说 ln 和 hn 链是复制出来的，而非原来的链表迁移过去的，所以原来 hash 桶上的链表并没有受到影响，因此从迁移开始到迁移结束这段时间都是可以正常访问原数组 hash 桶上面的链表，迁移结束后放置上fwd，往后的访问请求就直接转发到扩容后的数组去了 。

 

4、如果 lastRun 节点正好在一条全部都为高位或者全部都为低位的链表上，会不会形成死循环？

答：

- 在数组长度为64之前会导致一直扩容，但是到了64或者以上后就会转换为红黑树，因此不会一直死循环 。首先要搞明白lastRun之前的节点和之后的节点有什么区别，lastRun的意思并不是最后一个和首节点的runBit不一样的节点，而是和前一个节点的runbit不一样的节点，仔细看runBit在遍历比较的过程中是变化的，只要当前节点的h和前一个runbit不一样就把lastRun赋值为当前节点，runbit赋值为当前节点的hash，这样最后找到的lastRun就能把一条链表分成两条，一条是lastRun前面的节点和一条lastRun后面的节点，lastRun后面的节点的hash都和lastRun相等，所以只需要复制lastRun到新数组就可以了，lastRun后面的节点会顺带过去，这算是作者的优化吧。lastRun前面的节点hash值不一定相等，所以需要一个个再hash复制到新数组（这是java1.7中ConcurrentHashMap的复制 迁移思路）

- 1.8中又进行了优化，在找lastRun基础上又把一条链表分为高低位两条链表，这样复制迁移的时候只需要复制两条链表就可以了。而为什么java1.7不找高低位呢，这里就涉及到java1.7在解决哈希冲突时，新节点是以头插法的方式，复制迁移的时候也是头插法，这样一条链表就倒序迁移了，新数组中的占位节点的位置是变化的，不过我觉得java1.7强行找高低位也是可以的。

 

5、扩容后 ln 和 hn 链不用经过 hash 取模运算，分别被直接放置在新数组的 i 和 n + i 的位置上，那么如何保证这种方式依旧可以用过 h & (n - 1) 正确算出 hash 桶的位置？

答：如果 fh & n-1 = i ，那么扩容之后的 hash 计算方法应该是 fh & 2n-1 。 因为 n 是 2 的幂次方数，所以 如果 n=16， n-1 就是 1111(二进制)， 那么 2n-1 就是 11111 (二进制) 。 其实 fh & 2n-1 和 fh & n-1 的值区别就在于多出来的那个 1 => fh & (10000) 这个就是两个 hash 的区别所在 。而 10000 就是 n 。所以说 如果 fh 的第五 bit 不是 1 的话 fh & n = 0 => fh & 2n-1 == fh & n-1 = i 。 如果第5位是 1 的话 。fh & n = n => fh & 2n-1 = i+n 。

 

6、我们都知道，并发情况下，各线程中的数据可能不是最新的，那为什么 get 方法不需要加锁？

答：get操作全程不需要加锁是因为Node的成员val是用volatile修饰的 。

 

7、ConcurrentHashMap 的数组上插入节点的操作是否为原子操作，为什么要使用 CAS 的方式？ 

答：待解决 。

 

8、扩容完成后为什么要再检查一遍？

答：为了避免遗漏hash桶，至于为什么会遗漏hash桶。

# concurrentHashMap1.7和1.8的区别

|  区别点  |                            JDK1.7                            |                            JDK1.8                            |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 整体结构 |   分段锁segment + HashEntry + CAS + ReentrantLock + Unsafe   | synchronized + CAS + volatile + Node + TreeNode + ForwardNode + Unsafe + TreeBin |
| 锁的对象 |                      Segment锁的粒度大                       |                Node（桶位的头节点）锁的粒度小                |
|   put    | 先定位segment，再定位桶，put全程加锁，没有获取锁的线程提前找桶的位置，并最多自旋（CPU核数大于1时否则自旋1次）64次获取锁，超过则挂起 | 类似HashMap，可以直接定位到桶，拿到first节点后进行判断，1、为空则CAS插入；2、为-1则说明在扩容，则跟着一起扩容；3、else则加锁put（类似1.7） |
|   get    |                        需要使用Unsafe                        | 由于value声明为volatile，保证了修改的可见性，因此不需要加锁  |
|  resize  | （rehash）当segment中的HashEntry数组大于扩容阈值是进行扩容，只扩容当前segment的HashEntry，全程加锁（因为segment中的put方法加锁） | 支持并发扩容，HashMap扩容在1.8中由头插（为了避免死循环问题），ConcurrentHashmap也是，迁移也是从尾部开始头插法插入高位和低位链表然后接入新的散列表的相应位置，扩容前在桶的头部放置一个hash值为-1的节点，这样别的线程访问时就能判断是否该桶已经被其他线程处理过了。 |
|   size   | 很经典的思路：计算两次，如果不变则返回计算结果，若不一致，则锁住所有的Segment求和。 | 用baseCount来存储当前的节点个数，这就设计到baseCount并发环境下修改的问题 |
|          |                                                              |                                                              |
|          |                                                              |                                                              |
|          |                                                              |                                                              |



# HashMap（1.8）

## 1、什么是哈希

核心理论：Hash也称散列、哈希、对应的英文单词都是Hash，基本原理就是把任意长度的输入，通过Hash算法变成固定长度的输出。这个映射的规则就是对应的Hash算法，而原始的数据映射后的二进制串就是哈希值。

>Hash的特点

- 从hash值不可以反向推导出原始的数据
- 输入的数据微小变化都会得到完全不同的hash值，相同的数据会得到相同的值
- 哈希算法的执行效率要高效，长文本也能快速的计算出哈希值
- hash算法的冲突概率要小
  - 由于hash的原理是将输入空间的值映射成hash空间内，而hash值的空间远小于输入的空间。根据**抽屉原理**，一定会存在不同的输入被映射成相同输出情况

>抽屉原理：桌上有10个苹果，要把这10个苹果放入9个抽屉中，无论怎么放，至少会有一个抽屉里面放不少于两个苹果

## 2、HashMap继承关系

![hashmap继承关系](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162806.png)

## 3、Node数据结构

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + val
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(val
    }
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

## 4、低层存储结构

数组 + 链表 + 红黑树

![hashmap底层存储结构](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162807.png)

## 5、put数据原理分析

![put实现原理](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162808.png)

>路由寻址公式

```java
(table.length - 1) & node.hash
```

 n & (m - 1)  相当于 n  % m（路由寻址其实就是对长度取余）

## 6、什么是hash碰撞

>当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被`其他元素占用`了，其实这就是所谓的**哈希冲突**，也叫**哈希碰撞**。

哈希函数的设计至关重要，好的哈希函数会尽可能地保证 计算简单和散列地址分布均匀,但是，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。那么哈希冲突如何解决呢？
哈希冲突的解决方案有多种:

- **开放地址法**（发生冲突，继续寻找下一块未被占用的存储地址）
- **再散列函数法**
- **链地址法**，而HashMap即是采用了链地址法，也就是数组+链表的方式

简单来说，HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好。

## 7、HashMap的源码分析

### 7.1、属性

>默认table大小

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

>table最大长度

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
```

>负载因子

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

>树化阈值

```java
static final int TREEIFY_THRESHOLD = 8;
```

>链化阈值

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

>最小树化的元素个数

```java
static final int MIN_TREEIFY_CAPACITY = 64;
```

>扩容阈值

```java
int threshold;
threshold = capacity * loadFactor (扩容阈值 = 当前容量 * 负载因子)
```

### 7.2、构造方法

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

>tableSizeFor的作用：返回一个大于当前的cap的一个数字，这个数字一定是2的次方数

```java
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

将任何一个二进制数 0001 1101 1100 => 0001 1111 1111 将任何一个二进制数第一位1之后的数字全部置1

>HashMap 的底层数组长度为何总是2的n次方

- 使数据分布均匀，减少碰撞
- 当length为2的n次方时，h&(length - 1) 就相当于对length取模，而且在速度、效率上比直接取模要快得多

### 7.3、put方法

```java
public V put(K key, V value) {
	return putVal(hash(key), key, value, false, true);
}
```

>hash函数

```java
static final int hash(Object key) {
	int h;
	return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

>为什么是异或？

因为两个二进制数进行异或后产生0和1的概率都为0.5，而区域操作产生的概率不同

hash结果的返回值：key的hashCode值高16位和低16位异或后的结果

为什么要这样做？

因为在HashMap未扩容之前，table的长度非常小（16,32,64...）这时根据路由寻址算法(table.length - 1) & node.hash只能使用到较低的位数，之所以高16位和低16位异或，是为了让key的高16位也参与到路由选址的过程中。减少hash碰撞。

为什么要用异或运算符?

保证了对象的`hashCode`的32位值只要有一位发生改变,整个`hash()`返回值就会改变。尽可能的减少碰撞。

>putVal方法

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    //tab引用当前HashMap的散列表
    //p当前散列表的元素
    //n散列表长度
    //i路由寻址的结果
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    
    //如果当前散列表未初始化(散列表为空或者长度为0),则初始化散列表
    //懒加载，在put是才会初始化（节省内存）
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    
    //如果寻找到桶位，此时这个桶为null，直接将k，v -> node放入桶中
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        
        //桶位不为空
        //e与当前key相等的节点
        Node<K,V> e; K k;
        //当前桶位的头节点key值和插入key值相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //当前节点已经树化
        else if (p instanceof TreeNode)
            //e不为null找到了key相同的节点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //当前节点还未树化
        else {
            //循环链表
            for (int binCount = 0; ; ++binCount) {
                //循环到链表末尾，直接在末尾添加node
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //找到和key相等的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //找到了和key相等的节点，替换
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //没有找到和key相等的节点
    ++modCount;
    //是否需要扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

#### putTreeVal

>putTreeVal红黑树版的插入

```java
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                               int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    //判断当前节点是否为根节点然后拿到当前根节点
    TreeNode<K,V> root = (parent != null) ? root() : this;
    //遍历红黑树
    for (TreeNode<K,V> p = root;;) {
        //p当前节点
        //ph当前节点的hash
        //pk当前节点的key
        int dir, ph; K pk;
        if ((ph = p.hash) > h)
            dir = -1;
        else if (ph < h)
            dir = 1;
        //根节点等于插入节点
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
       	/**
		*下面的步骤主要就是当发生冲突 也就是hash相等的时候
		* 检验是否有hash相同 并且equals相同的节点。
		* 也就是检验该键是否存在与该树上
		*/
		//说明进入下面这个判断的条件是 hash相同 但是equal不同
		// 没有实现Comparable<C>接口或者 实现该接口 并且 k与pk Comparable比较结果相同
        else if ((kc == null &&
                  //如果对象k的类是C，如果C实现了Comparable<C>接口，那么返回k的类，否则返回null
                  (kc = comparableClassFor(k)) == null) ||
                 //kc和pk类型一样返回比较值，否则返回0
                 (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                //ch：p的子节点
                //kc当前k的类
                //k插入节点的key
                TreeNode<K,V> q, ch;
                searched = true;
                //在当前节点的左子树和右子树中寻找与插入节点相同的点
                //找到返回
                if (((ch = p.left) != null &&
                     (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            //说明红黑树中没有与之equals相等的  那就必须进行插入操作
			//打破平衡的方法的 分出大小 结果 只有-1 1 
            dir = tieBreakOrder(k, pk);
        }
        //下列操作进行插入节点
		//xp 保存当前节点
        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            Node<K,V> xpn = xp.next
            //创建出一个新的节点;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
             //小于父亲节点  新节点放左孩子位置
             xp.left = x;
             else
         //大于父亲节点  放右孩子位置
                xp.right = x;
                    
                //维护双链表关系
                xp.next = x;
				x.parent = x.prev = xp;
				if (xpn != null)
					((TreeNode<K,V>)xpn).prev = x;
				//将root移到table数组的i 位置的第一个节点
				//插入操作过红黑树之后 重新调整平衡。
				moveRootToFront(tab, balanceInsertion(root, x));
				return null;
        }
    }
}
```

#### find

>find

```java
 /**
  * 从根节点p开始查找指定hash值和关键字key的结点
  * 当第一次使用比较器比较关键字时，参数kc储存了关键字key的 比较器类别
  */
 final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
     TreeNode<K,V> p = this;
     do {
         int ph, dir; K pk;
         TreeNode<K,V> pl = p.left, pr = p.right, q;
         if ((ph = p.hash) > h)			//如果给定哈希值小于当前节点的哈希值，进入左节点
             p = pl;
         else if (ph < h)				//如果大于，进入右结点
             p = pr;
         else if ((pk = p.key) == k || (k != null && k.equals(pk)))	//如果哈希值相等，且关键字相等，则返回当前节点
             return p;
         else if (pl == null)		//如果左节点为空，则进入右结点
             p = pr;
         else if (pr == null)		//如果右结点为空，则进入左节点
             p = pl;
         else if ((kc != null ||
                 (kc = comparableClassFor(k)) != null) &&
                 (dir = compareComparables(kc, k, pk)) != 0)		//如果不按哈希值排序，而是按照比较器排序，则通过比较器返回值决定进入左右结点
             p = (dir < 0) ? pl : pr;
         else if ((q = pr.find(h, k, kc)) != null)		//如果在右结点中找到该关键字，直接返回
             return q;
         else
             p = pl;								//进入左节点
     } while (p != null);
     return null;
 }
```

#### treeifyBin

> treeifyBin 将当前链表转换为一个双链表

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    //e链表当前节点
    //index路由寻址的结果
    //n桶长
    int n, index; Node<K,V> e;
    //如果桶位为空或者桶的长度小于最小树化的元素个数先扩容
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    //当前桶位不为空
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        //hd双向链表的头节点
        //双向链表的尾节点
        TreeNode<K,V> hd = null, tl = null;
        //循环当前桶位的链表创建一个双向链表
        do {
            //创建一个和p相同的TreeNode，next、left、right都为空
            TreeNode<K,V> p = replacementTreeNode(e, null);
            //填充prev和next
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

#### replacementTreeNode

>replacementTreeNode 获得一个TreeNode 参数和 Node值相等的

```java
//创建一个和p相同的TreeNode节点，next、left、right都为空
TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    return new TreeNode<>(p.hash, p.key, p.value, next);
}
```

#### treeify

>treeify 真正的树化操作将双向链表插入以root为根节点的的红黑树

```java
/**
 * Forms tree of the nodes linked from this node.
 * @return root of tree
 */
final void treeify(Node<K,V>[] tab) {
    TreeNode<K,V> root = null;
    //循环当前tab
    for (TreeNode<K,V> x = this, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;
        //如果当前红黑树为空，将第一个节点赋值给根节点
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        }
        else {
            //k当前插入红黑树节点的key
            K k = x.key;
            int h = x.hash;
            //kc当前k的类型（是否实现Comparable）
            Class<?> kc = null;
            for (TreeNode<K,V> p = root;;) {
                //ph当前节点的hash值
                //pk当前节点的key值
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                          //如果对象k的类是C，如果C实现了Comparable<C>接口，那么返回k的类，否则返回null
                          (kc = comparableClassFor(k)) == null) ||
                         	// 如果pk所属的类是kc，返回k.compareTo(x)的比较结果
							// 如果pk为空，或者其所属的类不是kc，返回0
                         //插入节点pk的类型不是当前节点kc的类型
                         (dir = compareComparables(kc, k, pk)) == 0)
                    //比较插入节点和当前树节点的key值
                    dir = tieBreakOrder(k, pk);
                TreeNode<K,V> xp = p;
                //dir小于等于0插入红黑树的左边，反之插入右边
                //如果p的左边或者右边为空直接插入当前节点x
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    //插入平衡返回当前根节点
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }
    moveRootToFront(tab, root);
}
```

#### comparableClassFor

>comparableClassFor 判断类是否实现了Comparable接口

```java
//如果对象x的类是C，如果C实现了Comparable<C>接口，那么返回C，否则返回null
static Class<?> comparableClassFor(Object x) {
    if (x instanceof Comparable) {
        Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
        //为什么如果x是个字符串就直接返回c了呢 ? 因为String  实现了 Comparable 接口，可参考如下String类的定义
        if ((c = x.getClass()) == String.class) // bypass checks
            return c;
        //如果 c 不是字符串类，获取c直接实现的接口（如果是泛型接口则附带泛型信息）
        if ((ts = c.getGenericInterfaces()) != null) {
            for (int i = 0; i < ts.length; ++i) {
                // 如果当前接口t是个泛型接口 
                // 如果该泛型接口t的原始类型p 是 Comparable 接口
                // 如果该Comparable接口p只定义了一个泛型参数
                // 如果这一个泛型参数的类型就是c，那么返回c
                if (((t = ts[i]) instanceof ParameterizedType) &&
                    ((p = (ParameterizedType)t).getRawType() ==
                     Comparable.class) &&
                    (as = p.getActualTypeArguments()) != null &&
                    as.length == 1 && as[0] == c) // type arg is c
                    return c;
            }
        }
    }
    return null;
}
```

#### compareComparables

>compareComparables 如果类型相同返回比较值，如果不同返回0

```java
// 如果x所属的类是kc，返回k.compareTo(x)的比较结果
// 如果x为空，或者其所属的类不是kc，返回0
static int compareComparables(Class<?> kc, Object k, Object x) {
    return (x == null || x.getClass() != kc ? 0 :
            ((Comparable)k).compareTo(x));
}
```

#### tieBreakOrder

>tieBreakOrder 强行比较两个节点的大小（一定返回比较值）

```java
用这个方法来比较两个对象，返回值要么大于0，要么小于0，不会为0
也就是说这一步一定能确定要插入的节点要么是树的左节点，要么是右节点，不然就无继续满足二叉树结构了
先比较两个对象的类名，类名是字符串对象，就按字符串的比较规则
如果两个对象是同一个类型，那么调用本地方法为两个对象生成hashCode值，再进行比较，hashCode相等的话返回-1
static int tieBreakOrder(Object a, Object b) {
    int d;
    if (a == null || b == null ||
        (d = a.getClass().getName().
         compareTo(b.getClass().getName())) == 0)
        d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
             -1 : 1);
    return d;
}
```

#### balanceInsertion

>balanceInsertion 插入平衡，维持红黑树的平衡

```java
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                            TreeNode<K,V> x) {
    x.red = true;
    //xp x的父节点
    //xpp x的祖父节点
    //xppl x的左叔叔节点
    //xppr x的右叔叔节点
    for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
        //插入节点的父节点为空，空树
        if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }
        //父节点为黑节点并且祖父节点不存在
        //直接插入不需要做任何平衡操作
        else if (!xp.red || (xpp = xp.parent) == null)
            return root;
        //插入节点的父节点是祖父节点的左节点
        if (xp == (xppl = xpp.left)) {
            //右叔叔节点存在且为红色
            //叔叔节点和父节点变为黑色，祖父节点变为红色，将祖父节点设置为当前插入节点
            if ((xppr = xpp.right) != null && xppr.red) {
                xppr.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            else {//叔叔节点为空或为黑节点
                //插入节点是父节点的右节点LR双红
                if (x == xp.right) {
                    //对父节点进行左旋
                    root = rotateLeft(root, x = xp);
                    //重新赋值xpp
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                //LL双红
                if (xp != null) {
                    //父节点和祖父节点改为红色对祖父节点进行右旋
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        root = rotateRight(root, xpp);
                    }
                }
            }
        }//插入的节点的父节点是祖父节点的右节点
        else {
            //左叔叔节点不为空，且为红色
            //叔叔节点和父节点变为黑色，祖父节点变为红色，将祖父节点设置为当前插入节点
            if (xppl != null && xppl.red) {
                xppl.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            else {//叔叔节点为空或为黑节点
                //插入节点是父节点的左节点RL双红
                if (x == xp.left) {
                     //对父节点进行右旋
                    root = rotateRight(root, x = xp);
                    //重新赋值xpp
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                //LL双红
                if (xp != null) {
                    //父节点和祖父节点改为红色对祖父节点进行右旋
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        root = rotateLeft(root, xpp);
                    }
                }
            }
        }
    }
}
```

>rotateLeft

```java
左旋
static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                      TreeNode<K,V> p) {
    TreeNode<K,V> r, pp, rl;
    if (p != null && (r = p.right) != null) {
        if ((rl = p.right = r.left) != null)
            rl.parent = p;
        if ((pp = r.parent = p.parent) == null)
            (root = r).red = false;
        else if (pp.left == p)
            pp.left = r;
        else
            pp.right = r;
        r.left = p;
        p.parent = r;
    }
    return root;
```

>rotateRight

```java
右旋
static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                       TreeNode<K,V> p) {
    TreeNode<K,V> l, pp, lr;
    if (p != null && (l = p.left) != null) {
        if ((lr = p.left = l.right) != null)
            lr.parent = p;
        if ((pp = l.parent = p.parent) == null)
            (root = l).red = false;
        else if (pp.right == p)
            pp.right = l;
        else
            pp.left = l;
        l.right = p;
        p.parent = l;
    }
    return root;
}
```

#### moveRootToFront

>moveRootToFront 确保当前红黑树仍是一条有序的双链表

```java
把红黑树的根节点设为  其所在的数组槽 的第一个元素
首先明确：TreeNode既是一个红黑树结构，也是一个双链表结构
这个方法里做的事情，就是保证树的根节点一定也要成为链表的首节点

static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
    int n;
    if (root != null && tab != null && (n = tab.length) > 0) { // 根节点不为空 并且 HashMap的元素数组不为空
        int index = (n - 1) & root.hash; // 根据根节点的Hash值 和 HashMap的元素数组长度  取得根节点在数组中的位置
        TreeNode<K,V> first = (TreeNode<K,V>)tab[index]; // 首先取得该位置上的第一个节点对象
        if (root != first) { // 如果该节点对象 与 根节点对象 不同
            Node<K,V> rn; // 定义根节点的后一个节点
            tab[index] = root; // 把元素数组index位置的元素替换为根节点对象
            TreeNode<K,V> rp = root.prev; // 获取根节点对象的前一个节点
            if ((rn = root.next) != null) // 如果后节点不为空 
                ((TreeNode<K,V>)rn).prev = rp; // root后节点的前节点  指向到 root的前节点，相当于把root从链表中摘除
            if (rp != null) // 如果root的前节点不为空
                rp.next = rn; // root前节点的后节点 指向到 root的后节点
            if (first != null) // 如果数组该位置上原来的元素不为空
                first.prev = root; // 这个原有的元素的 前节点 指向到 root，相当于root目前位于链表的首位
            root.next = first; // 原来的第一个节点现在作为root的下一个节点，变成了第二个节点
            root.prev = null; // 首节点没有前节点
        }
 
        /*
         * 这一步是防御性的编程
         * 校验TreeNode对象是否满足红黑树和双链表的特性
         * 如果这个方法校验不通过：可能是因为用户编程失误，破坏了结构（例如：并发场景下）；也可能是TreeNode的实现有问题（这个是理论上的以防万一）；
         */ 
        assert checkInvariants(root); 
    }	
}
```

#### checkInvariants

>checkInvariants 验证红黑树的准确性

```java
验证红黑树的准确性，checkInvariants（该方法需要在启动参数中加 -ae 才能生效）
static <K, V> boolean checkInvariants(HashMap.TreeNode<K, V> t) {
            // tp-父节点，tl-左子节点，tr-右子节点，tp-前驱节点，tn-后继节点
            HashMap.TreeNode<K, V> tp = t.parent, tl = t.left, tr = t.right,
                    tb = t.prev, tn = (HashMap.TreeNode<K, V>) t.next;
            // 当出现以下任一情况时，红黑树或者双向链表不正确
            // 1、如果前驱节点存在，但是前驱节点的后继节点不是当前节点
            if (tb != null && tb.next != t)
                return false;
            // 2、如果后继节点存在，但是后继节点的前驱节点不是当前节点
            if (tn != null && tn.prev != t)
                return false;
            // 3、父节点存在，但是父节点的左子节点、右子节点均不是当前节点
            if (tp != null && t != tp.left && t != tp.right)
                return false;
            // 4、左子节点存在。但是左子节点的父节点不是当前节点或者左子节点的hash值大于当前节点的hash值
            if (tl != null && (tl.parent != t || tl.hash > t.hash))
                return false;
            // 5、右子节点存在。但是右子节点的父节点不是当前节点或者右子节点的hash值小于当前节点的hash值
            if (tr != null && (tr.parent != t || tr.hash < t.hash))
                return false;
            // 6、当前节点是红色，孩子节点也是红色
            if (t.red && tl != null && tl.red && tr != null && tr.red)
                return false;
            // 递归验证左子树
            if (tl != null && !checkInvariants(tl))
                return false;
            // 递归验证右子树
            if (tr != null && !checkInvariants(tr))
                return false;
            // 都正确返回true
            return true;
}
```

### 7.4、resize方法

```java
final Node<K,V>[] resize() {
    //oldTab引用指向扩容前的散列表
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    
    /*<========================newCap、newThr赋值==========================>*/
    //扩容之前有初始容量
    if (oldCap > 0) {
        //如果初始容量大于table最大长度
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //扩容后table容量小于table最大长度并且扩容前的table容量大于默认table长度
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    //调用了
    //HashMap(int initialCapacity, float loadFactor)
    //HashMap(map)
    //初始扩容阈值被赋值但是table没有初始化(第一次扩容)
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // HashMap()
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //newThr为零，计算一个newThr
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    /*<========================newCap、newThr赋值==========================>*/
 
    
    /*<========================扩容==========================>*/
    @SuppressWarnings({"rawtypes","unchecked"})
    	//创建一个新的散列表
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //本次扩容之前原来的散列表有数据
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {	
            //当前节点
            Node<K,V> e;
            //当前桶位有数据
            if ((e = oldTab[j]) != null) {
                //方便JVM GC回收
                oldTab[j] = null;
                //第一种情况，当前桶位只有一个节点，直接重新路由寻址赋值
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                //第二种情况，当前桶位已经树化
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                //第三种情况，当前桶位已经莲链化
                else { // preserve order
                    //原桶位的链表拆为两个链表，分为低位和高位链表
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        //使用尾插法按顺序拆分为两个链表
                        next = e.next;
                        //oldCap = 16
                        //e.hash & 10000
                        //等于0表示最高位为0插入低位链表
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
                    //低位链表
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    //高位链表
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    /*<========================扩容==========================>*/
    return newTab;
}
```

>16桶位链化节点扩容为32桶位

15号桶位的节点有一个共同特征，hash值的低四位为1111所以	分为两种情况**01111**或者**11111**,在32桶位的散列表中路由寻址计算公式(table.length - 1) & hash = (11111) & hash可以看出**01111**重新寻址桶位为15,**11111**重新寻址桶位为31(15 + oldCap)。

![image-20210415224732200](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162809.png)

>jdk1.7 resize()方法在多线程访问下会出现扩容死循环问题

![image-20210417172612423](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162810.png)

#### split

```java
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    //tab新table
    //index为当前桶位
    //bit原数组的table大小
    TreeNode<K,V> b = this;
    // Relink into lo and hi lists, preserving order
    //低位和高位
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0;
    //循环当前桶位的节点（双链表）
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;
        //低位（尾插法）
        if ((e.hash & bit) == 0) {
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
            ++lc;
        }
        //高位（尾插法）
        else {
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
            ++hc;
        }
    }
    if (loHead != null) {
        //当前链表小于链化阈值
        if (lc <= UNTREEIFY_THRESHOLD)
            //链化
            tab[index] = loHead.untreeify(map);
        else {
            //重新树化当前双链表
            tab[index] = loHead;
            if (hiHead != null) // (else is already treeified)
                loHead.treeify(tab);
        }
    }
    if (hiHead != null) {
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            tab[index + bit] = hiHead;
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

#### untreeify

>untreeify

```java
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    //循环双链表(尾插法)
    for (Node<K,V> q = this; q != null; q = q.next) {
        Node<K,V> p = map.replacementNode(q, null);
        if (tl == null)
            hd = p;
        else
            tl.next = p;
        tl = p;
    }
    return hd;
}
```

>replacementNode

```java
返回一个和TreeNode完全相等的Node
Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    return new Node<>(p.hash, p.key, p.value, next);
}
```

### 7.5、get方法

```java
public V get(Object key) {
	Node<K,V> e;
	return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

>getNode()方法

```java
final Node<K,V> getNode(int hash, Object key) {
    //tab引用指向当前散列表
    //first桶位的第一个节点
    //n table长度
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //散列表不为空并且hash(key)路由寻址后的桶位上第一个节点存在
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //第一种情况：第一个节点就是要找的节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //第二种情况：当前节点已经树化
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //第三种情况：当前节点已经链化
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

### 7.6、remove方法

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
```

>removeNode()方法

```java
//matchValue 删除是否需要匹配value
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    //tab引用指向当前散列表
    //p当前桶位的第一个节点
    //index寻址结果
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //散列表不为空并且hash(key)路由寻址后的桶位上第一个节点存在
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        //要删除的节点
        Node<K,V> node = null, e; K k; V v;
        //如果当前节点恰好是要删除的节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        //当前节点不是要删除的节点
        else if ((e = p.next) != null) {
            //节点已经树化
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            //节点已经链化
            else {
                do {
                    //找到指定节点
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    //当前节点的前驱结点
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //找到了要删除的节点
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            //如果树化了
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            //要删除的节点是桶位的第一个节点
            else if (node == p)
                tab[index] = node.next;
            //要删除的节点是链表中的其他节点
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    //没有找到
    return null;
}
```

#### removeTreeNode

>removeTreeNode

```java
final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab,
                          boolean movable) {
    //n当前table的长度
    int n;
    //如果，当前散列表未初始化或者当前长度为0直接返回
    if (tab == null || (n = tab.length) == 0)
        return;
    //路由寻址当前前节点的桶位
    int index = (n - 1) & hash;
    //first当前双链表的头链表
    //root当前红黑树的根节点
    //rl 根节点的左节点
    //succ当前节点的后继节点
    //pred当前节点的前驱节点
    TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl;
    TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;
    //前驱节点为空 删除的节点为头节点
    if (pred == null)
        tab[index] = first = succ;
    else
        //直接删除当前节点
        pred.next = succ;
    //succ的前驱节点指向pred
    if (succ != null)
        succ.prev = pred;
    //删除的节点是头节点 直接返回
    if (first == null)
        return;
    //root节点的父节点不为空
    if (root.parent != null)
        //找到父节点赋值给root
        root = root.root();
    //判断--->链化
    if (root == null || root.right == null ||
        (rl = root.left) == null || rl.left == null) {
        tab[index] = first.untreeify(map);  // too small
        return;
    }
    //p 当前删除节点
    //pl 删除节点的左节点
    //pr 删除节点的右节点
    //replacement
    TreeNode<K,V> p = this, pl = left, pr = right, replacement;
    if (pl != null && pr != null) {
        TreeNode<K,V> s = pr, sl;
        while ((sl = s.left) != null) // find successor
            s = sl;
        boolean c = s.red; s.red = p.red; p.red = c; // swap colors
        TreeNode<K,V> sr = s.right;
        TreeNode<K,V> pp = p.parent;
        if (s == pr) { // p was s's direct parent
            p.parent = s;
            s.right = p;
        }
        else {
            TreeNode<K,V> sp = s.parent;
            if ((p.parent = sp) != null) {
                if (s == sp.left)
                    sp.left = p;
                else
                    sp.right = p;
            }
            if ((s.right = pr) != null)
                pr.parent = s;
        }
        p.left = null;
        if ((p.right = sr) != null)
            sr.parent = p;
        if ((s.left = pl) != null)
            pl.parent = s;
        if ((s.parent = pp) == null)
            root = s;
        else if (p == pp.left)
            pp.left = s;
        else
            pp.right = s;
        if (sr != null)
            replacement = sr;
        else
            replacement = p;
    }
    else if (pl != null)
        replacement = pl;
    else if (pr != null)
        replacement = pr;
    else
        replacement = p;
    if (replacement != p) {
        TreeNode<K,V> pp = replacement.parent = p.parent;
        if (pp == null)
            root = replacement;
        else if (p == pp.left)
            pp.left = replacement;
        else
            pp.right = replacement;
        p.left = p.right = p.parent = null;
    }
    TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);
    if (replacement == p) {  // detach
        TreeNode<K,V> pp = p.parent;
        p.parent = null;
        if (pp != null) {
            if (p == pp.left)
                pp.left = null;
            else if (p == pp.right)
                pp.right = null;
        }
    }
    if (movable)
        moveRootToFront(tab, r);
}
```

#### root

>root  返回当前的根节点

```java
/**
 * Returns root of tree containing this node.
 */
final TreeNode<K,V> root() {
    for (TreeNode<K,V> r = this, p;;) {
        if ((p = r.parent) == null)
            return r;
        r = p;
    }
}
```



### 7.7、replace方法

```java
@Override
public boolean replace(K key, V oldValue, V newValue) {
    Node<K,V> e; V v;
    if ((e = getNode(hash(key), key)) != null &&
        ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
        e.value = newValue;
        afterNodeAccess(e);
        return true;
    }
    return false;
}
@Override
public V replace(K key, V value) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) != null) {
        V oldValue = e.value;
        e.value = value;
        afterNodeAccess(e);
        return oldValue;
    }
    return null;
}
```

# JDK1.7 和1.8的HashMap区别

|       区别点       |                            JDK1.7                            |                            JDK1.8                            |
| :----------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    Hash方法不同    | 如果key为字符串直接返回处理结果。key的hashCode无规则扰乱，可以自己控制通过设置环境变量中的哈希种子 |                key的hashCode高16位异或低16位                 |
|  put与resize顺序   |                         先扩容后插入                         |                         先插入后扩容                         |
|      数据结构      |                          数组+链表                           |                     数组 + 链表 + 红黑树                     |
| 扩容后数据转移不同 | 通过hash和newCap - 1重新计算下标，头插法（多线程形成循环链表，遍历链表的时候产生死循环）(通过控制扩容来防止)1.7是通过更新hashSeed来修改hash值达到分散的目的 |      通过hash和newCap计算最高位1和0插入方法不同，尾插法      |
|      扩容条件      |        size大于扩容阈值，并且当前table[index]不为null        | size大于扩容阈值、当前桶位不为空并且table长度小于等于64（不树化），直接扩容 |
|  table的加载方式   |                  构造方法中直接初始化table                   |            懒加载，在第一次put是对table进行初始化            |



# 红黑树

>为什么有了平衡树还需要红黑树？

- 虽然平衡树解决了二叉树退化为近似链表的缺点，能够把查找时间控制在O(logn)，不过却不是最佳的
- 因为平衡树要求每个节点的左子树和右子树的高度差至多等于1，这个要求实在是太严了，导致每次进行插入/删除节点的时候，几乎都会破坏平衡树的的第二个规则，进而我们需要进行左旋和右旋来进行调整，使之再次成为一颗符合要求的平衡树。
- 显然，如果在插入、删除很频繁的场景中，平衡树需要频繁的进行调整，这会使平衡树的性能大打折扣，为了解决这个问题，产生了红黑树。

## [红黑树的概述](https://blog.csdn.net/qq_35190492/article/details/109503539?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162044246816780264081609%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162044246816780264081609&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-109503539.pc_search_result_before_js&utm_term=%E7%BA%A2%E9%BB%91%E6%A0%91)

### 先谈平衡树

> 红黑树本质其实是对概念模型：2-3-4树的一种实现

2-3-4树的阶数是4的B树，B树，全名Balance Tree，平衡树，这种结构主要用来做查询，它最重要的特性是在于平衡，这使得我们能够在最坏情况下也保持O(LogN)的时间复杂度实现查找

由于2-3-4树是一颗阶数为4的B树，所以他会存在以下节点：

- 2节点

- 3节点
- 4节点

2节点中存放着一个key[X]，两个指针，分别指向小于X的子节点和大于X的子节点；3节点中存放在两个key[X,Y],三个指针，分别指向小于X的子节点，介于X~Y之间的子节点和大于Y的子节点；4节点可依此类推。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210508155724.jpeg" alt="节点介绍" style="zoom:80%;" />

### 2-3-4树到红黑树的转化

红黑树是对概念模型2-3-4树的一种实现，由于直接进行不同节点间的转化会造成较大的开销，所以选择以二叉树为基础，在二叉树的属性中加入一个颜色属性来表示2-3-4树中不同的节点。

![B树到红黑树的转化](https://gitee.com/zhang-songyao/blog-images/raw/master/20210508164209.jpeg)

### 2-3树到红黑树的转化（左倾红黑树）

> 左倾红黑树：2-3树中较为特殊的一种转化-左倾红黑树。顾名思义，左倾红黑树限制了如果树中出现了红色节点，那么这个节点必须是左儿子

![B树到红黑树的转化](https://gitee.com/zhang-songyao/blog-images/raw/master/20210508164724.jpeg)

只要把左倾红黑树中的红色节点**顺时针方向旋转45°**使其与黑父平行，然后再将它们看作一个整体，你就会发现，这不就是一颗2-3树吗？

![B树到红黑树的转化](https://gitee.com/zhang-songyao/blog-images/raw/master/20210508164743.jpeg)

红黑树其实就是对概念模型2-3树（或者2-3-4树）的一种实现。

## 红黑树的性质

| 红黑树的性质                                                 |
| :----------------------------------------------------------- |
| 性质1：每个节点要么是黑色，要么是红色                        |
| 性质2：根节点是黑色，2-3树中如果根节点为2节点，那么它本来就对应红黑树中黑节点；如果根节点为3节点，也可以用黑色节点表示较大的那个元素，然后较小的元素作为左倾红节点存在于红黑树中】 |
| 性质3：每个叶子节点（NIL）是黑色                             |
| 性质4：每个红色节点的两个子节点一定都是黑色，不能有两个红色节点相连【2-3树中本来就规定没有4节点，2-3-4树中虽然有4节点，但是要求在红黑树中体现为一黑色节点带两红色儿子，分布左右，所以也不会有连续红节点】 |
| 性质5：任意一节点到每个叶子节点的路径都包含数量相同的黑节点，俗称**黑高**（红黑树中的红接点和黑色父节点绑定的，在2-3树中本来就是同一层，只有黑色节点才会在2-3树中真正贡献高度，由于2-3树的任一节点得到空链接举例相同，因此反映在红黑树中就是黑色完美平衡） |
| 从性质5可以推出：如果一个节点存在黑子节点，那么该节点肯定有两个子节点 |

![image-20210417100837326](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162811.png)

>黑色完美平衡

左子树和右子树的黑节点层数是相等的，也即任意一个节点到每个叶子节点的路径都包含数量相同的黑节点。

>红黑树如何保证自我平衡

- **变色**：节点的颜色由红变黑或由黑变红。
- **左旋**：以某个节点作为支点旋转节点，其右节点变为旋转节点的父节点，右子节点的左子节点变为旋转节点的右子节点，左子节点保持不变。
- **右旋**：以某个节点作为支点旋转节点，其左子节点变为旋转节点的父节点，左子节点的右子节点变为旋转节点的左子节点，右子节点保持不变。

### 左旋

```java
public void leftRotate(){
    Node newNode = new Node(val);
    newNode.left = left;
    newNode.right = right.left;
    val = right.val;
    right = right.right;
    left = newNode;
}
```

![image-20210417101806160](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162812.png)

### 右旋

```java
public void rightRotate(){
    Node newNode = new Node(val);
    newNode.right = right;
    newNode.left = left.right;
    val = left.val;
    left = left.left;
    right = newNode;
}
```

![image-20210417101722329](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162813.png)

## 红黑树的插入

插入操作的步骤：

1. 查找插入的位置
2. 插入后自平衡

>注意

插入节点，必须为红色，理由很简单：红色在父节点（如果存在）为黑节点时，红黑树的黑色平衡没被破坏，不需要做自平衡。但如果插入节点是黑色，那么插入位置所在的子树黑色节点总是多1，需要做自平衡

### 情景1

>红黑树为空树

直接把插入节点作为根节点

根据红黑树的性质2：根节点是黑色，还需要把插入的节点设置为红色

### 情景2

> 插入的节点的key已经存在

更新当前节点的值为插入节点的值

![image-20210417102430488](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162814.png)

### 情景3

>插入节点的父节点为黑节点

由于插入的节点是红色，当插入的节点的父节点为黑色是，不会影响红黑树的平衡，直接插入即可

![image-20210417102540369](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162815.png)

### 情景4

>插入节点的父节点为红色

由于性质2：根节点一定是黑色，如果插入节点的父节点为红色，那么该父节点不可能为根节点，所以插入节点总是存在祖父节点

![image-20210417103115945](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162816.png)

#### 情景4.1

>叔叔节点存在并且为红色

根据红黑树的性质4：红色节点不能相连

处理：

1. 将P和U节点改为黑色
2. 将PP改为红色
3. 将PP设置为当前节点，进行后续操作

![image-20210417103502087](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162817.png)

#### 情景4.2

>叔叔节点不存在或为黑色，并且插入节点的父亲节点是祖父节点的左子节点

![image-20210417104124534](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162818.png)

##### 情景4.2.1

>新插入节点，为其父亲节点的左子节点 （LL双红）

处理：

1. 将P变为黑色，将PP变为红色
2. 对PP进行右旋

![image-20210417104352802](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162819.png)

##### 情景4.2.2

>新插入节点，为其父亲节点的右子节点 （LR双红）

处理：

1. 对P进行左旋
2. 将P设置为当前节点，得到LL双红
3. 按照LL双红处理
   1. 将P变为黑色，将PP变为红色
   2. 对PP进行右旋 

![image-20210417104558335](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162820.png)

#### 情景4.3

>叔叔节点不存在或为黑色，并且插入节点的父亲节点是祖父节点的右子节点

![image-20210417104857160](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162821.png)

##### 情景4.3.1

>新插入节点，为其父亲节点的右子节点 （RR双红）

处理：

1. 将P变为黑色，将PP变为红色
2. 对PP进行左旋

![image-20210417105010676](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162822.png)

##### 情景4.3.2

>新插入节点，为其父亲节点的左子节点 （RL双红）

处理：	

1. 对P进行右旋
2. 将P设置为当前节点，得到RR双红
3. 按照RR双红处理
   1. 将P变为黑色，将PP变为红色
   2. 对PP进行左旋

![image-20210417105217054](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162823.png)

#### 插入案例

![红黑树插入案例](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162824.png)

## 红黑树的删除

### 节点命名约定

- D表示要被删除的节点。即：取 Delete 的首字母；
- P 表示父节点。即：取 Parent 的首字母；
- S表示兄弟姐妹节点。即：取 Sibling的首字母；
- U表示叔伯节点。即：取Uncle的首字母；
- G表示祖父节点。即：取 Grandfather的首字母；
- L表示左树。即：取Left的首字母；
- R表示右树。即：取Right的首字母；
- Nil表示叶子节点。即所谓的空节点；注意：红黑树中的叶子节点与其他树中所述说的叶子节点不是同一概念。而且红黑树中的叶子节点(即：Nil节点)永远是被定义为黑色的。
-  DR表示要被删除的节点的右子树，即：右子节点；
-  SL表示兄弟节点的左子树，即：左子节点；

### 情景1

> 被删除的**D节点为红色**	

根据红黑树的定义，被删除的节点D(即：上文所述的(Real)D节点)不论如何都一定有一个“右子树”，只是该右子树要不为非空树(即：真正存在的节点，不为Nil节点)，要不就必为空树(即：D的两个子节点都为Nil)。下面称D的该右子节点(或称为右子树)为DR。

处理：

1. P指向DR

   ![image-20210417214711967](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162825.png)

分析：因为D为红色，所以P必为黑色，同时DR不可能为红色(否则违反性质4)。同时由于性质5，则DR必为Nil，否则就D树来说，经过DR与不经过DR的路径的黑节点数必不相同。现在要删除D节点，只需要直接将D节点删除，并将DR作为P的左子节点即可。因此删除后，变成上图右侧所示。

### 情景2

> 被删除的节点D为黑色 且DR不为NIL，则DR必为红色

处理

1. P指向DR 
2. DR变为黑色



![image-20210417214926178](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162826.png)

分析：由于删除的D为黑色，删除后P的左子树的黑节点数必少1，此时刚好DR为黑色，并且删除后DR可以占据D的位置(这样仍是一棵二叉搜索树，只是暂时还不是合格的红黑树罢了)，然后再将DR的颜色改为黑色，刚好可以填补P左子树所减少的黑节点数。从而P树又平衡了。因此，平衡处理后，最终变成上图右侧的图示。

### 情景3

>被删除的D为黑色，且DR为Nil。

#### 情景3.1

>S为红结点

处理

1. P指向DR
2. P节点左旋
3. S变为黑色 P变为红色

![image-20210417215325446](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162827.png)



![image-20210417215355675](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162828.png)

## 手写红黑树

步骤：

1. 创建RBTree，定义颜色
2. 创建RBNode
3. 辅助方法定义：parentOf(node)、isRed()、setRed()、setBlack()、inOrderPrint()
4. 左旋右旋方法定义：leftRotate(node)、rightRotate(node)
5. 公开插入接口方法定义：insert(K key,  V value);
6. 内部插入接口方法定义：insert(RBNode node);
7. 修正插入导致红黑树失衡的方法定义：insertFlxUp(RBNode node);
8. 测试



```java
package school.xauat.datastructres.rbtree;

/**
 * @author ：zsy
 * @date ：Created 2021/4/17 11:36
 * @description：红黑树
 */

import java.util.Base64;

/**
 * 步骤：
 *
 * 1. 创建RBTree，定义颜色
 * 2. 创建RBNode
 * 3. 辅助方法定义：parentOf(node)、isRed()、setRed()、setBlack()、inOrderPrint()
 * 4. 左旋右旋方法定义：leftRotate(node)、rightRotate(node)
 * 5. 公开插入接口方法定义：insert(K key,  V value);
 * 6. 内部插入接口方法定义：insert(RBNode node);
 * 7. 修正插入导致红黑树失衡的方法定义：insertFlxUp(RBNode node);
 * 8. 测试
 */
public class RBTree<K extends Comparable<K>, V> {
    private static final boolean RED = true;
    private static final boolean BLACK = false;
    private RBNode root;

    public void insert(K key, V value) {
        RBNode newNode = new RBNode();
        newNode.setKey(key);
        newNode.setValue(value);
        newNode.setColor(RED);
        insert(newNode);
    }

    /**
     * 插入节点
     * @param node
     */
    private void insert(RBNode node) {
        if (root == null) {
            root = node;
            setBlack(root);
            return;
        }
        //寻找插入的父节点
        RBNode tmp = root;
        RBNode parent = null;
        while (tmp != null) {
            parent = tmp;
            int cmp = node.key.compareTo(tmp.key);
            if (cmp == 0) {
                //插入的节点已经存在情景2
                tmp.setValue(node.getValue());
                return;
            }
            if (cmp < 0) {
                tmp = tmp.left;
            } else {
                tmp = tmp.right;
            }
        }
        node.parent = parent;
        if (parent != null) {
            int cmp = node.key.compareTo(parent.key);
            if (cmp > 0) {
                parent.right = node;
            } else {
                parent.left = node;
            }
        }
        //调用修复红黑树平衡的方法
        insertFlxUp(node);
    }

    /**
     * 修复红黑树平衡性
     * @param node
     */
    private void insertFlxUp(RBNode node) {
        //根节点染黑
        setBlack(root);
        RBNode parent = parentOf(node);
        RBNode pp = parentOf(parent);
        RBNode uncle = null;
        if (parent != null && isRed(parent)) { //情景4
            //寻找叔叔节点
            if (pp.left != null && pp.left.key.equals(parent.key)) {
                uncle = pp.right;
            } else {
                uncle = pp.left;
            }

            if(uncle != null && isRed(uncle)) {//情景4.1：叔叔存在且为红色
                setBlack(parent);
                setBlack(uncle);
                setRed(pp);
                insertFlxUp(pp);
                return;
            } else { //情景4.2:叔叔节点不存在或者存在为黑节点
                if (pp.left != null && pp.left == parent) { //情景4.2:插入节点的父亲节点是祖父节点的左子节点
                    if (parent.left != null && parent.left == node) {//情景4.2.1LL双红:插入节点为父亲的左节点
                        setBlack(parent);
                        setRed(pp);
                        rightRotate(pp);
                    } else { //情景4.2.2LR双红
                        leftRotate(parent);
                        insertFlxUp(parent);
                        return;
                    }
                } else { //情景4.3:插入节点的父亲节点是祖父节点的右子节点
                    if(parent.right != null && parent.right == node) {//情景4.3.1RR双红:新插入节点，为其父亲节点的右子节点
                        setBlack(parent);
                        setRed(pp);
                        leftRotate(pp);
                        return;
                    } else { //情景4.3.2：RL双红
                        rightRotate(parent);
                        insertFlxUp(parent);
                        return;
                    }
                }
            }
        }
    }

    /**
     * 左旋
     *
     *      p                       p
     *      |                       |
     *      x                       y
     *     / \        ------>      / \
     *   lx   y                   x   ry
     *       / \                 / \
     *     ly  ry               lx  ly
     * @param x
     */
    private void leftRotate(RBNode x) {
        RBNode y = x.right;
        //将x的右子节点指向y的左子节点，将左子节点的父节点更新为x
        x.right = y.left;
        if (y.left != null) {
            y.left.parent = x;
        }
        //当x的父节点不为空时，更新y的父节点为x的父节点，并将x的父节点指向为y
        if (x.parent != null) {
            y.parent = x.parent;
            if (x == x.parent.left) {
                x.parent.left = y;
            } else {
                x.parent.right = y;
            }
        } else { //x为根节点，更新根节点
            this.root = y;
        }
        //将x的父节点更新为y，将y的左子节点更新为x
        x.parent = y;
        y.left = x;
    }

    /**
     * 右旋
     *
     *       p                       p
     *       |                       |
     *       x                       y
     *      / \        ------>      / \
     *     y  lx                   ly  x
     *    / \                         / \
     *  ly  ry                       ry  lx
     * @param x
     */
    private void rightRotate(RBNode x) {
        RBNode y = x.left;
        //将x的左子节点指向y的右子节点,并将右子节点的父节点更新为x
        x.left = y.right;
        if (y.right != null) {
            y.right.parent = x;
        }
        //当x的父节点不为空时，更新y的父节点为x的父节点，并将x的父节点指向为y
        if (x.parent != null) {
            y.parent = x.parent;
            if (x == x.parent.left) {
                x.parent.left = y;
            } else {
                x.parent.right = y;
            }
        } else { //x为根节点，更新根节点
            this.root = y;
        }
        //将x的父节点更新为y，将y的左子节点更新为x
        x.parent = y;
        y.right = x;
    }

    public void inOrderPrint() {
        inOrderPrint(root);
    }

    public void preOrderPrint() {
        preOrderPrint(root);
    }

    /**
     * 中序打印红黑树
     * @param node
     */
    private void inOrderPrint(RBNode node) {
        if (node == null)return;
        inOrderPrint(node.left);
        System.out.println(node);
        inOrderPrint(node.right);
    }

    /**
     * 中序打印红黑树
     * @param node
     */
    private void preOrderPrint(RBNode node) {
        if (node == null)return;
        System.out.println(node);
        preOrderPrint(node.left);
        preOrderPrint(node.right);
    }

    /**
     * 获取节点的父节点
     * @param node
     * @return
     */
    public RBNode parentOf(RBNode node) {
        if (node != null) {
            return node.parent;
        }
        return null;
    }

    /**
     * 判断节点是否是红色
     * @param node
     * @return
     */
    private boolean isRed(RBNode node) {
        if (node != null) {
            return node.color == RED;
        }
        return false;
    }

    /**
     * 设置当前节点为红色
     * @param node
     */
    private void setRed(RBNode node) {
        if (node != null) {
            node.color = RED;
        }
    }

    /**
     * 设置当前节点为黑色
     * @param node
     */
    private void setBlack(RBNode node) {
        if (node != null) {
            node.color = BLACK;
        }
    }

    /**
     * 判断节点是否是黑色
     * @param node
     * @return
     */
    private boolean isBlack(RBNode node) {
        if (node != null) {
            return node.color == BLACK;
        }
        return false;
    }

    static class RBNode<K extends Comparable<K>, V> {
        private RBNode parent;
        private RBNode left;
        private RBNode right;
        private boolean color;
        private K key;
        private V value;

         public RBNode() {}


         public RBNode(RBNode parent, RBNode left, RBNode right, boolean color, K key, V value) {
             this.parent = parent;
             this.left = left;
             this.right = right;
             this.color = color;
             this.key = key;
             this.value = value;
         }

         public RBNode getParent() {
             return parent;
         }

         public void setParent(RBNode parent) {
             this.parent = parent;
         }

         public RBNode getLeft() {
             return left;
         }

         public void setLeft(RBNode left) {
             this.left = left;
         }

         public RBNode getRight() {
             return right;
         }

         public void setRight(RBNode right) {
             this.right = right;
         }

         public boolean isColor() {
             return color;
         }

         public void setColor(boolean color) {
             this.color = color;
         }

         public K getKey() {
             return key;
         }

         public void setKey(K key) {
             this.key = key;
         }

         public V getValue() {
             return value;
         }

         public void setValue(V value) {
             this.value = value;
         }

        @Override
        public String toString() {
            return "RBNode{" +
                    "color=" + color +
                    ", key=" + key +
                    ", value=" + value +
                    '}';
        }
    }
}
```

## 思考

如果我构造一颗只有黑色节点的红黑树，这样子可行吗？因为这样子没有破坏任何一条红黑树的规则。如果可行，红节点的真实意义是什么？

# [HashMap面试题](https://thinkwon.blog.csdn.net/article/details/104863992)

### HashMap线程安全方面会出现什么问题

- 在jdk1.7中，在多线程环境下，扩容时会造成环形链或数据丢失
- 在jdk1.8中，在多线程环境下，会发生数据覆盖的情况
  - jdk1.8中HashMap中put操作的主函数，如果没有hash碰撞则会直接插入元素。如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给**覆盖**，发生线程不安全。

### 为什么HashMap的底层数组长度为何总是2的n次方

我们计算桶的位置完全可以使用h % length，如果这个length是随便设定值的话当然也可以，但是如果你对它进行研究，设计一个合理的值得话，那么将对HashMap的性能发生翻天覆地的变化。

- 第一：当length为2的N次方的时候，h & (length-1) = h % length
  为什么&效率更高呢？因为位运算直接对内存数据进行操作，不需要转成十进制，所以位运算要比取模运算的效率更高
- 第二：当length为2的N次方的时候，数据分布均匀，减少冲突