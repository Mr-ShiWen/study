# Spring

spring在容器中创建的bean是通过依赖注入才使得它们具有完整的结构。这样使得bean能够被复用，例如一个A类的bean与一个B类的bean可以依赖同一个C类的bean，而普通方式创建的话，它们是依赖不同的C类bean的。在用spring创建bean时，我们是通过注入一个完整的bean来创建另一个完整的bean的。与微服务的思想类似。



普通方式创建bean，则其所有依赖也会被创建出来，可以认为其所有依赖的bean是本bean自身结构的一部分。每次创建普通bean都会创建其所有依赖。

## 使用入门

 1、下载 spring ：

​         （5.2.6 稳定版：https://repo.spring.io/release/org/springframework/spring/5.2.6.RELEASE/）



2、创建普通java工程

![image-20210408151219344](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210408151219344.png)



3、导入 Spring5 相关 jar 包

![coreContainer](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/coreContainer.png)

![springJar包](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/springJar包.png)

![依赖](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/依赖.png)

4、创建普通类，在这个类创建普通方法

```java
package com.atguigu.spring5;

public class User {
    public void add(){
        System.out.println("add....");
    }
}
```



5、创建 Spring 配置文件，在配置文件配置创建的对象

（1）Spring 配置文件使用 xml 格式，先创建 xml 文件，再配置，配置如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置User类对象创建-->
    <bean id="user" class="com.atguigu.spring5.User">
    </bean>
    
</beans>
```



6、进行测试代码编写

```java
public class TestSpring5 {

    @Test
    public void testAdd(){
        //1 加载spring配置文件 ( 以下两种方法都可以加载spring配置文件，仅仅是路径解析规则不同 )
        ApplicationContext context=new ClassPathXmlApplicationContext("bean1.xml");

        ApplicationContext context1 = new FileSystemXmlApplicationContext("D:\\IntelliJ IDEA\\IntelliJ IDEA 2020.1\\spring5_demo1\\src\\bean1.xml");


        //2 获取配置中创建的对象
        User user = context.getBean("user", User.class);

        System.out.println(user);
        user.add();
    }
}
```



## 配置（注解、xml）

spring有两种配置形式，即注解和xml。本质上都是配置，只是表现形式不同。

​		spring里面需要一个原点配置，可以是xml文件，也可以是配置类（全注解开发）。对bean的管理是痛过原点配置出发的，如果需要用到注解配置，还需要在原点配置中配置组件扫描，以把注解配置连接进来。当然注解并非都独立生效，有些注解的生效还需要其他配置配合。

​		例如AOP中 `@Aspect` 的生效还需要开启Aspect

```xml
    <!--开启Aspect，生成代理对象（根据注解设置的增强与被增强关系）-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

​		另外事务注解也需要开启事务注解，事务注解需要事务管理器，事务管理器需要数据库连接池



## IOC

### 基本概念

​	（1）控制反转，把对象创建和对象之间的调用过程，交给 Spring 进行管理

​	（2）使用 IOC 目的：为了耦合度降低

​	（3）做入门案例就是 IOC 实现

### 底层原理

容器：创建所有的bean以及构造所有的依赖关系

具体实现：每遍历一个类时，创建该类bean以及后续所有依赖的bean，并构造依赖关系。这可以通过递归实现。

 	主要技术：xml 解析、工厂模式、反射

![image-20210408212218515](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210408212218515.png)



### BeanFactory 接口

1、IOC 思想基于 IOC 容器完成，IOC 容器底层就是对象工厂



2、Spring 提供 IOC 容器实现两种方式：（两个接口）

​	（1）**BeanFactory**：IOC 容器基本实现，是 Spring 内部的使用接口，不提供开发人员进行使用

​			  *加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象*

   （2）**ApplicationContext**：BeanFactory 接口的子接口，提供更多更强大的功能，一般由开发人员进行使用

​			 *加载配置文件时候就会把在配置文件对象进行创建*

如把 Application 换成 BeanFacotry ，效果是一样的：

```java
public class TestSpring5 {

    @Test
    public void testAdd(){
        //1 加载spring配置文件 ( 以下两种方法都可以加载spring配置文件，仅仅是路径解析规则不同 )
        BeanFactory context=new ClassPathXmlApplicationContext("bean1.xml");
        

        //2 获取配置中创建的对象
        User user = context.getBean("user", User.class);

        System.out.println(user);
        user.add();
    }
}
```



3、ApplicationContext 接口有实现类

![image-20210408214523687](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210408214523687.png)



### 操作 Bean 管理

1、Bean 管理指的是两个操作

​	（1）Spring 创建对象

​	（2）Spirng 注入属性



2、Bean 管理操作有两种方式

​	（1）基于 xml 配置文件方式实现

​	（2）基于注解方式实现



#### 基于xml的bean管理

##### bean创建

![image-20210410164706615](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210410164706615.png)

（1）在 spring 配置文件中，使用 bean 标签，标签里面添加对应属性，就可以实现对象创建

（2）在 bean 标签有很多属性，介绍常用的属性

​		id 属性：唯一标识，相当于对象的身份证

​		class 属性：类全路径（包类路径）

（3）创建对象时候，默认也是执行无参数构造方法完成对象创建



##### 属性注入



######  一、 set 方法注入

   （1）创建类，定义属性和对应的 set 方法

```java
package com.atguigu.spring5.collectiontype;


public class Book {

    private String bname;

    private String bauthor;

    public void setBname(String bname) {
        this.bname = bname;
    }

    public void setBauthor(String bauthor) {
        this.bauthor = bauthor;
    }
    
}

```


（2）在 spring 配置文件配置对象创建，配置属性注入

```xml
    <!--set注入属性-->
    <bean id="book" class="com.atguigu.spring5.Book">
        <!--使用property完成属性注入,name表示类里面的属性名，value表示类里面该属性要赋予的值-->
        <property name="bname" value="易筋经"></property>
        <property name="bauthor" value="达摩老祖"></property>
    </bean>
```



   

  ######   二、 有参构造函数注入

   （1）创建类，定义属性，创建属性对应有参数构造方法

 ```java
package com.atguigu.spring5;

public class Order {
    
    private String oname;
    private String address;

    public Order(String oname, String address) {
        this.oname = oname;
        this.address = address;
    }

}
 ```



（2）在 spring 配置文件配置对象创建，配置属性注入。这里可以体现出

```xml
    <!--有参构造注入属性,name表示形参数名/参数index，value表示实参值-->
    <bean id="order" class="com.atguigu.spring5.Order">
        <constructor-arg name="oname" value="电脑"></constructor-arg>
        <constructor-arg name="address" value="中国"></constructor-arg>
    </bean>

    <!--有参构造注入属性,name表示形参数名/参数index，value表示实参值-->
    <bean id="order" class="com.atguigu.spring5.Order">
        <constructor-arg name="0" value="电脑"></constructor-arg>
        <constructor-arg name="1" value="中国"></constructor-arg>
    </bean>
```



##### 属性注入其他类型值

###### 一、字面量

​	（1）null

```xml
    <!--set注入属性-->
    <bean id="book" class="com.atguigu.spring5.Book">
        <!--使用property完成属性注入,name表示类里面的属性名，value表示类里面该属性要赋予的值-->
        <property name="bname" value="易筋经"></property>
        <property name="bauthor" value="达摩老祖"></property>

        <!--属性设置空值-->
        <property name="address" >
            <null></null>
        </property>

    </bean>
```



​	(2) 属性值包含特殊符号

​			把 < > 进行转义  &lt ;  &gt ;

​			或者把带特殊符号内容写到 CDATA

```xml
    <!--set注入属性-->
    <bean id="book" class="com.atguigu.spring5.Book">
        <!--使用property完成属性注入,name表示类里面的属性名，value表示类里面该属性要赋予的值-->
        <property name="bname" value="易筋经"></property>
        <property name="bauthor" value="达摩老祖"></property>

        <!--属性设置特殊符号-->
        <property name="address">
            <value><![CDATA[ <<南京>> ]]></value>
        </property>

    </bean>
```



###### 二、外部bean

​	（1）创建两个类 service 类和 dao 类 

​	（2）在 service 调用 dao 里面的方法

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.UserDao;
import com.atguigu.spring5.dao.UserDaoImpl;

public class UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void add(){
        System.out.println("service add.......");

        //创建UserDao
        userDao.update();
    }

}
```

​	（3）在 spring 配置文件中进行配置

```xml
    <!--Service和Dao对象创建-->
    <bean id="userService" class="com.atguigu.spring5.service.UserService">
        <!--注入UserDao对象
            name表示属性名
            ref表示对象引用（另一个bean的id）
         -->
        <property name="userDao" ref="userDaoImpl"></property>
    </bean>


    <bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>
```



###### 三、内部bean

​	一对多关系：部门和员工（部门是一，员工是多）。在实体类之间表示一对多关系，员工表示所属部门，使用对象类型属性进行表示

**部门类**

```java
package com.atguigu.spring5.bean;

public class Dept {
    private String dname;

    public void setDname(String dname) {
        this.dname = dname;
    }
}
```



**员工类**

```java
package com.atguigu.spring5.bean;
//员工类
public class Emp {

    private String ename;
    private String gender;
    //员工属于某一个部门
    private Dept dept;

    public void setDept(Dept dept) {
        this.dept = dept;
    }

    public void setEname(String ename) {
        this.ename = ename;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }
}

```



**xml配置**

```xml
    <!--内部bean-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--两个普通属性-->
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>
        <!--对象类型属性-->
        <property name="dept">
            <bean id="dept" class="com.atguigu.spring5.bean.Dept">
                <property name="dname" value="安保部"></property>
            </bean>
        </property>
    </bean>
```

内部bean与外部bean的效果是一样的，可根据实际需要选择内部bean还是外部bean



###### 四、级联赋值

所谓级联赋值，就是对对象类型的属性，赋值其属性（即属性的属性）



第一种写法

```xml
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--两个普通属性-->
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>

        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
    </bean>

    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
        <property name="dname" value="财务部"></property>
    </bean>
```



第二种写法

```xml
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--两个普通属性-->
        <property name="ename" value="lucy"></property>
        <property name="gender" value="女"></property>

        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
        
        <!--下面这个需要有dept的get方法，否则报错。因为需要得到dept才能设置它的dname-->
        <property name="dept.dname" value="技术部"></property>
    </bean>

    <bean id="dept" class="com.atguigu.spring5.bean.Dept"></bean>
```



###### 五、注入集合



创建类，定义数组、list、map、set 类型属性，生成对应 set 方法

```java
package com.atguigu.spring5.collectiontype;

import java.util.List;
import java.util.Map;
import java.util.Set;

public class Stu {
    //1.数组类型
    private String[] courses;

    //2.list集合类型属性
    private List<String> list;

    //3.map集合类型属性
    private Map<String,String> map;

    //4.set集合类型属性
    private Set<String> set;

    public void setCourses(String[] courses) {
        this.courses = courses;
    }

    public void setList(List<String> list) {
        this.list = list;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public void setSet(Set<String> set) {
        this.set = set;
    }
}

```



xml配置

```xml
    <bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">
        
        <!--数组类型属性注入-->
        <property name="courses">
            <array>
                <value>java课程</value>
                <value>mysql课程</value>
            </array>
        </property>

        <!--list类型属性注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>李四</value>
            </list>
        </property>

        <!--map类型属性注入-->
        <property name="map">
            <map>
                <entry key="JAVA" value="java"></entry>
                <entry key="PHP" value="php"></entry>
            </map>
        </property>

        <!--set类型属性注入-->
        <property name="set">
            <set>
                <value>mysql</value>
                <value>redis</value>
            </set>
        </property>
        
    </bean>
```

由上面的学习可知，value标签一般表示String对象，list标签表示List对象，map标签表示Map对象，set标签表示Set对象，entry标签表示Entry对象。当然，这些对象仅仅在设置bean属性的时候用的，本身不是容器里的一个bean。如果需要创建集合类型的Bean，需要先引入命名空间util，详见下面的**注意二**。



**注意一：集合元素是对象时**

```xml
    <bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">

        <!--元素是对象的List注入-->
        <property name="courseList">
            <list>
                <ref bean="course1"></ref>
                <ref bean="course2"></ref>
            </list>
        </property>

    </bean>

    <bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="Spring5框架"></property>
    </bean>

    <bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="MyBatis框架"></property>
    </bean>
```



**注意二：即创建集合的bean**

（1）在 spring 配置文件中引入名称空间 util

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">


</beans>
```



（2）使用 util 标签完成 list 集合注入提取

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

    <!--1 创建list集合的bean，可以被其他bean引用-->
    <util:list id="bookList">
        <value>易筋经</value>
        <value>九阳神功</value>
        <value>九阴真经</value>
    </util:list>

    <!--2 list集合属性注入，引用外部bean-->
    <bean id="book" class="com.atguigu.spring5.collectiontype.Book">
        <property name="list" ref="bookList"></property>
    </bean>

</beans>
```



##### FactoryBean

Spring 有两种类型 bean，一种普通 bean，另外一种工厂 bean（FactoryBean）

​        普通 bean：在配置文件中定义 bean 类型就是返回类型

​		工厂 bean：在配置文件定义 bean 类型可以和返回类型不一样
​				第一步 创建类，让这个类作为工厂 bean，**实现接口 FactoryBean**
​				第二步 实现接口里面的方法，在实现的方法中定义返回的 bean 类型

​		**普通bean是按配置得到bean，工厂bean是按配置得到bean后，调用其getObject（）得到产品对象，以这个产品对象作为最终返回结果。**

​        **FactoryBean与普通bean的配置一样的，容器里装的也确实FactoryBean本身，但通过ApplicationContext得到时，得到的是FactoryBean的产品对象。**



定义一个工厂bean类

```java
package com.atguigu.spring5.factorybean;

import com.atguigu.spring5.collectiontype.Course;
import org.springframework.beans.factory.FactoryBean;

public class MyBean implements FactoryBean<Course> {

    //定义返回bean
    @Override
    public Course getObject() throws Exception {
        Course course=new Course();
        course.setCname("abc");
        return course;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }

}
```



xml配置

```xml
    <bean id="myBean" class="com.atguigu.spring5.factorybean.MyBean">

    </bean>
```



得到工厂bean的产品对象（即getObject（）返回的对象）

```java
    public void testCollection3(){

        ApplicationContext context=new ClassPathXmlApplicationContext("bean3.xml");

        //这里不是得到MyBean类对象，而是MyBean类对象的产品对象
        Course myBean = context.getBean("myBean", Course.class);

        System.out.println(myBean);

    }
```



##### Bean作用域

​		在 Spring 里面，设置创建 bean 实例是单实例还是多实例。默认情况下，bean 是单实例对象。

​		在 spring 配置文件 bean 标签里面有 scope 属性用于设置单实例还是多实例。

​				singleton，表示是单实例对象（即全局作用域）

​				prototype，表示是多实例对象（即单点作用域）

​				request，每次请求一个bean，不同请求不同的bean

​				session，每个session域一个bean，不同session域不同的bean

​     一般来说，最常用的是singleton、prototype

​        区别：

​				设置 scope 值是 singleton 时候，加载 spring 配置文件时候就会创建单实例对象。

​				设置 scope 值是 prototype 时候，不是在加载 spring 配置文件时候创建 对象，在调用getBean 方法时候创建多实例对象。



##### Bean的生命周期

从对象创建到对象销毁的过程

###### 无后置处理器过程：

​	（1）通过构造器创建 bean 实例（无参数构造）

​	（2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）

​	*（3）调用 bean 的初始化的方法（需要进行配置初始化的方法）*

​	（4）bean 可以使用了（对象获取到了）

​	*（5）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）*

bean的java代码：

```java
package com.atguigu.spring5.bean;

public class Orders {

    private String oname;

    //2.set方法
    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("2.调用set，设置属性");
    }

    //1.无参构造
    public Orders() {
        System.out.println("1.执行无参构造，创建bean实例");
    }

    //3.执行初始化
    public void initMethod(){
        System.out.println("3.执行初始化方法");
    }

    //5.执行销毁方法
    public void destroyMethod(){
        System.out.println("5.执行销毁方法");
    }
}

```



xml配置（注意，初始化和销毁方法都需要配置）：

```xml
    <bean id="orders" class="com.atguigu.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
        <property name="oname" value="手机"></property>
    </bean>
```



###### 有后置处理器过程：

​		（1）通过构造器创建 bean 实例（无参数构造）

​		（2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）

 		**（3.pre) 把 bean 实例传递 bean 后置处理器的方法 postProcessBeforeInitialization**

​		*（3）调用 bean 的初始化的方法（需要进行配置初始化的方法）*

​    	**（3.after) 把 bean 实例传递 bean 后置处理器的方法 postProcessAfterInitialization**

​		（4）bean 可以使用了（对象获取到了）

​		*（5）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）*



操作过程：

  （1）创建类，实现接口 BeanPostProcessor，创建后置处理器

```java
package com.atguigu.spring5.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.lang.Nullable;


public class MyBeanPost implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化前置方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化后置方法");
        return bean;
    }
}

```



（2）后置处理器 xml 配置

```xml
    <!--配置后置处理器-->
    <bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
```

由于这个bean实现了 BeanPostProcessor 接口，因此 spring 知道它是个后置处理器，不需要再特殊声明。

配置了后置处理器后，所有bean的生命周期都会加入后置处理器的处理逻辑，即在初始化前后都经过后置处理器处理。



##### xml自动装配

​		根据指定装配规则（属性名称或者属性类型），Spring 自动将匹配的属性值进行注入。

Emp的java代码：

```java
package com.atguigu.spring5.autowire;

public class Emp {

    private Dept dept;

    public void setDept(Dept dept) {
        this.dept = dept;
    }

}

```



Dept的java代码：

```java
package com.atguigu.spring5.autowire;

public class Dept {

}

```



根据名称自动注入（属性名称要和需要注入的bean的id相同）：

```xml
    <bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byName">
    </bean>

    <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
```



根据类型自动注入（目标类型的bean不能有多个，否则由于不知道要注入哪个会报错）：

```xml
    <bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byType">
    </bean>

    <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
```



##### 外部属性文件

方法1、直接配置数据库信息

​	（1）配置德鲁伊连接池

​	（2）引入德鲁伊连接池依赖 jar 包

xml配置：

```xml
    <!--直接配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
```



方法2、引入外部属性文件配置数据库连接池

（1）创建外部属性文件，properties 格式文件，写数据库信息

![image-20210412214203024](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210412214203024.png)



（2）把外部 properties 属性文件引入到 spring 配置文件中

​     先引入 context 名称空间，具体和引入util名称空间一样

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    
</beans>
```



（3）引入外部属性文件

```xml
   <!--引入外部属性文件，即把外部属性文件的键值对放入context的域中-->
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>

    <!--通过el表达式，引入属性值，表示式是外部属性中的key，以期得到对应的value-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.passWord}"></property>
    </bean>
```



#### 基于注解的bean管理

##### 注解

1、概念

​	（1）注解是代码特殊标记，格式：@注解名称(属性名称=属性值, 属性名称=属性值..)

​	（2）使用注解，注解作用在类上面，方法上面，属性上面

​	（3）使用注解目的：简化 xml 配置



2、Spring 针对 Bean 管理中创建对象提供注解

​	（1）@Component

​	（2）@Service

​	（3）@Controller

​	（4）@Repository

上面四个注解功能是一样的，都可以用来创建 bean 实例（当然，为了可读性，建议controller注解用于web层、service注解用于service层，repository注解用于dao层）



##### bean创建

1、引入aop依赖

![image-20210413103611687](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210413103611687.png)

2、第二步 开启组件扫描（需要在xml里配置所要扫描的包，这里需要先引入context名称空间）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启组件扫描
       1、如果要扫描多个包，多个包之间用逗号隔开
       2、或者指定上层的包，那这个上层包下的所有包都会被扫描
     -->
    <context:component-scan base-package="com.atguigu.spring5.dao,com.atguigu.spring5.service"></context:component-scan>


</beans>
```



3、 创建类，在类上面添加创建对象注解

```java
package com.atguigu.spring5.service;

import org.springframework.stereotype.Component;

//等价于<bean id="userService class="com.atguigu.spring5.service.UserService" >
//注解的value属性值就是bean的id，可以不写，默认是类名称(首字母小写)

@Component(value = "userService")
public class UserService {

    public void add(){
        System.out.println("service add ... ");
    }

}
```



注意，组件扫描的细节设置

```xml
    <!--示例1
        use-default-filters="false"  表示不使用默认filter，即不全部扫描，需要自己设置扫描内容
        context:include-filter  设置扫描哪些内容 (如只扫描带Controller的类)
    -->
    <context:component-scan base-package="com.atguigu" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--示例2
   	 	全部扫描，可以配合 exclude-filter 来排除某些不扫描的类
    	context:exclude-filter  设置不扫描哪些内容 (如不扫描带Controller的类)
	-->
    <context:component-scan base-package="com.atguigu">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```





##### 属性注入

​		**通过注解进行属性注入不需要 set 方法，因为底层是通过反射进行依赖注入的**

###### @Autowired：

根据属性类型进行自动装配

第一步 把 service 和 dao 对象创建，在 service 和 dao 类添加创建对象注解

第二步 在 service 注入 dao 对象，在 service 类添加 dao 类型属性，在属性上面使用注解

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

//等价于<bean id="userService class="com.atguigu.spring5.service.UserService" >
//注解的value属性值就是bean的id，可以不写，默认是类名称(首字母小写)

@Service(value = "userService")
public class UserService {

    //定义dao属性
    //不需要添加set方法
    //添加注入属性注解
    @Autowired
    private UserDao userDao;

    public void add(){
        System.out.println("service add ... ");
        userDao.add();
    }

}

```





###### @Qualifier：

根据名称进行注入

这个@Qualifier 注解的使用，和上面@Autowired 一起使用

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

//等价于<bean id="userService class="com.atguigu.spring5.service.UserService" >
//注解的value属性值就是bean的id，可以不写，默认是类名称(首字母小写)

@Service(value = "userService")
public class UserService {

    //定义dao属性
    //不需要添加set方法
    //添加注入属性注解
    @Autowired  //根据类型注入
    @Qualifier(value = "userDaoImpl1") //在目标类型的所有bean中，根据名称注入
    private UserDao userDao;

    public void add(){
        System.out.println("service add ... ");
        userDao.add();
    }

}

```



###### @Resource：

可以根据类型注入，可以根据名称注入

根据类型注入

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

//等价于<bean id="userService class="com.atguigu.spring5.service.UserService" >
//注解的value属性值就是bean的id，可以不写，默认是类名称(首字母小写)

@Service(value = "userService")
public class UserService {

    @Resource  //根据类型注入
    private UserDao userDao;

    public void add(){
        System.out.println("service add ... ");
        userDao.add();
    }

}
```



根据名称注入

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

//等价于<bean id="userService class="com.atguigu.spring5.service.UserService" >
//注解的value属性值就是bean的id，可以不写，默认是类名称(首字母小写)

@Service(value = "userService")
public class UserService {

    @Resource(name = "userDaoImpl1")  //根据名字注入
    private UserDao userDao;

    public void add(){
        System.out.println("service add ... ");
        userDao.add();
    }

}
```



###### @Value：

注入普通类型属性

```java
package com.atguigu.spring5.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

//等价于<bean id="userService class="com.atguigu.spring5.service.UserService" >
//注解的value属性值就是bean的id，可以不写，默认是类名称(首字母小写)

@Service(value = "userService")
public class UserService {

    @Value(value = "abc")
    private String name;
    

    public void add(){
        System.out.println("service add ... "+name);
    }

}

```





##### 完全注解开发

（1）创建配置类，替代 xml 配置文件

```java
package com.atguigu.spring5.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration  //作为配置类，替代xml文件
@ComponentScan(basePackages = {"com.atguigu"})  //配置扫描注解
public class SpringConfig {

}
```



（2）编写测试类

```java
package com.atguigu.spring5.testDemo;

import com.atguigu.spring5.config.SpringConfig;
import com.atguigu.spring5.service.UserService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class TestSpring5Demo1 {

    @Test
    public void testCollection2(){

        ApplicationContext context=new AnnotationConfigApplicationContext(SpringConfig.class);

        UserService userService = context.getBean("userService", UserService.class);

        System.out.println(userService);

        userService.add();

    }

}
```



**注意：**完全注解开发多了一种bean创建方式，那就是在配置类下，用 @Bean 标记方法，表示通过此方法创建一个bean，方法的参数可以是其他bean，按照bean名字来对应即可。



### 扩展接口

ioc容器提供了扩展接口给我们，使得在ioc容器创建所有bean完毕后，调用接口。bean可以实现这些接口。详见：

https://blog.csdn.net/weixin_41627757/article/details/103802890

https://www.jianshu.com/p/4bd3f68cb179



## AOP

### 基本概念

（1）面向切面编程（方面），利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

（2）通俗描述：不通过修改源代码方式，在主干功能里面添加新功能。

（3）使用登录例子说明 AOP。

![image-20210413211125699](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210413211125699.png)



### 大致实现

ioc创建bean，增强bean里面有增强信息（切点+通知）。通过开启aspect来生成相应的代理bean，从而实现所描述的增强。



### 底层原理

#### JDK动态代理

有接口情况，使用 JDK 动态代理。创建接口实现类代理对象，增强类的方法

![image-20210413211736023](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210413211736023.png)







#### CGLIB动态代理

没有接口情况，使用 CGLIB 动态代理。创建子类的代理对象，增强类的方法

![image-20210413212537417](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210413212537417.png)



#### 原理

![动态代理](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/动态代理.jpg)





### 相关术语

#### 1、连接点

类里可以被增强的方法，可以叫做连接点。



#### 2、切入点

实际被增强的方法，称为切入点。切入点也可以理解为描述被增强的连接点的模式，即用于筛选哪些连接点来做增强处理。



#### 3、通知

实际增加的逻辑部分称为通知或增强

通知有多种类型：

​		前置通知（@Before，方法之前执行）

​		后置通知（@After，方法之后执行，无论有没有异常）

​		环绕通知（@Around，方法之前，方法之后执行，需要用到ProceedingJoinPoint）

​		异常通知（@AfterThrowing，有异常才执行，没异常就不执行）

​		返回通知（@AfterReturning，有异常就不执行）



#### 4、切面

通知与切入点的结合，通知说明了干什么，切入点说明了在哪干。



### 操作

#### 准备

1、Spring 框架一般都是基于 AspectJ 实现 AOP 操作
 （1）AspectJ 不是 Spring 组成部分，独立 AOP 框架，一般把 AspectJ 和 Spirng 框架一起使
用，进行 AOP 操作



2、基于 AspectJ 实现 AOP 操作
	（1）基于 xml 配置文件实现
	（2）基于注解方式实现（推荐）



3、在项目工程里面引入 AOP 相关依赖

​		`com.springsource.net.sf.cglib-2.2.0.jar`

​		`com.springsource.org.aopalliance-1.0.0.jar`

​		`com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar`

​		`spring-aspects-5.2.6.RELEASE.jar`



4、切入点表达式

​		（1）切入点表达式作用：知道对哪个类里面的哪个方法进行增强

​		（2）语法结构： execution(   [权限修饰符]   [返回类型]   [类全路径].[方法名称] ([参数列表])   )，其中返回类型可以不写

​		**（3）多个切入点可以用 `||` 连接起来，如 `execution(xxx)||execution(yyy)`**



举例 1：对 com.atguigu.dao.BookDao 类里面的 add 进行增强

execution(* com.atguigu.dao.BookDao.add(..) )



举例 2：对 com.atguigu.dao.BookDao 类里面的所有的方法进行增强

execution( * com.atguigu.dao.BookDao.* (..) )



举例 3：对 com.atguigu.dao 包里面所有类，类里面所有方法进行增强

execution( * com.atguigu.dao.*.* (..) )



关于切入点表达式的具体语法：https://blog.csdn.net/qq_39720594/article/details/105321664



#### 基于AspectJ的注解操作

##### 1、创建被增强类

```java
package com.atguigu.spring5.aopanno;

//被增强类
@Component
public class User {
    public void add(){
        System.out.println("add...");
    }
}
```



##### 2、创建增强类（编写增强逻辑）

在增强类里面，创建方法，让不同方法代表不同通知类型。增强需要从类对类的角度出发理解，而非从对象的角度理解。

@Component  //创建bean
@Aspect   //用于标记这是一个增强类，否则spring不认为这是增强类

@通知类型（value=“哪一类的哪一个方法”），value指明被增强的某一类对象的某方法





```java
package com.atguigu.spring5.aopanno;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

//增强类
@Component //创建bean，实际是通过【增强对象】来增强【被增强对象】的
@Aspect //表明含有切面（通知+切点），ioc可用来做增强
public class UserProxy {

    //@Before注解表示作为前置通知
    @Before(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void before(){
        System.out.println("before...");
    }
    
	//@After注解表示作为后置通知
    @After(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void after(){
        System.out.println("after...");
    }
    
    //@AfterReturning注解表示作为返回通知
    @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterReturning(){
        System.out.println("afterReturning...");
    }

    //@AfterThrowing注解表示作为异常通知
    @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterThrowing(){
        System.out.println("afterThrowing...");
    }
    
    //@Around注解表示作为环绕通知
    @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前...");
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后...");
    }

}

```



##### 3、进行xml配置

（1）在 spring 配置文件中，开启注解扫描（需要引入context、aop名称空间）、开启Aspect（生成代理对象）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启组件扫描，以生成bean，获取增强与被增强关系-->
    <context:component-scan base-package="com.atguigu.spring5.aopanno">			</context:component-scan>

    <!--开启Aspect，生成代理对象（根据注解设置的增强与被增强关系）-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

</beans>
```



##### 注意：

###### 通知的执行顺序

```java
try{
    try{
        //环绕前
        //@Before
        method.invoke(..);
        //环绕后
    }finally{
        //@After
    }
    
    //@AfterReturning
    
}catch(){
    //@AfterThrowing
}
```



###### 相同切入点抽取

通过 `@Point` 注解把相同切入点抽取出来，如下例子所示：value = "point()" 表示切入点与point()方法相同。当然point（）方法可以是自定义的任意方法，不会被绑定到目标方法上。

```java
package com.atguigu.spring5.aopanno;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

//增强类
@Component
@Aspect //生成代理对象
public class UserProxy {

    @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void point(){

    }

    //@Before注解表示作为前置通知
    @Before(value = "point()")
    public void before(){
        System.out.println("before...");
    }

    @After(value = "point()")
    public void after(){
        System.out.println("after...");
    }

    @AfterReturning(value = "point()")
    public void afterReturning(){
        System.out.println("afterReturning...");
    }

    @AfterThrowing(value = "point()")
    public void afterThrowing(){
        System.out.println("afterThrowing...");
    }

    @Around(value = "point()")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前...");
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后...");
    }

}
```



###### 一个通知多个切点

通常利用或逻辑连接多个切点，如下:

或逻辑连接多个execution

```java
    @Before(value ="execution(* com.atguigu.spring5.aopanno.Printer.sayHello(..))||execution(* com.atguigu.spring5.aopanno.Printer.happy(..))" )
    public void before(){
        System.out.println("before....");
    }
```



或逻辑连接多个公共pointCut

```java
   @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.Printer.sayHello(..))")
    public void pointCut(){

    }



    @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.Printer.happy(..))")
    public void pointCut1(){

    }

	@Before(value = "pointCut()||pointCut1()")
    public void before(){
        System.out.println("before....");
    }
```







###### 增强类优先级

有多个增强类多同一个方法进行增强，设置增强类优先级

在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高

```java
@Component
@Aspect
@Order(1)
public class PersonProxy {

    @Before(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void before(){
        System.out.println("person before...");
    }

}

//增强类
@Component
@Aspect //生成代理对象
@Order(3)
public class UserProxy {

    //@Before注解表示作为前置通知
    @Before(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void before(){
        System.out.println("before...");
    }
    
}
```



##### 完全注解开发

```java
package com.atguigu.spring5.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration  //作为配置类，代替xml
@ComponentScan(value = "com.atguigu")  //开启组件扫描
@EnableAspectJAutoProxy(proxyTargetClass = true)  //开启Aspect，生成代理对象，实现增强
public class ConfigAop {

}
```





#### 基于AspectJ的xml操作

配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--创建对象-->
    <bean id="book" class="com.atguigu.spring5.aopxml.Book"></bean>
    <bean id="bookProxy" class="com.atguigu.spring5.aopxml.BookProxy"></bean>

    <!--配置增强-->
    <aop:config>

        <!--切入点-->
        <aop:pointcut id="p" expression="execution(* com.atguigu.spring5.aopxml.Book.buy())"/>

        <!--配置切面-->
        <aop:aspect ref="bookProxy">
            <!--配置增强作用的具体位置-->
            <aop:before method="before" pointcut-ref="p"></aop:before>
            <aop:after method="after" pointcut-ref="p"></aop:after>
        </aop:aspect>

    </aop:config>
    
</beans>
```



#### 异同（注解式AOP、xml式AOP）

​		无论注解式AOP，还是xml式AOP。增强描述的都是对象与对象之间的关系，而非类与类的关系。因此增强的具体情况与对象的具体情况密切相关，这是通过动态代理实现的。



![AOP](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/AOP-1618801653962.jpg)





## JdbcTemplate

### 概念

Spring 框架对 JDBC 进行封装，使用 JdbcTemplate 方便实现对数据库操作



### 准备

#### 1、引入相关依赖

![image-20210414204148242](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210414204148242.png)





#### 2、配置数据库连接池

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--数据库连接池的配置-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3308/user_db"></property>
        <property name="username" value="root"></property>
        <property name="password" value="199402160579shch"></property>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    </bean>

</beans>
```



#### 3、配置 JdbcTemplate 对象，注入数据库连接池

```xml
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource"></property>
     </bean>
```



#### 4、创建 service、 dao 类，在 dao 注入 JdbcTemplate 对象

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.BookDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BookService {

    //注入dao
    @Autowired
    private BookDao bookDao;
}


@Repository
public class BookDaoImpl implements BookDao{

    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;

}
```



xml配置

```xml
    <!--开启组件扫描-->
    <context:component-scan base-package="com.atguigu"></context:component-scan>
```



### 操作数据库

#### 1、创建实体类

```java
package com.atguigu.spring5.entity;

public class User {
    String userId;
    String username;
    String userstatus;

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getUserstatus() {
        return userstatus;
    }

    public void setUserstatus(String userstatus) {
        this.userstatus = userstatus;
    }
}

```



#### 2、编写 service 和 dao

（1）在 dao 进行数据库添加操作



#### 3、利用JdbcTemplate实现增删改查

##### 增删改

```java
package com.atguigu.spring5.dao;

import com.atguigu.spring5.entity.Book;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class BookDaoImpl implements BookDao{

    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;

    //添加
    @Override
    public void addBook(Book book) {
        String sql="insert into t_book (user_id, username, userstatus) VALUES (?,?,?);";
        int update = jdbcTemplate.update(sql, book.getUserId(), book.getUsername(), book.getUserstatus());
        System.out.println(update);
    }

    //修改
    @Override
    public void update(Book book) {
        String sql="update t_book set username=?,userstatus=? where user_id=?";
        int update = jdbcTemplate.update(sql, book.getUsername(), book.getUserstatus(), book.getUserId());
        System.out.println(update);
    }

    //删除
    @Override
    public void delete(String userId) {
        String sql="delete from t_book where user_id=?;";
        int update = jdbcTemplate.update(sql, userId);
        System.out.println(update);
    }

}

```



##### 查

###### 查询返回值（记录数）

​		queryForObject（）有两个参数： sql 语句、返回类型 Class

```java
    //查询表中记录数
    @Override
    public int selectCount() {
        String sql="select count(*) from t_book;";
        Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
        return count;
    }
```



###### 查询返回对象（一行数据）

​		queryForObject（）有三个参数：sql 语句、RowMapper接口、占位符值（变长参数）。

RowMapper 是接口，针对返回不同类型数据，使用这个接口里面实现类完成数据封装

```java
    //查询表中一行
    @Override
    public Book findBookInfo(String id) {
        String sql="select user_id as userId,username,userstatus from t_book where user_id=?";
        Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Book.class), id);
        return book;
    }
```



###### 查询返回集合

​		query（）有三个参数：sql 语句、RowMapper接口、sql 语句值（变长参数）。

RowMapper 是接口，针对返回不同类型数据，使用这个接口里面实现类完成数据封装

```java
    //查询表中多行
    @Override
    public List<Book> findAllBook() {
        String sql="select * from t_book";
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Book.class));
        return bookList;
    }
```



###### 注意

JdbcTemplate的单对象查询与多对象查询差别体现在方法名上

```java
//单对象查询
jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Book.class), args);

//多对象查询
jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Book.class),args);

```





QueryRunner（属于DButils包）的单对象查询与多对象查询差别体现在方法的参数上。

通过自定义的BaseDao可以封装出JdbcTemplate的效果

```java
//单对象查询
query(conn,sql,new BeanHandler<T>(type),args);
    
//多对象查询
query(conn,sql,new BeanListHandler<T>(type),args);
```



##### 批量操作

###### 批量操作的方法函数

​		batchUpdate（）有两个参数：sql语句、List<Object[]> batchArgs

​		通过sql语句（带占位符）执行，batchArgs的每一个数组元素代表一次占位符的值。

​		因此底层里是通过遍历 batchArgs 的所有数组，每个数组里的值作为sql语句的占位符的值执行一次。



###### 批量增

```java
    //批量增加
    @Override
    public void batchAddBook(List<Object[]> batchArgs) {
        String sql="insert into t_book (user_id, username, userstatus) VALUES (?,?,?)";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

batchArgs如下：

```java
        List<Object[]> batchArgs=new ArrayList<>();

        Object[] o1={"4","JavaScript","a"};
        Object[] o2={"5","JSP","a"};
        Object[] o3={"6","MySql","a"};

        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
```





###### 批量删

```java
    //批量删除
    @Override
    public void batchDeleteBook(List<Object[]> batchArgs) {
        String sql="delete from t_book where user_id=?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

batchArgs如下：

```java
        List<Object[]> batchArgs=new ArrayList<>();

        Object[] o1={"4"};
        Object[] o2={"5"};
        Object[] o3={"6"};

        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
```





###### 批量改

```java
    //批量修改
    @Override
    public void batchUpdateBook(List<Object[]> batchArgs) {
        String sql="update t_book set  username=?, userstatus=? where user_id=?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

batchArgs如下：

```java
        List<Object[]> batchArgs=new ArrayList<>();

        Object[] o1={"JavaScript888","a","4"};
        Object[] o2={"JSP888","a","5"};
        Object[] o3={"MySql888","a","6"};

        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
```



#### 4、事务

事务是数据库操作最基本单元，逻辑上一组操作，要么都成功，如果有一个失败所有操作都失败。

事务四个特性（ACID）

（1）原子性

（2）一致性

（3）隔离性

（4）持久性



##### 非事务操作

![image-20210415204313487](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210415204313487.png)

1、创建数据库表，添加记录

![image-20210415204341190](D:%5Ctypora%5Cmarkdown%E5%9B%BE%E7%89%87%5Cimage-20210415204341190.png)



2、创建 service，搭建 dao，完成对象创建和注入关系

​		service 注入 dao，在 dao 注入 JdbcTemplate，在 JdbcTemplate 注入 DataSource

```java
@Service
public class UserService {

    //注入dao
    @Autowired
    private UserDao userDao;
    
}

@Repository
public class UserDaoImpl implements UserDao {

    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;

}
```



3、在 dao 创建两个方法：多钱和少钱的方法，在 service 创建方法（转账的方法）

dao:

```java
package com.atguigu.spring5.dao;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class UserDaoImpl implements UserDao {

    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;

    //多钱
    @Override
    public void addMoney() {
        String sql="update t_account set money=money+? where username=? ";
        jdbcTemplate.update(sql,100,"mary");
    }

    //少钱
    @Override
    public void reduceMoney() {
        String sql="update t_account set money=money-? where username=? ";
        jdbcTemplate.update(sql,100,"lucy");
    }
}

```



service：

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    //注入dao
    @Autowired
    private UserDao userDao;

    //转账方法
    public void accountMoney(){

        //lucy少100
        userDao.reduceMoney();

        //mary多100
        userDao.addMoney();

    }
}
```



上面代码，如果正常执行没有问题的，但是如果代码执行过程中出现异常，就有问题。因此需要进行事务操作。

```java
    //转账方法
    public void accountMoney(){

        try {
            //开启事务

            //进行业务操作
            userDao.reduceMoney(); //lucy少100
            userDao.addMoney();//mary多100

            //事务提交

        }catch (Exception e){
            //事务回滚

        }

    }
```



##### 事务管理

1、事务添加到 JavaEE 三层结构里面 Service 层（业务逻辑层）

2、在 Spring 进行事务管理操作
		有两种方式：编程式事务管理（不推荐）和声明式事务管理（推荐）

3、声明式事务管理
	（1）基于注解方式（使用）
	（2）基于 xml 配置文件方式

4、**在 Spring 进行声明式事务管理，底层使用 AOP 原理，即spring的事务管理功能是通过AOP实现的**



###### Spring事务管理API



提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类

![image-20210415213415342](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210415213415342.png)



###### 1、注解的声明式事务管理

（1）在 spring 配置文件配置事务管理器

```xml
    <!--数据库连接池的配置-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3308/user_db"></property>
        <property name="username" value="root"></property>
        <property name="password" value="199402160579shch"></property>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    </bean>

    <!--创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源，也就是数据库连接池-->
        <property name="dataSource" ref="dataSource"></property>
     </bean>
```



（2）在 spring 配置文件，开启事务注解（需要引入名称空间 tx）

```xml

     <!--开启事务注解-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>

```



（3）在 service 类上面（或者 service 类里面方法上面）添加事务注解

​			@Transactional，这个注解添加到类上面，也可以添加方法上面
​					如果把这个注解添加类上面，这个类里面所有的方法都添加事务
​					如果把这个注解添加方法上面，为这个方法添加事务

```java
package com.atguigu.spring5.service;

import com.atguigu.spring5.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class UserService {

    //注入dao
    @Autowired
    private UserDao userDao;

    //转账方法
    public void accountMoney(){

        userDao.reduceMoney(); //lucy少100

        int i=10/0; //模拟异常

        userDao.addMoney();//mary多100

    }

}

```



###### 2、完全注解的声明式事务管理

与注解的方式类似，唯一不同的地方是使用配置类来代替配置文件，配置类如下：

```java
package com.atguigu.spring5.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

@Configuration  //配置类，替代xml文件
@ComponentScan(basePackages = "com.atguigu")  //开启组件扫描
@EnableTransactionManagement  //开启事务管理，自动在IOC容器找到事务管理器对象
public class TxConfig {

    //创建数据库连接池
    @Bean
    public DruidDataSource getDruidDataSource(){

        DruidDataSource druidDataSource=new DruidDataSource();
        druidDataSource.setDriverClassName("com.mysql.jdbc.Driver");
        druidDataSource.setUrl("jdbc:mysql://localhost:3308/user_db");
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("199402160579shch");

        return druidDataSource;
    }

    //创建 JdbcTemplate 对象
    //实参将从IOC容器获取，就是前面创建的DruidDataSource对象
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource){
        JdbcTemplate jdbcTemplate=new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);

        return jdbcTemplate;
    }

    //创建事务管理器
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource){
        DataSourceTransactionManager dataSourceTransactionManager=new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }


}

```







###### 3、xml的声明式事务管理

第一步 配置事务管理器

```xml
    <!--创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"></property>
     </bean>
```

第二步 配置通知

```xml
    <!--配置通知-->
    <tx:advice id="txadvice" transaction-manager="transactionManager">

        <!--配置事务参数-->
        <tx:attributes>
            <!--指定在哪种规则上添加事务，功能类似于切点-->
            <tx:method name="accountMoney" propagation="REQUIRED"/>  <!--名称为accountMoney的所有方法-->
            <tx:method name="account*" propagation="REQUIRED"/> <!--名称前缀为account的所有方法-->
        </tx:attributes>

    </tx:advice>
```

第三步 配置切入点和切面

```xml
    <!--配置切入点与切入面-->
    <aop:config>

        <!--配置切入点-->
        <aop:pointcut id="pt" expression="execution(* com.atguigu.spring5.service.UserService.*(..))"/>

        <!--配置切面-->
        <aop:advisor advice-ref="txadvice" pointcut-ref="pt"></aop:advisor>

    </aop:config>
```





##### @Transactional  注解的参数配置

![image-20210416090702128](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416090702128.png)



###### 1、propagation：事务传播行为

​		多事务方法直接进行调用，这个过程中事务 是如何进行管理的。

![image-20210416093524967](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416093524967.png)



具体看：https://blog.csdn.net/weixin_39625809/article/details/80707695

重点掌握  REQUIRED 与 REQUIRED_NEW

设置：

```java
@Transactional(propagation = Propagation.REQUIRED)
```





###### 2、isolation：隔离级别

![image-20210416095906964](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416095906964.png)

设置：

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```



###### 3、timeout：超时时间

​		事务需要在一定时间内进行提交，如果不提交进行回滚。默认值是 -1 ，设置时间以秒单位进行计算。

```java
@Transactional(timeout = 5)
```



###### 4、readOnly：是否只读

​		readOnly 默认值 false，表示可以查询，可以添加修改删除操作

​		设置 readOnly 值是 true，设置成 true 之后，只能查询



###### 5、rollbackFor：回滚

​		设置出现哪些异常进行事务回滚



###### 6、noRollbackFor：不回滚

​		设置出现哪些异常不进行事务回滚



## spring5 新功能

### 1、java8

整个 Spring5 框架的代码基于 Java8，运行时兼容 JDK9，许多不建议使用的类和方
法在代码库中删除



### 2、日志

Spring 5.0 框架自带了通用的日志封装

​	（1）Spring5 已经移除 Log4jConfigListener，官方建议使用 Log4j2

​	（2）Spring5 框架整合 Log4j2



​    第一步 引入 jar 包：

   ![image-20210416160028238](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416160028238.png)



​	第二步 创建 log4j2.xml 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration 后面的 status 用于设置 log4j2 自身内部的信息输出，可以不设置，
当设置成 trace 时，可以看到 log4j2 内部各种详细输出-->
<configuration status="debug">

    <!--先定义所有的 appender-->
    <appenders>
        <!--输出日志信息到控制台-->
        <console name="Console" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </console>
        
    </appenders>

    <!--然后定义 logger，只有定义 logger 并引入的 appender，appender 才会生效-->
    <!--root：用于指定项目的根日志，如果没有单独指定 Logger，则会使用 root 作为默认的日志输出-->
    <loggers>
        <root level="info">
            <appender-ref ref="Console"/>
        </root>
    </loggers>

</configuration>

```



### 3、@Nullable 注解

@Nullable 注解可以使用在方法上面，属性上面，参数上面。表示方法返回可以为空，属性值可以

为空，参数值可以为空。



注解用在方法上面，方法返回值可以为空

```java
    @Nullable
    String getId();
```



注解使用在方法参数里面，方法参数可以为空

```java
    public <T> void registerBean(@Nullable String beanName, Class<T> beanClass, @Nullable Supplier<T> supplier, BeanDefinitionCustomizer... customizers) {
        this.reader.registerBean(beanClass, beanName, supplier, customizers);
    }
```



注解使用在属性上面，属性值可以为空

```java
    @Nullable
    private String bookName;
```



### 4、函数式风格

Spring5 核心容器支持函数式风格 GenericApplicationContext

```java
 //1、创建GenericApplicationContext对象
        GenericApplicationContext context=new GenericApplicationContext();

        //2、调用context方法注册对象
        context.refresh();

        context.registerBean("user1",User.class,()->new User());

        //3、获取注册的对象
//        User user = (User) context.getBean("com.atguigu.spring5.test.User");  bean没有命名时才可以使用这个方法
        User user1 = (User) context.getBean("user1");
        System.out.println(user1);
```



### 5、测试（JUnit5）

#### （1）整合 JUnit4

  第一步 引入 Spring 相关针对测试依赖

![image-20210416202237749](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416202237749.png)



![image-20210416202248493](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416202248493.png)





第二步 创建测试类，使用注解方式完成

```java
package com.atguigu.spring5.test;

import com.atguigu.spring5.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)  //单元测试框架
@ContextConfiguration("classpath:bean1.xml")  //加载配置文件

public class JTest4 {

    @Autowired
    private UserService userService;

    @Test
    public void test1(){
        userService.accountMoney();
    }

}
```

注意：@ContextConfiguration("classpath:bean1.xml")  ，加载配置文件得到的 IOC 容器中不含有测试类本身的bean



#### （2）整合 JUnit5

​	第一步 引入 JUnit5 的 jar 包

![image-20210416204250658](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416204250658.png)



​	第二步 创建测试类，使用注解完成

```java
package com.atguigu.spring5.test;


import com.atguigu.spring5.service.UserService;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.TestExecutionListeners;
import org.springframework.test.context.junit.jupiter.SpringExtension;

@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:bean1.xml") //加载配置文件

public class JTest5 {

    @Autowired
    private UserService userService; //由上边的加载配置文件得到容器，在容器里找到相应的bean注入

    @Test
    public void test1(){
        userService.accountMoney();
    }


}

```

注意：@ContextConfiguration("classpath:bean1.xml")  ，加载配置文件得到的 IOC 容器中不含有测试类本身的bean



使用一个复合注解替代上面两个注解完成整合

```java
package com.atguigu.spring5.test;


import com.atguigu.spring5.service.UserService;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.TestExecutionListeners;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

/*@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:bean1.xml") //加载配置文件*/

@SpringJUnitConfig(locations = "classpath:bean1.xml")

public class JTest5 {

    @Autowired
    private UserService userService; //由上边的加载配置文件得到容器，在容器里找到相应的bean注入

    @Test
    public void test1(){
        userService.accountMoney();
    }

}

```

注意：@ContextConfiguration("classpath:bean1.xml") ，加载配置文件得到的 IOC 容器中不含有测试类本身的bean



### 6、Webflux（未完待续）

#### 1、SpringWebflux 介绍

​		（1）是 Spring5 添加新的模块，用于 web 开发的，功能和 SpringMVC 类似的，Webflux 使用当前一种比较流程响应式编程出现的框架。

![image-20210416213651876](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210416213651876.png)



（2）使用传统 web 框架，比如 SpringMVC，这些基于 Servlet 容器，Webflux 是一种异步非阻塞的框架，异步非阻塞的框架在 Servlet3.1 以后才支持，核心是基于 Reactor 的相关 API 实现的。



（3）解释什么是异步非阻塞

​			异步和同步

​			非阻塞和阻塞

​			上面都是针对对象不一样

**异步和同步针对调用者**，调用者发送请求，如果等着对方回应之后才去做其他事情就是同步，如果发送请求之后不等着对方回应就去做其他事情就是异步。

**阻塞和非阻塞针对被调用者**，被调用者受到请求之后，做完请求任务之后才给出反馈就是阻塞，受到请求之后马上给出反馈然后再去做事情就是非阻塞。



（4）Webflux 特点：

​			第一 非阻塞式：在有限资源下，提高系统吞吐量和伸缩性，以 Reactor 为基础实现响应式编程

​			第二 函数式编程：Spring5 框架基于 java8，Webflux 使用 Java8 函数式编程方式实现路由请求



（5）比较 SpringMVC

![image-20210417121948402](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210417121948402.png)



第一 两个框架都可以使用注解方式，都运行在 Tomcat 等容器中

第二 SpringMVC 采用命令式编程，Webflux 采用异步响应式编程



#### 2、响应式编程

（1）定义

​		响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。

​		电子表格程序就是响应式编程的一个例子。单元格可以包含字面值或类似"=B1+C1"的公式，而包含公式的单元格的值会依据其他单元格的值的变化而变化。



（2）Java8 及其之前版本

提供的**观察者模式**两个类 Observer 和 Observable

```java
package com.atguigu.demoreactor8.reactor8;

import java.util.Observable;
import java.util.Observer;

public class ObserverDemo extends Observable {

    public static void main(String[] args) {


        ObserverDemo observerDemo=new ObserverDemo();
        //添加观察者
        observerDemo.addObserver((o,arg)->{
            System.out.println("发生变化");
        });

        observerDemo.addObserver((o,arg)->{
            System.out.println("收到被观察者通知，准备发生变化");
        });


        observerDemo.addObserver(new Observer() {
            @Override
            public void update(Observable o, Object arg) {
                System.out.println("收到通知，准备响应");
            }
        });


        observerDemo.setChanged();  //数据变化

        observerDemo.notifyObservers(); //通知，只有数据变化了，观察者才会响应，否则不响应
    }

}
```





# SpringMVC

本质是基于 mvc 技术的servlet web应用，通过dispatcherServlet处理所有请求。

另一个角度，我们也可以理解为通过tomcat来实现mvc。



dispatcherServlet在创建后调用init时，利用spring创建bean（有些bean上面有url信息表明这是一个对应该url的handler），然后利用ioc里面的bean信息建立url到handler的映射。这是利用了sevlet提供的init扩展。



也就是说servlet采用springMVC模式来实现相应的功能。



感悟：servlet作为一个资源管理者去理解比作为一个资源去理解的适用范围更广。请求的是资源，servlet的url-pattern表示它管理的资源的路径。



## 项目结构

springMVC项目本质上是一个tomcat项目，结构如下：

![SpringMVC 项目结构](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/SpringMVC 项目结构.jpg)





## 整体工作流程

本质还是servlet，只是由 dispatcherServlet 处理所有的请求，具体的处理方法：

  1、找到handler与对应的adapter

  2、利用adapter调用handler，获取ModuleAndView对象

  3、利用视图解析器解析出真正的View

  4、把Module的数据填入View中，返回给客户端



​		**总的来说，就是浏览器请求handler，dispatcherServlet找到相应handler后，执行（配合适配器，适配器主要是把handler的执行结果翻译成ModuleAndView的形式返回）得到ModuleAndView对象，并反回ModuleAndView对象所对应结果。**



具体流程图如下：

![img](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/7896890-65ef874ad7da59a2.png)

**DispatcherServlet与ioc容器的联系**：

​    1、springmvc的ioc容器WebApplicationContext在DispatcherServlet初始化时建立。

​    2、springmvc的ioc容器，会包含普通bean如controller，以及handler映射表。

​    3、DispatcherServlet利用springmvc的ioc容器里面的bean做相应的初始化。



入门教程：https://www.jianshu.com/p/91a2d0a1e45a

DispatcherServlet具体工作流程：https://www.cnblogs.com/tengyunhao/p/7518481.html



## Handler

主要有以下几种：

### 1、@RequestMapping

一个方法作为一个handler。这是最常用的handler。实现上用一个HandlerMethod对象保存这个handler对象信息（包括Bean，以及相应的Method），HandlerMethod可以看做是这个Handler的指针。

![requestmapping](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/requestmapping.PNG)

**该handler返回值的意义：**

* 情况一：不使用@ResponseBody

​	1、ModelAndView。返回该视图。

​	2、String。返回viewName是该值的视图。

​	3、void。不返回任何信息。

* 情况二：使用@ResponseBody

​	1、任意类型。返回该类型值的json字符串。



### 2、Controller接口

一个bean作为一个Handler

![controller](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/controller.PNG)



### 3、HttpRequestHandler接口。
一个bean作为一个Handler

![httpRequestHandler](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/httpRequestHandler.PNG)



### 4、Servlet接口。

一个bean作为一个Handler。<u>新版的springmvc已经不支持了，没有相应的adapter了。</u>



handler：https://blog.csdn.net/wang0907/article/details/109154902

handlerAdapter：https://blog.csdn.net/u013219087/article/details/80649654



## HandlerAdapter

handlerAdapter主要用来屏蔽不同handler的差异，对外界提供统一的处理接口。

​	1、RequestMappingHandlerAdapter。用于配合method型的handler。

​	2、SimpleControllerHandlerAdapter。用于配合Controller。

​	3、HttpRequestHandlerAdapter。用于配合HttpRequestHandler。

​	4、SimpleServletHandlerAdapter。用于配合Servlet，*新版SpringMvc不用了*。



一般来说，如果没有自定义adapter，那默认加载三个adapter。

如果有自定义adapter，那只加载自定义的adapter。因此，如果自定义adapter，那最好自定义所有需要的adapter。



## Url到Handler映射的注册

DiapatcherServlet利用spring创建bean后，需要建立HandlerMapping。建立url与handler的映射：

​       遍历所有bean。每个bean，找出其所有名字（包括别名），对于以'/'开始的名字，认为是一个url，把该url与bean的映射保存即可。一个bean可以有多个url映射到它。

​      所以，关键就是bean名字，一个'/'开头的名字认为是一个url，代表着一个handler。



这里说的是普通handler，@RequestMapping也差不多，都是利用bean的信息寻找handler的。

不同的地方是：

​	普通handler直接是url到handler映射

​	@RequestMapping则是url到HandlerMethod（可认为是handler指针）映射



详见 AbstractDetectingUrlHandlerMapping 类的 detectHandlers() 方法。就是在这里建立url到handler的映射的。

HandlerMethod的注册过程：https://www.bianchengquan.com/article/596660.html



## 常见问题

1、使用@ResponseBody但无法报500错误，显示**无法转换**为json。

原因：对应的HandlerAdapter缺少转换器

解决方法，自定义带有转换器的HandlerAdapter即可。

第一步：pom.xml添加依赖

```xml
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.12.5</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.12.5</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.12.5</version>
    </dependency>
```

第二步：注解方式，自定义adapter的bean，加入转换器

```java
@Component
public class UtilBean {

    @Bean
    public MappingJackson2HttpMessageConverter getMappingJackson2HttpMessageConverter(){
        MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter=new MappingJackson2HttpMessageConverter();
        List<MediaType> supportedMediaTypes=new ArrayList<>();
        supportedMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        supportedMediaTypes.add(MediaType.TEXT_HTML);
        mappingJackson2HttpMessageConverter.setSupportedMediaTypes(supportedMediaTypes);
        return mappingJackson2HttpMessageConverter;
    }

    @Bean
    public StringHttpMessageConverter getStringHttpMessageConverter(){
        StringHttpMessageConverter stringHttpMessageConverter=new StringHttpMessageConverter();
        List<MediaType> supportedMediaTypes=new ArrayList<>();
        supportedMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        supportedMediaTypes.add(MediaType.TEXT_HTML);
        stringHttpMessageConverter.setSupportedMediaTypes(supportedMediaTypes);
        return stringHttpMessageConverter;
    }
    
        @Bean
    public RequestMappingHandlerAdapter getRequestMappingHandlerAdapter(MappingJackson2HttpMessageConverter cv1,StringHttpMessageConverter cv2){
        RequestMappingHandlerAdapter requestMappingHandlerAdapter=new RequestMappingHandlerAdapter();
        List<HttpMessageConverter<?>> messageConverters=new ArrayList<>();
        messageConverters.add(cv1);
        messageConverters.add(cv2);
        requestMappingHandlerAdapter.setMessageConverters(messageConverters);
        return requestMappingHandlerAdapter;
    }


}
```

或者在dispatcher-servlet.xml里面创建自定义adapter，加入转换器

```xml
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="messageConverters">
            <list>
                <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                    <property name="supportedMediaTypes">
                        <list>
                            <value>text/html; charset=UTF-8</value>
                            <value>application/json;charset=UTF-8</value>
                        </list>
                    </property>
                </bean>
                <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                    <property name="supportedMediaTypes">
                        <list>
                            <value>text/html; charset=UTF-8</value>
                            <value>application/json;charset=UTF-8</value>
                        </list>
                    </property>
                </bean>
            </list>
        </property>
    </bean>
```



2、jsp的el表达式无法解析

原因：idea创建的web工程的web.xml的问题，主要设计到起始标签和servlet版本

解决办法：

第一步：起始标签设为

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

第二步：把servlet的版本改为更高，比如4.0

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

</web-app>
```



参考：https://www.daimafans.com/article/d4344963293773824-p1-o1.html





# SpringBoot

由于前后端分离，大多数情况下，使用springBoot时，我们只需要返回数据就行，不需要返回整个页面。一般在@RequestMapping基础上加上@ResponseBody（如果类使用@RestController则不需要再加@ResponseBody了），使得handler返回json数据。



springBoot启动web时，通过内置的tomcat来启动，启动的是springMVC项目（本质上是tomcat项目），这个项目估计也是内置的，我们只需要提供classes即可（需要在类上面加上注解来表示bean创建），运行端口与工程项目可以在application.properties里面配置。具体见：https://blog.csdn.net/u012100371/article/details/76409388。



比如一棵树：

spring时代：项目是树干，spring是树叶

springboot时代：springboot是树干，项目是树叶



