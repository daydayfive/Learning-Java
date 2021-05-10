# TreeMap 与 TreeSet

## 概念

* TreeMap 是一个**有序**的**key-value 集合**，它是通过**红黑树**实现的，红黑树结构天然支持排序，默认情况下通过key值的自然顺序进行排序；
* TreeMap 是按照**键**而不是值有序，都是对键进行比较
* TreeMap 继承了NavigableMap接口，NavigableMap 接口继承了 SortedMap 接口，可支持一系列的导航定位以及导航操作的方法，当然只是提供了接口，需要 TreeMap 自己去实现。


## 基本API与使用
(1) 构造方法
TreeMap():创建一个空TreeMap，keys按照自然排序，要求Map中的键实现Comparable接口。
```java
TreeMap<Integer, String> treeMap = new TreeMap<>();
```

TreeMap(Comparator comparator): 创建一个空  TreeMap， 按照指定的comparator 排序。即创建自定义的比较器

```java
TreeMap<Integer, String> map = new TreeMap<>(Comparator.reverseOrder());
map.put(3, "val");
map.put(2, "val");
map.put(1, "val");
map.put(5, "val");
map.put(4, "val");
System.out.println(map); // {5 = val, 4 = val, 3 = val, 2 = val, 1 = val} 逆序
```

```java
// String.CASE_INSENSITIVE_ORDER是String类中的一个忽略大小写的Comparator
Map<String, String> map = new TreeMap<>(String.CASE_INSENSITIVE_ORDER);
// 逆序并忽略大小写
Map<String, String> map = new TreeMap<>(Collections.reverseOrder(String.CASE_INSENSITIVE_ORDER));
```


TreeMap(Map m):由给定的map创建一个TreeMap，keys按照自然排序。Hutool中对Map的排序就是用的这个方法将原本的map转为TreeMap。
```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "val");
...
TreeMap<Integer, String> treeMap = new TreeMap<>(map);
```


(2) 使用实例
```java
TreeMap<Integer, String> treeMap = new TreeMap<>();
treeMap.put(1, "a");
treeMap.put(2, "b");
treeMap.put(3, "c");
treeMap.put(4, "d"); 	// treeMap: {1 = a, 2 = b, 3 = c, 4 = d}

treeMap.remove(4); // treeMap: {1 = a, 2 = b, 3 = c}
int sizeOfTreeMap = treeMap.size(); // sizeOfTreeMap: 3

treeMap.replace(2, "e"); // treeMap: {1 = a, 2 = e, 3 = c}

Map.Entry entry = treeMap.firstEntry(); // entry: 1 -> a
Integer key = treeMap.firstKey(); // key: 1
entry = treeMap.lastEntry(); // entry: 3 -> c
key = treeMap.lastKey(); // key: 3
String value = treeMap.get(3); // value: c
SortedMap sortedMap = treeMap.headMap(2); // sortedMap: {1 = a}
sortedMap = treeMap.subMap(1, 3); // sortedMap: {1 = a, 2 = e}

Set setOfEntry = treeMap.entrySet(); // setOfEntry: [1 = a, 2 = e, 3 = c]
Collection<String> values = treeMap.values(); // values: [a, e, c]
treeMap.forEach((integer, s) -> System.out.println(integer + "->" + s)); 
// output：
// 1 -> a
// 2 -> e
// 3 -> c
```

(3)遍历方式
for循环
```java
// 与HashMap迭代类似
for (Map.Entry entry : treeMap.entrySet()) {
      System.out.println(entry);
}
```
迭代器循环
```java
Iterator iterator = treeMap.entrySet().iterator();
while (iterator.hasNext()) {
      System.out.println(iterator.next());
}
```

### 源码解析

这部分可参照：https://www.cnblogs.com/LiaHon/p/11221634.html
