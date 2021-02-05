**位运算**

> 定义：
>
> 程序中的所有数在计算机内存中都是以二进制的形式储存的。位运算就是直接对整数在内存中的二进制位进行操作。

```java
public class demoMain {
    public static void main(String[] args) {
        System.out.println(5&5);
        System.out.println(5&6);
        System.out.println(5&7);
        System.out.println(5&8);
    }
}
         /**
         * 运行结果
         * 5  过程：（5&5）->(0101 & 0101 = 0101 ->5)
         * 4  过程：（5&6）->(0101 & 0110 = 0100 ->4)
         * 5  过程：（5&7）->(0101 & 0111 = 0101 ->5)
         * 0  过程：（5&8）->(0101 & 1000 = 0000 ->0)
         */
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> （and） & 运算原理：
>
> 相同位的两个数字都为1，则为1；若有一个不为1，则为0。

```java
public class demoMain {
    public static void main(String[] args) {
        System.out.println(5|5);
        System.out.println(5|6);
        System.out.println(5|7);
        System.out.println(5|8);
        /**
         * 运行结果
         * 5  过程：（5|5）->(0101 | 0101 = 0101 ->5)
         * 7  过程：（5|6）->(0101 | 0110 = 0111 ->7)
         * 7  过程：（5|7）->(0101 | 0111 = 0111 ->7)
         * 13 过程：（5|8）->(0101 | 1000 = 1101 ->13)
         */
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> （or）| 运算原理
>
> 相同位只要一个为1即为1。

```java
public class demoMain {
    public static void main(String[] args) {
        System.out.println(5^5);
        System.out.println(5^6);
        System.out.println(5^7);
        System.out.println(5^8);
        /**
         * 0  过程：（5^5）->(0101 | 0101 = 0000 ->0)  
         * 3  过程：（5^6）->(0101 | 0110 = 0011 ->3)
         * 2  过程：（5^7）->(0101 | 0111 = 0010 ->2)
         * 13 过程：（5^8）->(0101 | 1000 = 1101 ->13)
         */
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> (xor) ^ 异或运算 --同为假，异为真
>
> 按位异或运算, 对**等长**二进制模式按位或二进制数的每一位执行逻辑按位异或操作. 操作的结果是如果某位不同则该位为1, 否则该位为0.

> 其他：
>
> 二进制的最末位为0表示该数为偶数，最末位为1表示该数为奇数。

```java
public class demoMain {
    public static void main(String[] args) {
        System.out.println(~5);
        System.out.println(~6);
        System.out.println(~7);
        System.out.println(~8);
          /**
         * -6  过程：（~5）->(0101 ->1010)  
         * -7  过程：（~6）->(0110 ->1001)
         * -8  过程：（~7）->(0111 ->1000)
         * -9  过程：（~8）->(0010 ->1101)
         */
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> not 运算 ~

# hash

# ![img](https://img-blog.csdnimg.cn/20201209192605976.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



> hash计算



 ![img](https://img-blog.csdnimg.cn/20201209192624952.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDIyOTY1,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> hash冲突原因



 ![img](https://img-blog.csdnimg.cn/202012091927207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDIyOTY1,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> 解决方法



![img](https://img-blog.csdnimg.cn/20201209192802888.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDIyOTY1,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> **后移法**
>
> 如果想要放置的key位置被放置了，则将当前的key对应数据放在后面最近的一个位置上。记住，hash的本质就是数组，此时的数组并不是每个位置都是满的，会有空位，这个位置被占了，就往后面一个位置挪就好了。



![img](https://img-blog.csdnimg.cn/20201209192817804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDIyOTY1,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



> 链拉法
>
> 将对应同一个hash值的key都放在同一个节点后续，使用链表的方式进行保存