#### 概览

------

容器主要包括Collection和Map两种，Collection存储着对象的集合，而Map存储着键值对（两个对象）的映射表。

**Collection**

[![img](https://camo.githubusercontent.com/c0506ba8f5134d89ed6a398e1c165865d50c68f6c7af4e01b75248a95e0da37d/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383232303934383038342e706e67)](https://camo.githubusercontent.com/c0506ba8f5134d89ed6a398e1c165865d50c68f6c7af4e01b75248a95e0da37d/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383232303934383038342e706e67)

**Set**

- TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如HashSet，HashSet查找的时间复杂度为O(1),TreeSet则为O(logN)。
- HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用Iterator遍历HashSet得到的结果是不确定的。
- LinkedHashSet：具有HashSet的查找效率，并且内部使用双向链表维护元素的插入顺序。

**List**

- ArrayList：基于动态数组实现，支持随机访问
- Vector：和ArrayList类似，但它是线程安全的
- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速的在列表中间插入和删除元素。不仅如此，LinkedList还可以用作栈、队列、和双向队列

**Queue**

- LinkedList：可以用它来实现双向队列。
- PriorityQueue：基于堆结构实现，可以用它来实现优先队列



**Map**

[![img](https://camo.githubusercontent.com/ce6470fc8cfd0f0c74ba53bd16ee9467b21c5ca7fc0566413df4701342a96a15/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303230313130313233343333353833372e706e67)](https://camo.githubusercontent.com/ce6470fc8cfd0f0c74ba53bd16ee9467b21c5ca7fc0566413df4701342a96a15/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303230313130313233343333353833372e706e67)

- TreeMap：基于红黑树实现。
- HashMap：基于哈希表实现。
- HashTable：和hashMap类似，但它是线程安全的，这意味着同一时刻多个线程同时写入HashTable不会导致数据不一致。它是一流泪，不应该去使用它，而是使用ConcurrentHashMap来支持线程安全，ConcurrentHashMap的效率会更高，因为ConcurrentHashMap引入了分段锁。
- LinkedHashMap：使用双向列表来维护元素的顺序，顺序为插入顺序或最近最少使用顺序

#### 容器中的设计模式

------

**迭代器模式**

[![img](https://camo.githubusercontent.com/e44932dbf2dc015828f878428f58df99b9e808a6fb5cc76ab65b8baab7c0d2bb/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383232353330313937332e706e67)](https://camo.githubusercontent.com/e44932dbf2dc015828f878428f58df99b9e808a6fb5cc76ab65b8baab7c0d2bb/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383232353330313937332e706e67)

Collection继承了Iterable接口，其中Iterator（）方法能够产生一个Iterator对象，通过这个对象可以迭代遍历Collection中的元素。

从jdk1.5 之后可以使用foreach方法来编辑实现了Iterator接口的聚合对象

```java
package dataStructure.chaper2;

import java.util.ArrayList;
import java.util.List;

public class Test {

    public static void main(String[] args) throws CloneNotSupportedException {

        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");

        for (String str : list) {
            System.out.println(str);
        }

    }


}

```

**适配器模式**

java.util.Arrays # asList() 可以把数组类型转换为List类型。

```java
@SafeVarages
public static <T> List<T> asList(T... a)
```

应该注意的是asList的参数为泛型的边长参数，不能用基本数据类型数组作为参数，只能使用响应的包装类型数组。

```java
package dataStructure.chaper2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Test {

    public static void main(String[] args) throws CloneNotSupportedException {

        Integer[] arr = new Integer[]{
                1, 2, 3, 4, 5
        };
        List<Integer> list = Arrays.asList(arr);
        for (Integer str : list) {
            System.out.println(str);
        }

    }

}

```

也可以使用一下方式调用asList():

```java
List list = Arrays.asList(1,2,3);
```

#### 源码分析

------

如果没有特别说明，以下源码分析基于JDK1.8。

在IDEA中double shift调出Search EveryWhere，查找源码文件，找到之后就可以阅读源码。

**ArrayList**

- 概览

  因为ArrayList是基于数组实现的，所以支持快速随机访问。RandomAccess接口标识着该类支持快速随机访问。

  ```java
  public class ArrayList<E> extends AbstractList<E> implements List<E>,RandomAccess,Cloneable,java.io.Serializable
  ```

  数组默认大小为10。

  ```JAVA
  private static final int DEFAULT_CAPACITY = 10;
  ```

  [![img](https://camo.githubusercontent.com/dfd2ffd9b9f090b35d2a2ae1af4e3a700dfc234b2f892d310806534001cd0885/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383233323232313236352e706e67)](https://camo.githubusercontent.com/dfd2ffd9b9f090b35d2a2ae1af4e3a700dfc234b2f892d310806534001cd0885/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383233323232313236352e706e67)

  

- 扩容

  添加元素时使用ensueCapacityInternal（）方法来保证容量足够，如果不够时，需要使用grow（）方法进行扩容，新容量的大小为oldCapacity+（oldCapacity>>1）,即oldCapacity+oldCapacity/2。其中oldCapacity>>1需要取整，所以新容量大约是旧容量的1.5倍左右。（oldCapacity 为偶数就是 1.5 倍，为奇数就是 1.5 倍-0.5）

  ```java
  public boolean add(E e){
    ensureCapacityInternal(size+1);
    elementData[size++] = e;
    return true;
  }
  private void ensureCapacityInternal(int minCapacity){
    if(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA){
      minCapacity = Math.max(minCapacity,DEFAULT_CAPACITY);
    }
    ensureExplicitCapacity(minCapacity);
  }
  
  private void ensureExplicitCapacity(int minCapacity){
    modCount ++;
    if(minCapacity - elementData.length >0){
      grow(minCapacity);
    }
  }
  
  private void grow(int minCapacity){
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity>>1);
    if(newCapacity -minCapacity<0) newCapacity = minCapacity;
    if(newCapacity-MAX_ARRAY_SIZE >0) newCapacity = hugeCapacity(minCapacity);
    
    elementData =Arrays.copyOf(elementData,newCapacity);
  }
  ```

  

- 删除元素

  需要调用System.arraycopy()将index+1后面的元素都复制到index位置上，该操作的时间复杂度为O(N),可以看到ArrayList删除元素的代价非常高

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

  

- 序列化

  ArrayList基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定会被使用，那么久没必要全部序列化。

  保存元素的数组elementData使用 transient修饰，该关键字声明数组默认不会被序列化。

  ```java
  transient Object[] elementData; // non-private to simplify nested class access
  
  ```

  ArrayList实现了writeObject和readObject来控制之序列化数组中有元素填充那部分内容。

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
          ensureCapacityInternal(size);
  
          Object[] a = elementData;
          // Read in all elements in the proper order.
          for (int i=0; i<size; i++) {
              a[i] = s.readObject();
          }
      }
  }
  
  ```

  ```
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

  序列化时需要使用ObjectOutputSystem的writeObject将对象转换为字节流并输出。而writeObject（）方法在传入的对象存在writeObject的时候回去反射调用该对象的writeObject来实现序列化。反序列化使用的是ObjectInputSystem的readObject（）方法，原理类似

- 

