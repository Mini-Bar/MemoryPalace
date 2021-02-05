# 						java反射机制（reflection）



![image-20201204173339521](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201204173339521.png)

#### 								简单的说，反射机制就是指程序在运行时可以获取自身的信息！！！



## 为什么需要使用 

（比如开发一个文本编辑器：支持txt,pdf,doc三种模式，我们将txt,pdf,doc定义为三个子模块）

**静态编译：**一次性编译的时候将所有的模块都编译进去。

（我想看个txt,点击图标后，三个模块全部加载到系统，另外两个模块并未使用，所以占用了系统资源）

**动态编译：** 按需要编译你用到的模块，并不需要一次性全部编译出来。

（我想看那个，就加载判断属于什么格式就加载那个格式，其他的两个模块并未使用，这样<u>编译速度加快，节省系统资源，利于今后拓展</u>）

<hr/>

#### 如何获得某个class文件对应的class对象：

`1.类名.class`

`2.对象.getClass()       --Object类提供`

`3.Class.forName("包名.类名")`

``java``

```java
//例如有一个Person对象
package cn.mini.reflect.test;
import org.junit.jupiter.api.Test;

public class ClassTest {
    @Test
    /**
     * 获得Class对象 1.通过类名.class 2.对象.getClass() 3.Class.forName();
     */
    public void demo1() throws ClassNotFoundException {
        // 1.类名.class方式
        Class class1=Person.class;
        // 2.通过对象.getClass()方式
        Person person = new Person();
        Class class2 = person.getClass();
        // 3.Class类forName();获得(推荐，因为使用反射一般都不会知道对象的)
        Class class3=Class.forName("cn.mini.reflect.test.Person");
    }
}
```

## 使用步骤

1.获取目标类型的Class对象

2.通过获取的class对象获取Constructer类对象，Method类对象，Filed类对象

3.通过上面获取的三种不同对象分别获取累的构造函数，方法，属性的具体信息。

![4997216-badc52a62b817876](C:\Users\Administrator\Desktop\4997216-badc52a62b817876.png)

<hr/>

#### 获取目标类型的class对象（四种方式）

##### 方法一：Object.getClass();

```java
//方法一：Object.getClass();
 Boolean carson = true; 
 Class<?> classType = carson.getClass(); // Object类中的getClass()返回一个Class类型的实例
 System.out.println(classType); // 输出结果：class java.lang.Boolean  
```

##### 方法二：T.class语法

``` java
 // T = 任意Java类型（泛型）
  Class<?> classType = Boolean.class; 
  System.out.println(classType); // 输出结果：class java.lang.Boolean  
```

##### 方法三：Static method Class.fotName

```java
 Class<?> classType = Class.forName("java.lang.Boolean"); 
  // 使用时应提供异常处理器
 System.out.println(classType);  // 输出结果：class java.lang.Boolean  
```

##### 方法四:TYPE语法

```java
  Class<?> classType = Boolean.TYPE; 
  System.out.println(classType);  // 输出结果：boolean  
```

<hr/>

## 实例应用讲解

### 1：获取类的属性和赋值

项目结构<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201207115151039.png" alt="image-20201207115151039"  />

```java
package test;
public class Student {
    public Student() {
        System.out.println("创建了一个Student实例");
    }
    private String name;
}
```

```java
package test;
import java.lang.reflect.Field;

class demo{
    public static void main(String[] args) {
        //获取student的class对象
        Class<Student> studentClass = Student.class;
        try {
            //通过获取到的对象实例化对象
            Student student = studentClass.newInstance();
            //通过class对象获取name属性
            Field f = studentClass.getDeclaredField("name");
            //设置私有访问权限（true为私有权限可以访问，否则相反）
            f.setAccessible(true);
            //通过实例化的对象设置name属性值
            f.set(student,"minibar");
            //输出测试结果
            System.out.println(f.get(student));
        } catch (NoSuchFieldException | IllegalAccessException |
                 InstantiationException e) {
            e.printStackTrace();
        }
    }
}
```

运行结果：![image-20201207115411630](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201207115411630.png)

### 2：利用反射调用类的构造函数

项目结构：![image-20201207120454989](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201207120454989.png)

```java
package test2;
public class Student {
    // 无参构造函数
    public Student() {
        System.out.println("调用了无参构造函数");
    }
    // 有参构造函数
    public Student(String str) {
        System.out.println("调用了有参构造函数: "+str);
    }
    private String name;
}
```

```java
package test2;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class test {
    public static void main(String[] args) {
        //获取Student的class对象
        Class<Student> studentClass1 = Student.class;
        try {
            //通过构造器获取class对象
            Constructor<Student> constructor = studentClass1.getConstructor();
            //通过class对象获取Constructor类对象（调用的是无参构造方法）
            Student student = constructor.newInstance();
            //通过class对象获取Constructor对象 (调用的是有参构造方法)
            Constructor<Student> constructor1 = studentClass1.getConstructor(String.class);
            Student student1 = constructor1.newInstance("hello");
        } catch (NoSuchMethodException | InstantiationException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

运行结果：![image-20201207121528792](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201207121528792.png)

### 3：调用类对象的方法

项目结构：![image-20201207141255218](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201207141255218.png)

```java
package test2;
public class Student {
    public Student() {
        System.out.println("创建了一个Student实例");
    }
    // 无参数方法
    public void setName1 (){
        System.out.println("调用了无参方法：setName1（）");
    }
    // 有参数方法
    public void setName2 (String str){
        System.out.println("调用了有参方法：setName2（String str）:" + str);
    }
}
```

```java
package test2;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class test {
    public static void main(String[] args) {
        //获取Student的class对象
        Class<Student> studentClass = Student.class;
        try {
            //实例化Student对象
            Student student = studentClass.newInstance();
            //获取class对象获取方法setName1的Method对象
            Method setName1 = studentClass.getMethod("setName1");
            setName1.invoke(student);
            //获取class对象获取方法setName2的Method对象
            Method setname2 = studentClass.getMethod("setName2", String.class);
            setname2.invoke(student,"hello Method");
        } catch (InstantiationException | IllegalAccessException |
                NoSuchMethodException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}

```

运行结果：![image-20201207141432906](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201207141432906.png)
#### 反射中动态创建数组

通过Array.newinstance()方法

``` java
// Array.java
public static Object newInstance(Class<?> componentType, int... dimensions)
        throws IllegalArgumentException, NegativeArraySizeException {
        return multiNewArray(componentType, dimensions);
    }
```

```java
//比如我们呢要创建 int [][] ints = new int[2][3];
第一个参数（数组类元素类型）第二个（相应维度中数组的长度）
Array.newInstance(int.class,2,3);
```
#### Array的读取与赋值

```java
import java.lang.reflect.Array;

public class ArrayList {
    /**main*/
    public static void main(String[] args) throws Exception {
        ArrayList testArray = new ArrayList();
        testArray.testOneObject();
        testArray.testOne();
        testArray.testTwo();
    }

    /**放置数据
     * @throws ClassNotFoundException **/
    public void testOneObject() throws ClassNotFoundException{
        Object object = Array.newInstance(Object.class,10);
        Array.set(object,3,"00000");
        System.out.println(Array.get(object,3));
    }

    /**一维素组*/
    public void testOne() throws ClassNotFoundException{
        Class<?> classType = Class.forName("java.lang.String");
        Object object = Array.newInstance(classType,10);  //数组0,9
        Array.set(object,5,"11111");
        System.out.println(Array.get(object, 5));
        System.out.println(Array.getLength(object));
    }

    /**二维数组*/
    public void testTwo() throws ClassNotFoundException {
        Class<?> classType = Class.forName("java.lang.String");
        Object object = Array.newInstance(classType, 10,10);  //数组0,9
        Array.set(Array.get(object,5),5,"22222");  //[5][5] 为"123",先获取一维，在通过一维获取二维的数据
        System.out.println(Array.get(Array.get(object,5),5));
    }
}
```

运行结果：![image-20201207145000747](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201207145000747.png)

<hr/>









