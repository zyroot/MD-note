# **springboot高阶技术**

# 九、缓存

JSR-107 缓存规范

springboot缓存抽象

redis 缓存

## 1:jsr-107

```properties
Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry
和 Expiry。
• CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个		   CachingProvider。
• CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下   文中。一个CacheManager仅被一个CachingProvider所拥有
• Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
• Entry是一个存储在Cache中的key-value对。
• Expiry 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目	 将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy
```

## 2:springboot提供的缓存对象

```properties
二、Spring缓存抽象Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用JCache（JSR-107）注解简化我们开发；
• Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
• Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等；
• 每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
• 使用Spring缓存抽象时我们需要关注以下两点；1、确定方法需要被缓存以及他们的缓存策略2、从缓存中读取之前缓存存储的
```

## 3.缓存注解：

```properties
三、几个重要概念&缓存注解
Cache 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、
ConcurrentMapCache等
CacheManager  缓存管理器，管理各种缓存（Cache）组件
@Cacheable    主要针对方法配置，能够根据方法的请求参数对其结果进行缓存
@CacheEvict   清空缓存
@CachePut     保证方法被调用，又希望结果被缓存。
@EnableCaching 开启基于注解的缓存
keyGenerator 缓存数据时key生成策略
serialize 缓存数据时value序列化策略
```

![缓存注解主要参数](images\缓存注解主要参数.png)

![sqel语法](images\sqel语法.png)

## 4.整合redis缓存：

```properties
1. 引入spring-boot-starter-data-redis
2. application.yml配置redis连接地址
3. 使用RestTemplate操作redis
	1. redisTemplate.opsForValue();//操作字符串
	2. redisTemplate.opsForHash();//操作hash
	3. redisTemplate.opsForList();//操作list
	4. redisTemplate.opsForSet();//操作set
	5. redisTemplate.opsForZSet();//操作有序set
4. 配置缓存、CacheManagerCustomizers
5. 测试使用缓存、切换缓存、 CompositeCacheManag
```

## 5.缓存初体验:

​	1）、开启基于注解的缓存

​		在启动类上加上：@EnableCaching

​	2）、(service层)标注缓存注解即可

​	3）、默认使用的是ConcurrentMapCacheMananger组件

@Cacheable

```java
    /**
     * @Cacheable
     * 注意：方法运行后
     * 将方法的运行结果进行缓存，以后再要相同的数据，直接从缓存中获取
     * CacheManager管理多个Cache组件的，对缓存的真正crud操作在Cache组件中，
     * 每个缓存组件都有自己的一个名字；
     * 几个属性：
     *      cacheNames/value;指定缓存组件的名字；可以指定多个缓存
     *      key:缓存数据时的key;可以用它来指定。默认是使用方法参数的值，1-方法的返回值
     *          编写sqel; eg: #id; 参数id的值 （等同于 #a0 #p0 #root.args[0]）
     *      keyGenerator: key的生成器；可以自己指定key的生成器的id
     *          key和keyGenerator 二选一
     *      cacheManager：指定缓存管理器；或者指定缓存解析器cacheResolver；二选一
     *      condition：指定符合条件的情况下；
     *      unless：否定缓存 eg:unless="#result==null" 如果结果为空则不缓存
     *      sync：是否使用异步模式
     * cache运行流程：
     *      1、方法运行之前，先去查cache(缓存组件)，按照cacheName指定的名字获取；
     *          （CacheManager现获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     *      2、区cache中查找缓存的内容，使用一个key,默认就是方法的参数；
     *          key是按照某种策略生成的；默认是使用keyGenerator生成的。默认使用simpleKeyGnenerator生成key
     *      3、没有查到缓存九调用目标方法；
     *      4、将目标方法返回的结果，放入缓存中。
     * @param id
     * @return
     */
    @Cacheable(cacheNames = {"emp"},keyGenerator = "myKeyGenerator")
    public Employee getEmployee(Integer id){
        Employee one = employeeDao.findOne(id);
        System.out.println("查询"+id+"号员工");
        return  one;
    }
```

@CachePut

```java
    /**
     * @CachePut：既调用了方法，有更新了缓存数据；同步更新缓存
     * 修改了数据库的某个数据，同时又更新缓存；
     * 运行时机：
     * 1、先调用方法；
     * 2、将目标方法的结果缓存起来（方法运行以后给缓存中放入数据）
     *  @Cacheable的key是不能用 #result,运行时机不同，
     */
    @CachePut(value = "emp",key = "#employee.id")
    public Employee updateEmployee(Employee employee){
        Employee one = employeeDao.save(employee);
        System.out.println("更新了"+one.getId()+"号员工");
        return  one;
    }
```

@CacheEvict:

```java
    /**
     * @CacheEvict 清除缓存
     * 参数：
     *       key:指定要清除的数据
     *      allEntries=true 删除emp 清除所有的数据
     *      beforeInvocation=false ：缓存的清除是否在方法之前执行
     *      默认代表是缓存清除操作在方法执行之后执行；
     *  key 指定要清除的数据
     * @param id
     * @return
     */
    @CacheEvict(value = "emp",key = "#id")
    public void deleteEmployee(Integer id){
        System.out.println("删除了"+id+"号员工");
    }
```

@CacheConfig:标注在类上，作用抽取缓存中公共配置

```java
//cacheManager : 是自定义的cacheManager的方法名称
@CacheConfig(cacheNames = "emp",cacheManager = "emptCacheManager")
```

## 6.redis缓存之初体验

### 1）、引入redis的start

spring-boot-starter-data-redis

```
<!--引入reids-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>

```



```properties
#redis
spring.redis.host=172.17.0.154
```

原理：

​        a.引入redis的starter,容器中保存的是 redisCacheManager,

​	b.redisCacheManager帮我们创建redisCache来作为缓存组件；redisCache通过redisl来缓存数据

​	c.默认保存数据k-v都是Object;利于序列化保存的；如何保存json?

​		解析：1.引入了redis的starter，容器保存的是RedisCacheManager;

​			    2.默认创建的RedisCacheManager 操作redis的时候使用的是 RedisTemplate<Object,Object>

​			    3.RedisTemplate默认使用jdk序列化机制

​			   4.自定义CacheManager;	

springboot提供了两个操作redis的工具类

​	a:  RedisTemplate ;  //k-v都是对象的

​	b:  StringRedisTemplate; //操作k-v都是字符串的

```java
	/**
	 * redis5大数据类型
	 * String（字符串）、list（列表）、set（集合）、hash（散列）、ZSet(有序集合)
	 *	stringRedisTemplate.opsForValue() 简化操作字符串的
	 *  ....
	 */
	@Test
	public void test(){
		//保存
//		stringRedisTemplate.opsForValue().append("msg","hello");
		//获取
		String msg = stringRedisTemplate.opsForValue().get("msg");
	}
```

### 2）、对象序列化：

```java
	//测试保存对象
	@Test
	public void testSaveObj(){
		Department one = departmentDao.findOne(1);
		//默认保存对象，默认使用jdk序列化机制，序列化后的数据保存到redis中
//		redisTemplate.opsForValue().set("dp-01",one);
		//使用自定义的 模板操作类
		deRedisTemplate.opsForValue().set("dp-02",one);
	}
```

```java
RedisAutoConfiguration//自动配置类 注册了两个操作redis 类
  1、RedisTemplate
  2、StringRedisTemplate
各自类中有自己的默认序列化规则：jdk

//为满足实际需要，重新配置序列化机制：

package com.eim.config;

import com.eim.entity.Department;
import com.eim.entity.Employee;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import java.net.UnknownHostException;

/**
 * redis自定义配置类
 * Created by Zy on 2018/12/6.
 */
@Configuration
public class MyRedisConfig {

    /**
     * 新建一个redis操作工具类，修改其默认(jdk)的序列化器为 json
     指定泛型：Department
     新建：Jackson2JsonRedisSerializer
     设置默认序列化器
     * @param redisConnectionFactory
     * @return
     * @throws UnknownHostException
     */
    @Bean
    public RedisTemplate<Object, Department> deRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
      
        Jackson2JsonRedisSerializer<Department> employeeJackson2JsonRedisSerializer = new 		         Jackson2JsonRedisSerializer<Department>(Department.class);
      
        template.setDefaultSerializer(employeeJackson2JsonRedisSerializer);
      
        return template;
    }
}

```

### 3）、自定义cachemanager;

```java
package com.eim.config;

import com.eim.entity.Department;
import com.eim.entity.Employee;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

import java.net.UnknownHostException;
import java.util.List;

/**
 * redis自定义配置类
 * Created by Zy on 2018/12/6.
 */
@Configuration
public class MyRedisConfig {

    /**
     * 新建一个redis操作工具类，修改其默认(jdk)的序列化器为 json
     * 指定泛型：dept
     * @param redisConnectionFactory
     * @return
     * @throws UnknownHostException
     */
    @Bean
    public RedisTemplate<Object, Department> deRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> employeeJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(employeeJackson2JsonRedisSerializer);
        return template;
    }

    /**
     * 新建一个redis操作工具类，修改其默认(jdk)的序列化器为 json
     * 指定泛型：emp
     * @param redisConnectionFactory
     * @return
     * @throws UnknownHostException
     */
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> employeeJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(employeeJackson2JsonRedisSerializer);
        return template;
    }

    /**
     * 模仿自：RedisCacheConfiguration类
     * 当出现自定义两个及其以上的cache时，需要指定一个是主（默认）cacheManager
     * 解决：使用：@primary注解
     * 自定义 缓存管理器 dept
     */
    @Primary
    @Bean
    public RedisCacheManager depCacheManager( RedisTemplate<Object, Department> deRedisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(deRedisTemplate);
        //使用前缀 默认将cacheName作为key
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }
    /**
     * 模仿自：RedisCacheConfiguration类
     * 自定义 缓存管理器 emp
     */
    @Bean
    public RedisCacheManager emptCacheManager( RedisTemplate<Object, Employee> empRedisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        //使用前缀 默认将cacheName作为key
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }
}

```

### 4）、反序列化问题：

```java
/**
 * Created by Zy on 2018/12/6.
 */
@Service
public class DepartmentService {

    @Autowired
    private DepartmentDao departmentDao;

    /**
     * //因为修改了序列化机制，所以存入json都没问题，
     * 但是不同的类型，反序列化（从redis数据库中）取出数据时，如果不指定cacheManager,
     * 则使用系统默认一个，导致反序列化出错
     * 解决办法：根据不同缓存对象，指定不同的cacheManager
     * @param id
     * @return
     */
    @Cacheable(value = "dept",cacheManager = "depCacheManager")
    public Department getDepartment(Integer id){
        return departmentDao.findOne(id);
    }
}
```

# 十、消息队列：

## ==一、概述：==

```javascript
1、大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力
2、消息服务中两个重要概念：
       a:消息代理（message broker）---消息服务器
       b:目的地（destination）
3、当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。
消息队列主要有两种形式的目的地
	1）、队列（queue）：点对点消息通信（point-to-point）
	2）、主题（topic）：发布（publish）/订阅（subscribe）消息通信

```

异步处理：



传统方式：

![图片1](images\传统方式.png)



多线程方式：

![图片2](images\多线程方式.png)

消息队列方式：

![图片3](images\消息队列方式.png)

==主要用途：==

![应用解耦](images\应用解耦.png)

应用解耦：

​	利用微服务，将系统进行划分和隔离，并且通过消息队列（订阅）进行耦合。

流量削峰：

​	秒杀，100个--限定10到消息队列，完了以后，从消息队列中取出10个消息，进行业务逻辑处理

```javascript
4、点对点式： **消费及删除**
消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列
消息只有唯一的发送者和接受者，但并不是说只能有一个接收者

5、发布订阅式：
发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息

6、JMS（Java Message Service）JAVA消息服务：
基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

7、AMQP（Advanced Message Queuing Protocol）--跨平台性更好
高级消息队列协议，也是一个消息代理的规范，兼容JMS
RabbitMQ是AMQP的实现
```

![jms&amqp](images\jms&amqp.png)

```java
8、Spring支持
spring-jms提供了对JMS的支持
spring-rabbit提供了对AMQP的支持
需要ConnectionFactory的实现来连接消息代理
提供JmsTemplate、RabbitTemplate来发送消息
@JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息
@EnableJms、@EnableRabbit开启支持

9、Spring Boot自动配置
JmsAutoConfiguration
RabbitAutoConfiguration
```

## 二、RabbitMQ简介：

```javascript
RabbitMQ简介：
RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。

核心概念
Message
消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

Publisher
消息的生产者，也是一个向交换器发布消息的客户端应用程序。

Exchange
交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别
Queue
消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

Binding
绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
Exchange 和Queue的绑定可以是多对多的关系。

Connection
网络连接，比如一个TCP连接。

Channel
信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

Consumer
消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

Virtual Host
虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

Broker
表示消息队列服务器实体
```

![rabbit全局图](images\rabbit全局图.png)



## 三、RabbitMQ运行机制

```javascript
AMQP 中的消息路由
AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。
```

![rabbitmq运行机制](images\rabbitmq运行机制.png)

### Exchange 类型

### 1）、Direct Exchange 类型

```javascript
Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers 。headers 匹配 AMQP 消息的 header 而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：
```

![DirectExchange类型](images\DirectExchange类型.png)

```javascript
消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。
```

### 2）、fanout  Exchange 类型

![fanoutExchange](images\fanoutExchange.png)

```javascript
每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。
```



### 3）、 topic  Exchange 类型

![topicExchange类型](images\topicExchange类型.png)

```javascript
topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*”。#匹配0个或多个单词，*匹配一个单词。
```

## 四、RabbitMQ整合

安装RabbitMQ:

```javascript
 1、选择docker中国镜像加速，并且下载management版本（有web控制页面）
 doker pull registry.docker-cn.com/library/rabbitmq:3-management
 2、运行镜像 暴露连接端口：5672；暴露web访问端口：15672 ；d69a5113ceae（镜像id）
 [root@192 ~]# docker run d -p 5672:5672 -p 15672:15672 --name myrabbitmq d69a5113ceae
3、外部浏览器访问：
  http://192.168.0.113:15672/
  用户名密码 默认都是 guest
```



```javascript
1、引入 spring-boot-starter-amqp
2、application.yml配置
3、测试RabbitMQ
4、AmqpAdmin：管理组件
5、RabbitTemplate：消息发送处理组件
```

### 1、引入 spring-boot-starter-amqp

```xml
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 2、application.properties配置

```properties
spring.rabbitmq.host=192.168.0.113
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
#默认是5672 （可以不用配置）
#spring.rabbitmq.port=5672
#默认是 / (可以不用配置)
#spring.rabbitmq.virtual-host=
```

### 3、自动配置原理

```java
/**
 *rabbitmq 自定配置类
 * 1、RabbitAutoConfiguration
 * 2、自动配置了链接工厂：RabbitConnectionFactoryCreator
 * 3、RabbitProperties 封装了 rabbit所有配置
 * 4、RabbitTemplate 操作rabbit的（接受和发送消息）
 * 5、AmqpAdmin ：rabbit系统管理组件（增加exchange、queues,绑定exchange和queues）
 */
@SpringBootApplication
public class SpringbootRabbitApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootRabbitApplication.class, args);
	}

}
```

### 4、rabbitTemplate 操作rabbit

发送：

```java
@Autowired
private RabbitTemplate rabbitTemplate;

/**
	 * 发送
	 */
@Test
public void contextLoads() {

  //send(String exchange(交换器), String routingKey（路由键）, Message message（消息）)
  //rabbitTemplate.send(exchange,rotuteKye,message);

  //简单常用的、
  //只需要传入要发送的对象，自动序列化发送给rabbitmq
  //convertAndSend(String exchange（交换器）, String routingKey（路由键）, Object object（发送的对象）)
  //rabbitTemplate.convertAndSend(exchange,rotuteKye,object);

  HashMap<String, Object> map = new HashMap<>();
  map.put("msg","消息");
  map.put("date", Arrays.asList("dd",123,true));
  //对象默认序列化（java自己的序列化机制）后发送出去
  rabbitTemplate.convertAndSend("exchange.direct","atguigu.news",map);

}
```

接受：

```java
	/**
	 * 接受
	 */
	@Test
	public void receive(){
		//填入要收取的消息队列的名字
		Object o = rabbitTemplate.receiveAndConvert("atguigu.news");
		System.out.println(o.getClass());
		System.out.println(o.toString());
	}
```

### 5、指定rabbit序列化机制

```java
/**
 * rabbit配置项
 * Created by Administrator on 2018/12/15.
 */
@Configuration
public class RabbitConfig {

    /**
     * 配置自定义的消息转换序列化机制
     * @return
     */
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

### 6、广播发送

```java
	/**
	 * 广播类型的发送
	 */
	@Test
	public void message(){
		//填入要发送的exchange交换器，object消息对象，因为是广播无需指定路由键，
		Object object = new Object();
		rabbitTemplate.convertAndSend("exchange",object);

	}
```

### 7、监听消息队列

==@EnableRabbit  +  @RabbitListener  监听消息队列的内容==

#### 1）、主配置类开启注解

```java
package com.eim;

import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 *rabbitmq 自定配置类
 * 1、RabbitAutoConfiguration
 * 2、自动配置了链接工厂：RabbitConnectionFactoryCreator
 * 3、RabbitProperties 封装了 rabbit所有配置
 * 4、RabbitTemplate 操作rabbit的（接受和发送消息）
 * 5、AmqpAdmin ：rabbit系统管理组件（增加exchange、queues,绑定exchange和queues）
 */
@EnableRabbit //开启基于注解的rabbitmq
@SpringBootApplication
public class SpringbootRabbitApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootRabbitApplication.class, args);
	}

}


```

#### 2）、监听队列

```java
package com.eim.service;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;

/**
 * Created by Administrator on 2018/12/16.
 */
@Service
public class RabbitService {


    /**
     * @RabbitListener 监听消息队列的注解
     * queues  添加要监听的消息队列的名字
     *只获取消息内容，直接反序列化到Object对象中
     */
    @RabbitListener(queues = {"atguigu.news"})
    public void receive(Object object){
        System.out.println("消息队列有消息，处理");
    }

    /**
     * 获取更多，eg:消息头
     * 使用Message
     * @param message
     */
    @RabbitListener(queues = {"atguigu.news"})
    public void receive02(Message message){
        //消息体
        message.getBody();
        //消息头
        message.getMessageProperties();
        System.out.println("消息队列有消息，处理");
    }
}

```

### 8、AmqpAdmin 消息管理系统组件

```java
package com.eim;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * Created by Administrator on 2018/12/16.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class AmqbAdminTest {

    @Autowired
    private AmqpAdmin amqpAdmin;

    @Test
    public void test(){
        /**
         * 利用组件创建 交换器
         */
        amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));

        /**
         * 利用组件创建 消息队列
         */
        amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));

        /**
         * 利用组件 绑定 消息队列
         * 参数集合：
             * 1.destination:目的地--队列
             * 2.绑定的的类型：消息队列
             * 3.exchange:交换器
             * 4.routingKey:路由键
             * 5.arguments map类型的参数（没有传入null）
         */
        amqpAdmin.declareBinding(new Binding("qpadmin.queue",
                                Binding.DestinationType.QUEUE,
                                "amqpadmin.exchange","haha",null));
    }
}

```

# 十一、Apache shiro 框架使用

## 一、shiro框架核心Api

​	1）、subject: 用户主体（把操作交给securityManager）

​	2）、securityManager: 安全管理器（关联realm）

​	3）、Realm：shiro连接数据的桥梁

## 二、整合shiro

### 1）、依赖导入：

```xml
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring</artifactId>
			<version>1.4.0</version>
		</dependency>
```

## 三、编写shiro配置类：

### 1）、编写自定义realm类：

```java
package com.eim.realm;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

/**
 * 自定义realm
 * 此类用于关键性的授权和认证逻辑
 * Created by Zy on 2018/12/29.
 */
public class UserRealm  extends AuthorizingRealm{

    /**
     * 执行授权逻辑
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权逻辑");
        return null;
    }

    /**
     * 执行认证逻辑
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证逻辑");
        return null;
    }
}

```

### 2）、shiro配置类

​	三大核心：

```java
package com.eim.config;

import com.eim.realm.UserRealm;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * shiro配置类
 * Created by Zy on 2018/12/29.
 */
@Configuration
public class ShiroConfig {

    /**
     * 创建shiroFilterFactoryBean
     */
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(getDefaultWebSecurityManager());
        return shiroFilterFactoryBean;
    }

    /**
     * 创建DefaultWebSecurityManager
     */
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(getRealm());
        return defaultWebSecurityManager;
    }

    /**
     * 创建Realm对象
     */
    @Bean
    public UserRealm getRealm(){
        return new UserRealm();
    }

}
```

### 3）使用shiro控制器来拦截请求

```java
    /**
     * 创建shiroFilterFactoryBean
     */
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(getDefaultWebSecurityManager());
        //添加Shiro内置过滤器
        /**
         * Shiro内置过滤器：可以实现权限相关的拦截器
         *  常用的过滤器：
         *      anon: 无需认证（登录）可以访问、
         *      authc:必须认证才可以访问、
         *      user：如果使用remember的功能，可以直接访问
         *      perms:该资源必须得到资源权限才可以访问
         *      role:该资源必须得到角色权限才可以访问
         */
        Map<String, String> filterMap = new LinkedHashMap<>();
        filterMap.put("/user/**","authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);

        //修改自定义login路径
        shiroFilterFactoryBean.setLoginUrl("/user/login.do");
        return shiroFilterFactoryBean;
    }
```

### 4）、编写自定义登录页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    登录页面
    <font color="red">[[${msg}]]</font>
    <h3>登录</h3>
    <form method="post" th:action="@{/dologin.do}">
        用户名：
        <input name="username" type="text"/><br>
        密码：
        <input name="password"  type="text"/><br>
        <button type="submit" value="点我登录哦">点我登录哦</button>
    </form>
</body>
</html>
```

### 5）、shiro编写登录逻辑

```java
    /**
     * 登录逻辑
     * @param username
     * @param password
     * @return
     */
    @PostMapping("/dologin.do")
    public String dologin(@RequestParam("username")String username,
                          @RequestParam("password") String password,
                          Map map){
        /**
         * 使用shiro 编写认证操作
         */
        //1.获取Subject
        Subject subject = SecurityUtils.getSubject();

        //2.封装用户数据
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(username, password);

        //3.执行登录方法
        try {
          //会调用UserRealm的doGetAuthenticationInfo方法 执行认证逻辑
            subject.login(usernamePasswordToken);
            //登录成功
            return "redirect:/add.do";
        } catch (UnknownAccountException e) {
            //登录失败
            //UnknownAccountException 捕获用户名不存在的异常
            map.put("msg","用户名不存在呀！！");
            return  "login";
        }catch (IncorrectCredentialsException e) {
            //登录失败
            //IncorrectCredentialsException 捕获密码错误的异常
            map.put("msg","密码呀！！");
            return  "login";
        }
    }
```

### 6）、修改realm类，userRealm

```java
package com.eim.realm;

import com.eim.mapper.UserMapper;
import com.eim.pojo.User;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

/**
 * 自定义realm
 * 此类用于关键性的授权和认证逻辑
 * Created by Zy on 2018/12/29.
 */
public class UserRealm  extends AuthorizingRealm{

    @Autowired
    private UserMapper userMapper;

    /**
     * 执行授权逻辑
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权逻辑");
        return null;
    }

    /**
     * 执行认证逻辑
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //编写 认证逻辑（判断用户名和密码是否正确）
        //1.获取用户名密码（从入参中获取authenticationToken）
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        User user = userMapper.findbyUserName(token.getUsername());
        //2.判断是否相等
        if(!token.getUsername().equals(user.getName())){
            //用户名不存在
            return null;//返回null时，shiro底层会抛出UnknownAccountException 异常
        }
        //3.判断密码
        /**
         * 返回一个 AuthenticationInfo的子类 SimpleAuthenticationInfo
         * @param principal   返回给 subject.login(usernamePasswordToken); 的参数（可以为空）
         * @param credentials 数据库查出的密码.
         * @param realmName   realm的名字（可以为空）
         */
        return new SimpleAuthenticationInfo("",user.getPassword(),"");
    }
}
```

## 四、Shiro授权

### 1）、书写 授权过滤器 ，过滤资源

//授权过滤器

```java
	 /**
     * 当授权资源 被拦截后，会调用 doGetAuthorizationInfo 执行授权逻辑
     * 没有授权时，会调到 未授权页面
     */
      filterMap.put("/user/add.do","perms[user:add]");
```

==//设置未授权时 跳转的页面==

  ==shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorize.do");==

```java
package com.eim.config;

import com.eim.realm.UserRealm;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * shiro配置类
 * Created by Zy on 2018/12/29.
 */
@Configuration
public class ShiroConfig {

    /**
     * 创建shiroFilterFactoryBean
     */
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(getDefaultWebSecurityManager());
        //添加Shiro内置过滤器
        /**
         * Shiro内置过滤器：可以实现权限相关的拦截器
         *  常用的过滤器：
         *      anon: 无需认证（登录）可以访问、
         *      authc:必须认证才可以访问、
         *
         *      user：如果使用remember的功能，可以直接访问
         *      perms:该资源必须得到资源权限才可以访问
         *      role:该资源必须得到角色权限才可以访问
         */
        Map<String, String> filterMap = new LinkedHashMap<>();

        //授权过滤器
        /**
         * 当授权资源 被拦截后，会调用 doGetAuthorizationInfo 执行授权逻辑
         * 没有授权时，会调到 未授权页面
         【】 中的 user:add 就是授权字符串
         */
        filterMap.put("/user/add.do","perms[user:add]");

        //认证过滤器
        filterMap.put("/user/dologin.do","anon");
        filterMap.put("/user/*","authc");



        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);

        //修改自定义login路径
        shiroFilterFactoryBean.setLoginUrl("/user/login.do");
        //设置未授权时 跳转的页面
        shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorize.do");
        return shiroFilterFactoryBean;
    }

    /**
     * 创建DefaultWebSecurityManager
     */
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(getRealm());
        return defaultWebSecurityManager;
    }

    /**
     * 创建Realm对象
     */
    @Bean
    public UserRealm getRealm(){
        return new UserRealm();
    }

}

```

### 2.书写授权逻辑（UserRealm）：

当要访问需要授权的页面时，会调用UserRealm的doGetAuthorizationInfo方法，

判断是否拥有权限，

如果拥有权限返回：simpleAuthorizationInfo

```java
   /**
     * 执行授权逻辑
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //获取用户资源权限
        Subject subject = SecurityUtils.getSubject();
        //getPrincipal  认证时  注入的 principal
        User user = (User) subject.getPrincipal();
      	//添加用户权限
        simpleAuthorizationInfo.addStringPermission(user.getPerms());
        return simpleAuthorizationInfo;
    }
```

## 五、thymeleaf和shiro 联合使用：

官方文档：https://github.com/theborakompanioni/thymeleaf-extras-shiro

### 1）、maven

```xml
<!--thymeleaf-shiro-extras-->
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>1.1.0</version>
</dependency>

```

### 2）、在shiro的configuration中配置

```java
@Bean
public ShiroDialect shiroDialect() {
  return new ShiroDialect();
}
```

### 3）、在html中加入xmlns命名空间

```html
<html lang="zh_CN" xmlns:th="http://www.thymeleaf.org"
      xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
```

### 4）、使用：