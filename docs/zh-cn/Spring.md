# Spring

### bean注入方式

##### 🚕 Car对象

```java
//Car对象
public class Car {
    // 只包含基本数据类型的属性
    private int speed;
    private double price;
    
    public Car() {
    }
    public Car(int speed, double price) {
        this.speed = speed;
        this.price = price;
    }
    
    public int getSpeed() {
        return speed;
    }
    public void setSpeed(int speed) {
        this.speed = speed;
    }
    public double getPrice() {
        return price;
    }
    public void setPrice(double price) {
        this.price = price;
    }
    @Override
    public String toString() {
        return "Car{" +
                "speed=" + speed +
                ", price=" + price +
                '}';
    }
}
```
##### 🥳User对象

```java
//User对象
public class User {
	
    private String name;
    private int age;
    // 除了上面两个基本数据类型的属性，User还依赖Car
    private Car car;
    
    public User() {
    }
    public User(String name, int age, Car car) {
        this.name = name;
        this.age = age;
        this.car = car;
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
    public Car getCar() {
        return car;
    }
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", car=" + car +
                '}';
    }
}
```

#### 1.构造器注入

##### 例子1

```java
 /*带参数，方便利用构造器进行注入*/ 
 public CatDaoImpl(String message){ 
 	this. message = message; 
 } 
<bean id="CatDaoImpl" class="com.CatDaoImpl"> 
	<constructor-arg value=" message "></constructor-arg> 
</bean>
```

##### 例子2

###### 	**（一）匹配构造器的参数名称**

```xml

//使用最上面的User和Car
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <!-- 通过constructor-arg的name属性，指定构造器参数的名称，为参数赋值 -->
    <constructor-arg name="speed" value="100" />
    <constructor-arg name="price" value="99999.9"/>
</bean>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <constructor-arg name="name" value="aaa" />
    <constructor-arg name="age" value="123" />
    <!-- 
         和之前一样，基本数据类型或Java包装类型使用value，
         而引用类型使用ref，引用另外一个bean的id 
    -->
    <constructor-arg name="car" ref="myCar" />
</bean>
```

这样就完成了，测试代码和之前一样，运行结果也一样，我这里就不贴出来了。有人看完之后，可能会觉得这里的配置和`set`注入时的配置几乎一样，除了一个使用`property`，一个使用`constructor-arg`。确实，写法上一样，但是表示的含义却完全不同。**property的name属性，是通过set方法的名称得来；而constructor-arg的name，则是构造器参数的名称**。

###### **（二）匹配构造器的参数下标**

上面是通过构造器参数的名称，匹配需要传入的值，那种方式最为直观，而`Spring`还提供另外两种方式匹配参数，这里就来说说通过参数在参数列表中的下标进行匹配的方式。下面的配置，请结合`2.2`节中`User`和`Car`的构造方法一起阅读，配置方式如下：

```xml
<bean id="car" class="cn.tewuyiang.pojo.Car">
    <!-- 下标编号从0开始，构造器的第一个参数是speed，为它赋值100 -->
    <constructor-arg index="0" value="100" />
    <!-- 构造器的第二个参数是price，为它赋值99999.9 -->
    <constructor-arg index="1" value="99999.9"/>
</bean>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- 与上面car的配置同理 -->
    <constructor-arg index="0" value="aaa" />
    <constructor-arg index="1" value="123" />
    <constructor-arg index="2" ref="car" />
</bean>
```

  上面就是通过参数的下标为构造器的参数赋值，需要注意的是，**参实的下标从0开始**。使用上面的方式配置，若赋值的类型与参数的类型不一致，将会在容器初始化`bean`的时候抛出异常。如果`bean`存在多个参数数量一样的构造器，`Spring`容器会自动找到类型匹配的那个进行调用。比如说，`Car`有如下两个构造器，`Spring`容器将会调用第二个，因为上面的配置中，`index = 1`对应的`value`是`double`类型，与第二个构造器匹配，而第一个不匹配：

```java
public Car(double price, int speed) {
    this.speed = speed;
    this.price = price;
}
// 将使用匹配这个构造器
public Car(int speed, double price) {
    this.speed = speed;
    this.price = price;
}
```

  还存在另外一种特殊情况，那就是多个构造器都满足`bean`的配置，此时选择哪一个？假设当前`car`的配置是这样的：

```java
<bean id="car" class="cn.tewuyiang.pojo.Car">
    <!-- 两个下标的value值都是整数 -->
    <constructor-arg index="0" value="100" />
    <constructor-arg index="1" value="999"/>
</bean>
```

  假设`Car`还是有上面两个构造器，两个构造器都是一个`int`类型一个`double`类型的参数，只是位置不同。而配置中，指定的两个值都是`int`类型。但是，`int`类型也可以使用`double`类型存储，所以上面两个构造器都是匹配的，此时调用哪一个呢？结论就是调用第二个。自己去尝试就会发现，**若存在多个构造器匹配bean的定义，Spring容器总是使用最后一个满足条件的构造器**。

###### **（三）匹配构造器的参数类型**

  下面说最后一种匹配方式——匹配构造器的参数类型。直接看配置文件吧：

```xml
<bean id="car" class="cn.tewuyiang.pojo.Car">
    <!-- 使用type属性匹配类型，car的构造器包含两个参数，一个是int类型，一个是double类型 -->
    <constructor-arg type="int" value="100" />
    <constructor-arg type="double" value="99999.9"/>
</bean>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- 对于引用类型，需要使用限定类名 -->
    <constructor-arg type="java.lang.String" value="aaa" />
    <constructor-arg type="int" value="123" />
    <constructor-arg type="cn.tewuyiang.pojo.Car" ref="car" />
</bean>
```

  上面应该不难理解，直接通过匹配构造器的参数类型，从而选择一个能够完全匹配的构造器，调用这个构造器完成`bean`的创建和属性注入。需要注意的是，上面的配置中，类型并不需要按构造器中声明的顺序编写，`Spring`也能进行匹配。这也就意味着可能出现多个能够匹配的构造器，和上一个例子中一样。比如说，`Car`还是有下面两个构造器：



```java
public Car(double price, int speed) {
    // 输出一句话，看是否调用这个构造器
    System.out.println(111);
    this.speed = speed;
    this.price = price;
}
// 将使用匹配这个构造器
public Car(int speed, double price) {
    // 输出一句话，看是否调用这个构造器
    System.out.println(222);
    this.speed = speed;
    this.price = price;
}
```

  上面两个构造器都是一个`int`，一个`double`类型的参数，都符合xml文件中，`car`这个`bean`的配置。通过测试发现，**Spring容器使用的永远都是最后一个符合条件的构造器**，这和上面通过下标匹配是一致的。**需要说明的一点是，这三种使用构造器注入的方式，可以混用**。

#### 2.set方法注入

##### 例子1

```java
public class Id { 
  private int id; 
  public int getId() { return id; } 
  public void setId(int id) { this.id = id; } } 
<bean id="id" class="com.id ">
    <property name="id" value="123"></property> 
</bean>
```

##### 例子2


 有了上面两个类，我们就可以演示**set**注入了。需要注意一点，如果我们需要使用**set**注入，那么必须要为属性提供**set**方法，Spring容器就是通过调用**bean**的**set**方法为属性注入值的。而在**xml**文件中，使用set注入的方式就是通过**property**标签，如下所示：

```xml
<!-- 定义car这个bean，id为myCar -->
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <!-- 
        为car的属性注入值，因为speed和price都是基本数据类型，所以使用value为属性设置值；
        注意，这里的name为speed和price，不是因为属性名就是speed和price，
        而是set方法分别为setSpeed和setPrice，名称是通过将set删除，然后将第一个字母变小写得出；
    -->
    <property name="speed" value="100"/>
    <property name="price" value="99999.9"/>
</bean>

<!-- 定义user这个bean -->
<bean id="user" class="cn.tewuyiang.pojo.User">
    <property name="name" value="aaa" />
    <property name="age" value="123" />
    <!-- car是引用类型，所以这里使用ref为其注入值，注入的就是上面定义的myCar 
         基本数据类型或Java包装类型使用value，
         而引用类型使用ref，引用另外一个bean的id 
    -->
    <property name="car" ref="myCar" />
</bean>
```

通过上面的配置，就可以为`Car`和`User`这两个类型的`bean`注入值了。需要注意的是，**property的name属性，填写的不是属性的名称，而是set方法去除set，然后将第一个字符小写后的结果。对于基本数据类型，或者是Java的包装类型（比如String），使用value注入值，而对于引用类型，则使用ref，传入其他bean的id。**接下来我们就可以测试效果了：

```java
@Test
public void test1() {
    ApplicationContext context =  new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    // 获取user这个bean
    User user = context.getBean(User.class);
    // 输出产看结果
    System.out.println(user);
}
```

由于`user`包含`car`的引用，所以我们直接输出`user`，也能够看到`car`的情况，输入结果如下：

```java
User{name='aaa', age=123, car=Car{speed=100, price=99999.9}}
```

#### 3.静态工厂注入

##### 例子1

```java
public class DaoFactory { //静态工厂 
 public static final FactoryDao getStaticFactoryDaoImpl(){ 
 return new StaticFacotryDaoImpl(); 
 } 
} 
public class SpringAction { 
 private FactoryDao staticFactoryDao; //注入对象
 //注入对象的 set 方法 
 public void setStaticFactoryDao(FactoryDao staticFactoryDao) { 
 this.staticFactoryDao = staticFactoryDao; 
 } 
} 
//factory-method="getStaticFactoryDaoImpl"指定调用哪个工厂方法
 <bean name="springAction" class=" SpringAction" > 
 	<!--使用静态工厂的方法注入对象,对应下面的配置文件--> 
	 <property name="staticFactoryDao" ref="staticFactoryDao"></property> 
 </bean> 
 <!--此处获取对象的方式是从工厂类中获取静态方法--> 
 <bean name="staticFactoryDao" class="DaoFactory" 
	factory-method="getStaticFactoryDaoImpl">
 </bean>
```

##### 例子2

```java
public class SimpleFactory {

    /**
     * 静态工厂，返回一个Car的实例对象
     */
    public static Car getCar() {
        return new Car(12345, 5.4321);
    }
}
```

```xml
<!-- 
	注意，这里的配置并不是创建一个SimpleFactory对象，取名为myCar，
    这一句配置的意思是，调用SimpleFactory的getCar方法，创建一个car实例对象，
    将这个car对象取名为myCar。
-->
<bean id="car" class="cn.tewuyiang.factory.SimpleFactory" factory-method="getCar"/>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- name和age使用set注入 -->
    <property name="name" value="aaa"/>
    <property name="age" value="123"/>
    <!-- 将上面配置的car，注入到user的car属性中 -->
    <property name="car" ref="car"/>
</bean>
```

![](https://ftp.bmp.ovh/imgs/2021/02/86238c8de7a3dc85.png) 

以上就配置成功了，测试方法以及执行效果如下，注意看`car`的属性值，就是我们在静态工厂中配置的那样，这说明，`Spring`容器确实是使用我们定义的静态工厂方法，创建了`car`这个`bean`：

```java
@Test
public void test1() {
    ApplicationContext context =
        new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    // 获取静态工厂创建的car
    Car car = (Car) context.getBean("car");
    // 获取user
    User user = context.getBean(User.class);
    System.out.println(car);
    System.out.println(user);
}
```

  输出如下所示：

```java
Car{speed=12345, price=5.4321}
User{name='aaa', age=123, car=Car{speed=12345, price=5.4321}}
```

#### 4.实例工厂注入

##### 例子1

```java
public class DaoFactory { //实例工厂 
 public FactoryDao getFactoryDaoImpl(){ 
	 return new FactoryDaoImpl(); 
 } 
} 
public class SpringAction { 
 private FactoryDao factoryDao; //注入对象 
 public void setFactoryDao(FactoryDao factoryDao) { 
 	this.factoryDao = factoryDao; 
 } 
} 
 <bean name="springAction" class="SpringAction"> 
	 <!--使用实例工厂的方法注入对象,对应下面的配置文件--> 
 	<property name="factoryDao" ref="factoryDao"></property> 
 </bean> 
 <!--此处获取对象的方式是从工厂类中获取实例方法--> 
 <bean name="daoFactory" class="com.DaoFactory"></bean> 
 <bean name="factoryDao"
    factory-bean="daoFactory"
	factory-method="getFactoryDaoImpl">
 </bean>
```


##### 例子2

实例工厂与静态工厂类似，不同的是，静态工厂调用工厂方法不需要先创建工厂类的对象，因为静态方法可以直接通过类调用，所以在上面的配置文件中，并没有声明工厂类的`bean`。但是，实例工厂，需要有一个实例对象，才能调用它的工厂方法。我们先看看实例工厂的定义：

```java
public class SimpleFactory {

    /**
     * 实例工厂方法，返回一个Car的实例对象
     */
    public Car getCar() {
        return new Car(12345, 5.4321);
    }

    /**
     * 实例工厂方法，返回一个String
     */
    public String getName() {
        return "tewuyiang";
    }

    /**
     * 实例工厂方法，返回一个int，在Spring容器中会被包装成Integer
     */
    public int getAge() {
        return 128;
    }
}
```

  在上面的工厂类中，共定义了三个工厂方法，分别用来返回`user`所需的`car`，`name`以及`age`，而配置文件如下：



```xml
<!-- 声明实例工厂bean，Spring容器需要先创建一个SimpleFactory对象，才能调用工厂方法 -->
<bean id="factory" class="cn.tewuyiang.factory.SimpleFactory" />

<!-- 
    通过实例工厂的工厂方法，创建三个bean，通过factory-bean指定工厂对象，
    通过factory-method指定需要调用的工厂方法
-->
<bean id="name" factory-bean="factory" factory-method="getName" />
<bean id="age" factory-bean="factory" factory-method="getAge" />
<bean id="car" factory-bean="factory" factory-method="getCar" />

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- 将上面通过实例工厂方法创建的bean，注入到user中 -->
    <property name="name" ref="name"/>
    <property name="age" ref="age"/>
    <property name="car" ref="car"/>
</bean>
```
![](https://ftp.bmp.ovh/imgs/2021/02/1c18c792421edee9.png)

  我们尝试从`Spring`容器中取出`name`，`age`，`car`以及`user`，看看它们的值，测试代码如下：

```java
@Test
public void test1() {
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    // 获取静态工厂创建的car，name和age这三个bean
    Car car = (Car) context.getBean("car");
    String name = (String) context.getBean("name");
    Integer age = (Integer) context.getBean("age");
    // 获取user这个bean
    User user = context.getBean(User.class);
    System.out.println(car);
    System.out.println(name);
    System.out.println(age);
    System.out.println(user);
}
```

  以下就是输出结果，可以看到，我们通过工厂创建的`bean`，都在`Spring`容器中能够获取到：

```java
Car{speed=12345, price=5.4321}
tewuyiang
128
User{name='tewuyiang', age=128, car=Car{speed=12345, price=5.4321}}
```

#### 使用注解注入

##### 配置包扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
 
    <!--扫描对象-->
    <context:component-scan base-package="com.health.IOC.L_注解开发.b_注入属性"></context:component-scan>
</beans>
```



##### @Autowired：![image-20210111182131125.png](https://i.loli.net/2021/02/05/fvFdecZNbkYChxX.png)

根据属性类型自动装配

##### @Qualifier：![image-20210111182152858.png](https://i.loli.net/2021/02/05/8XmyhvRcMLTwpaW.png)

根据属性名称进行注入（需要和@Autowired一起使用）

##### @Resource：![image-20210111182211833.png](https://i.loli.net/2021/02/05/EWhHJ6AQIDP35ox.png)	

可以根据类型也可以根据属性

##### @Value：

```java
@Value(value = "值")
private String name;
```

注入普通类型属性