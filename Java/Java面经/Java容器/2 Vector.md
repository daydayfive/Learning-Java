# Vector

## 同步
它的实现与ArrayList类似，但是使用了synchronized对各种方法进行同步
```java
public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }

public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }

public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        E oldValue = elementData(index);

        int numMoved = elementCount - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--elementCount] = null; // Let gc do its work

        return oldValue;
}

```

## 扩容
Vector 的构造函数可以传入 capacityIncrement 参数，它的作用是在扩容时使容量capacity 增长 capacityIncreament。 如果这个参数的值小于等于0，扩容时每次都令capacity 为原来的两倍。

```java
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
调用没有capacityIncrement的构造函数时，默认capacityIncrement为0，默认情况下Vecotr每次扩容容量都会翻倍。
```java
public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}

public Vector() {
    this(10);
}
```


## 与ArrayList的比较
* Vector 是同步的，因此开销就比ArrayList要大，访问速度更慢。最好使用ArrayList而不是Vector，因为同步操作完全可以由程序员自己来控制；
* Vector 每次扩容请求其大小的2倍，而ArrayList是1.5倍


## 替代方案
可以用Collection.synchronizedList(),得到一个线程安全的ArrayList。
```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);
```
也可以使用concurrent并发包下的CopyOnWriteArrayList类。
```java
List<String> list=new CopyOnWriteArrayList<>();
```