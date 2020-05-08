#  AOP切面的实践
### 一、下面是一个在spring mvc中关于切面如何使用的例子，可以指直观的理解切面到底有什么作用

#### 1、引用 AspectJ jar 包依赖
pom.xml 文件添加依赖
```java
<!-- aspectjrt --> 
<dependency>
 <groupId>org.aspectj</groupId>
 <artifactId>aspectjrt</artifactId>
 <version>1.9.2</version>
</dependency>
```
 2、创建两个新的包
 ```java
com.hbxy.course.c03
test.com.hbxy.course.c03
```
3、创建一个业务类 UserXML
```java
package com.hbxy.course.c03;
public class UserXML {
 private int id= 20190101;
 private String name ="AspectJXML 测试用户";
 private String sex;
 private String email;
 public int getId() {
 return id;
 }
 public void setId(int id) {
 this.id = id;
 }
 public String getName() {
 return name;
 }
 public void setName(String name) {
 this.name = name;
 }
 public String getSex() {
 return sex;
 }
 public void setSex(String sex) {
 this.sex = sex;
 }
 public String getEmail() {
 return email;
 }
 public void setEmail(String email) {
 this.email = email;
 }
 public String toString(){
 StringBuffer stringBuffer=new StringBuffer();
 stringBuffer.append("id=");
 stringBuffer.append(id);
 stringBuffer.append("\n");
 stringBuffer.append("name=");
 stringBuffer.append(name);
 return stringBuffer.toString();
 }
 public void saveUser() {
 System.out.println("保存用户信息...");
 }
 public void queryUser() {
 System.out.println("查看用户信息...");
 System.out.println(this.toString());
 } }
 ```
4、创建一个 AOP 切面类 UserXMLAspectJ
```java
package com.hbxy.course.c03;
//定义代理通知类(切面类)
public class UserXMLAspectJ {
 public void aspectMethod1() {
 System.out.println("在切点之前，执行 aspectMethod1 方法...");
 }
 public void aspectMethod2() {
 System.out.println("在切点之后，执行 aspectMethod2 方法...");
 } }
 ```
 5、创建配置类 spring-config-aop.xml
 ```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xmlns:aop="http://www.springframework.org/schema/aop" 
 xsi:schemaLocation="http://www.springframework.org/schema/beans 
 http://www.springframework.org/schema/beans/spring-beans.xsd 
 http://www.springframework.org/schema/aop 
 http://www.springframework.org/schema/aop/spring-aop.xsd">
 <!-- 1.定义业务 bean（目标类） -->
 <bean id="userxml" class="com.hbxy.course.c03.UserXML"/>
 <!-- 2.定义代理类（切面类） -->
 <bean id="myAspectJ" class="com.hbxy.course.c03.UserXMLAspectJ"/>
 <!-- 3.定义 aspectj-->
 <aop:config>
 <aop:aspect id="userAspect" ref="myAspectJ">
 <!--3.1 定义切点-->
 <aop:pointcut id="userQueryUser" 
 expression="execution(* com.hbxy.course.c03.UserXML.queryUser(..))" />
 <!--3.2 定义通知-->
 <!--aop:before 配置了切点之前执行 aspectMethod1 方法-->
 <aop:before pointcut-ref="userQueryUser" method="aspectMethod1" />
 <!--aop:after-returning 配置了切点之后执行 aspectMethod2 方法-->
 <aop:after-returning pointcut-ref="userQueryUser" method="aspectMethod2"/>
 </aop:aspect>
 </aop:config>
</beans>
```

6、新建一个测试类 AspecJXMLTest
```java
package test.com.hbxy.course.c03;
import com.hbxy.course.c03.UserXML;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class AspecJXMLTest {
 public static void main(String[] args) {
 ApplicationContext ctx =
 new ClassPathXmlApplicationContext("spring-config-aop.xml");
 UserXML user = (UserXML) ctx.getBean("userxml");
 //执行业务方法，该方法配置为 AOP 切点
 //当执行时，将会在这个方法之前和之后执行切面类（UserXMLAspectJ）的方法
 user.queryUser();
 } }
 ```

 7、运行测试类
 ```java
运行结果：
在切点之前，执行 aspectMethod1 方法...
查看用户信息...
id= 20190101
name ="AspectJXML 测试用户"
在切点之后，执行 aspectMethod2 方法...
```
对比运行结果你可以发现切点在此的作用，就是在运行系统或者说主类前先经过他，或运行完要经过他。
### 二、在Spring Boot中定义的话会更加简单免除了这些个配置类，下面我简单说一下在Spring Boot中对于日志处理的整体逻辑结构。
==同样的我们要先添加依赖Aspect依赖，因为SpringBoot2.0之后新建项目的时候没有aop这个模块了，你需要自己手动加载依赖，2.0之前新建项目有aop模块，你勾选上就可以，只是简单说一下，怕小伙伴迷路啊。让后你就可以编写日志类了，下面我简单说下里面用到的注解。内容的话比较简单，这些注解你知道用在什么地方，有什么作用，其实你已经学会了。==

**1.定义一个切面类，加上@Component @Aspect这两个注解**
@Aspect：作用就是把当前类标识为一个切面（必须）

@Component ：组件扫描。spring boot可以通过这个注解才可以找到这个切面类。

**2.定义切点，需要一个是表达式，一个是方法签名这两个通俗的讲，第一个就是在哪个地方定义切点，第二个是切点空方法，之后会使用到。**

@Pointcut("execution(* com.Mwxw.web.*.*(..))") （切入你可以理解为拦截）

public void log() {}

**在上面切点表达式中，上面表达式代表切入com.Mwxw.web包下的所有类的所有方法，第一个*表示不限返回类型，第二个*表示那个包下的所有类，第三个*表示包下所有类的所有方法，..两个点表示方法里的参数不限。必须用@Pointcut切点注解，写在一个空方法上面，等下在Advice通知中可以直接调用这个空方法就行了。**

**3.Advice通知增强**
@Before("log()"）：在方法上标注这个注解，可以在系统执行到切点之前执行你注解的这个方法。

@After（"log()"）:在系统方法执行之后执行你注解的这个方法。

@AfterReturning(returning = "result",pointcut = "log()"）：这个是代表系统方法执行完，要返回数据的时候执行这个方法。里面的参数第一个就是你在方法里定义的要返回的参数，第二个就是切点方法。

@AfterThrowing ()切点方法抛异常执行。

**JoinPoint为连接点对象，它可以获取当前切入的方法的参数等**

joinPoint.getSignature().getName(); // 获取目标方法名

joinPoint.getSignature().getDeclaringType().getSimpleName(); // 获取目标方法所属类的简单类名

joinPoint.getSignature().getDeclaringTypeName(); // 获取目标方法所属类的类名

joinPoint.getSignature().getModifiers(); // 获取目标方法声明类型(public、private、protected)

Object[] args = joinPoint.getArgs(); // 获取传入目标方法的参数，返回一个数组

joinPoint.getTarget(); // 获取被代理的对象

joinPoint.getThis(); // 获取代理对象自己

**这就是最基本的日志框架的东西，你可以简单理解一下，更深一些的东西下一次会分享到。**

##### 面向切面编程是一种思想，在我们日常实践中最常用的就是在做日志处理的时候，如果没有切面我们会在每个方法中添加日志处理的代码，如下代码内容是一样的，但是代码复用性差并且单独写成一个日志处理的方法也需要手动调用。![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321134951130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDE2MjAwMA==,size_16,color_FFFFFF,t_70#pic_center)
###### 我们可以通过动态代理，可以在指定位置执行对应流程。这样就可以将一些横向的功能抽离出来形成一个独立的模块，然后在指定位置插入这些功能。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321135500895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDE2MjAwMA==,size_16,color_FFFFFF,t_70#pic_center)
之后就是例子中的定义切点并通知系统在切点的哪个位置运行。
通知类型和介绍
|Before  |目标方法调用之前执行  |
|--|--|
|After  |目标方法调用之后执行  |
| After-returning |	目标方法执行成功后执行  |
|Around|相当于合并了前置和后置|
| After-throwing | 目标方法抛出异常后执行 |
#### 今天就分享到这里，之后会继续更新，路过的小伙伴感觉不错的可以帮忙点赞，有其他想法的同学可以评论留言互相交流，本人看到后会第一时间回复的。
