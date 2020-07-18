# （未完）Java集合框架梳理（基于JDK1.8）

Java集合类主要由两个接口`Collection`和`Map`派生出来的，`Collection`派生出了三个子接口：`List`、`Set`、`Queue`（Java5新增的队列），因此Java集合大致也可分成`List`、`Set`、`Queue`、`Map`四种接口体系

这篇内容总结的容器都是非同步的，即线程不安全的。常用的线程安全的类：
- CopyOnWriteArrayList
- CopyOnWriteArraySet
- ConcurrentHashMap
- ConcurrentLinkedQueue
- 等等


Java集合框架大致示意图（包含常用的集合类和大致的接口继承关系）：

根接口Collection：
![集合框架-Collection](http://img.longzhuang.top/集合框架-Collection.png)

根接口Map：
![](http://img.longzhuang.top/15937083930052.jpg)


四种体系分别代表了如下内容：
- `List` : 有序可重复集合
- `Queue` : 队列集合
- `Set` : 无序不可重复集合
- `Map` : 键值对集合

其中, 所有Collection接口的实现类均可以使用的方法有
`contains(Object o)` : 判断集合中是否存在指定元素
`containsAll(Collection<?> c)` : 判断集合中是否包含指定Collection的所有元素
`isEmpty()` : 判断集合是否为空
`iterator()` : 获取Collection对象的迭代器
`size()` : 获取集合中的元素个数
`toArray()` : 返回包含所有集合元素的Object数组
`toArray(T[] a)` : 返回指定类型的数组，如果a足够大，则使用该数组存储元素，并把多余的空间设置为null；否则返回一个新的同类型数组。例如可以使用下面的语句获得一个String类型数组:
```java
String[] strs = collection.toArray(new String[0]);
```

## List接口

基于List的集合允许元素的重复，同时提供了随机访问的方法，可以对集合中的任意元素进行操作。

### ArrayList

![-w1026](http://img.longzhuang.top/15939690988785.jpg)


#### ArrayList简介

ArrayList 是一个数组队列，内部基于数组实现，相当于动态数组。与Java中的数组相比，它的容量能动态增长。它继承于`AbstractList`，实现了`List`, `RandomAccess`, `Cloneable`, `java.io.Serializable`这些接口。

ArrayList 实现了`RandmoAccess`接口，即提供了随机访问功能。`RandmoAccess`是java中用来被List实现，为List提供快速访问功能的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。

由于ArrayList基于数组实现，那么它也有数组的一些性质，例如：
- 支持快速随机访问，即对元素的查、改可以在$O(1)$时间完成
- 在数组中间的增删操作需要移动数组，时间效率不佳
- 需要占据连续的内存空间

和Vector不同，**ArrayList中的操作不是线程安全的！**所以，建议在单线程中才使用ArrayList，而在多线程中可以选择`Vector`或者`CopyOnWriteArrayList`。

#### ArrayList构造函数

ArrayList中提供了三种构造函数：

- `ArrayList()` : 默认构造函数
- `ArrayList(int initialCapacity)` : 传入一个初始的容量（默认值为10）
- `ArrayList(Collection c)` : 使用指定的Collection构造ArrayList

#### ArrayList源码分析

##### 1. 成员属性`elementData`和`size`

`elementData`是一个Object类型的数组，它就是ArrayList保存元素的方式。它的初始容量默认为10。当集合中的元素超出这个容量，便会进行扩容操作。需要扩容时，ArrayList会把数组容量扩大一半。

扩容操作也是ArrayList 的一个性能消耗比较大的地方，所以若我们可以提前预知数据的规模，应该通过`ArrayList(int initialCapacity)`构造方法，指定集合的大小，去构建ArrayList实例，以减少扩容次数，提高效率。

`size`就是动态数组的**实际大小**，即保存了多少个元素。


##### 2. 常用操作

###### 1. 增

- 先判断是否越界，是否需要扩容。 
    - 如果扩容， 就复制数组。 
    - 然后设置对应下标元素值。

值得注意的是： 
- 如果需要扩容的话，默认扩容一半。如果扩容一半不够，就用目标的size作为扩容后的容量。 

###### 2. 删

- 删除操作会修改modCount，且可能涉及到数组的复制，相对低效。 
- 批量删除中，涉及高效的保存两个集合公有元素的算法，可以留意一下。

###### 3. 改

不会修改modCount，相对增删是高效的操作。
```java
public E set(int index, E element) {
    rangeCheck(index);//越界检查
    E oldValue = elementData(index); //取出元素 
    elementData[index] = element;//覆盖元素
    return oldValue;//返回元素
}
```

###### 4. 查

数组本身支持快速随机访问，效率很高。
```java
public E get(int index) {
    rangeCheck(index);//越界检查
    return elementData(index); //下标取数据
}
E elementData(int index) {
    return (E) elementData[index];
}
```

##### 3. 小结

- 扩容操作会导致数组复制，批量删除会导致 找出两个集合的交集，以及数组复制操作，因此，增、删都相对低效。 而 改、查都是很高效的操作。
- 与ArrayList相似的Vector的内部也是数组做的，区别在于Vector在API上都加了synchronized所以它是线程安全的，以及Vector扩容时，是翻倍size，而ArrayList是扩容50%。

---

### LinkedList

![-w931](http://img.longzhuang.top/15939691374302.jpg)


#### LinkedList简介

除了List接口外，LinkedList还实现了`Deque`接口，提供了add/remove、offer/poll等方法，可以作为一个队列或双端队列来使用。

顾名思义，LinkedList是基于链表实现，这也使它具有链表的特点，即对结点的插入和删除效率更高，但随机访问性能较差。同时由于使用链表实现，不需要连续的内存空间，也不需要考虑扩容的问题。

和ArrayList一样，**LinkedList也不是线程安全的**。

#### LinkedList源码分析

##### 1. 类的定义

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

从这段代码中我们可以清晰地看出 `LinkedList` 继承 `AbstractSequentialList`，实现 `List`、`Deque`、`Cloneable`、`Serializable`。其中 **AbstractSequentialList** 提供了 List 接口的骨干实现，从而最大限度地减少了实现受“连续访问”数据存储（如链接列表）支持的此接口所需的工作,从而以减少实现 List 接口的复杂度。Deque是一个线性 collection，支持在两端插入和移除元素，定义了双端队列的操作。

LinkedList类中有三个属性，分别为`size`、`first`、`last`
- `int size` : 保存当前列表中的元素个数
- `Node<E> first` : 保存双向链表的第一个节点
- `Node<E> last` : 保存双向链表的最后一个节点

##### 2. 提供的方法

作为List，LinkedList可以使用`get(int index)`获取指定位置的元素，使用`add(E e)`向尾部添加元素，使用`remove(int index)`来删除指定位置的元素。

LinkedList还可以作为双端队列使用。它实现了`Deque`接口，因此提供了队列和双端队列的操作方法。

由于LinkedList基于链表实现，它不能实现快速随机访问，而是需要用遍历链表的方式进行。获取指定位置的元素的实现方式如下：
```java
// 获取index位置的结点
Node<E> node(int index) {
    // assert isElementIndex(index);
    // 前半部分的结点从头部开始遍历；后半部分的结点从尾部开始遍历
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
```

LinkedList还可以用于队列或双端队列，这部分内容在Queue接口的总结中介绍。

---

## Queue接口

在Queue接口下有三个常用的类，分别为`LinkedList`, `ArrayDeque`, `PriorityQueue`。

其中，Queue有一个子接口`Deque`，`LinkedList`和`ArrayDeque`都是基于该接口实现。

### Queue和Deque接口的通用方法：

作为队列，常用的操作为从队列的一端入队、从另一端出队，以及查看下一个要出队的元素。

Queue中有两套操作的方法，都能实现相同的功能，但其中一套是遇到问题时（例如从空队列取元素或向满队列添加元素）抛出异常，另一套则是返回特殊值（false/null）

1. 抛出异常型

    `add()`方法向末尾添加元素，`remove()`从头部取出元素，`element()`查看头部元素。

2. 返回特殊值型

    `offer()`方法向末尾添加元素，`poll()`从头部取出元素，`peek()`查看头部元素。

双端队列Deque继承了Queue接口，因此它也能够进行上述操作，不过它提供了更为具体的方法，可以自由选择对头部还是对尾部操作。可以直接体现在方法名上。对于上述方法，直接在其后加上First/Last，例如：
-  `addLast()`, `removeFirst()`, `offerLast()`, `pollFirst()`, `peekFirst()`, `getFirst()`
-  `addFirst()`, `removeLast()`, `offerFirst()`, `pollLast()`, `peekLast()`, `getLast()`

其中第一行对应了Queue中的方法，第二行则是Queue中无法实现的操作。除了`getFirst()`和`getLast()`替代了Queue中的`element()`外，其余的方法都是在原有方法上加后缀。

---

### LinkedList

LinkedList实现了List和Deque两个接口，基于链表，允许插入null。关于List接口的部分已经介绍完毕，这里介绍它对Deque接口的实现。

LinkedList中关于添加和删除元素的操作都是基于`linkFirst()`, `linkLast()`, `unlinkFirst()`, `unlinkLast()`完成的。

`linkFirst()`的实现如下：
```java
private void linkFirst(E e) {
    final Node<E> f = first;    // 获取头结点
    final Node<E> newNode = new Node<>(null, e, f); // 根据元素值创建新结点
    first = newNode; // 将新结点设为头结点
    // 建立头结点和原头结点的连接
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

而`linkLast()`则是对尾结点进行操作，其余和上述实现相同。除此之外，还有一个`linkBefore()`方法，实现向链表的中间插入，它除了需要多建立一道连接外，与上述方法也类似。

从链表中删除结点的方法有`unlinkFirst()`和`unlinkLast()`以及`unlink()`，其中`unlink()`用于删除任意结点，其他两个用于删除首位结点。

例如，`unlinkFirst()`的实现如下：
```java
// 这里的f是上一个调用函数传入的，并且一定为头结点。
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

其他两个方法的实现类似。

对于操作头结点的public方法，例如`addFirst()`, `removeFirst()`, `offerFirst()`, `pollFirst()`, `remove()`和`poll()`等，最终都会去调用`linkFirst()`和`unlinkFirst()`进行实际的结点操作，区别在于在操作前可以对集合状态以及输入参数的合法性进行检查。
反之那些对尾结点进行操作的方法，最终都会调用`linkLast()`和`unlinkLast()`进行实际的结点操作。

---

### ArrayDeque

![-w849](http://img.longzhuang.top/15939691845774.jpg)


ArrayDeque是Deque的另一个实现类，这个类从JDK1.6开始引入，它和ArrayList类似，底层都基于数组实现。ArrayDeque把数组看作**循环数组**进行处理，以便能够更好地支持双端队列的特性，即从头部和尾部都能够方便地操作元素。

注意：
- **ArrayDeque不支持插入null值**
- **ArrayDeque不是线程安全的**

在实现队列和堆栈的时候，ArrayDeque的性能要优于LinkedList，非常重要的原因是LinkedList每插入一个元素都要使用new来创建一个结点，这会带来很大的开销；同时因为链表结构的内存地址不连续，很难命中缓存，这就导致了从主存读取的又一大的时间开销。因此如果只需要实现双端队列的功能，使用ArrayDeque通常可以获得更好的性能。

但需要注意的是，并不是所有情况下都适合使用ArrayDeque，例如当你必须在队列中保存null值时就需要使用LinkedList（尽管官方不推荐null值）；另外由于ArrayDeque没有实现List接口，它也没有提供随机访问的方法，因此如果需要使用List的相关功能，ArrayDeque就无法提供。

#### ArrayDeqeu构造函数

ArrayDeque提供了三种构造函数，分别为默认构造、从已有Collection中构造，以及指定默认容量。下面主要介绍第三种。

在ArrayDeque源码中该构造函数实现如下：
```java
public ArrayDeque(int numElements) {
    allocateElements(numElements);
}

// 创建一个数组用于存放元素
private void allocateElements(int numElements) {
    elements = new Object[calculateSize(numElements)];
}

// 根据元素数量来确定初始的数组容量
private static int calculateSize(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;
    // Find the best power of two to hold elements.
    // Tests "<=" because arrays aren't kept full.
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1); // 将高2位设置为1
        initialCapacity |= (initialCapacity >>>  2); // 将高4位设置为1
        initialCapacity |= (initialCapacity >>>  4); // 将高8位设置为1
        initialCapacity |= (initialCapacity >>>  8); // 将高16位设置为1
        initialCapacity |= (initialCapacity >>> 16); // 将高32位设置为1
        initialCapacity++;

        if (initialCapacity < 0)   // Too many elements, must back off
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
    }
    return initialCapacity;
}
```

我们重点关注`private static int calculateSize(int numElements)`这一方法。这个方法的作用是将数组的实际容量控制为大于且最接近指定值的2的整数次方。在位运算中，距离一个数最近的2的整数次幂就是比这个数高一位的比特为1，其余为0。
通过五次逻辑右移操作，可以确保把最高的1bit位和它的右边全部设置为1。然后再+1，就可以得到一个距离它最近的而且大于它的2的整数次幂了。

#### ArrayDeque属性和方法

在ArrayDeque中有用来存放集合元素的Object类的数组`elements`；头元素的下标`head`和尾元素的下标`tail`。

ArrayDeque的元素个数并不是用专门的成员变量来保存，而是使用如下方法计算返回：
```java
public int size() {
    return (tail - head) & (elements.length - 1);
}
```

由于数组的容量是比元素数大的2的整数次幂，那么`elements.length - 1`就是将最高位的右侧全部置1，其他位为0。这个与运算实际上相当于取模的操作。因为ArrayDeque作为循环数组，头元素是会向前增长过0，从而循环到到数组的末尾的，所以头结点的下标可能比尾结点更大。

##### 1. 插入元素

ArrayDeque提供了双端队列提供的所有插入方法，实际起作用的方法有两个，分别是在头部插入的`addFirst()`和在尾部插入的`addLast()`。

`addFirst()`方法的源码如下：
```java
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}
```

在头部插入，会先把`head`减1，并且取模，保证了下标可以循环。然后将元素放入数组的对应下标位置。

接下来判断首位结点是否相遇，如果相遇说明需要进行扩容，这时把容量扩充为原来的2倍。`doubleCapacity()`方法的实现如下：
```java
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    int newCapacity = n << 1;
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    head = 0;
    tail = n;
}
```

扩容的过程大概分为以下步骤[^double]：

[^double]: ArrayDeque部分内容参考自简书： https://www.jianshu.com/p/b29a516eb322

> 1. 检测tail是否等于head；
> 2. 创建一个原数组容量2倍的新数组；
> 3. 将原数组从head位置一分为二，分别拷贝到新数组中；比如数组容量16，head位置是7，则先将原数组下标从7开始，拷贝16-7个元素到新的下标从0开始，元素个数为16-7的数组中。然后将原数组下标从0开始到7的元素拷贝到新数组下标从（16-7）开始，元素个数是7的数组中。这么做的目的就是为了拷贝到新数组的时候保持元素原来的顺序。
> 4. 重新确定头尾指针的位置。

向尾部插入的`addLast()`方法类似：
```java
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[tail] = e;
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}
```

##### 2. 删除元素

删除元素使用的方法为`pollFirst()`和`pollLast()`，分别为从头部和从尾部删除元素，在队列为空时返回null（这也是ArrayDeqeu不允许存放空值的原因之一）。其他删除方法直接调用这两个方法或在此基础上检查队列为空时抛出异常。

`pollFirst()`的实现如下：

```java
public E pollFirst() {
    int h = head;
    @SuppressWarnings("unchecked")
    E result = (E) elements[h];
    // Element is null if deque empty
    if (result == null)
        return null;
    elements[h] = null;     // Must null out slot
    head = (h + 1) & (elements.length - 1);
    return result;
}
```

获取元素的方式很简单，利用头部下标从数组中取出即可。同时将head+1并取模（循环数组），与插入恰好相反。而`pollLast()`与这个方法区别不大，就不再赘述。

#### 小结

总体来说, ArrayDeque是实现栈和队列以及双端队列的首选，它不需要从中间删除结点，因此通过循环数组下标保存头尾结点的结构保障了它的高性能。它从首位读取元素的时间都是$O(1)$，并且不需要创建新结点，而且由于数组内存空间连续的特点，连续的读取也会有较高的缓存命中率。

它的缺点是由于容量必须为2的幂次，也一定程度上降低了空间的利用率，在最坏的情况下，如果元素最大个数为$2^k$，就要实际创建一个容量为$2^{k+1}$的数组，浪费了一半的空间。

---

### PriorityQueue

![-w1025](http://img.longzhuang.top/20200715174019.jpg)


PriorityQueue，即优先级队列，继承自AbstractQueue，即可以实现Queue接口的所有方法。Java中的PriorityQueue是通过堆来实现的，可以通过构造方法传入比较器来决定如何判定优先级。默认为最小值堆，即队列首部的元素就是整个队列的最小值，如果有多个就取其中之一。

堆是一棵完全二叉树，而由于完全二叉树的性质决定了它可以使用顺序存储结构通过随机访问来高效实现，因此PriorityQueue的底层数据结构是数组，也有capacity，有扩容操作。

同样，PriorityQueue也不是线程安全的。

#### 属性

```java
private static final int DEFAULT_INITIAL_CAPACITY = 11; //默认的初始化容量

transient Object[] queue; //存放元素的数组，非私有方便嵌套类访问

private int size = 0; //元素个数

private final Comparator<? super E> comparator;  // 比较器

transient int modCount = 0; // 用于实现 fast-fail 机制的
```

#### 构造方法

PriorityQueue提供了多达7个构造方法，如下：

```Java
//默认无参构造器，采用初始化容量 11，无 comparator
public PriorityQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}

//只传入指定的初始化容量，不传入 comparator
public PriorityQueue(int initialCapacity) {
    this(initialCapacity, null);
}

//只传入 comparator，采用默认容量 11
public PriorityQueue(Comparator<? super E> comparator) {
    this(DEFAULT_INITIAL_CAPACITY, comparator);
}

//传入用户指定的初始化容量和 comparator
public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}

//传入一个集合（非SortedSet/PriorityQueue或它的子类对象）
@SuppressWarnings("unchecked")
public PriorityQueue(Collection<? extends E> c) {
    // 如果传入的集合可以被SortedSet实例化，就获取它的比较器
    if (c instanceof SortedSet<?>) {
        // 先进行类型转换
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
        this.comparator = (Comparator<? super E>) ss.comparator();
        initElementsFromCollection(ss); 
    }
    // 如果传入的集合可以被PriorityQueue示例化，那么也获取比较器
    else if (c instanceof PriorityQueue<?>) {
        PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        initFromPriorityQueue(pq);
    }
    // 其他情况就没有比较器
    else {
        this.comparator = null;
        initFromCollection(c);
    }
}

//传入一个 PriorityQueue 类型的集合
@SuppressWarnings("unchecked")
public PriorityQueue(PriorityQueue<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initFromPriorityQueue(c);
}

//传入 SortedSet 类型的集合
@SuppressWarnings("unchecked")
public PriorityQueue(SortedSet<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initElementsFromCollection(c);
}
```

在上述构造方法中，使用了三种方法来从传入的集合中初始化PriorityQueue，分别是`initElementsFromCollection()`，`initFromCollection()`和`initFromPriorityQueue()`

```java
// 从PriorityQueue中初始化，如果类型相同就直接从原队列中拿数据
private void initFromPriorityQueue(PriorityQueue<? extends E> c) {
    if (c.getClass() == PriorityQueue.class) {
        this.queue = c.toArray();
        this.size = c.size();
    } else {
        initFromCollection(c);
    }
}

// 从原来的集合中初始化，由于直接拿过来的元素不满足优先队列顺序，初始化之后需要建堆
private void initFromCollection(Collection<? extends E> c) {
    initElementsFromCollection(c);
    heapify(); //建堆
}

// 从集合中初始化元素的方法
private void initElementsFromCollection(Collection<? extends E> c) {
    Object[] a = c.toArray(); //集合中的数组内容
    // If c.toArray incorrectly doesn't return Object[], copy it.
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, a.length, Object[].class);
    int len = a.length;
    if (len == 1 || this.comparator != null) 
        for (int i = 0; i < len; i++)
            if (a[i] == null) //检查是否有空元素
                throw new NullPointerException();
    this.queue = a;
    this.size = a.length;
}
```

前面提到了堆是完全二叉树，可以使用数组实现。把堆的结点按照层次顺序存储到数组中，就可以通过计算下标的方式访问到它的父结点和左右子结点。对于下标为$i$ 的结点：
1. 父结点为：$i/2\ (i ≠ 1)$，若i = 1，则i是根节点。
2. 左子结点：$2i\ (2i ≤ n)$， 若不满足则无左子结点。
3. 右子结点：$2i + 1\ (2i + 1 ≤ n)$，若不满足则无右子结点。

在PriorityQueue中，为了维持堆的特性，最重要的两个操作就是`shiftUp`和`shiftDown`。

首先我们来看`shiftUp()`相关方法：
```java
private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}

//有比较器的时候用这个
@SuppressWarnings("unchecked")
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1; //找到当前节点 k 的父节点
        Object e = queue[parent]; //取父节点
        if (comparator.compare(x, (E) e) >= 0)
            break;
        // 如果父节点优先级小于它，就和它交换，并继续判断父节点是否需要上移，
        // 直到它的优先级小于父结点或者到达了根结点
        queue[k] = e; 
        k = parent;
    }
    queue[k] = x;
}

//没有比较器的时候用这个，与前面类似
@SuppressWarnings("unchecked")
private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (key.compareTo((E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = key;
}
```

再来看`shiftDown()`相关方法：

```java
// 把下标为k的元素x下拉
private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
        siftDownComparable(k, x);
}

// 有比较器的情况
@SuppressWarnings("unchecked")
private void siftDownUsingComparator(int k, E x) {
    // 根据完全二叉树性质，size / 2就是第一个叶子结点的坐标。
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;   // 2k+1，即左子节点
        Object c = queue[child]; 
        int right = child + 1;      // 取右子节点
        //比较左右子结点找出优先级更高的一个
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0) 
            c = queue[child = right]; 
        if (comparator.compare(x, (E) c) <= 0)
            break;
        // 如果子结点有优先级高于当前结点的，就交换
        queue[k] = c;
        k = child;
    } // 终止条件是k已经是叶节点或者k小于其所有子节点
    queue[k] = x;
}

@SuppressWarnings("unchecked")
private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo((E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = key;
}
```

有了上述两个方法之后，我们再来看构造方法中的建堆操作`heapify()`:

```java
private void heapify() {
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        siftDown(i, (E) queue[i]);
}
```

可以看出，建堆的过程就是从非叶子结点开始，自底向上地将每个结点执行一次下拉操作。这样就可以保证每次都把优先级高的结点交换到上层，直到优先级最高的被交换到堆顶。

#### 堆的插入和删除

##### 1. 插入结点（入队）

新结点的插入，具体的实现在`offer()`方法中：

```java
public boolean offer(E e) {
    if (e == null)          //如果需要插入的数据为 null 抛异常
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)  // 如果队列满了就扩容
        grow(i + 1);
    size = i + 1;
    if (i == 0)             // 如果队列里尚且没有元素，
        queue[0] = e;       // 就插入到队列头部，也就是堆的根节点
    else
        siftUp(i, e);       // 否则先插入尾部，然后通过shiftUp找到对应位置。
    return true;
}
```

总的来说，新结点的插入总是在尾部进行，但是插入之后会通过shiftUp向上找到合适的位置。

##### 2. 删除结点（出队）

优先队列的出队每次都是取出头结点，即索引为0的结点。

```java
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    E result = (E) queue[0];    // 取出头结点
    // 用队尾结点替代头结点
    E x = (E) queue[s];
    queue[s] = null;
    // 下拉到合适位置
    if (s != 0)
        siftDown(0, x);
    return result;
}
```

从具体的实现方式可以看出，删除头结点后需要找到一个新的头结点。而最方便的做法就是先将尾结点提到头结点，然后再依次执行shiftDown，就可以让每个结点都到对应的位置来。

##### 3. 删除任意元素

在PriorityQueue中，还实现了一个`remove(Object o)`方法，可以删除堆中的指定元素。

```java
public boolean remove(Object o) {
    // 找到元素的下标
    int i = indexOf(o);
    if (i == -1)
        return false;
    else {
        removeAt(i);
        return true;
    }
}

// 删除指定下标的元素
private E removeAt(int i) {
    // assert i >= 0 && i < size;
    modCount++;
    int s = --size;                 // 获取队尾元素，并调整size
    if (s == i)                     // 如果只有一个元素了，就直接置null
        queue[i] = null;
    else {
        E moved = (E) queue[s];     // 用队尾元素替代被删除元素
        queue[s] = null;
        siftDown(i, moved);         // shiftDown到合适位置
        if (queue[i] == moved) {    
            siftUp(i, moved);       // 如果没有被下拉，就尝试进行shiftUp
            if (queue[i] != moved)
                return moved;
        }
    }
    return null;
}
```

删除堆中指定元素的步骤大致概括为：
1. 找到要删除的元素的坐标
2. 用队尾元素替换要被删除的元素
3. 从删除位置开始执行shiftDown
4. 如果shiftDown没有效果（即它不会比目前位置更低），就执行shiftUp

---

## Map接口

Map集合与Collection集合不同，它具有如下特点：

1. Map集合是一个双列集合，一个元素包含两值（一个key，一个value）
2. Map集合中的元素，key和value的数据类型可以相同，也可以不同
3. Map集合中的元素，key是不允许重复的，value是可以重复的
4. Map集合中的元素，key和value是一一对应的

在Java中常用的Map的实现类有`HashMap`, `LinkedHashMap`, `TreeMap`, 以及线程安全的`ConcurrentHashMap`。其中TreeMap还实现了`SortedMap`这一Map的子接口

### Map接口中的常用方法：

- **put(K key, V value)** : 将指定的值与此映射中的指定键关联
- **get(Object key)** : 返回指定键所映射的值；如果此映射不包含该键的映射关系，则返回 null。
- **getOrDefault(Object key, V defaultValue)** : 返回指定键所映射的值；如果此映射不包含该键的映射关系，则返回指定的defaultValue
- **remove(Object key)** : 如果存在一个键的映射关系，则将其从此映射中移除
- **containsKey(Object key)** : 如果此映射包含指定键的映射关系，则返回 true
- **entrySet()** : 返回此映射中包含的**映射关系**的 Set 视图
- **keySet()** : 返回此映射中包含的**键**的 Set 视图

Map中还存在一个内部接口`Entry`：`Map.Entry<K,V>`，它包含了一对映射关系（key和value)
作用：当Map集合一创建，那么就会在Map集合中创建一个Entry对象，用来记录键与值（键值对对象,键与值的映射关系)，可以使用EntrySet来对Map进行遍历。

---

### HashMap

![-w892](http://img.longzhuang.top/20200715174221.jpg)


HashMap是最常用的Map实现之一，在JDK1.7及以前它底层基于数组+链表的实现方式，在JDK1.8开始，底层使用数组+链表+红黑树来实现。

HashMap根据存储对象的hashCode值来决定存储位置，由于使用了数组，在大多数情况下都能够快速定位到元素，因而具有非常高的时间效率。HashMap允许使用null作为一个key，这一点要注意，因为在其他的Map实现中例如`ConcurrentHashMap`，可能不支持null作为key。另外，HashMap的遍历顺序是不确定的。

**HashMap不是线程安全的**

#### HashMap属性字段

首先是HashMap中的一些预设值：
```java
// 默认的初始容量为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认的负载因子为0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 这是JDK1.8在底层做的一个优化，当数组上一个Entry挂载的节点超过8个，
// 就会将当前Entry的链表结构转化为红黑树的数据结构
static final int TREEIFY_THRESHOLD = 8;
// 将红黑树退化成链表的阈值
static final int UNTREEIFY_THRESHOLD = 6;
// 转换红黑树的最小容量，如果当前的数组容量还没有到达64，
// 那么就不会转换成红黑树，而是先进行扩容操作。
static final int MIN_TREEIFY_CAPACITY = 64;
```

HashMap中的属性：
```Java
// HashMap保存结点的数组
transient Node<K,V>[] table;
// 一个entry集合，用于获取所有entry
transient Set<Map.Entry<K,V>> entrySet;
// 结点的个数
transient int size;
transient int modCount;
// 扩容的阈值，计算为Capacity * loadFactor，size达到阈值会触发扩容。
int threshold;
// 负载因子，用于控制扩容的时机
final float loadFactor;
```

接下来我们来关注一下HashMap内部的具体实现。

首先是数组，在HashMap中的`table`字段是一个Node数组，这个数组就是哈希桶数组：
```java
transient Node<K,V>[] table;
```
而其中这个Node就是对Map接口中的Entry的实现:
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
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
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

可以看到，一个Node本身就是一个键值对，同时它被定义称为链表的形式，因此当发生哈希冲突的时候，HashMap采用链地址法，多个结点可以以链表的形式放在同一个桶中。在JDK1.8以后，HashMap得到进一步的优化，当一个哈希桶的链表长度超过了8，就把这个链表转换成红黑树。红黑树的结点定义如下：

```java
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        // 方法省略
        ...
    }
```

这个TreeNode是继承自LinkedHashMap.Entry，而LinkedHashMap.Entry又继承自HashMap.Node，也就是说TreeNode是Node的一个子类，因此需要转换成红黑树的时候，只需要把转换后的根结点放在原来的哈希桶中即可。

#### HashMap构造方法

HashMap一共有4个构造方法，主要的工作就是完成初始容量和负载因子的赋值。Hash表都是采用的懒加载方式，构造HashMap时不会被创建，当第一次插入数据时才会创建。

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

我们从上面的字段可以看出默认的capacity是16，而如果我们指定了capacity，HashMap并不会直接使用这个容量，而是找到一个最接近它的大于等于它的2的整数次幂作为容量，即上述构造方法的代码中第11行调用了如下方法获取threshold：

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

这个方法和ArrayDeque中指定初始容量的方法十分类似。这样保证了经过扩容，table的容量始终为2的整数次幂，为hash计算下标提供了方便。

这里还有一个问题，虽然给threshold的定义是`capacity * factory`，即它并不等价于capacity，但是使用这种构造方法对threshold赋值后，对table的初始化操作就会直接使用threshold当作初始的capacity。这个操作可以在`resize()`方法中看到

#### HashMap获取元素

HashMap通常可以在$O(1)$时间内获取元素。获取的步骤可以概括如下：
1. 计算要找的key的哈希值，定位到数组的下标
2. 如果该位置有结点，就与该位置的头结点的key使用equals进行比较，如果比较成功，就返回该位置的结点value值。如果该位置没有结点，返回null
3. 如果equals比较失败，就检查是否有链表或者红黑树：
    1. 如果有红黑树，就调用红黑树中的方法寻找结点，并返回查找结果
    2. 如果有链表，就依次和链表中的结点key进行equals比较，直到找到对应结点返回value，或者到了链表末尾返回null

获取元素的`get()`以及相关方法的实现如下：

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // 通过hash先定位到头结点
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) { // 形成了链表或红黑树
            if (first instanceof TreeNode)  // 在红黑树中查找
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {    // 在链表中查找
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```


#### HashMap插入

HashMap使用`put(K key, V value)`方法插入一个键值对。如果在原Map中不存在这个key，就插入，否则就更新这个key的值为value。

```java
public V put(K key, V value) {
    // hash(key)计算出key的hash值,
    return putVal(hash(key), key, value, false, true);
}

// put方法实际调用putVal来完成插入。
// onlyIfAbsent决定是否覆盖已有值
// evict用于LinkedHashMap
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)     // 懒加载 table
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)  // 哈希桶中无结点，直接插入
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))  // 当前位置是已存在的相同key，保存该结点后续根据onlyIfAbsent决定是否更新
            e = p;
        else if (p instanceof TreeNode)     // 当前哈希桶中是红黑树结构，使用红黑树的putTreeVal方法插入结点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {                              // 当前哈希桶中是链表结构
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) { // 没有equals当前key的结点，说明是新结点，使用尾插法添加(jdk1.8)
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);  // 超过阈值，转红黑树(如果满足条件)
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) // 找到了equals结点，保存该结点后续根据onlyIfAbsent决定是否更新
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e); // 预留的回调方法，在HashMap中留空
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)     // 如果元素个数超过阈值，进行扩容
        resize();
    afterNodeInsertion(evict);  // 预留的回调方法，在HashMap中留空
    return null;
}
```

#### HashMap扩容

在每次插入元素后，HashMap都会检查元素的个数是否达到了阈值，如果达到了阈值就需要使用`resize()`方法进行扩容。

我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，经过rehash之后，**元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置**。

我们把resize()方法分成两部分来看。第一部分对阈值和容量做了更新操作，并且创建了一个新的table。如果是初始化，在这一部分就已经可以完成:

```java
Node<K,V>[] oldTab = table;                         // 原来的table
int oldCap = (oldTab == null) ? 0 : oldTab.length;  // 原来的table容量
int oldThr = threshold;                             // 原来的扩容阈值
int newCap, newThr = 0;
if (oldCap > 0) {
    if (oldCap >= MAXIMUM_CAPACITY) {   // 旧table已经达到了最大容量，不能再扩容了
        threshold = Integer.MAX_VALUE;
        return oldTab;
    }
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && // 容量和阈值都扩大为原来的2倍
             oldCap >= DEFAULT_INITIAL_CAPACITY)
        newThr = oldThr << 1; // double threshold
}   // 下面是初始化table的情形
else if (oldThr > 0) // initial capacity was placed in threshold (构造方法指定了容量)
    newCap = oldThr;
else {               // zero initial threshold signifies using defaults (构造方法未指定容量)
    newCap = DEFAULT_INITIAL_CAPACITY;
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
}
if (newThr == 0) {
    float ft = (float)newCap * loadFactor;
    newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
              (int)ft : Integer.MAX_VALUE);
}
threshold = newThr;     // 更新阈值
@SuppressWarnings({"rawtypes","unchecked"})
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
table = newTab;         // 使用新的table保存元素
```

第二部分是整个扩容的关键。HashMap的下标计算方式是将元素的hash码与数组的容量进行与运算。而前面提到了table的capacity总是2的整数次幂, 因此`capacity - 1`的低位全部为1。也就是截取了hash码的后若干位作为数组下标。

于是，当数组扩容之后，把capacity变成原来的2倍，即右移1位，那么再与hash做与运算的时候，hash就会又暴露出一个高比特位，此时根据这个比特位的0和1，就可以决定将它放到原位置还是增加了原capacity之后的位置。

举个例子(只举int的后8位为例):
```
当前capacity(二进制):0000 1000
key1  hash: 1010 0110 
key2  hash: 1101 1110
在扩容之前，二者计算的下标相同，即
hash & (capacity - 1) : 0000 0110

扩容之后，capacity变成 0001 0000
key1 hash & (capacity - 1): 0000 0110
key2 hash & (capacity - 1): 0000 1110
因此key1下标不变，而key2下标增加了原capacity这么多
```

我们再来分析`resize()`方法的后半部分：

```java
for (int j = 0; j < oldCap; ++j) {
    Node<K,V> e;
    if ((e = oldTab[j]) != null) {  // 取出每一个非空元素结点 e
        oldTab[j] = null;
        if (e.next == null)     // 没有形成链表/树，直接转移到新哈希表对应下标
            newTab[e.hash & (newCap - 1)] = e;
        else if (e instanceof TreeNode)     // 形成了红黑树，那么利用红黑树的split方法处理
            ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
        else { 
            // 重点：当该结点形成链表时的处理
            Node<K,V> loHead = null, loTail = null; // 呆在原坐标位置的链表结点
            Node<K,V> hiHead = null, hiTail = null; // 进入高位坐标位置的链表结点
            Node<K,V> next;
            do { // 对当链表的每个结点进行判断
                next = e.next;
                if ((e.hash & oldCap) == 0) {   // hash新暴露的高位为0
                    if (loTail == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                }
                else {  // hash新暴露的高位为1
                    if (hiTail == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                }
            } while ((e = next) != null);
            if (loTail != null) {   // 设置新数组原坐标的结点
                loTail.next = null;
                newTab[j] = loHead;
            }
            if (hiTail != null) {   // 设置新数组高位坐标的结点
                hiTail.next = null;
                newTab[j + oldCap] = hiHead;
            }
        }
    }
}
```

可以看到，在JDK1.8中，扩容后链表的插入顺序与原链表一致，它避免了JDK1.7及之前的并发死循环问题。但这并不代表它可以在并发环境中使用，如果要在并发环境下使用，应该使用`ConcurrentHashMap`

#### 小结

1. JDK1.8的HashMap中引入了红黑树, 使它的结构变成了数组+链表+红黑树，让HashMap的性能得到了提升。
2. HashMap不是线程安全的，并发环境须使用ConcurrentHashMap
3. HashMap的扩容操作比较复杂，如果容量设置过小导致频繁发生扩容会对性能产生一定影响。因此如果能预估集合的规模，可以设置一个合适的容量，尽量减少扩容的触发。

---

### LinkedHashMap

![-w885](http://img.longzhuang.top/20200715174232.jpg)


LinkedHashMap继承自HashMap，它在HashMap的基础上实现了**有序**的操作。所谓的“有序”，是指通过迭代器遍历集合的顺序。

LinkedHashMap中有两种排序方式，默认只按照元素的插入顺序进行排序，即按照put的顺序进行排序，而如果我们指定了`accessOrder`为true时，它还会根据元素的访问来更新排序，即get操作也会更新元素的顺序。

其中的第二种排序方式也使得LinkedHashMap非常适合作为LRU的实现方式。

#### LinkedHashMap顺序存取原理

LinkedHashMap与HashMap最本质的区别就是是否有序，而LinkedHashMap实现有序的方式是继承了HashMap.Node，并增加了双向链表的指针`before`和`after`，形成了一个哈希表中所有结点的链表，迭代的时候就通过这个链表来迭代，而不是根据原数组。

LinkedHashMap中对结点的定义：
```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;   // 增加了双向链表的指针
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

通过查看LinkedHashMap的源码我们可以看到，它并没有重写put方法，只是重写了增加结点的`newNode()`方法，并实现了HashMap预留的三个回调方法。

其中，`newNode()`的实现如下：
```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}
```

首先根据key-value创建一个结点，并且调用`linkNodeLast(p)`来完成有序的实现，即把新结点加入链表尾：
```java
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```

在LinkedHashMap中实现的三个回调方法：

```java
// 插入新结点之后会调用该方法，这一步是判断是否要逐出头结点，是实现LRU的重要步骤
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

// 获取或尝试更新已有结点的时候会调用该方法，这一步会更新访问的结点到链表结尾
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}

// 移除结点后，维护链表的连接关系
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

#### LinkedHashMap实现LRU

前面提到，LinkedHashMap有一个`accessOrder`属性，当它为true时，就会把每次获取或者尝试更新的结点移动到链表尾部。这个特点符合了LRU的排列机制。

前面还介绍了LinkedHashMap实现的回调方法，其中`afterNodeInsertion()`就是用于逐出头结点的。再来看这个方法：

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

可以看出，要想逐出头结点，必须满足三个条件：
1. `evict`为true。这个变量在我们调用HashMap的`put()`方法时，就会自动地被设置为true：
    ![-w600](http://img.longzhuang.top/20200715174240.jpg)
2. 头结点不为空
3. `removeEldestEntry(first)`返回true。这个方法就是我们实现LRU的关键，在LinkedHashMap的实现中，这个函数直接返回false。

因此，如果我们要实现一个LRU，只需要继承LinkedHashMap，并且重写第3步判断方法`removeEldestEntry()`即可。而作为一个LRU，我们就只需要让这个方法在结点个数达到设定的最大值的时候返回true，这样就可以在执行put之后调用`afterNodeInsertion()`，自动地删除头结点。

例如，一个简单的LRU实现：
```java
public class LRUCache<K,V> extends LinkedHashMap<K,V> {
        
    private int cacheSize;
      
    public LRUCache(int cacheSize) {
        super(16, 0.75f, true); // 调用LinkedHashMap的构造方法
        this.cacheSize = cacheSize;
    }
    
    /**
    * 判断元素个数是否超过缓存容量
    */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > cacheSize;
    }
}
```

---

### TreeMap

TreeMap是Map的一个实现，它基于**红黑树**，是一种有序的key-value集合

TreeMap的继承关系如下：
![-w701](http://img.longzhuang.top/20200716172853.jpg)

TreeMap的基本属性有4个：

```java
//这是一个比较器，方便插入查找元素等操作
private final Comparator<? super K> comparator;
//红黑树的根节点：每个节点是一个Entry
private transient Entry<K,V> root;
//集合元素数量
private transient int size = 0;
//集合修改的记录
private transient int modCount = 0;
```

TreeMap的每一个结点是一个Entry，实现了Map.Entry，这个Entry的实现是红黑树结点,在结点中提供了一些基本的方法:

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;

    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    public K getKey() {
        return key;
    }

    public V getValue() {
        return value;
    }

    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;

        return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
    }

    public int hashCode() {
        int keyHash = (key==null ? 0 : key.hashCode());
        int valueHash = (value==null ? 0 : value.hashCode());
        return keyHash ^ valueHash;
    }

    public String toString() {
        return key + "=" + value;
    }
}
```

#### TreeMap构造方法

TreeMap有四种构造方法：

```java
//构造方法1：默认构造方法，比较器为空
public TreeMap() {
    comparator = null;
}
//构造方法2：指定一个比较器
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
//构造方法3：从指定的Map创建，无比较器
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}
//构造方法4：从指定的Sorted
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

如果我们传入了比较器，那么后续维持红黑树的时候使用的就是传入的比较器。反之如果没有定义比较器，即comparator为null，那么红黑树的排序就是按照自然排序。同时由于TreeMap实现自SortedMap，而SortedMap接口中就定义了比较器，因此传入一个SortedMap的对象时，就可以直接从其中获取比较器。

TreeMap的插入和删除操作都是基于红黑树进行的，这部分内容可以直接查看对红黑树的介绍，结合红黑树来看TreeMap的源码就很容易理解。

---

### ConcurrentHashMap[^chm]

[^chm]: ConcurrentHashMap参考连接：[https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)

![-w789](http://img.longzhuang.top/20200716172842.jpg)

