# CopyOnWriteArrayList

## 读写分离

写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。


写操作需要加锁，防止并发写入时导致写入数据丢失。


写操作结束之后需要把原始数组指向**新的复制数组**。

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

然而 CopyOnWriteArrayList 的数组是volatile 修饰的，具有可见性
```java
private transient volatile Object[] array;
```

## 适用场景
CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

* 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；


所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。
