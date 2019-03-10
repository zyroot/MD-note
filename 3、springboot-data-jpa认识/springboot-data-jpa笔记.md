# springdata jpa

# 1、SpringData简介

jsp是什么？

```properties
JPA(Java Persistence API)是Sun官方提出的Java持久化规范。它为Java开发人员提供了一种对象/关联映射工具来管理Java应用中的关系数据。他的出现主要是为了简化现有的持久化开发工作和整合ORM技术，结束现在Hibernate，TopLink，JDO等ORM框架各自为营的局面。
```

springdatajpa是什么？

```properties
Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！
```

JPA 与 hibernate关系

```properties
1,JPA(Java Persistence API)是Sun官方提出的Java持久化规范。它为Java开发人员提供了一种对象/关系映射工具来管理Java应用中的关系数据。，而Hibernate是它的一种实现。除了Hibernate，还有EclipseLink(曾经的toplink)，OpenJPA等可供选择，所以使用Jpa的一个好处是，可以更换实现而不必改动太多代码。

2,Hibernate作为JPA的一种实现,jpa的注解已经是hibernate的核心，hibernate只提供了一些补充，而不是两套注解。hibernate对jpa的支持够足量，在使用hibernate注解建议使用jpa。
```

# 2、整合SpringData JPA

**JPA:ORM（**Object Relational Mapping）；

## 1）、创建映射类

**编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；**  

```java
//使用JPA注解配置映射关系
@Entity //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user") //@Table来指定和哪个数据表对应;如果省略默认表名就是user；
public class User {

    @Id //这是一个主键
  	//@GeneratedValue主键生成策略。默认使用的GenerationType.AUTO
    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增主键
    private Integer id;

    @Column(name = "last_name",length = 50) //这是和数据表对应的一个列
    private String lastName;
    @Column //省略默认列名就是属性名
    private String email;
```

### 实体类注解：

```java
@Id 声明属性为主键 

@GeneratedValue //表示主键是自动生成策略，一般该注释和 @Id 一起使用 

@Entity // 任何 hibernte 映射对象都要有次注释 

@Table(name = “tablename”) //类声明此对象映射到哪个表 

@Column(name = “Name”,nullable=false,length=32) //声明数据 库字段和类属性对应关系 

@Lob// 声明字段为 Clob 或 Blob 类型

@OneToMany(mappedBy=”order”,cascade = CascadeType.ALL, fetch = FetchType.LAZY) 
   @OrderBy(value = “id ASC”) 
  // 一对多声明，和 ORM 产品声明类似，一看就明白了。 
   @ManyToOne(cascade=CascadeType.REFRESH,optional=false) 
   @JoinColumn(name = “order_id”) 
  // 声明为双向关联 



@Temporal(value=TemporalType.DATE) //做日期类型转换。 

@OneToOne(optional= true,cascade = CascadeType.ALL, mappedBy = “person”) 
  // 一对一关联声明 
   @OneToOne(optional = false, cascade = CascadeType.REFRESH) 
   @JoinColumn(name = “Person_ID”, referencedColumnName = “personid”,unique = true) 
   //声明为双向关联 



 @ManyToMany(mappedBy= “students”) 
   //多对多关联声明。 
  @ManyToMany(cascade = CascadeType.PERSIST, fetch = FetchType.LAZY) 
  @JoinTable(name = “Teacher_Student”, 
    joinColumns = {@JoinColumn(name = “Teacher_ID”, referencedColumnName = “teacherid”)}, 
    inverseJoinColumns = {@JoinColumn(name = “Student_ID”, referencedColumnName = 
    “studentid”)}) 
  // 多对多关联一般都有个关联表，是这样声明的！ 



 @Transiten//表示此属性与表没有映射关系，是一个暂时的属性 

@Cache(usage= CacheConcurrencyStrategy.READ_WRITE)//表示此对象应用缓存 
```

```java
 @Transient   //表示此数据不在数据库表里建立属性
 private String temp;
 
@Temporal(TemporalType.TIMESTAMP) //这个是带时分秒的类型
private Date date;
 
@OneToOne(cascade = CascadeType.ALL, mappedBy = "x")
private A a;

```



### =='@DynamicUpdate'==

//自动更新时间 (操作表数据，会更新表中设置的 

举例：

`update_time timestamp not null default current_timestamp  on update current_timestamp) `

属于hibernate包

### @GeneratedValue

```properties
@GeneratedValue提供了主键的生成策略。@GeneratedValue注解有两个属性,分别是strategy和generator,其中generator属性的值是一个字符串,默认为"",其声明了主键生成器的名称(对应于同名的主键生成器@SequenceGenerator和@TableGenerator)。

 JPA为开发人员提供了四种主键生成策略,其被定义在枚举类GenerationType中,包括以四种：
 
1.GenerationType.TABLE
使用一个特定的数据库表格来保存主键,持久化引擎通过关系数据库的一张特定的表格来生成主键

2.GenerationType.SEQUENCE
在某些数据库中,不支持主键自增长,比如Oracle,其提供了一种叫做"序列(sequence)"的机制生成主键。此时,GenerationType.SEQUENCE就可以作为主键生成策略

3.GenerationType.IDENTITY
此种主键生成策略就是通常所说的主键自增长,数据库在插入数据时,会自动给主键赋值,比如MYSQL可以在创建表时声明"auto_increment" 来指定主键自增长。

4.GenerationType.AUTO
把主键生成策略交给持久化引擎(persistence engine),持久化引擎会根据数据库在以上三种主键生成策略中选择其中一种。此种主键生成策略比较常用,由于JPA默认的生成策略就是GenerationType.AUTO,所以使用此种策略时.可以显式的指定@GeneratedValue(strategy = GenerationType.AUTO)也可以直接@GeneratedValue

```

## 2）、实操

**编写一个Dao接口来操作实体类对应的数据表（Repository）** 

```java
//继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}

```

repository继承关系：

```properties
=Repository （空接口）
=CrudRepository （增删改查）
=PagingAndSortingRepository （分页和排序）
=JpaRepository （扩展增删改查、批量操作 ）   四个是一一继承的关系
JpaSpecificationExecutor： 用来做负责查询的接口
Specification：是Spring Data JPA提供的一个查询规范， 要做复杂的查询，类似hibernate QBC查询
```



### 关键字查询

List<User> findByEmailLike(String email);

User findByUserNameIgnoreCase(String userName);
​    
List<User> findByUserNameOrderByEmailDesc(String email);

具体的关键字，使用方法和生产成SQL如下表所示:

| Keyword           | Sample                                  | JPQL snippet                             |
| ----------------- | --------------------------------------- | ---------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2 |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2 |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                 |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2    |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                       |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age ⇐ ?1                       |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                       |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                      |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                 |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                 |
| IsNull            | findByAgeIsNull                         | … where x.age is null                    |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                   |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1              |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1          |
| StartingWith      | findByFirstnameStartingWith             | … where x.firstname like ?1 (parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining               | … where x.firstname like ?1 (parameter bound wrapped in %) |
| OrderBy           | findByAgeOrderByLastnameDesc            | … where x.age = ?1 order by x.lastname desc |
| Not               | findByLastnameNot                       | … where x.lastname <> ?1                 |
| In                | findByAgeIn(Collection ages)            | … where x.age in ?1                      |
| NotIn             | findByAgeNotIn(Collectionage)           | … where x.age not in ?1                  |
| TRUE              | findByActiveTrue()                      | … where x.active = true                  |
| FALSE             | findByActiveFalse()                     | … where x.active = false                 |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)    |

### 分页查询

分页查询在实际使用中非常普遍了，spring data jpa已经帮我们实现了分页的功能，在查询的方法中，需要传入参数`Pageable`,当查询中有多个参数的时候`Pageable`建议做为最后一个参数传入

Page<User> findALL(Pageable pageable);
​    
Page<User> findByUserName(String userName,Pageable pageable);

`Pageable` 是spring封装的分页实现类，使用的时候需要传入页数、每页条数和排序规则

`注意`，分页查询page是从 ==0== 开始的(0表示第一页)。

```java
@Test
public void testPageQuery() throws Exception {
    int page=1,size=10;
    Sort sort = new Sort(Direction.DESC, "id");
    Pageable pageable = new PageRequest(page, size, sort);
    userRepository.findALL(pageable);
    userRepository.findByUserName("testName", pageable);
}
```

### **限制查询**

有时候我们只需要查询前N个元素，或者支取前一个实体。

```java
ser findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

### 自定义SQL查询

其实Spring data 觉大部分的SQL都可以根据方法名定义的方式来实现，但是由于某些原因我们想使用自定义的SQL来查询，spring data也是完美支持的；在SQL的查询方法上面使用`@Query`注解，如涉及到删除和修改在需要加上`@Modifying`.也可以根据需要添加 `@Transactional` 对事物的支持，查询超时的设置等

```java
@Modifying
@Query("update User u set u.userName = ?1 where c.id = ?2")
int modifyByIdAndUserId(String  userName, Long id);
    
@Transactional
@Modifying
@Query("delete from User where id = ?1")
void deleteByUserId(Long id);
  
@Transactional(timeout = 10)
@Query("select u from User u where u.emailAddress = ?1")
    User findByEmailAddress(String emailAddress);
```

### 开启本地注解：

还可以使用@Query来指定本地查询，只要设置nativeQuery为true，比如：

```java
@Query(value="select * from tbl_user where name like %?1" ,nativeQuery=true)    
public List<UserModel> findByUuidOrAge(String name);
```

举例：

```java
    @Query(nativeQuery = true,value = "SELECT  * from user")
    List<User> getUsers();
```

### 索引参数与命名参数

索引参数如下所示，索引值从1开始，查询中 ”?X” 个数需要与方法定义的参数个数相一致，并且顺序也要一致 

```java
@Query("select u from User u where u.emailAddress = ?1")
    User findByEmailAddress(String emailAddress);
```

命名参数（==推荐使用这种方式==）：可以定义好参数名，赋值时采用==@Param("参数名")==，而不用管顺序。

```java
@Query(value = "select * from user where name=:name and age=:age",nativeQuery = true)
User UserByNameAndAge(@Param("age") Integer age,@Param("name") String name);
```

如果是 @Query 中有 LIKE 关键字，后面的参数需要前面或者后面加 %，这样在传递参数值的时候就可以不加 %：

```java
@Query("select o from UserModel o where o.name like ?1%")    
public List<UserModel> findByUuidOrAge(String name);

@Query("select o from UserModel o where o.name like %?1")    
public List<UserModel> findByUuidOrAge(String name);

@Query("select o from UserModel o where o.name like %?1%")    
public List<UserModel> findByUuidOrAge(String name);
```

### 多表查询

多表查询在spring data jpa中有两种实现方式，第一种是利用hibernate的级联查询来实现，第二种是创建一个结果集的接口来接收连表查询后的结果，这里主要第二种方式。

首先需要定义一个结果集的接口类。

```java
public interface HotelSummary {

    City getCity();

    String getName();

    Double getAverageRating();

    default Integer getAverageRatingRounded() {

        return getAverageRating() == null ? null : (int) Math.round(getAverageRating());

    }

}
```

查询的方法返回类型设置为新创建的接口

```java
@Query("select h.city as city, h.name as name, avg(r.rating) as averageRating "
        - "from Hotel h left outer join h.reviews r where h.city = ?1 group by h")
Page<HotelSummary> findByCity(City city, Pageable pageable);

@Query("select h.name as name, avg(r.rating) as averageRating "
        - "from Hotel h left outer join h.reviews r  group by h")
Page<HotelSummary> findByCity(Pageable pageable);
```

使用

```java
Page<HotelSummary> hotels = this.hotelRepository.findByCity(new PageRequest(0, 10, Direction.ASC, "name"));
for(HotelSummary summay:hotels){
        System.out.println("Name" +summay.getName());
    }
```

在运行中Spring会给接口（HotelSummary）自动生产一个代理类来接收返回的结果，代码汇总使用`getXX`的形式来获取

### 本地查询多表查询：

==这种形式可以查任何多表，任意返回字段，只不过我们自己多写一点转换代码==  

`注意`  不能用select  * ，会报错。

```java
@Query(value = "select e.id as eid,e.deptId as edid from tbl_emp e join tbl_dept t on e.deptId=t.id ",nativeQuery = true)
List<Object> getInfo();

/**
或者，不取别名
*/
@Query(value = "select e.id,e.deptId from tbl_emp e join tbl_dept t on e.deptId=t.id ",nativeQuery = true)
List<Object> getInfo();
```

封装参数的类：

```java
@Data
public class Info {
    private Integer eid;

    private Integer edid;

    private Integer tid;

    public Info() {
    }

    public Info(Integer eid, Integer edid, Integer tid) {
        this.eid = eid;
        this.edid = edid;
        this.tid = tid;
    }
}
```

①封装对象属性到Info中，然后，在添加到list集合中

```java
	@Test//链表查询测试
	public void testJoin(){
		List<Object> info = deptRepository.getInfo();
		ArrayList<Info> infos = new ArrayList<>();
		for(Object o : info){
			Object[] rowArray = (Object[]) o;
			Info info1 = new Info((Integer) rowArray[0], (Integer) rowArray[1], null);
			infos.add(info1);
			System.out.println(infos);
		}
	}
```

```json
[Info(eid=1, edid=1, tid=null), Info(eid=2, edid=1, tid=null), Info(eid=3, edid=1, tid=null), Info(eid=4, edid=2, tid=null), Info(eid=5, edid=2, tid=null), Info(eid=6, edid=3, tid=null), Info(eid=7, edid=4, tid=null)]
```

②封装属性到Map中，然后再添加到list集合中

```java
@Test//链表查询测试
public void testJoin2(){
   List<Object> info = deptRepository.getInfo();
   ArrayList infos = new ArrayList<>();
   for(Object o : info){
      Object[] rowArray = (Object[]) o;
      Map<String, Object> mapArr = new HashMap<String, Object>();
      mapArr.put("eid", rowArray[0]);//根据查询字段的顺序排列的
      mapArr.put("edid", rowArray[1]);
      infos.add(mapArr);
      System.out.println(infos);
   }
}
```

```json
[{eid=1, edid=1}, {eid=2, edid=1}, {eid=3, edid=1}, {eid=4, edid=2}, {eid=5, edid=2}, {eid=6, edid=3}, {eid=7, edid=4}]
```



## 3）、基本的配置JpaProperties

```yaml
spring:  
 jpa:
    hibernate:
#     更新或者创建数据表结构
      ddl-auto: update
#    控制台显示SQL
    show-sql: true
#在配置文件中取消自动将驼峰命名转为下划线形式
 jpa:
    hibernate:
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
#配置自动建表：updata:没有表新建，有表更新操作,控制台显示建表语句
spring:
	jpa:
		hibernate:
			ddl-auto: update
```

# 2.1 JpaSpecificationExecutor查询

参考地址：https://blog.csdn.net/qq_30054997/article/details/79420141

## 1、单条件查询

继承接口

```java
/**
 * Created by Administrator on 2019/2/23.
 * 注意：JpaSpecificationExecutor不能单独使用，要和继承repository的接口一起使用
 */
public interface DeptRepository extends JpaRepository<Dept,Integer>,JpaSpecificationExecutor<Dept>{
```

测试代码

```java
	@Test
	public void testManyToOne(){
		Specification<Dept> spc = new Specification<Dept>() {
			/**
			 *
			 * @param root    			根对象，封装了查询条件的对象
			 * @param criteriaQuery		定义一个基本查询，一般不用
			 * @param criteriaBuilder	创建一个查询对象
			 * @return Predicate 	·	定义了查询对象
			 */
			@Override
			public Predicate toPredicate(Root<Dept> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
				//从根对象中去取字段
				Path<Object> deptName = root.get("deptName");
				Predicate predicate = criteriaBuilder.equal(deptName, "RD");
				return predicate;
			}
		};
		List<Dept> all = deptRepository.findAll(spc);
		System.out.println(all);
	}
```

## 2、多条件查询

### ①给定查询条件多条件查询

```java
	/**
	 * 多条件查方式一
	 * 需求：deptName以及locAdd进行数据查询
	 */
	@Test
	public void testManyQ(){
		Specification<Dept> spec = new Specification() {
			@Override
			public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {
				ArrayList<Predicate> list = new ArrayList<>();
				list.add(criteriaBuilder.equal(root.get("deptName"),"RD"));
				list.add(criteriaBuilder.equal(root.get("locAdd"),11));
				Predicate[] predicates = new Predicate[list.size()];
				//criteriaBuilder.and()中的参数是predicates可变数组，所有需要如此转换
				//使用cb 进行条件拼接
				return criteriaBuilder.and(list.toArray(predicates));
			}
		};
		List<Dept> all = deptRepository.findAll(spec);
		System.out.println(all);
	}
```

### ②多条件查询方二

```java
	/**
	 * 多条件查询二
	 * 需求：deptName或者locAdd
	 */
	@Test
	public void testManyQor(){
		Specification<Dept> spec = new Specification() {
			@Override
			public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {

				return criteriaBuilder.or(criteriaBuilder.equal(root.get("deptName"),"RD"),
											criteriaBuilder.equal(root.get("locAdd"),"12"));
			}
		};
		List<Dept> all = deptRepository.findAll(spec);
		System.out.println(all);
	}
```

## 3、分页查询

```java
	/**
	 * 需求 ：查询王姓用户并做分页处理
	 */
	@Test
	public void test5(){
		Specification<Dept> spc = new Specification() {
			@Override
			public Predicate toPredicate(Root root, CriteriaQuery criteriaQuery, CriteriaBuilder criteriaBuilder) {
				return criteriaBuilder.like(root.get("deptName").as(String.class),"%R%");
			}
		};
		Pageable pageable = new PageRequest(0,2);
		Page<Dept> all = deptRepository.findAll(spc, pageable);
		System.out.println(all);
	}
```

## 4、排序类似：

## 5、排序+分页

# 2.2关联映射操作

## ①一对一的关联关系

​	用户与角色的一对一关系。

用户：一方

角色：一方

创建实体：

```java
/**
 * Created by Administrator on 2019/3/2.
 */
@Data
@Entity
@Table(name = "t_roles")
public class Roles {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "roleid")
    private Integer roleid;

    @Column(name = "rolename")
    private String rolename;
    
    //mappedBy  关联的那个对象？ private Roles roles;
    @OneToOne(mappedBy = "roles")
    private Users users;

}
```

```java
@Data
@Entity
@Table(name = "t_user")
public class Users {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "userid")
    private Integer userid;

    @Column(name = "username")
    private String username;

    @Column(name = "userage")
    private Integer userage;

    //建立主外键关系  一对一
    @OneToOne(cascade = CascadeType.PERSIST)//cascade 级联操作，在保存用户的同时保存角色
    //JoinColumn就是维护外键（roles对应），列名可以相同，也可以自己取一个名字
    @JoinColumn(name = "roles_id")
    private Roles roles;
}
```

一对一关联关系操作：

级联保存：

```java
    @Autowired
    private UsersRepository repository;

    /**
     * 添加用户的同时添加角色
     */
    @Test
    public void test01(){
        //创建角色
        Roles roles = new Roles();
        roles.setRolename("管理员");
        //创建用户
        Users users = new Users();
        users.setUserage(30);
        users.setUsername("赵小刚");
        //建立关系
        users.setRoles(roles);
        roles.setUsers(users);
        //保存数据
        repository.save(users);
    }
```

级联查询：

```java
    /**
     * 根据用户ID查询用户，同时查询用户角色
     */
    @Test
    public void test03(){
        Users one = repository.findOne(1);
        System.out.println(one);
        System.out.println(one.getRoles());
    }

//输出：
Users{userid=1, username='赵小刚', userage=30, 
Roles{roleid=1, rolename='管理员'
```

## ②一对多关联关系

需求：从角色到用户的一对多的关联关系

角色：一方

用户：多方

创建实体users

```java
package com.eim.pojo;

import lombok.Data;

import javax.persistence.*;

/**
 * Created by Administrator on 2019/3/2.
 */
//@Data
@Entity
@Table(name = "t_user")
public class Users {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "userid")
    private Integer userid;

    @Column(name = "username")
    private String username;

    @Column(name = "userage")
    private Integer userage;

    @ManyToOne(cascade = CascadeType.PERSIST)//开启级联操作
    @JoinColumn(name = "roles_id")//此处用roles_id维护外键关系，多方设置外键
    private Roles roles;



}

```

创建实体roles

```java
package com.eim.pojo;

import lombok.Data;

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

/**
 * Created by Administrator on 2019/3/2.
 */
//@Data
@Entity
@Table(name = "t_roles")
public class Roles {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "roleid")
    private Integer roleid;

    @Column(name = "rolename")
    private String rolename;


    @OneToMany(mappedBy = "roles")//放那些用户对象：private Roles roles;
    private Set<Users> users = new HashSet<>();


}

```

测试代码：

```java
 /**
     * 添加用户同时添加角色
     */
    @Test
    public void test01(){
        //创建角色
        Roles roles = new Roles();
        roles.setRolename("7超级管理员");
        //创建用户
        Users users = new Users();
        users.setUsername("7lisi");
        users.setUserage(16);
        //建立关系
        roles.getUsers().add(users);
        users.setRoles(roles);
        //保存数据
        repository.save(users);
    }
```

一对多查询：

```java
    /**
     *根据用户ID查询用户信息，同时查询角色
     */
    @Test
    public void test03(){
        Users one = repository.findOne(5);
        System.out.println(one);
        System.out.println(one.getRoles());
    }
    
//结果
Users{userid=5, username='4无极', userage=88}
Roles{roleid=5, rolename='4超级管理员'}
```

## ③多对多的关联关系

需求：一个角色可以拥有多个菜单，一个菜单可以分配多个角色，多对多的关联关系。

角色：多方

菜单：多方

创建实体类：

角色：

```java
package com.eim.pojo;

import com.fasterxml.jackson.annotation.JsonIdentityInfo;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.ObjectIdGenerators;
import lombok.Data;
import org.hibernate.annotations.Proxy;
import org.springframework.context.annotation.Lazy;

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

/**
 * Created by Administrator on 2019/3/2.
 */
@Data
@Entity
@Table(name = "t_roles")
public class Roles {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "roleid")
    private Integer roleid;

    @Column(name = "rolename")
    private String rolename;

//设置为积极加载
    @ManyToMany(mappedBy = "set",fetch = FetchType.EAGER)
    private Set<Menus> set = new HashSet<>();

}

```

菜单：

```java
package com.eim.pojo;

import lombok.Data;

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

/**
 * Created by Administrator on 2019/3/3.
 */
@Data
@Entity
@Table(name = "t_menus")
public class Menus {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "menusid")
    private Integer menusid;

    @Column(name = "menusname")
    private String menusname;

    @Column(name = "menusurl")
    private String menusurl;

    @Column(name = "fatherid")
    private Integer fatherid;

  //设置为积极加载
    @ManyToMany(cascade = CascadeType.PERSIST,,fetch = FetchType.EAGER)
    //JoinTable指定多对多中间表如何维护外键
    @JoinTable(name = "t_menus_roles",
            joinColumns = @JoinColumn(name = "menus_id"),
            inverseJoinColumns = @JoinColumn(name = "roles_id"))
    private Set<Roles> set = new HashSet<>();
}
```

操作多对多测试：

```java
    @Test
    public void test01(){
        //创建角色
        Roles roles = new Roles();
        roles.setRolename("多多");

        Roles roles2 = new Roles();
        roles2.setRolename("吉吉");

        //创建菜单
        Menus menus = new Menus();
        menus.setFatherid(12);
        menus.setMenusname("meu多多");
        menus.setMenusurl("http");

        //建立关系
        menus.getSet().add(roles);
        menus.getSet().add(roles2);

        roles.getSet().add(menus);

        roles2.getSet().add(menus);


        Menus save = menusRepository.save(menus);

        Assert.assertNotNull(save);
    }
```

查询多对多：

```java
    @Test
    public void test02(){
        Menus one = menusRepository.findOne(1);
        System.out.println(one);
        Set<Roles> set = one.getSet();
        for(Roles roles:set){
            System.out.println(roles);
        }

    }

查询结果：
Menus{menusid=1, menusname='meu多多', menusurl='http', fatherid=12}
Roles{roleid=10, rolename='吉吉'}
Roles{roleid=9, rolename='多多'}

```

## ④碰到的异常

使用Gson将对象转换成字符串的时候报java.lang.StackOverflowError

**原因**：该对象A中有一个有一个组件B，组件B中有这个对象A的属性（==关联查询一对多==），而造成了一个循环 
解决办法：去掉组件B中A的属性，这样做就不能通过B获取A了但是可以正常的转换JSON

**解决方法**：设置组件B为null,即可转换成功

# 3、事务

Spring Data 提供了默认的事务处理方式，即所有的查询均声明为`只读事务`。
对于自定义的方法，如需改变 Spring Data 提供的事务默认方式，可以在方法上注解 @Transactional 声明 
进行多个 Repository 操作时，也应该使它们在同一个事务中处理，按照分层架构的思想，这部分属于业务逻辑层，因此，==需要在 Service 层实现对多个 Repository 的调用，并在相应的方法上声明事务。==   

