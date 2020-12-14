# Java 容器

## 1. 概览

- 容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对的映射表。

### 1.1 Collection

<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144352.png" alt="image-20200920174603425" style="zoom:33%;" />

#### 1.1.1 Set

- **TreeSet：** 基于红黑树实现，支持有序性操作，遍历集合元素时，是有顺序的（从小到大）。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
- **HashSet：** 基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
- **LinkedHashSet：**具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。

#### 1.1.2 List

- **ArrayList：**基于动态数组实现，支持随机访问。
- **Vector：**和 ArrayList 类似，但它是线程安全的。
- **LinkedList：**基于双向链表实现，只能顺序访问，但是可以快速的在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

#### 1.1.3 Queue

- **LinkedList：** 可以用它来实现双向队列。
- **PriorityQueue：** 基于堆结构实现，可以用来实现优先队列。

### 1.2 Map

<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144409.png" alt="image-20200920175549409" style="zoom:33%;" />

- **TreeMap：** 基于红黑树实现。
- **HashMap：** 基于哈希表实现。
- **HashTable：** 和 HashMap 类似，但它是线程安全的。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。
- **LinkedHashMap：** 使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用顺序。

## 2. 容器中的设计模式

### 2.1 迭代器模式

<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144427.png" alt="image-20200920180203215" style="zoom:33%;" />

- Collection 继承了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。

### 2.2 适配器模式

- java.util.Arrays#asList() 可以把数组类型转换为 List 类型。

	```java
	@SafeVarargs 
	public static <T> List<T> asList(T... a)
	```

- 应该注意的是 asList() 的参数为泛型的变长参数，不能使用基本类型数组作为参数，只能使用相应的包装类型数组。

	```java
	Integer[] arr = {1, 2, 3}; 
	List list = Arrays.asList(arr);
	```

- 也可以使用以下方式调用 asList()： 

	```java
	List list = Arrays.asList(1, 2, 3);
	```

## 3. 源码分析

### 3.1 ArrayList

#### 3.1.1 概览

- 因为 ArrayList 是基于数组实现的，所以支持快速随机访问。RandomAccess 接口标识着该类支持快速随机访问。

	```java
	public class ArrayList<E> extends AbstractList<E> 
	  implements List<E>, RandomAccess, Cloneable, java.io.Serializable
	```

- 数组的默认大小为 10。 

	```java
	private static final int DEFAULT_CAPACITY = 10;
	```

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144450.png" alt="image-20200920180933581" style="zoom:33%;" />

#### 3.1.2 扩容

- 添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 oldCapacity + (oldCapacity >> 1) ，也就是旧容量的 1.5 倍。

- 扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

	```java
	public boolean add(E e) { 
	    ensureCapacityInternal(size + 1); // Increments modCount!! 
	    elementData[size++] = e; 
	    return true; 
	}
	
	private void ensureCapacityInternal(int minCapacity) { 
	    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) { 
	        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity); 
	    }
	    ensureExplicitCapacity(minCapacity); 
	}
	
	private void ensureExplicitCapacity(int minCapacity) { 
	    modCount++; // overflow-conscious code 
	    if (minCapacity - elementData.length > 0) 
	      	grow(minCapacity); 
	}
	
	private void grow(int minCapacity) {
	    // overflow-conscious code 
	    int oldCapacity = elementData.length; 
	    int newCapacity = oldCapacity + (oldCapacity >> 1); 
	    if (newCapacity - minCapacity < 0) 
	      	newCapacity = minCapacity; 
	    if (newCapacity - MAX_ARRAY_SIZE > 0) 
	      	newCapacity = hugeCapacity(minCapacity); 
	    // minCapacity is usually close to size, so this is a win: 
	    elementData = Arrays.copyOf(elementData, newCapacity); 
	}
	```

#### 3.1.3 删除元素

- 需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看出 ArrayList 删除元素的代价是非常高的。

	```java
	public E remove(int index) { 
	    rangeCheck(index); 
	    modCount++; 
	    E oldValue = elementData(index); 
	    int numMoved = size - index - 1; 
	    if (numMoved > 0) 
	        System.arraycopy(elementData, index+1, elementData, index, numMoved); 
	    elementData[--size] = null; // clear to let GC do its work 
	    return oldValue; 
	}
	```

#### 3.1.4 Fail-Fast

- modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

- 在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出ConcurrentModifificationException 。

	```java
	private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{ 
	    // Write out element count, and any hidden stuff 
	    int expectedModCount = modCount; 
	    s.defaultWriteObject(); 
	
	    // Write out size as capacity for behavioural compatibility with clone() 
	    s.writeInt(size); 
	
	    // Write out all elements in the proper order. 
	    for (int i=0; i<size; i++) { 
	      	s.writeObject(elementData[i]); 
	    }
	
	    if (modCount != expectedModCount) { 
	      	throw new ConcurrentModificationException(); 
	    } 
	}
	```

#### 3.1.5 序列化

- ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

- 保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。

	```java
	transient Object[] elementData; // non-private to simplify nested class access
	```

- ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。

	```java
	private void readObject(java.io.ObjectInputStream s) throws java.io.IOException, ClassNotFoundException {
	    elementData = EMPTY_ELEMENTDATA; 
	
	    // Read in size, and any hidden stuff 
	    s.defaultReadObject(); 
	
	    // Read in capacity 
	    s.readInt(); // ignored 
	
	    if (size > 0) { 
	        // be like clone(), allocate array based upon size not capacity 
	        ensureCapacityInternal(size); 
	
	        Object[] a = elementData; 
	        // Read in all elements in the proper order. 
	        for (int i=0; i<size; i++) { 
	          	a[i] = s.readObject(); 
	        } 
	    } 
	}
	```

	```java
	private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{ 
	    // Write out element count, and any hidden stuff 
	    int expectedModCount = modCount; 
	    s.defaultWriteObject(); 
	
	    // Write out size as capacity for behavioural compatibility with clone() 
	    s.writeInt(size); 
	
	    // Write out all elements in the proper order. 
	    for (int i=0; i<size; i++) { 
	      	s.writeObject(elementData[i]); 
	    }
	
	    if (modCount != expectedModCount) { 
	      	throw new ConcurrentModificationException(); 
	    } 
	}
	```

- 序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去反射调用该对象的 writeObject() 来实现序列化。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

	```java
	ArrayList list = new ArrayList(); 
	ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file)); 
	oos.writeObject(list);
	```

### 3.2 Vector

#### 3.2.1 同步

- 它的实现与 ArrayList 类似，但是使用了 synchronized 进行同步。

	```java
	public synchronized boolean add(E e) { 
	    modCount++; 
	    ensureCapacityHelper(elementCount + 1); 
	    elementData[elementCount++] = e; 
	    return true; 
	}
	
	public synchronized E get(int index) {
	    if (index >= elementCount) throw new ArrayIndexOutOfBoundsException(index);
	    return elementData(index);
	}
	```

#### 3.2.2 与 ArrayList 的比较

- Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制；
- Vector 每次扩容请求其大小的 2 倍空间，而 ArrayList 是 1.5 倍。

#### 3.2.3 替代方案

- 可以使用 Collections.synchronizedList(); 得到一个线程安全的 ArrayList。 

	```java
	List<String> list = new ArrayList<>(); 
	List<String> synList = Collections.synchronizedList(list);
	```

- 也可以使用 concurrent 并发包下的 CopyOnWriteArrayList 类。

	```java
	List<String> list = new CopyOnWriteArrayList<>();
	```

### 3.3 CopyOnWriteArrayList

#### 3.3.1 读写分离

- 写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。
- 写操作需要加锁，防止并发写入时导致写入数据丢失。

- 写操作结束之后需要把原始数组指向新的复制数组。

	```java
	public boolean add(E e) {
	    final ReentrantLock lock = this.lock;
	    lock.lock();
	    try {
	        Object[] elements = getArray();
	        int len = elements.length;
	        Object[] newElements = Arrays.copyOf(elements, len + 1);
	        newElements[len] = e;
	        setArray(newElements);
	        return true;
	    } finally {
	        lock.unlock();
	    }
	}
	
	final void setArray(Object[] a) {
	    array = a;
	}
	```

	```java
	@SuppressWarnings("unchecked") 
	private E get(Object[] a, int index) { 
	  return (E) a[index]; 
	}
	```

#### 3.3.2 使用场景

- CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。
- 但是 CopyOnWriteArrayList 有其缺陷：
	- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
	- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。
- 所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。

### 3.4 LinkedList

#### 3.4.1 概览

- 基于双向链表实现，使用 Node 存储链表节点信息。

	```java
	private static class Node<E> { 
	  	E item; 
	  	Node<E> next; 
	  	Node<E> prev; 
	}
	```

- 每个链表存储了 fifirst 和 last 指针：

	```java
	transient Node<E> first; 
	transient Node<E> last;
	```

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144504.png" alt="image-20200921101121833" style="zoom:33%;" />

#### 3.4.2 与 ArrayList 的比较

- ArrayList 基于动态数组实现，LinkedList 基于双向链表实现。
- ArrayList 支持随机访问，LinkedList 不支持。
- LinkedList 在任意位置添加删除元素更快。

### 3.5 HashMap

- 为了便于理解，以下源码分析以 JDK 1.7 为主。

#### 3.5.1 存储结构

- 内部包含了一个 Entry 类型的数组 table。

	```java
	transient Entry[] table;
	```

- Entry 存储着键值对。它包含了四个字段，从 next 字段我们可以看出 Entry 是一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap 使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的 Entry。

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144519.png" alt="image-20200921103006586" style="zoom:33%;" />

	```java
	static class Entry<K, V> implements Map.Entry<K, V> {
	    final K key;
	    V value;
	    Entry<K, V> next;
	    int hash;
	
	    Entry(int h, K k, V v, Entry<K, V> n) {
	        value = v;
	        next = n;
	        key = k;
	        hash = h;
	    }
	
	    public final K getKey() {
	        return key;
	    }
	
	    public final V getValue() {
	        return value;
	    }
	
	    public final V setValue(V newValue) {
	        V oldValue = value;
	        value = newValue;
	        return oldValue;
	    }
	
	    public final boolean equals(Object o) {
	        if (!(o instanceof Map.Entry)) return false;
	        Map.Entry e = (Map.Entry) o;
	        Object k1 = getKey();
	        Object k2 = e.getKey();
	        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
	            Object v1 = getValue();
	            Object v2 = e.getValue();
	            if (v1 == v2 || (v1 != null && v1.equals(v2))) return true;
	        }
	        return false;
	    }
	
	    public final int hashCode() {
	        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
	    }
	
	    public final String toString() {
	        return getKey() + "=" + getValue();
	    }
	}
	```

#### 3.5.2 拉链法的工作原理

```java
HashMap<String, String> map = new HashMap<>(); 
map.put("K1", "V1"); 
map.put("K2", "V2"); 
map.put("K3", "V3");
```

- 新建一个 HashMap，默认大小为 16。

- 插入 <K1,V1> 键值对，先计算 K1 的 hashCode 为 115，使用除留余数法得到所在的桶下标 115%16=3。

- 插入 <K2,V2> 键值对，先计算 K2 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6。

- 插入 <K3,V3> 键值对，先计算 K3 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6，插在 <K2,V2> 前面。

- 应该注意到链表的插入是以头插法方式进行的，例如上面的 <K3,V3> 不是插在 <K2,V2> 后面，而是插入在链表头部。

- 查找需要分成两步进行：

	- 计算键值对所在的桶；

	- 在链表上顺序查找，时间复杂度显然和链表的长度成正比。

		<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144531.png" alt="image-20200921104657132" style="zoom:33%;" />

#### 3.5.3 put 操作

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    // 键为 null 单独处理 
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    // 确定桶下标 
    int i = indexFor(hash, table.length);
    // 先找出是否已经存在键为 key 的键值对，如果存在的话就更新这个键值对的值为 value 
    for (Entry<K, V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    
    modCount++;
    // 插入新键值对 
    addEntry(hash, key, value, i);
    return null;
}
```

- HashMap 允许插入键为 null 的键值对。但是因为无法调用 null 的 hashCode() 方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。HashMap 使用第 0 个桶存放键为 null 的键值对。

	```java
	private V putForNullKey(V value) {
	    for (Entry<K, V> e = table[0]; e != null; e = e.next) {
	        if (e.key == null) {
	            V oldValue = e.value;
	            e.value = value;
	            e.recordAccess(this);
	            return oldValue;
	        }
	    }
	    modCount++;
	    addEntry(0, null, value, 0);
	    return null;
	}
	```

- 使用链表的头插法，也就是新的键值对插在链表的头部，而不是链表的尾部。

	```java
	void addEntry(int hash, K key, V value, int bucketIndex) {
	    if ((size >= threshold) && (null != table[bucketIndex])) {
	        resize(2 * table.length);
	        hash = (null != key) ? hash(key) : 0;
	        bucketIndex = indexFor(hash, table.length);
	    }
	    createEntry(hash, key, value, bucketIndex);
	}
	
	void createEntry(int hash, K key, V value, int bucketIndex) {
	    Entry<K, V> e = table[bucketIndex];
	    // 头插法，链表头部指向新的键值对 
	    table[bucketIndex] = new Entry<>(hash, key, value, e);
	    size++;
	}
	
	Entry(int h, K k, V v, Entry<K,V> n) { 
	    value = v; 
	    next = n; 
	    key = k; 
	    hash = h; 
	}
	```

#### 3.5.4 确定桶下标

- 很多操作都需要先确定一个键值对所在的桶下标。

	```java
	int hash = hash(key); 
	int i = indexFor(hash, table.length);
	```

- 计算 hash 值

	```java
	final int hash(Object k) {
	    int h = hashSeed;
	    if (0 != h && k instanceof String) {
	        return sun.misc.Hashing.stringHash32((String) k);
	    }
	    
	    h ^= k.hashCode();
	    
	    // This function ensures that hashCodes that differ only by 
	    // constant multiples at each bit position have a bounded 
	    // number of collisions (approximately 8 at default load factor). 
	    h ^= (h >>> 20) ^ (h >>> 12);
	    return h ^ (h >>> 7) ^ (h >>> 4);
	}
	```

	```java
	public final int hashCode() { 
	  	return Objects.hashCode(key) ^ Objects.hashCode(value); 
	}
	```

- 取模

	- 令 x = 1<<4，即 x 为 2 的 4 次方，它具有以下性质：

		```
		x   : 00010000 
		x-1 : 00001111
		```

	- 令一个数 y 与 x-1 做与运算，可以去除 y 位级表示的第 4 位以上数：

		```java
		y       : 10110010 
		x-1     : 00001111 
		y&(x-1) : 00000010
		```

	- 这个性质和 y 对 x 取模效果是一样的：

		```java
		y   : 10110010 
		x   : 00010000 
		y%x : 00000010
		```

	- 我们知道，位运算的代价比求模运算小的多，因此在进行这种计算时用位运算的话能带来更高的性能。

	- 确定桶下标的最后一步是将 key 的 hash 值对桶个数取模：hash%capacity，如果能保证 capacity 为 2 的 n 次方，那么就可以将这个操作转换为位运算。

		```java
		static int indexFor(int h, int length) { 
		  	return h & (length-1); 
		}
		```

#### 3.5.5 扩容-基本原理

- 设 HashMap 的 table 长度为 M，需要存储的键值对数量为 N，如果哈希函数满足均匀性的要求，那么每条链表的长度大约为 N/M，因此平均查找次数的复杂度为 O(N/M)。

- 为了让查找的成本降低，应该尽可能使得 N/M 尽可能小，因此需要保证 M 尽可能大，也就是说 table 要尽可能大。HashMap 采用动态扩容来根据当前的 N 值来调整 M 值，使得空间效率和时间效率都能得到保证。

- 和扩容相关的参数主要有：capacity、size、threshold 和 load_factor。

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144542.png" alt="image-20200921112704480" style="zoom:33%;" />

	```java
	static final int DEFAULT_INITIAL_CAPACITY = 16; 
	static final int MAXIMUM_CAPACITY = 1 << 30; 
	static final float DEFAULT_LOAD_FACTOR = 0.75f; 
	transient Entry[] table; 
	transient int size; int threshold; 
	final float loadFactor; 
	transient int modCount;
	```

- 从下面的添加元素代码中可以看出，当需要扩容时，令 capacity 为原来的两倍。

	```java
	void addEntry(int hash, K key, V value, int bucketIndex) { 
	    Entry<K,V> e = table[bucketIndex]; 
	    table[bucketIndex] = new Entry<>(hash, key, value, e); 
	    if (size++ >= threshold) 
	      	resize(2 * table.length); 
	}
	```

- 扩容使用 resize() 实现，需要注意的是，扩容操作同样需要把 oldTable 的所有键值对重新插入 newTable 中，因此这一步是很费时的。

	```java
	void resize(int newCapacity) {
	    Entry[] oldTable = table;
	    int oldCapacity = oldTable.length;
	    if (oldCapacity == MAXIMUM_CAPACITY) {
	        threshold = Integer.MAX_VALUE;
	        return;
	    }
	    Entry[] newTable = new Entry[newCapacity];
	    transfer(newTable);
	    table = newTable;
	    threshold = (int) (newCapacity * loadFactor);
	}
	
	void transfer(Entry[] newTable) {
	    Entry[] src = table;
	    int newCapacity = newTable.length;
	    for (int j = 0; j < src.length; j++) {
	        Entry<K, V> e = src[j];
	        if (e != null) {
	            src[j] = null;
	            do {
	                Entry<K, V> next = e.next;
	                int i = indexFor(e.hash, newCapacity);
	                e.next = newTable[i];
	                newTable[i] = e;
	                e = next;
	            } while (e != null);
	        }
	    }
	}
	```

#### 3.5.6 扩容-重新计算桶下标

- 在进行扩容时，需要把键值对重新放到对应的桶上。HashMap 使用了一个特殊的机制，可以降低重新计算桶下标的操作。

- 假设原数组长度 capacity 为 16，扩容之后 new capacity 为 32： 

	```java
	capacity     : 00010000 
	new capacity : 00100000
	```

- 对于一个 Key，
	- 它的哈希值如果在第 5 位上为 0，那么取模得到的结果和之前一样；
	- 如果为 1，那么得到的结果为原来的结果 +16。

#### 3.5.7 计算数组容量

- HashMap 构造函数允许用户传入的容量不是 2 的 n 次方，因为它可以自动地将传入的容量转换为 2 的 n 次方。

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

#### 3.5.8 链表转红黑树

- 从 JDK 1.8 开始，一个桶存储的链表长度大于 8 时会将链表转换为红黑树。

#### 3.5.9 与 HashTable 的比较

- HashTable 使用 synchronized 来进行同步。
- HashMap 可以插入键为 null 的 Entry。
- HashMap 的迭代器是 fail-fast 迭代器。
- HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

### 3.6 ConcurrentHashMap

#### 3.6.1 存储结构

```java
static final class HashEntry<K,V> { 
    final int hash; 
    final K key; 
    volatile V value; 
    volatile HashEntry<K,V> next; 
}
```

- ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。

- Segment 继承自 ReentrantLock。 

	```java
	static final class Segment<K, V> extends ReentrantLock implements Serializable {
	    private static final long serialVersionUID = 2249069246763182397L;
	    static final int MAX_SCAN_RETRIES = Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;
	    transient volatile HashEntry<K, V>[] table;
	    transient int count;
	    transient int modCount;
	    transient int threshold;
	    final float loadFactor;
	}
	```

	```java
	final Segment<K,V>[] segments;
	```

- 默认的并发级别为 16，也就是说默认创建 16 个 Segment。 

	```java
	static final int DEFAULT_CONCURRENCY_LEVEL = 16;
	```

#### 3.6.2 size 操作

- 每个 Segment 维护了一个 count 变量来统计该 Segment 中的键值对个数。

	```java
	/**
	 * The number of elements. Accessed only either within locks 
	 * or among other volatile reads that maintain visibility. 
	 */
	transient int count;
	```

- 在执行 size 操作时，需要遍历所有 Segment 然后把 count 累计起来。

- ConcurrentHashMap 在执行 size 操作时先尝试不加锁，如果连续两次不加锁操作得到的结果一致，那么可以认为这个结果是正确的。

- 尝试次数使用 RETRIES_BEFORE_LOCK 定义，该值为 2，retries 初始值为 -1，因此尝试次数为 3。

- 如果尝试的次数超过 3 次，就需要对每个 Segment 加锁。

	```java
	static final int RETRIES_BEFORE_LOCK = 2;
	
	public int size() {
	    // Try a few times to get accurate count. On failure due to 
	    // continuous async changes in table, resort to locking. 
	    final Segment<K, V>[] segments = this.segments;
	    int size;
	    boolean overflow; // true if size overflows 32 bits 
	    long sum; // sum of modCounts 
	    long last = 0L; // previous sum 
	    int retries = -1; // first iteration isn't retry 
	    try {
	        for (; ; ) {
	            // 超过尝试次数，则对每个 Segment 加锁 
	            if (retries++ == RETRIES_BEFORE_LOCK) {
	                for (int j = 0; j < segments.length; ++j)
	                    ensureSegment(j).lock(); // force creation 
	            }
	            sum = 0L;
	            size = 0;
	            overflow = false;
	            for (int j = 0; j < segments.length; ++j) {
	                Segment<K, V> seg = segmentAt(segments, j);
	                if (seg != null) {
	                    sum += seg.modCount;
	                    int c = seg.count;
	                    if (c < 0 || (size += c) < 0) overflow = true;
	                }
	            }
	            // 连续两次得到的结果一致，则认为这个结果是正确的 
	            if (sum == last) break;
	            last = sum;
	        }
	    } finally {
	        if (retries > RETRIES_BEFORE_LOCK) {
	            for (int j = 0; j < segments.length; ++j) segmentAt(segments, j).unlock();
	        }
	    }
	    return overflow ? Integer.MAX_VALUE : size;
	}
	```

#### 3.6.3 JDK 1.8 的改动

- JDK 1.7 使用分段锁机制来实现并发更新操作，核心类为 Segment，它继承自重入锁 ReentrantLock，并发度与 Segment 数量相等。
- JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。
- 并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。

### 3.7 LinkedHashMap

#### 3.7.1 存储结构

- 继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。

	```java
	public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
	```

- 内部维护了一个双向链表，用来维护插入顺序或者 LRU 顺序。

	```java
	/**
		* The head (eldest) of the doubly linked list. 
		*/ 
	transient LinkedHashMap.Entry<K,V> head; 
	
	/**
		* The tail (youngest) of the doubly linked list. 
		*/ 
	transient LinkedHashMap.Entry<K,V> tail;
	```

- accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。

	```java
	final boolean accessOrder;
	```

- LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

	```java
	void afterNodeAccess(Node<K,V> p) { } 
	void afterNodeInsertion(boolean evict) { }
	```

#### 3.7.2 afterNodeAccess()

- 当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。

	```java
	void afterNodeAccess(Node<K, V> e) {  // move node to last 
	    LinkedHashMap.Entry<K, V> last;
	    if (accessOrder && (last = tail) != e) {
	        LinkedHashMap.Entry<K, V> p = (LinkedHashMap.Entry<K, V>) e, b = p.before, a = p.after;
	        p.after = null;
	        if (b == null) head = a;
	        elseb.after = a;
	        if (a != null) a.before = b;
	        elselast = b;
	        if (last == null) head = p;
	        else {
	            p.before = last;
	            last.after = p;
	        }
	        tail = p;
	        ++modCount;
	    }
	}
	```

#### 3.7.3 afterNodeInsertion()

- 在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 fifirst。 

- evict 只有在构建 Map 的时候才为 false，在这里为 true。 

	```java
	void afterNodeInsertion(boolean evict) { // possibly remove eldest 
	    LinkedHashMap.Entry<K, V> first;
	    if (evict && (first = head) != null && removeEldestEntry(first)) {
	        K key = first.key;
	        removeNode(hash(key), key, null, false, true);
	    }
	}
	```

- removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。

	```java
	protected boolean removeEldestEntry(Map.Entry<K,V> eldest) { 
	  	return false; 
	}
	```

#### 3.7.4 LRU 缓存

- 以下是使用 LinkedHashMap 实现的一个 LRU 缓存：

	- 设定最大缓存空间 MAX_ENTRIES 为 3；

	- 使用 LinkedHashMap 的构造函数将 accessOrder 设置为 true，开启 LRU 顺序；

	- 覆盖 removeEldestEntry() 方法实现，在节点多于 MAX_ENTRIES 就会将最近最久未使用的数据移除。

		```java
		class LRUCache<K, V> extends LinkedHashMap<K, V> {
		    private static final int MAX_ENTRIES = 3;
		
		    protected boolean removeEldestEntry(Map.Entry eldest) {
		        return size() > MAX_ENTRIES;
		    }
		
		    LRUCache() {
		        super(MAX_ENTRIES, 0.75f, true);
		    }
		}
		```

		```java
		public static void main(String[] args) {
		    LRUCache<Integer, String> cache = new LRUCache<>();
		    cache.put(1, "a");
		    cache.put(2, "b");
		    cache.put(3, "c");
		    cache.get(1);
		    cache.put(4, "d");
		    System.out.println(cache.keySet());
		}
		```

		```java
		[3, 1, 4]
		```

### 3.8 WeakHashMap

#### 3.8.1 存储结构

- WeakHashMap 的 Entry 继承自 WeakReference，被 WeakReference 关联的对象在下一次垃圾回收时会被回收。

- WeakHashMap 主要用来实现缓存，通过使用 WeakHashMap 来引用缓存对象，由 JVM 对这部分缓存进行回收。

	```java
	private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
	```

#### 3.8.2 ConcurrentCache

- Tomcat 中的 ConcurrentCache 使用了 WeakHashMap 来实现缓存功能。

- ConcurrentCache 采取的是分代缓存：

	- 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收（伊甸园）；
	- 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收。
	- 当调用 get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收。
	- 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。

	```java
	public final class ConcurrentCache<K, V> {
	    private final int size;
	    private final Map<K, V> eden;
	    private final Map<K, V> longterm;
	
	    public ConcurrentCache(int size) {
	        this.size = size;
	        this.eden = new ConcurrentHashMap<>(size);
	        this.longterm = new WeakHashMap<>(size);
	    }
	
	    public V get(K k) {
	        V v = this.eden.get(k);
	        if (v == null) {
	            v = this.longterm.get(k);
	            if (v != null) this.eden.put(k, v);
	        }
	        return v;
	    }
	
	    public void put(K k, V v) {
	        if (this.eden.size() >= size) {
	            this.longterm.putAll(this.eden);
	            this.eden.clear();
	        }
	        this.eden.put(k, v);
	    }
	}
	```

## 4. 一些问题

### 4.1 说说 List、Set、Map 三者的区别？

- **List 对付顺序的好帮手：** List 接口存储一组不唯一的、有序的对象。
- **Set 注重独一无二的性质：** 不允许重复的集合。不会有多个元素引用相同的对象。
- **Map 用 Key 来搜索的专家：** 使用键值对存储。Map 会维护与 Key 有关联的值。两个 key 可以引用相同的对象，但 key 不能重复，典型的 key 是 String 类型，但也可以是任何对象。

### 4.2 ArrayList 和 LinkedList 区别？

- 两者都不能保证线程安全。
- ArrayList 底层是 Object 数组，LinkedList 底层是双向链表。
- ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置影响。LinkedList 采用链表存储，所以插入、删除元素的时间复杂度不受元素位置的影响，如果是要在指定的位置插入和删除元素的话，时间复杂度为 O(n)，因为需要先移动到指定位置插入。
- LinkedList 不支持高效的随机元素访问，而 ArrayList 支持。
- ArrayList 的空间浪费主要体现在在 List 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间。

### 4.3 ArrayList 与 Vector 区别？为什么要用 ArrayList 取代 Vector 呢？

- Vector 类的所有方法都是同步的。可以由多个线程安全的访问一个 Vector 对象，但是一个线程访问 Vector 的话代码要在同步操作上耗费大量的时间。
- ArrayList 不是同步的，所以在不需要保证线程安全时建议使用 ArrayList。

### 4.4 HashMap 和 HashTable 的区别？

- **线程是否安全：** HashMap 是非线程安全的，HashTable 是线程安全的。HashTable 内部的方法基本都经过 synchronized 修饰。(如果要保证线程安全的话可以使用 ConcurrentHashMap )。
- **效率：** 因为线程安全的问题，HashMap 要比 HashTable 效率高。
- **对 Null key 和 Null value 的支持：** HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。但是在 HashTable 中 put 进的键值只要有一个 null，直接抛出 NullPointerException。
- **初始容量大小和每次扩充容量大小的不同：** HashMap 默认的初始化大小为 16，之后每次扩充，容量变为原来的 2 倍，创建时如果给定了容量初始值，那么 HashMap 会将其扩展为 2 的幂次方大小。HashTable 默认的初始化大小为 11，之后每次扩充，容量变为原来的 2n+1，创建时给定了容量初始值，那么 HashTable 会直接使用给定的大小。
- **底层数据结构：** JDK 1.8 之后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树，以减少搜索时间。

### 4.5 HashMap 和 HashSet 区别？

- HashSet 底层就是基于 HashMap 实现的。

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144625.png" alt="image-20200921151306715" style="zoom:33%;" />

### 4.6 HashSet 如何检查重复？

- HashSet 会先计算对象的 hashCode 值来判断对象加入的位置，同时也会与其他加入的对象的 hashCode 值比较，如果没有相符的 hashCode，HashSet 会假设对象没有重复出现过。但是如果发现有相同的 hashCode 值对象，这时会调用 equals 方法来检查 hashCode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让加入操作成功。

### 4.7 ConcurrentHashMap 和 HashTable 的区别？

- ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构：** ConcurrentHashMap 底层采用数据+链表/红黑树。HashTable 采用数组+链表。

- **实现线程安全的方式：** JDK 1.7 时，ConcurrentHashMap 采用分段锁，一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争。JDK 1.8 时，并发控制采用 synchronized 和 CAS 来操作。HashTable（同一把锁），使用 synchronized 来保证线程安全，效率十分低下。

- 对比图：

	- HashTable

		<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144643.png" alt="image-20200921153600381" style="zoom:33%;" />

	- JDK 1.7 的 ConcurrentHashMap：

		<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144657.png" alt="image-20200921153646506" style="zoom:33%;" />

	- JDK 1.8 的 ConcurrentHashMap：

		<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214144710.png" alt="image-20200921153732288" style="zoom:33%;" />

### 4.8 ConcurrentHashMap 线程安全的具体实现方式/底层实现？

#### 4.8.1 JDK 1.7

- 首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

- ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成。 

- Segment 实现了 Reentrantlock，所以 Segment 是一种可重入锁，扮演锁的角色。HashEntry 用于存储键值对数据。

	```java
	static class Segment<K,V> extends ReentrantLock implements Serializable {}
	```

- 一个 ConcurrentHashMap 里包含一个 Segmen 数组。 Segment 的结构和 HashMap 类似，是一种数组和链表结构，一个 Segment 包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素，每个 Segment 守护着一个 HashEntry 数组里的元素,当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment 的锁。

#### 4.8.2 JDK 1.8

- ConcurrentHashMap 取消了 Segment 分段锁，采用 CAS 和 synchronized 来保证并发安全。synchronized 只锁定当前链表或红黑树的首节点，这样只要 hash 不冲突，就不会产生并发，效率提升。

### 4.9 compareble 和 comparator 的区别？

- comparable 接口出自 java. lang 包，它有一个 compareTo(Object obj) 方法用来排序。 
- comparator 接口出自 java.util 包，它有一个 compare(Object obj1, Object obj2) 方法用来排序。

