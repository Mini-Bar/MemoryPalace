------

# ArrayList



## Comparable(排序接口)



```java
class Dog implements Comparable<Dog> {
    private String name;
    private int age;

    public Dog(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return '\n'+"Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}'+'\n';
    }

    // 看你自己默认排序哪一个
    @Override
    public int compareTo(Dog o) {
        //调用Arrays.sort() 默认排序狗的年龄 从小到大排序
        return this.age - o.age;
    }

}
```

## Comparator(比较器)

```java
//ReverSortDogName.java

import java.util.Comparator;
// 想弄实例化的排序就要调用Comparator
class ReverSortDogName implements Comparator<Dog> {
    @Override
    public int compare(Dog o1, Dog o2) {
        return o2.getName().compareTo(o1.getName());
    }
}
```

```java
import java.util.Arrays;
public class demoMain {
    public static void main(String[] args) {

        Dog[] dogs = new Dog[]{
                new Dog("111", 16),
                new Dog("333", 18),
                new Dog("222", 17)};

        //Arrays.sort(dogs);   //Comparable ，执行这个方法默认是按age从小到大排序
         Arrays.sort(dogs, new ReverSortDogName()); // Comparator，按name字符串编码从大到小排序
        System.out.println(Arrays.toString(dogs));
    }
}
```



## CompareTo源码分析

```java
    public int compareTo(String anotherString) {
        //获取两个需要比较的字符串的长度
        int len1 = value.length;
        int len2 = anotherString.value.length;
        //取两个字符串长度的最小值
        int lim = Math.min(len1, len2);
        //获取两个字符串的值
        char v1[] = value;
        char v2[] = anotherString.value;//（String底层使用char类型的数组保存类容）
		//从第一位开始，依次比较两个字符串每个字符的asc码
        int k = 0;
        while (k < lim) {
            char c1 = v1[k];//（举例：123）
            char c2 = v2[k];//（举例：124）
            //如果asc码不同，返回差值
            if (c1 != c2) {
                return c1 - c2;  //(举例：123-124=-1，从大到小排列)
            }
            k++;
        }
        //如果两个字符串的asc码比到最后，依然相同，则返回两个字符串长度的差值，举例（c1：123，c2:1234）
        return len1 - len2;
    }
/**
 * 正数：从小到大排列
 * 负数：从大到小排列
 * 相同：内容相同
 */
```

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class demoMain {
    public static void main(String[] args) {
        List<Integer> list1 = Arrays.asList(5, 3, 2, 4, 1);
        Collections.sort(list1);
        System.out.println(list1);//默认升序

        List<Integer> list2 = Arrays.asList(5, 3, 2, 4, 1);
        Collections.sort(list2, (a, b) -> b - a);
        System.out.println(list2);//自定义降序
    }
}
```



## 集合遍历的方式

区别：

​	1：fori是通过下标访问。

​	2：foreach是通过容器的itrator的next()方法来迭代。

### 一：使用lambda表达式便利集合

```java
import java.util.HashMap;
import java.util.HashSet;
public class demoMain {
    public static void main(String[] args) {
        //遍历set集合
        HashSet<String> books = new HashSet<>();
        books.add("第一本");
        books.add("第二本");
        books.add("第三本");
        books.forEach(str -> System.out.println("书名："  + str));
        //遍历map集合
        HashMap<String,String> mac=new HashMap<>();
        mac.put("1","1");
        mac.put("2","2");
        mac.put("3","3");
        mac.forEach((s, s2) -> System.out.println(s+"="+s2) );
    }
}
```



### 二：使用iterator遍历集合元素

```java
import java.util.HashSet;
import java.util.Iterator;

public class demoMain {
    public static void main(String[] args) {
        HashSet<String> books = new HashSet<>();
        books.add("第一本");
        books.add("第二本");
        books.add("第三本");
        //迭代器遍历集合
        Iterator<String> iterator=books.iterator();
        while(iterator.hasNext()){
            //hasNext()判断是否有下一个元素
            System.out.println(iterator.hasNext());
            //next() 取出下一个元素
            System.out.println(iterator.next());
        }
    }
}
```



#### remove

```java
import java.util.ArrayList;
import java.util.List;

public class demoMain {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("北京");
        list.add("上海");
        list.add("广州");
        list.add("深圳");
        for (String s : list) {
            System.out.println(s);
            if ("北京".equals(s)) {
                list.remove(s);
            }
        }
    }
}
```



说明：迭代器在使用的过程中，禁止修改迭代器中的内容，否则会怕抛出**java.util.ConcurrentModificationException**	

#### fail-fast

即快速失败机制,它是java集合中的一种错误检测机制, 当单个或多个线程在结构上对集合进行改变时,就有可能产生fail-fast机制.

```java
private class Itr implements Iterator<E> {
        // Android-changed: Add "limit" field to detect end of iteration.
        // The "limit" of this iterator. This is the size of the list at the time the
        // iterator was created. Adding & removing elements will invalidate the iteration
        // anyway (and cause next() to throw) so saving this value will guarantee that the
        // value of hasNext() remains stable and won't flap between true and false when elements
        // are added and removed from the list.
        protected int limit = ArrayList.this.size;

        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor < limit;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            /**
             * 通过代码得知,expectedModCount 这个值再对象创建时就被赋予了一个固定值modCont,
             * 所以expectedModCount 的值肯定是固定的,那问题就显而易见了,当迭代器遍历元素时, 
             * 如果modCount这个值发生了改变,那么就会报出ConcurrentModificationException异常
             */
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();//此处异常
            int i = cursor;
            if (i >= limit)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();//此处异常

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
                limit--;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

```

#### fail safe

fail safe机制的原理是会复制原集合的一份数据出来,然后操作复制后的数据,避免操作原数据.

使用  `CopyOnWriteArrayList`和`ConcurrentHashMap`无论改变值还是结构都不会报出`ConcurrentModificationException`

#### 源码分析（以ArrayList为例子）

```java
private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return(要返回的下一个元素的索引)
        int lastRet = -1; // index of last element returned; -1 if no such（最后要返回的元素索引，如果没有就为-1）
        int expectedModCount = modCount;

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

<hr/>

### 三：使用lambda表达式便利Iterator

```java
//JDK8 新增方法

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class demoMain {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("第一本书");
        list.add("第二本书");
        list.add("第三本书");
        Iterator<String> iterator = list.iterator();
        //方式一
        iterator.forEachRemaining(System.out::println);
        //方式二
        //iterator.forEachRemaining(s -> System.out.println(s));
    }
}
```

运行结果：![image-20201208173244444](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201208173244444.png)



### 四：使用 foreach 循环遍历集合元素

```java
import java.util.ArrayList;
import java.util.List;

public class demoMain {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("第一本书");
        list.add("第二本书");
        list.add("第三本书");
        for (String s:list) {
            System.out.println(s);
        }
    }
}
```





### 五：使用Stream遍历集合

```java
import java.util.ArrayList;
import java.util.List;

public class demoMain {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("北京");
        list.add("上海");
        list.add("广州");
        list.add("深圳");

      //list.stream().forEach(s -> System.out.println(s));//方式一
        list.stream().forEach(System.out::println);//方式二
    }
}
```

```java
import java.util.stream.Stream;

public class demoMain {
    public static void main(String[] args) {
        Stream stream=Stream.builder().add("第一本书").add("第二本书").add("第三本书").build();
        stream.forEach(System.out::println);
    }
}
```



## ArrayList源码分析:

### ArrayLsit默认参数：

```java
private static final long serialVersionUID = 8683452581122892189L;
 
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
 
    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
 
    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
 
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access
 
    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
     
 
    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

### ArrayList底层数据结构：

```java
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
 
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
 
    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```

> 底层结构就是一个数组

### ArrayList动态扩容：

```java
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
 
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

> 1. 每添加一个元素，检查数组剩余空间是否足够
> 2. 如果第一次添加元素，就创建容量为10的数组
> 3. 如果数组剩余空间不足，触发扩容
> 4. 每次扩容现有容量的50%
> 5. 把旧数组内容拷贝到新数组
> 6. 添加新元素

### Arrays.copyOf &System.arraycopy

```java
import java.util.Arrays;
 
public class demoMain {
    public static void main(String[] args) {
        /**
         * System.arraycopy  通过原数组的指定长度值改变目标数组中对应的值
         */
        String[] Array1 = {"a", "b", "c", "d"};
        String[] Array2 = {"1", "2", "3", "4"};
        System.arraycopy(Array1, 0, Array2, 0, 4);
        for (String s : Array1) {
            System.out.println("array1: " + s);
        }
        System.out.println("System.arraycopy后的结果：");
        for (String s : Array2) {
            System.out.println("array2: " + s);
        }
        /**
         *  Arrays.copyOf  只改变源数组的长度，对长度进行扩容
         */
        Object[] name = {"a", "b", "c", "d"};
        for (Object ojb : name) {
            System.out.println(ojb);
        }
        System.out.println("未改变之前的数组长度: " + name.length);
        name = Arrays.copyOf(name, 6);
        for (Object ojb : name) {
            System.out.println(ojb);
        }
        System.out.println("改变之后的数组长度: " + name.length);
    }
}
```

> ```html
> System.arraycopy  只改变源数组的长度，对长度进行扩容
> 
> Arrays.copyOf     只改变源数组的长度，对长度进行扩容
> ```

> ![img](https://img-blog.csdnimg.cn/20201209102826407.png)
>
> 将第一个数组从第一位开始截取到长度为length=2的位复制到第二个数组0位开始
>
> 结果：![img](https://img-blog.csdnimg.cn/20201209103232432.png)



