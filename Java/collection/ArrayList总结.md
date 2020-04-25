<!-- MarkdownTOC -->

- [简介](#arraylist简介)
- [扩容机制](#arraylist核心扩容技术)
- [System.arraycopy()和Arrays.copyOf\(\)方法](#systemarraycopy和arrayscopyof方法)
- [重要知识点](#重要知识点)

<!-- /MarkdownTOC -->


### ArrayList简介
ArrayList 的底层是数组队列，相当于动态数组。与 Java 中的数组相比，它的容量能动态增长。在添加大量元素前，应用程序可以使用`ensureCapacity`操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量。

在我们学数据结构的时候就知道了线性表的顺序存储，插入删除元素的时间复杂度为**O（n）**,求表长以及增加元素，取第 i   元素的时间复杂度为**O（1）**

它继承于 **AbstractList**，实现了 **List**, **RandomAccess**, **Cloneable**, **java.io.Serializable** 这些接口。

ArrayList 继承了**AbstractList**，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。

ArrayList 实现了**RandomAccess 接口**， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持**快速随机访问**的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。

ArrayList 实现了**Cloneable 接口**，即覆盖了函数 clone()，**能被克隆**。

ArrayList 实现**java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。

和 Vector 不同，**ArrayList 中的操作不是线程安全的**！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Vector 或者  CopyOnWriteArrayList。



### ArrayList 核心扩容技术

以无参构造函数为例，其扩容机制如下：

简略：

- 1，扩容10；
- 11~`MAX_ARRAY_SIZE`，1.5倍扩容；
- `MAX_ARRAY_SIZE + 1`扩容`Integer.MAX_VALUE`；
- `Integer.MAX_VALUE + 1`，异常`OutOfMemoryError`。

详细：

- 初始时，数组elementData长度为零；
- 扩容发生在`List#add(E)`和`List#add(int, E)`中。
- 添加第1个元素时，发生扩容，长度为默认初始容量（DEFAULT_CAPACITY）10；
- 添加第2~10个元素时，不发生扩容；
- 添加第11~`MAX_ARRAY_SIZE`个之间的元素时，按照之前1.5倍的容量进行扩容。
- 添加第`MAX_ARRAY_SIZE`（`Integer.MAX_VALUE - 8`）+ 1元素时，容量扩充为`Integer.MAX_VALUE`;
- 添加`MAX_ARRAY_SIZE + 1`~`Integer.MAX_VALUE`之间的元素时，不发生扩容；
- 添加第`Integer.MAX_VALUE + 1`元素时，抛出异常`OutOfMemoryError`。

如果使用了含参构造函数，则不存在DEFAULT_CAPACITY，直接添加第1~`MAX_ARRAY_SIZE`个之间的元素时，按照之前1.5倍的容量进行扩容。

扩容核心源码如下：

```java
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access    
		
		/**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
		
		/**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

		// 主要无参和含参构造函数的区别:
		// 前者使用了DEFAULTCAPACITY_EMPTY_ELEMENTDATA，后者使用了EMPTY_ELEMENTDATA
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
      	// 无参构造函数，会利用DEFAULT_CAPACITY，含参则不会利用
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
      	// 将新容量更新为旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
   

```
　　

需要注意的是：

1. java 中的**length 属性**是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
2. java 中的**length()方法**是针对字  符串String说的,如果想看这个字符串的长度则用到 length()这个方法.
3. .java 中的**size()方法**是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!



###  System.arraycopy()和Arrays.copyOf()方法

　　通过上面源码我们发现这两个实现数组复制的方法被广泛使用而且很多地方都特别巧妙。比如下面add(int index, E element)方法就很巧妙的用到了arraycopy()方法让数组自己复制自己实现让index开始之后的所有成员后移一个位置:

```java 
    /**
     * 在此列表中的指定位置插入指定的元素。 
     *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()方法实现数组自己复制自己
        //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量；
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
    }
```

又如toArray()方法中用到了copyOf()方法

```java
    /**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。 
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
        return Arrays.copyOf(elementData, size);
    }
```

**联系：**看两者源代码可以发现`copyOf()`内部调用了`System.arraycopy()`方法
**区别：**

1. arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置
2. copyOf()是系统自动在内部新建一个数组，并返回该数组。



### 重要知识点

**modCount的作用**

modCount表示列表的结构被改变的次数。在使用迭代器遍历的时候，用来检查列表中的元素是否发生结构性变化（列表元素数量发生改变）了，主要在多线程环境下需要使用，防止一个线程正在迭代遍历，另一个线程修改了这个列表的结构。

> The number of times this list has been structurally modified. Structural modifications are those that change the size of the list, or otherwise perturb it in such a fashion that iterations in progress may yield incorrect results.
> This field is used by the iterator and list iterator implementation returned by the iterator and listIterator methods. If the value of this field changes unexpectedly, the iterator (or list iterator) will throw a ConcurrentModificationException in response to the next, remove, previous, set or add operations. This provides fail-fast behavior, rather than non-deterministic behavior in the face of concurrent modification during iteration.
>
> ref：https://www.cnblogs.com/zuochengsi-9/p/7050351.html



**为什么modCount被transient关键字修饰**

modCount被transient关键字修饰，`protected transient int modCount = 0`，表示不被序列化，因为modCount的具体值没有意义，初始值可以用0表示。不被序列化可以节省存储空间。

> ref：https://www.cnblogs.com/chenpi/p/6185773.html



**为什么elementData[]被transient关键字修饰**

elementData[]是一个缓存数组，它通常会预留一些容量，那么有些空间可能就没有实际存储元素，这样如果序列化elementData[]就会浪费存储空间和时间。

那么被transient修饰后如何保证序列化呢？ArrayList在内部重写序列化writeObject和反序列化方法readObject。序列化的时候，直接将size和每个element写入ObjectOutputStream；反序列化的时候，从ObjectInputStream获取size和每个element，再恢复到elementData[]。这样就可以保证只序列化实际存储的那些元素，而不是整个数组，从而节省空间和时间。

> ref：https://blog.csdn.net/zero__007/java/article/details/52166306 



**fail-fast**

**fail-fast 机制是java集合(Collection)中的一种错误机制。**当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。在集合操作中使用modCount来检测这一机制。

> ref: 
>
> https://juejin.im/post/5cb683d6518825186d65402c#heading-4
>
> https://blog.csdn.net/ch717828/article/details/46892051

