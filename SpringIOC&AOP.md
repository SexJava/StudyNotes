# SpringIOC&AOP

### 1. IOC：控制反转（Inversion Of Control）

1. ##### 控制：资源的获取方式

   1. 主动式：要什么资源自己创建，复杂对象创建太复杂
   2. 被动式：资源的获取不是自己创建，而是交给容器创建和设置

2. ##### 容器：管理所有的组件（有功能的类），容器可以自动检查出哪些组件需要用到另一些组件（类）；容器帮我们创建组件对象并赋值过去。主动的new资源变成被动的接受资源。

### 2. DI：依赖注入（Dependency Injection）

容器能知道哪个组件运行的时候，需要另外一个组件，容器通过反射的形式，将容器中准备好的组件注入进去（利用反射给属性赋值）。

### 3.栗子（Demo）

以前都是自己new对象，现在所有的对象交给容器创建。（在容器中注册组件）

1. IOC配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--注册一个Person对象，Spring会自动创建这个Person对象-->
       <!--一个Bean标签可以注册一个组件（对象、类）
       class: 写要注册的组件的全类名
       id: 这个组件的唯一标识
       -->
       <bean id="person01" class="com.study.bean.Person">
           <!--使用property标签为person对象的属性赋值
           name="lastName":指定属性名
           value="张三":为这个属性赋值
           -->
           <property name="lastName" value="张三"></property>
           <property name="age" value="18"></property>
           <property name="gender" value="男"></property>
           <property name="email" value="123@qq.com"></property>
       </bean>
       <bean id="person02" class="com.study.bean.Person">
           <property name="lastName" value="李四"></property>
           <property name="age" value="10"></property>
           <property name="gender" value="男"></property>
           <property name="email" value="321@qq.com"></property>
       </bean>
        <bean id="person" class="com.study.bean.Person">
           <!--调用有参构造器进行创建对象并赋值
           可以省略name属性，但是要严格按照构造器参数进行赋值
           index：为参数指定索引，从零开始
           type：重载时指定参数类型
           -->
           <constructor-arg name="lastName" value="小明"></constructor-arg>
           <constructor-arg name="age" value="90"></constructor-arg>
           <constructor-arg name="gender" value="男"></constructor-arg>
           <constructor-arg name="email" value="111@qq.com"></constructor-arg>
       </bean>
       <!--通过p名称空间为bean赋值-->
       <!--名称空间：在xml中名称空间是用来防止标签重复的-->
       <!--导入名称空间xmlns:p="http://www.springframework.org/schema/p"-->
       <bean id="person03" class="com.study.bean.Person" p:age="19" p:email="xiji@qq.com" p:gender="女" p:lastName="小鸡"></bean>
   </beans>
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:p="http://www.springframework.org/schema/p"
          xmlns:util="http://www.springframework.org/schema/util"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
   
       <bean id="car01" class="com.study.bean.Car">
           <property name="carName" value="baoma"></property>
           <property name="color" value="red"></property>
           <property name="price" value="100"></property>
       </bean>
       <bean id="book00" class="com.study.bean.Book" p:bookName="aaaa" p:author="hyh"></bean>
       <bean id="person" class="com.study.bean.Person">
           <property name="lastName">
               <!--进行复杂赋值-->
               <null></null>
           </property>
           <!--ref:代表引用外面的一个值(严格引用) car= ioc.getBean("car01")-->
           <!--<property name="car" ref="car01">-->
           <property name="car">
               <!--对象可以使用bean标签创建 car =new Car();内部bean不能通过id获取到只能内部使用-->
               <bean class="com.study.bean.Car">
                   <property name="carName" value="benci"></property>
                   <property name="color" value="green"></property>
                   <property name="price" value="1000"></property>
               </bean>
           </property>
           <!--如何为list类型赋值-->
           <property name="books">
               <!--books = new ArrayList<Book>()-->
               <list>
                   <!--list标签体中添加每一个元素-->
                   <bean class="com.study.bean.Book" p:author="lyd" p:bookName="hahah"></bean>
                   <!--引用一个外部对象-->
                   <ref bean="book00"></ref>
               </list>
           </property>
           <property name="maps">
               <!--maps = new LinkedHashMap<>();-->
               <map>
                   <!--一个entry代表一个键值对-->
                   <entry key="key01" value="zhangsan"></entry>
                   <entry key="key02" value="18"></entry>
                   <entry key="key03" value-ref="book00"></entry>
                   <entry key="key04">
                       <bean class="com.study.bean.Car" p:carName="biyadi" p:color="black" p:price="50"></bean>
                   </entry>
                   <!--<entry key="key05">-->
                   <!--    <map></map>-->
                   <!--</entry>-->
               </map>
           </property>
           <property name="properties">
               <!--properties = new Properties();-->
               <props>
                   <!--所有的key value都是string类型 值都写在标签体内-->
                   <prop key="username">root</prop>
                   <prop key="password">123456</prop>
               </props>
           </property>
       </bean>
       <!--util名称空间创建集合类型的bean：方便引用-->
       <bean id="person01" class="com.study.bean.Person">
           <property name="maps" ref="myMap"></property>
       </bean>
       <!--相当于new LinkedHashMap<>()-->
       <util:map id="myMap">
           <entry key="key01" value="zhangsan"></entry>
           <entry key="key02" value="18"></entry>
           <entry key="key03" value-ref="book00"></entry>
           <entry key="key04">
               <bean class="com.study.bean.Car" p:carName="biyadi" p:color="black" p:price="50"></bean>
           </entry>
       </util:map>
       <!--级联属性赋值，级联属性：属性的属性-->
       <bean id="person02" class="com.study.bean.Person">
           <property name="car" ref="car01"></property>
           <!--为car赋值的时候改变car的价格-->
           <property name="car.price" value="30000000"></property>
       </bean>
   
   
       <!--通过继承实现bean的配置信息的重用-->
       <!--abstract="true"，控制这个bean的配置是一个抽象的，并不能获取他的实例，只能被别人继承-->
       <bean id="person03" class="com.study.bean.Person" abstract="true">
           <property name="lastName" value="哦哦"></property>
           <property name="age" value="20"></property>
           <property name="gender" value="男"></property>
           <property name="email" value="qwe@qq.com"></property>
       </bean>
       <!--parent指定当前bean的配置继承与哪个bean-->
       <bean id="person04" class="com.study.bean.Person" parent="person03">
           <property name="lastName" value="昂昂"></property>
       </bean>
   </beans>
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:p="http://www.springframework.org/schema/p"
          xmlns:util="http://www.springframework.org/schema/util"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
   
       <!--原来是按照配置的顺序创建bean-->
       <!--改变bean的创建顺序-->
       <!--depends-on="car,book",配置创建person的时候先创建car，book，depends-on属性值也是有顺序的-->
       <bean id="person" class="com.study.bean.Person" depends-on="car,book"></bean>
       <bean id="car" class="com.study.bean.Car"></bean>
       <bean id="book" class="com.study.bean.Book"></bean>
   
   
   
       <!--测试bean的作用域，分别创建单实例和多实例的bean-->
       <!--bean的作用域：指定bean是否单实例，默认是单实例的-->
       <!--
           prototype:多实例
               1.容器启动默认不会去创建多实例bean
               2.获取的时候创建bean
               3.每次获取都会创建一个新的容器（对象）
           singleton:单实例
               1.在容器启动完成之前就已经创建好对象，保存在容器中了
               2.任何时候要获取都是获取之前创建好的那个
           request:在web环境下，同一次请求创建一个bean实例(没用)
           session:在web环境下，同一次会话创建一个bean实例(没用)
       -->
       <bean id="book01" class="com.study.bean.Book" scope="prototype"></bean>
   
   
   
       <!--配置通过静态工厂方法创建的bean、实例工厂方法创建的bean、FactoryBean-->
       <!--bean的创建默认就是框架利用反射new出来的bean实例-->
       <!--工厂模式： 工厂帮我们创建对象:有一个专门帮我们创建对象的类，这个类就是工厂
           AirPlaneFactory.getAirPlane(String jzName)
   
           静态工厂：工厂本身不用创建对象，都是通过静态方法调用，工厂类，工厂方法名。
           实力工厂：工厂本事需要创建对象，工厂类 对象 = new 工厂类（）；
           工厂对象.getAirPlane(“张三”)；
       -->
       <!--1.静态工厂（不需要创建工厂本身）,class 指定静态工厂全类名，factory-method 指定创建对象的方法如果该方法需要传参则用constructor-arg指定参数-->
       <bean id="airPlane01" class="com.study.factory.AirPlaneStaticFactory" factory-method="getAirPlane">
           <!--可以为方法指定参数-->
           <constructor-arg name="jzName" value="lisi"></constructor-arg>
       </bean>
       <!--2.实例工厂使用 指定工厂方法factory-method-->
       <bean id="airPlane02" class="com.study.factory.AirPlaneInstanceFactory"></bean>
       <!--factoryBean指定当前对象创建使用哪个工厂
           先配置出实例工厂对象
           配置我们要创建的对象使用哪个工厂创建
               factory-bean指定使用哪个工厂实例
               factory-method指定使用哪个工厂方法
       -->
       <bean id="airPlane" class="com.study.bean.AirPlane" factory-bean="airPlane02" factory-method="getAirPlane">
           <constructor-arg value="wangwu"></constructor-arg>
       </bean>
   
       <!--FactoryBean()是Spring规定的一个接口，只要是这个接口的实现类，Spring都认为是一个工厂，
           ioc容器启动的时候不会创建实例
           FactoryBean获取的时候才会创建对象
       -->
       <bean id="myFactoryBean" class="com.study.factory.MyFactoryBeanImpl"></bean>
   </beans>
   ```

   

2. 实体类

   ```java
   public class Person {
       private String lastName;
       private Integer age;
       private String gender;
       private String email;
   	public Person() {
           System.out.println("我被创建了");
       }
       public Person(String lastName, Integer age, String gender, String email) {
           this.lastName = lastName;
           this.age = age;
           this.gender = gender;
           this.email = email;
           System.out.println("有参构造器被调用了");
       }
       @Override
       public String toString() {
           return "Person{" +
                   "lastName='" + lastName + '\'' +
                   ", age=" + age +
                   ", gender='" + gender + '\'' +
                   ", email='" + email + '\'' +
                   '}';
       }
   }
   public class Book {
       private String bookName;
       private String author;
   }
   public class Car {
       private String carName;
       private String color;
       private Integer price;
   }
   public class AirPlane {
       private String fdj;
       private String yc;// 机翼长度
       private Integer personNum;// 载客量
       private String jzName;// 机长名字
       private String fjsName;// 副驾驶
   }
   ```

   

3. 测试类

   ```java
   package com.study.test;
   
   import com.study.bean.Book;
   import com.study.bean.Car;
   import com.study.bean.Person;
   import org.junit.Test;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.support.ClassPathXmlApplicationContext;
   
   import java.util.ArrayList;
   import java.util.HashMap;
   
   /**
    * @Author Liuyunda
    * @Date 2020/6/22 20:56
    * @Email man021436@163.com
    * @Description: DOTO
    */
   
   public class IOCTest {
       // private  ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       // private  ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc2.xml");
       private  ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc3.xml");
   
       /**
        * @Description: 从容器中拿到这个组件
        * 细节：
        * 1.ApplicationContext（IOC容器的接口）
        * 2.给容器中注册一个组件，我们也从容器中按照id拿到了这个组件的对象
        *      组件的创建工作，是容器完成的
        *      Person对象是什么时候创建好的=在容器创建完成的时候就已经创建好了
        * 3.同一个组件在IOC容器中是单实例的，在容器创建完成时就已经创建了
        * 4.获取为注册的组件
        *      org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'none' available
        * 5.property怎么赋值的=IOC容器在创建这个组件对象的时候，（property）会利用setter方法为JavaBean的属性赋值
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/22
        */
       @Test
       public void test() {
           // ApplicationContext 代表IOC容器对象
           // 当前应用的xml配置文件在ClassPath下
           // 根据spring的配置文件得到IOC容器对象
           // ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
           // ApplicationContext ioc = new FileSystemXmlApplicationContext("D:\\Code\\Spring\\01_Spring_IOC\\conf\\ioc.xml");
           // 容器帮我们创建好了对象
           System.out.println("容器启动成功");
           Person person01 = (Person) ioc.getBean("person01");
           Person person02 = (Person) ioc.getBean("person01");
           System.out.println(person01);
           System.out.println(person02);
           System.out.println(person01 == person02);// true
           System.out.println("====================");
           // Person none = (Person) ioc.getBean("none");
           // System.out.println(none);
       }
       /**
        * @Description: 根据bean的类型从IOC容器中获取bean的实例
        * 细节：
        * 1.如果IOC容器中这个类型的bean有多个，查找就会失败
        *      org.springframework.beans.factory.NoUniqueBeanDefinitionException:
        *      No qualifying bean of type 'com.study.bean.Person' available: expected single matching bean but found 2: person01,person02
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/22
        */
       @Test
       public void test02() {
           // Person bean = ioc.getBean(Person.class);
           // System.out.println(bean);
           Person person02 = ioc.getBean("person02", Person.class);
           System.out.println(person02);
       }
       /**
        * @Description: 通过构造器为bean的属性赋值
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/22
        */
       @Test
       public void  test03() {
           // Person person = (Person) ioc.getBean("person");
           // System.out.println(person);
           Person person03 = (Person) ioc.getBean("person03");
           System.out.println(person03);
       }
   
       /**
        * @Description: 为各种属性赋值
        *      测试使用null值，默认引用类型就是null，基本类型是默认值
        *      Person{lastName='null', age=null, gender='null', email='null', car=null, books=null, maps=null, properties=null}
        *      ref引用，内部bean(内部bean不能被获取到)
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/22
        */
       @Test
       public void  test04() {
           Person person = (Person) ioc.getBean("person");
           System.out.println(person.getLastName() == null);
           System.out.println(person);
           System.out.println("===================");
           System.out.println(person.getCar());
           Car car01 = (Car) ioc.getBean("car01");
           System.out.println(car01 == person.getCar());
           System.out.println(person.getBooks());
           // Book book01 = (Book) ioc.getBean("book01");
           // System.out.println(book01);
           System.out.println(person.getMaps());
           System.out.println(person.getProperties());
           Person person02 = (Person) ioc.getBean("person02");
           System.out.println(person02);
   
   
       }
       /**
        * @Description: 测试级联属性
        *      级联属性可以修改属性的属性，原来的bean的属性的值也会修改
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/27
        */
       @Test
       public void test05(){
           Person person02 = (Person) ioc.getBean("person02");
           System.out.println(person02);
           Car car01 = (Car) ioc.getBean("car01");
           System.out.println(car01.getPrice());
       }
       /**
        * @Description: 通过继承实现bean的配置信息的重用
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/27
        */
       @Test
       public void test06(){
           Person person04 = (Person) ioc.getBean("person04");
           System.out.println(person04);
       }
   
       /**
        * @Description: 通过abstract属性创建一个模版bean，不能被获取实例，只能被继承
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/27
        */
       @Test
       public void test07(){
           //org.springframework.beans.factory.BeanIsAbstractException: Error creating bean with name 'person03': Bean definition is abstract
           Person person03 = (Person) ioc.getBean("person03");
           System.out.println(person03);
       }
   
       /**
        * @Description: bean之间的依赖关系
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/27
        */
       @Test
       public void test08(){
   
       }
   
       /**
        * @Description: 测试bean的作用域，分别创建单实例和多实例的bean
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/27
        */
       @Test
       public void test09(){
   
       }
   
       /**
        * @Description: 配置通过静态工厂方法创建的bean、实例工厂方法创建的bean、FactoryBeaan
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/6/27
        */
       @Test
       public void test10(){
   
       }
       /**
        * @Description: 配置通过静态工厂方法创建的bean、实例工厂方法创建的bean、FactoryBean
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/7/5
        */
       @Test
       public void test11(){
           // Object airPlane01 = ioc.getBean("airPlane01");
           // System.out.println(airPlane01);
           Object airPlane = ioc.getBean("airPlane");
           System.out.println(airPlane);
       }
   
       /**
        * @Description: FactoryBean()是Spring规定的一个接口，只要是这个接口的实现类，Spring都认为是一个工厂
        * @Param: []
        * @return: void
        * @Author: Liuyunda
        * @Date: 2020/7/5
        */
       @Test
       public void test12(){
           Object myFactoryBean = ioc.getBean("myFactoryBean");
           System.out.println(myFactoryBean);
       }
   }
   
   ```
   
4. 工厂类

   ```java
   package com.study.factory;
   
   import com.study.bean.AirPlane;
   
   /**
    * @Author Liuyunda
    * @Date 2020/7/5 13:07
    * @Email man021436@163.com
    * @Description: 实例工厂
    */
   public class AirPlaneInstanceFactory {
       // new AirPlaneInstanceFactory().getAirPlane
       public AirPlane getAirPlane(String jzName){
           System.out.println("AirPlaneInstanceFactory.....正在造飞机");
           AirPlane airPlane = new AirPlane();
           airPlane.setFdj("太行");
           airPlane.setFjsName("lyd");
           airPlane.setJzName(jzName);
           airPlane.setPersonNum(100);
           airPlane.setYc("198.98m");
           return airPlane;
       }
   }
   
   ```

   ```java
   package com.study.factory;
   
   import com.study.bean.AirPlane;
   
   /**
    * @Author Liuyunda
    * @Date 2020/7/5 13:07
    * @Email man021436@163.com
    * @Description: 静态工厂
    */
   public class AirPlaneStaticFactory {
       // AirPlaneStaticFactory.getAirPlane()
       public static AirPlane getAirPlane(String jzName){
           System.out.println("AirPlaneStaticFactory.....正在造飞机");
           AirPlane airPlane = new AirPlane();
           airPlane.setFdj("太行");
           airPlane.setFjsName("lyd");
           airPlane.setJzName(jzName);
           airPlane.setPersonNum(100);
           airPlane.setYc("198.98m");
           return airPlane;
       }
   }
   
   ```

   ```java
   package com.study.factory;
   
   import com.study.bean.Book;
   import org.springframework.beans.factory.FactoryBean;
   
   import java.util.UUID;
   
   /**
    * @Author Liuyunda
    * @Date 2020/7/5 13:36
    * @Email man021436@163.com
    * @Description: 实现了FactoryBean的类都是一个工厂类，spring会自动的调用工厂方法创建实例
    *      1.编写一个FactoryBean的实现类
    *      2.在spring配置文件中进行注册
    */
   public class MyFactoryBeanImpl implements FactoryBean<Book> {
       /**
        * @Description: 工厂方法，spring自动调用
        * @Param: []
        * @return: com.study.bean.Book
        * @Author: Liuyunda
        * @Date: 2020/7/5
        */
       @Override
       public Book getObject() throws Exception {
           System.out.println("MyFactoryBeanImpl....帮你创建了对象");
           Book book = new Book();
           book.setBookName(UUID.randomUUID().toString());
           return book;
       }
   
       /**
        * @Description: 返回创建的对象的类型，spring自动调用
        * @Param: []
        * @return: java.lang.Class<?>
        * @Author: Liuyunda
        * @Date: 2020/7/5
        */
       @Override
       public Class<?> getObjectType() {
           return Book.class;
       }
   
       /**
        * @Description: 是单例么
        * @Param: []
        * @return: boolean
        * @Author: Liuyunda
        * @Date: 2020/7/5
        */
       @Override
       public boolean isSingleton() {
           return false;
       }
   }
   
   ```

   

### 4.ApplicationContext继承关系图

![image-20200622212523200](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20200622212523200.png)

- new ClassPathXmlApplicationContext("ioc.xml"); IOC容器的配置文件在类路径下；
- new FileSystemXmlApplicationContext("D:\\Code\\Spring\\01_Spring_IOC\\conf\\ioc.xml"); IOC容器的配置文件在磁盘路径下；

### 5.通过注解分别创建