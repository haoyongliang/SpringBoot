# Spring Boot技术栈(Spring Data JPA)

> 本篇介绍 通过Spring Data JPA对数据库进行常见的操作:删除、修改、添加、分页查询、单表查询、多表查询。

# 1.Spring Data JPA介绍

 - Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套 JPA 应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！
 - Spring Data JPA 让我们解脱了 DAO 层的操作，基本上所有 CRUD 都可以依赖于它来实现。

# 2.开发环境搭建

## 1.添加依赖
```

	<dependency>
		<groupId>org.Springframework.boot</groupId>
	    <artifactId>Spring-boot-starter-data-jpa</artifactId>
	</dependency>
	 <dependency>
	    <groupId>mysql</groupId>
	    <artifactId>mysql-connector-java</artifactId>
	</dependency>

```
## 2.修改application.properties

添加配置

```
#数据库连接URL
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/heima
#数据库帐号
spring.datasource.username=root
#数据库密码
spring.datasource.password=root
#驱动名 
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#每次hibernate 时根据 model 类自动更新表结构
spring.jpa.properties.hibernate.hbm2ddl.auto=update
#告诉Hibernate，将HQL翻译成哪种数据库的SQL
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
#是否在控制台打印SQL语句,测试时设置为true
spring.jpa.show-sql= true
```

配置文件说明

> 1. spring.jpa.properties.hibernate.hbm2ddl.auto : 自动创建 | 更新 | 验证数据库表结构
>    - create : 每次加载hibernate，如果数据库中存在表，将所有表删除，然后重新生成表
>    - update 每次hibernate 时根据 model 类自动更新表结构，如果是第一次则创建表结构。之前表数据不会删除
>    - validate : 设置为validate:加载hibernate时，验证创建数据库表结构，这样 spring在加载之初，如果model层和数据库表结构不同，就会报错，这样有助于技术运维预先发现问题。例如：ProductInfoEntity这个实体有property1这个属性，而对应的数据库表product没有property1这个字段，就会在tomcat启动的时候报错：错误可能如下：Missing column: property1 in wjs.product
>    - create-drop : 如果一开始数据库没有表，启动tomcat的时候会生成表，当把tomcat关闭之后生成的表又会消除
> 2. spring.jpa.properties.hibernate.dialect ： 告诉Hibernate，将HQL翻译成哪种数据库的SQL,常见的有
>    - mysql : org.hibernate.dialect.MySQLDialect
>    - oracle : org.hibernate.dialect.OracleDialect
>    - sqlserver : org.hibernate.dialect.SQLServerDialect

## 3.创建实体类

 - 注意：
    - 实体类在命名时候不要用数据库中的关键字比如Order,可以定义成Orders
     - Entity 中不映射成列的字段得加 @Transient 注解，不加注解也会映射成列

```
/**
	账户
*/
@Entity
public class Account implements Serializable{
    @Id
    @GeneratedValue
    private Long id;		//ID主键
    @Column(nullable = false,unique = true)
    private String username;//用户名,不能为空，不能重复
    @Column(nullable = false)
    private String password;//密码，不能为空
    @Column(nullable = false)
    private String gender;  //性别，不能为空
    @Column()
    private String address; //地址
    @Column()
    private double balance;	//账户余额
    
    public Account(String username, String password, String gender, String address, double balance) {
        this.username = username;
        this.password = password;
        this.gender = gender;
        this.address = address;
        this.balance = balance;
    }
    //省略无参构造,getter,setter方法
}

```



## 4.创建DAO

 Dao 只要继承 JpaRepository 类就可以，几乎可以不用写方法，还有一个特别有个性的功能非常赞，就是可以根据方法名来自动的生产 SQL，如 findByUserName 会自动生产一个以 userName 为参数的查询方法，如 findAll 自动会查询表里面的所有数据，如自动分页等等

```
public interface AccountRepository extends JpaRepositoryAccount,Long {
    Account findAccountByUsername(String username);
    List<Account> findAllByAddressLike(String address);//比如查询地址包含"西"的 address的值="%西%"
    List<Account> findAllByBalanceGreaterThanEqual(double balance);//余额大于等于指定金额
    List<Account> findAllByBalanceGreaterThan(double balance);//余额大于指定金额
}
```



 - 说明：JpaRepository&lt;T,ID&gt;这个接口只是一个空的接口，目的是为了统一所有Repository的类型，其接口类型使用了泛型，泛型参数中T代表实体类型，ID则是实体中id的类型

## 5.编写测试代码

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class AccountRepositoryTests {
    @Autowired
    private AccountRepository accountRepository;
    @Test
    public void test()  {
    accountRepository.deleteAll();
    accountRepository.save(new Account("张三丰","123456","男","山西太原",5000.5));
    accountRepository.save(new Account("张四丰","123456","男","山西临汾",1000));
    accountRepository.save(new Account("张五丰","123456","男","山西忻州",1000));
    accountRepository.save(new Account("小明","123456","女","河北承德",3000));
    accountRepository.save(new Account("小丽","123456","女","河北保定",9000.5));

    Assert.assertEquals(5,accountRepository.findAll().size());//如果查询结果等于5
    Assert.assertEquals(3,accountRepository.findAllByAddressLike("%西%").size());//地址中包含西的记录
    Assert.assertEquals(3,accountRepository.findAllByBalanceGreaterThanEqual(3000).size());//大于等于3000的余额的记录
    Assert.assertEquals(2,accountRepository.findAllByBalanceGreaterThan(3000).size());//大于3000的余额的记录
    }

}
```

##6.运行时报错及解决方案

可能出现异常：

1. Caused by: java.lang.IllegalArgumentException: Not a managed type: class com.nsun.study.dao.model.UserInfo
  at org.hibernate.jpa.internal.metamodel.MetamodelImpl.managedType(MetamodelImpl.java:210)
2. Consider defining a bean of type '*Repository' in your configuration. 

原因：启动时没有扫描实体类和DAO,导致没有注入

解决方案：在Application 增加注解(测试类也添加)

```
@SpringBootApplication
@EntityScan(basePackages="cn.itcast.sprintBootDemo.domain")
@EnableJpaRepositories(basePackages = {"cn.itcast.sprintBootDemo.repository"})
@RunWith(SpringRunner.class)
public class SprintBootDemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(SprintBootDemoApplication.class, args);
	}
}
```



# 3.常见数据库操作

> 基本查询也分为两种，一种是 Spring Data 默认已经实现，一种是根据查询的方法来自动解析成 SQL。

## 1.基本查询

 > Spring Data JPA 默认预先生成了一些基本的 CURD 的方法，如增加，删除，修改，查询

 继承 JpaRepository：

```
public interface AccountRepository extends JpaRepository<Account,Long> {

}
```

 测试

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class AccountRepositoryTests {

    @Autowired
    private AccountRepository accountRepository;

    @Test
    public void testBaseQuery(){
        /*添加测试数据数据*/
        accountRepository.deleteAll();
        accountRepository.save(new Account("张三丰","123456","男","山西太原",5000.5));
        accountRepository.save(new Account("张四丰","123456","男","山西临汾",1000));
        
        /*通过findAll方法查询所有记录*/
        List<Account> accounts = accountRepository.findAll();
        for(Account account : accounts){
            System.out.println(account);
        }
        
        /*通过findOne查询一条记录,传入ID*/
        Account one = accountRepository.findOne(accountRepository.findAll().get(0).getId());
        System.out.println(one.getUsername());
        
        /*查询总记录条数*/
        long count = accountRepository.count();
        System.out.println(count);
        
        /*查询某条记录是否存在*/
        boolean exists = accountRepository.exists(accountRepository.findAll().get(0).getId());
        System.out.println(exists);
        
        /*更新记录,应该先查询到旧的记录在调用save方法，save方法的功能有两个：更新和添加*/
        Account account = accountRepository.findAll().get(0);
        account.setBalance(10000.4);
        accountRepository.save(account);
    }
}
```



## 2.自定义简单查询

 > 自定义的简单查询就是根据方法名来自动生成 SQL，主要的语法是 findXXBy、readAXXBy、queryXXBy、countXXBy、getXXBy 后面跟属性名称：

```
User findByUsername(String userName);
```

 > 也可以加一些关键字 And、Or：

```
 User findByUsernameOrAddress(String username, String address);
```

 > 修改、删除、统计也是类似语法：

```
Long deleteById(Long id);
Long countByUsername(String userName)
```

 > 基本上 SQL 体系中的关键词都可以使用，如 LIKE、IgnoreCase、OrderBy。

```
List<User> findByAddressLike(String address);
User findByAddressIgnoreCase(String address);
List<User> findByAddressOrderByBalanceDesc(String address);
```

> 具体的关键字，使用方法和生产成 SQL 如下表所示：

| Keyword           | Sample                                  | JPQL snippet                             |
| :---------------- | :-------------------------------------- | ---------------------------------------- |
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
| In                | findByAgeIn(Collection<Age> ages)       | … where x.age in ?1                      |
| NotIn             | findByAgeNotIn(Collection<Age> age)     | … where x.age not in ?1                  |
| TRUE              | findByActiveTrue()                      | … where x.active = true                  |
| FALSE             | findByActiveFalse()                     | … where x.active = false                 |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)    |



## 3.分页查询

 > 分页查询在实际使用中非常普遍了，Spring Data JPA 已经帮我们实现了分页的功能，在查询的方法中，需要传入参数 Pageable，当查询中有多个参数的时候 Pageable 建议做为最后一个参数传入：


 在AccountRepository类中添加

```
PageAccount findAllByAddressLike(String address, Pageable pageable);
```

 测试代码

```
@Test
public void pageTest(){
    accountRepository.deleteAll();
    accountRepository.save(new Account("张三丰","123456","男","山西太原",5000.5));
    accountRepository.save(new Account("张四丰","123456","男","山西临汾",1000));
    accountRepository.save(new Account("张五丰","123456","男","山西忻州",1000));
    accountRepository.save(new Account("张六丰","123456","女","山西taiyuan",3000));
    accountRepository.save(new Account("张七丰","123456","女","山西linfen",9000.5));
    accountRepository.save(new Account("张八丰","123456","女","山西xinzhou",9000.5));
    
    int currentPage = 0;//第几页,从0开始
    int size = 4;//每页多少条数据
    Sort sort = new Sort(Sort.Direction.DESC, "address");//按照address降序排序
    Pageable pageable = new PageRequest(currentPage, size, sort);
    
    Page<Account> page = accountRepository.findAllByAddressLike("%西%",pageable);
    System.out.println("一共有"+page.getTotalPages()+"页");
    System.out.println("一共有"+page.getTotalElements()+"条记录");
    System.out.println("本页的数据是有");
    for(Account account : page){
        System.out.println(account.getUsername()+"\t"+account.getAddress());
    }
}
```



 输出结果

```
一共有2页
一共有6条记录
本页的数据是有
张五丰	山西忻州
张三丰	山西太原
张四丰	山西临汾
张八丰	山西xinzhou
```



## 4.查询前N条数据

 > 有时候我们只需要查询前 N 个元素，或者只取前一个实体。可以在定义接口的时候使用firstBy,topBy,first4By,top4By

1.在AccountRepository接口中添加新方法

```
/**查询前三个*/
List<Account> findTop3ByOrderByBalanceDesc();
/**查询余额最高的*/
Account findFirstByOrderByBalanceDesc();
/**查询余额最高的*/
Account findTopByOrderByBalanceDesc();
```

2.测试代码

    @Test
    public void testTop(){
        accountRepository.deleteAll();
        accountRepository.save(new Account("张三丰","123456","男","山西太原",5000.5));
        accountRepository.save(new Account("张四丰","123456","男","山西临汾",1000));
        accountRepository.save(new Account("张五丰","123456","男","山西忻州",1000));
        accountRepository.save(new Account("小明","123456","女","河北承德",3000));
        accountRepository.save(new Account("小丽","123456","女","河北保定",9000.5));
    
        //查询前3条记录
        List<Account> accounts = accountRepository.findTop3ByOrderByBalanceDesc();
        for (Account account: accounts) {
        	System.out.println(account.getBalance());
        }
        
    	/**查询余额最高的*/
        Account firstAccount = accountRepository.findFirstByOrderByBalanceDesc();
        System.out.println(firstAccount);
        
    	/**查询余额最高的*/
        Account topAccount = accountRepository.findTopByOrderByBalanceDesc();
        System.out.println(topAccount);
    }


## 5.自定义 SQL 查询

 > Spring Data JPA 也可以完美支持自定义SQL语句操作：
 >
 > 1.在 SQL 的查询方法上面使用 @Query 注解。
 >
 > 2.1如果需要传参可以在SQL中通过":变量名"设置占位符，然后配合注解@Param("变量名")一起使用
 >
 > 2.2也可以使用？占位符，但是后面要跟1表示第一个参数如果多个问号只需要在每个?后面跟对应的数字就行了，从1开始
 >
 > 3.特别注意这里的 SQL 是 HQL，需要写类的名和属性，这块很容易出错。

1.在AccountRepository接口中添加新方法(使用 ":变量名" 的方式)

```
@Query("select a from Account a where a.username = :name")
Account selectByUsername(@Param("name") String username);

@Query("select a from Account a where a.address like %:address%")
ListAccount selectByAddressLike(@Param("address") String address);

```

2.在AccountRepository接口中添加新方法(使用 "?" 的方式)

```
@Query("select a from Account a where a.address like ?1")
ListAccount selectByAddressLike(String address);
```

3.测试代码

```
@Test
public void userQuery(){
    //通过username查找对应的账户
    Account zsf = accountRepository.selectByUsername("张三丰");
    System.out.println(zsf.getUsername()+"-"+zsf.getGender());

    //查找地址中包含"西"的账户
    List<Account> accounts = accountRepository.selectByAddressLike("西");
    for (Account account: accounts) {
    System.out.println(account.getAddress());
    }
}
```



## 6.自定义SQL 更新或删除数据

> 提示：更新或者删除的时候需要添加@Modifying否则报 Not supported for DML operations 异常，也需要添加@Transactional事物注解，否则报InvalidDataAccessApiUsageException异常

1.在AccountRepository接口中添加方法

```
@Query("update Account a set password = :password where username=:username")
@Modifying
@Transactional
int updatePassswordByUsername(@Param("username") String username, @Param("password") String password);
```

2.测试

```
@Test
public void userUpdateOrDelete(){
    accountRepository.deleteAll();
    accountRepository.save(new Account("张三丰","123456","男","山西太原",5000.5));
    accountRepository.save(new Account("张四丰","123456","男","山西临汾",1000));
    accountRepository.save(new Account("张五丰","123456","男","山西忻州",1000));
    accountRepository.save(new Account("小明","123456","女","河北承德",3000));
    accountRepository.save(new Account("小丽","123456","女","河北保定",9000.5));

    int rows = accountRepository.updatePassswordByUsername("张三丰","88888888");
    System.out.println("更新了"+rows+"行数据");
}
```

3.控制台结果

```
更新了1行数据
```



## 7.一对多关系(oneToMany)

需要在代表一的地方添加@OneToMany和@JoinColumn来标注(原理是在多的一方的表中增加一个外键列来保存关系)。代表多的实体不需要使用任何映射标注。

如以下关系：班级和学生，一个个班级对应多个学生，班级就是一的一方，学生就是多的一方

> Tips:cascade = CascadeType.ALL这个在后面讲解

> 班级代码 代表一的一方

```
@Entity
public class Classes {
    @Id
    @GeneratedValue
    private Long id;
    @Column(nullable = false,unique = true)
    private String name;

    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "classes_id")//表示Student表中指向本表外键名
    private List<Student> students;
}
```

> 学生代码 代表多的一方

```
@Entity
public class Student {
    @Id
    @GeneratedValue
    private Long id;
    @Column(nullable = false,unique = true)
    private String name;
}
```

> 测试代码：保存班级的同时保存学生

```
@Test
public void saveTest(){
	/*创建班级*/
    Classes bj = new Classes();
    bj.setName("一年级二班");
    /*创建学生*/
    Student stu1 = new Student();
    stu1.setName("小明");
    Student stu2 = new Student();
    stu2.setName("小红");
    
	/*将学生添加到班级中*/
    ArrayList<Student> students = new ArrayList<>();
    students.add(stu1);
    students.add(stu2);
    bj.setStudents(students);
    
 	//保存班级,同时会级联保存学生
    classesRepository.save(bj);
}
```

> 生成表的SQL

```
Hibernate: create table classes (id integer not null auto_increment, name varchar(255), primary key (id))
Hibernate: create table student (id integer not null auto_increment, name varchar(255), class_id integer, primary key (id))
Hibernate: alter table student add constraint FK5v50ed2bjh60n1gc7ifuxmgf4 foreign key (class_id) references classes (id)

```

> 插入数据的SQL

```
Hibernate: insert into classes (name) values (?)
Hibernate: insert into student (name) values (?)
Hibernate: insert into student (name) values (?)
Hibernate: update student set class_id=? where id=?
Hibernate: update student set class_id=? where id=?
```



## 8.多对一关系(ManyToOne)

需要在多的一方添加@ManyToOne

这里只需要在Student类中添加属性private Classes classes;生成get/set方法即可,并且在该属性上添加@ManyToOne(cascade={CascadeType.ALL})

> Tips:cascade = CascadeType.ALL这个在后面讲解

> 学生代码

```
@Entity
public class Student {
    @Id
    @GeneratedValue
    private Long id;
    @Column(nullable = false,unique = true)
    private String name;

    @ManyToOne(cascade={CascadeType.ALL})
    private Classes classes;
}
```

> 测试代码:查询所有学生对应的班级

```
@Test
public void testQuery(){
    List<Student> students = studentRepository.findAll();
    for(Student student: students){
        System.out.println(student.getClasses().getName());
    }
}
```
**可能产生问题:**

​	在controller返回数据到统一json转换的时候，出现了json infinite recursion stackoverflowerror的错误

​	具体的情况如下：

​	Classes类中，有个属性：private List<Student> students;， Classes与Student的关系为 OneToMany；在Student类中，有属性private Classes classes;,引用到Classes中的字段id，并作为外键。hibernate查询结果正常，可以看到返回的Classes对象中，有Student参数值，但在json转换的时候就出现了无限递归的情况。原因是json在序列化Classes中的students属性的时候，找到了Student类，然后序列化Student类，而Student类中有classes属性，因此，为了序列化classes属性，json又得去序列化Classes类，如此递归反复，造成该问题。

**解决方法：**

​	在Student类中classes的setter方法上加注解@JsonBackReference