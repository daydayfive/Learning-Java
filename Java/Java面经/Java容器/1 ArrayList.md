#ArrayList

##概览
 
 因为ArrayList 是基于数组实现的，所以支持快速随机访问。RandomAccess 接口标识着该类支持快速随机访问。

 ```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
 ```

数组的默认大小为10
 ```java
private static final int DEFAULT_CAPACITY = 10;
 ```

 ![](images/2021-04-21-13-24-34.png)


 ## 扩容
 
 ArrayList 添加元素时使用ensureCapacityInternal()来保证容量足够，如果不够时，需要使用grow()方法进行扩容，新容量的大小为```oldCapacity + (oldCapacity >> 1)``` ,即 oldCapacity+oldCapacity/2。 其中oldCapacity >>1 需要取整，所以新容量大约是旧容量的1.5倍左右。 （oldCapacity为偶数就是1.5倍，为奇数就是1.5倍-0.5）

 扩容操作需要调用 ```Arrays.copyOf()``` 把原数组整个复制到新数组中，这个操作代价很高，因次最好在创建ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

 ```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
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
    modCount++;
    // overflow-conscious code
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


 ## 删除元素
 需要调用 System.arraycopy()将index+1 后面的元素都复制到index位置上，该操作的事件复杂度为O(N)，可以看到ArrayList删除元素的代价是非常高的。

 ```java
public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

 ```


 ## 序列化

 ArrayList基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

 保存元素的elementData 使用 transient修饰，该关键字 声明数组默认不会被序列化。
 ```java
 transient Object[] elementData; // non-private to simplify nested class access
 ```

 ArrayList 实现了 writeObject()和readObject()来控制之序列化数组有元素填充那部分内容。

 ```java
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
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
 private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
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

 序列化时需要使用ObjectOutputStream的writeObject()将对象转化为字节流并输出。而writeObject()在传入的对象存在writeObject()的时候会去反射调用该对象的writeObject()来实现序列化。反序列化使用的是ObjectInputStream的readObject()方法，原理类似。
 ```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
 ```


 ## Fail-Fast
**modCount**用来记录ArrayList结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

 在进行序列化或者迭代等操作时，需要比较操作前后modCount是否改变，如果改变了需要抛出ConcurrentModificationException。
 