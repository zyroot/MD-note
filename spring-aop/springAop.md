# spring-aop

**简介**：

Aspect Oriented Programming  面向切面编程。解耦是程序员编码开发过程中一直追求的。AOP也是为了解耦所诞生。

具体思想是：定义一个切面，在切面的纵向定义处理方法，处理完成之后，回到横向业务流。

AOP 在Spring框架中被作为核心组成部分之一，的确Spring将AOP发挥到很强大的功能。最常见的就是事务控制。工作之余，对于使用的工具，不免需要了解其所以然。学习了一下，写了些程序帮助理解。

AOP 主要是利用代理模式的技术来实现的。



## **1、静态代理：** 

**就是设计模式中的proxy模式**

a.业务接口

![](aop_resources/psb.png)

b.业务实现

![](aop_resources/业务实现.png)

c.代理对象

![](aop_resources/代理对象.png)



d.测试类 

![](aop_resources/测试类.png)



从这段代码可以看出来，代理模式其实就是AOP的雏形。 上端代码中talk(String msg, String singname)是一个切面。在代理类中的sing(singname)方法是个后置处理方法。

这样就实现了，其他的辅助方法和业务方法的解耦。业务不需要专门去调用，而是走到talk方法，顺理成章的调用sing方法

再从这段代码看：1、要实现代理方式，必须要定义接口。2、每个业务类，需要一个代理类。



## 2、动态代理：

**jdk1.5中提供，利用反射。实现InvocationHandler接口。**



a.代理类

![](aop_resources/动态代理类.png)



b.测试类

![](aop_resources/动态代理测试.png)



输出结果会是：

切面之前执行
people talk业务说法
切面之后执行

说明只要在业务调用方法切面之前，是可以动态的加入需要处理的方法。

从代码来看，如果再建立一个业务模块，也只需要一个代理类。ITalk iTalk = (ITalk) new DynamicProxy().bind(new PeopleTalk());  将业务接口和业务类绑定到动态代理类。

但是这种方式：还是需要定义接口。



## **3、利用cglib** 

a.业务类

![](aop_resources/cglib业务类.png)

b.代理类

![](aop_resources/cglib代理类.png)



c.测试类

![](aop_resources/cglib测试.png)



最后输出结果：

事物开始
people talk业务方法
事物结束
事物开始
spreak chinese业务方法
事物结束

# Java AOP

**开发模式@Aspect注解说明**

## 2 注解说明

2.1 @Aspect

作用是把当前类标识为一个切面供容器读取

2.2 @Before
标识一个前置增强方法，相当于BeforeAdvice的功能，相似功能的还有

2.3 @AfterReturning

后置增强，相当于AfterReturningAdvice，方法正常退出时执行

2.4 @AfterThrowing

异常抛出增强，相当于ThrowsAdvice

2.5 @After

final增强，不管是抛出异常或者正常退出都会执行

2.6 @Around

环绕增强，相当于MethodInterceptor

2.7 @DeclareParents



## 3 execution切点函数


execution函数用于匹配方法执行的连接点，语法为：

execution(方法修饰符(可选)  返回类型  方法名  参数  异常模式(可选)) 

 

参数部分允许使用通配符：

*  匹配任意字符，但只能匹配一个元素

.. 匹配任意字符，可以匹配任意多个元素，表示类时，必须和*联合使用

+  必须跟在类名后面，如Horseman+，表示类本身和继承或扩展指定类的所有类



示例中的* chop(..)解读为：

方法修饰符  无

返回类型      *匹配任意数量字符，表示返回类型不限

方法名          chop表示匹配名称为chop的方法

参数               (..)表示匹配任意数量和类型的输入参数

异常模式       不限

 

更多示例：



void chop(String,int)

匹配目标类任意修饰符方法、返回void、方法名chop、带有一个String和一个int型参数的方法



public void chop(*)

匹配目标类public修饰、返回void、方法名chop、带有一个任意类型参数的方法



public String *o*(..)

 匹配目标类public修饰、返回String类型、方法名中带有一个o字符、带有任意数量任意类型参数的方法



public void *o*(String,..)

 匹配目标类public修饰、返回void、方法名中带有一个o字符、带有任意数量任意类型参数，但第一个参数必须有且为String型的方法

也可以指定类：



public void examples.chap03.Horseman.*(..)

匹配Horseman的public修饰、返回void、不限方法名、带有任意数量任意类型参数的方法



public void examples.chap03.*man.*(..)

匹配以man结尾的类中public修饰、返回void、不限方法名、带有任意数量任意类型参数的方法

指定包：



public void examples.chap03.*.chop(..)

匹配examples.chap03包下所有类中public修饰、返回void、方法名chop、带有任意数量任意类型参数的方法



public void examples..*.chop(..)

匹配examples.包下和所有子包中的类中public修饰、返回void、方法名chop、带有任意数量任意类型参数的方法
可以用这些表达式替换StorageAdvisor中的代码并观察效果



## 4 更多切点函数

除了execution()，Spring中还支持其他多个函数，这里列出名称和简单介绍，以方便根据需要进行更详细的查询



4.1 @annotation()

表示标注了指定注解的目标类方法

例如 @annotation(org.springframework.transaction.annotation.Transactional) 表示标注了@Transactional的方法



4.2 args()

通过目标类方法的参数类型指定切点

例如 args(String) 表示有且仅有一个String型参数的方法



4.3 @args()

通过目标类参数的对象类型是否标注了指定注解指定切点

如 @args(org.springframework.stereotype.Service) 表示有且仅有一个标注了@Service的类参数的方法



4.4 within()

通过类名指定切点

如 within(examples.chap03.Horseman) 表示Horseman的所有方法



4.5 target()

通过类名指定，同时包含所有子类

如 target(examples.chap03.Horseman)  且Elephantman extends Horseman，则两个类的所有方法都匹配



4.6 @within()

匹配标注了指定注解的类及其所有子类

如 @within(org.springframework.stereotype.Service) 给Horseman加上@Service标注，则Horseman和Elephantman 的所有方法都匹配



4.7 @target()

所有标注了指定注解的类

如 @target(org.springframework.stereotype.Service) 表示所有标注了@Service的类的所有方法



4.8 this()

大部分时候和target()相同，区别是this是在运行时生成代理类后，才判断代理类与指定的对象类型是否匹配



## 5 逻辑运算符

表达式可由多个切点函数通过逻辑运算组成

5.1 &&

与操作，求交集，也可以写成and

例如 execution(* chop(..)) && target(Horseman)  表示Horseman及其子类的chop方法

5.2 ||

或操作，求并集，也可以写成or

例如 execution(* chop(..)) || args(String)  表示名称为chop的方法或者有一个String型参数的方法

5.3 !

非操作，求反集，也可以写成not

作者：permike 
来源：CSDN 
原文：https://blog.csdn.net/permike/article/details/88863753 
版权声明：本文为博主原创文章，转载请附上博文链接！







