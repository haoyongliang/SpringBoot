# Spring Boot技术栈(Spring Data JPA)

 - 本篇介绍 Spring Data JPA

# 1.JPA介绍

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
## 2.修改application.properties配置文件

### 添加配置
<pre>
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/heima
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql= true
</pre>

### 配置文件说明

1. spring.datasource.url : 数据库连接URL
2. spring.datasource.username : 数据库帐号
3. spring.datasource.password : 数据库密码
4. pring.datasource.driver-class-name : 驱动名
	- mysql的驱动：com.mysql.jdbc.Driver
	- oracle的驱动：oracle.jdbc.OracleDriver
	- sqlserver的驱动：com.microsoft.sqlserver.jdbc.SQLServerDriver
5. spring.jpa.properties.hibernate.hbm2ddl.auto : 自动创建 | 更新 | 验证数据库表结构
	- create : 每次加载hibernate，如果数据库中存在表，将所有表删除，然后重新生成表
	- update : 每次hibernate 时根据 model 类自动更新表结构，如果是第一次则创建表结构。之前表数据不会删除
	- validate : 设置为validate:加载hibernate时，验证创建数据库表结构，这样 spring在加载之初，如果model层和数据库表结构不同，就会报错，这样有助于技术运维预先发现问题。例如：ProductInfoEntity这个实体有property1这个属性，而对应的数据库表product没有property1这个字段，就会在tomcat启动的时候报错：错误可能如下：Missing column: property1 in wjs.product
	- create-drop : 如果一开始数据库没有表，启动tomcat的时候会生成表，当把tomcat关闭之后生成的表又会消除
6. spring.jpa.properties.hibernate.dialect ： 告诉Hibernate，将HQL翻译成哪种数据库的SQL,常见的有
	- mysql : org.hibernate.dialect.MySQLDialect
	- oracle : org.hibernate.dialect.OracleDialect
	- sqlserver : org.hibernate.dialect.SQLServerDialect
7. show-sql ： 是否在控制台打印SQL语句，建议调试时启用，方便调试。
 
## 3.创建实体类

 - 注意：
	 - 实体类在命名时候不要用数据库中的关键字比如Order,可以定义成Orders
	 - Entity 中不映射成列的字段得加 @Transient 注解，不加注解也会映射成列
	 
<pre>
/**
 * 账户
 */
@Entity
public class Account implements Serializable{
    @Id
    @GeneratedValue
    private Long id;//ID主键
    @Column(nullable = false,unique = true)
    private String username;//用户名,不能为空，不能重复
    @Column(nullable = false)
    private String password;//密码，不能为空
    @Column(nullable = false)
    private String gender;    //性别，不能为空
    @Column()
    private String address; //地址
    @Column()
    private double balance;//账户余额

    public Account(String username, String password, String gender, String address, double balance) {
        this.username = username;
        this.password = password;
        this.gender = gender;
        this.address = address;
        this.balance = balance;
    }

    //省略无参构造,getter,setter方法
}

</pre>

## 4.创建DAO

 Dao 只要继承 JpaRepository 类就可以，几乎可以不用写方法，还有一个特别有个性的功能非常赞，就是可以根据方法名来自动的生产 SQL，如 findByUserName 会自动生产一个以 userName 为参数的查询方法，如 findAll 自动会查询表里面的所有数据，如自动分页等等

<pre>
public interface AccountRepository extends JpaRepository&lt;Account,Long&gt; {
    Account findAccountByUsername(String username);
    List&lt;Account&gt; findAllByAddressLike(String address);//比如查询地址包含"西"的 address的值="%西%"
    List&lt;Account&gt; findAllByBalanceGreaterThanEqual(double balance);//余额大于等于指定金额
    List&lt;Account&gt; findAllByBalanceGreaterThan(double balance);//余额大于指定金额
}
</pre>

 - 说明：JpaRepository&lt;T,ID&gt;这个接口只是一个空的接口，目的是为了统一所有Repository的类型，其接口类型使用了泛型，泛型参数中T代表实体类型，ID则是实体中id的类型

## 5.编写测试代码

<pre>
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
</pre>

# 3.基本查询

 基本查询也分为两种，一种是 Spring Data 默认已经实现，一种是根据查询的方法来自动解析成 SQL。

## 1.预先生成方法

 > Spring Data JPA 默认预先生成了一些基本的 CURD 的方法，如增、删、改等。

 继承 JpaRepository：

<pre>
public interface UserRepository extends JpaRepository<User, Long> {
}
</pre>

 测试

<pre>
@Test
public void testBaseQuery() {
    userRepository.findAll();
    userRepository.findOne(1l);
    userRepository.save(user);
    userRepository.delete(user);
    userRepository.count();
    userRepository.exists(1l);
    // ...
}
</pre>

## 2.自定义简单查询

 > 自定义的简单查询就是根据方法名来自动生成 SQL，主要的语法是 findXXBy、readAXXBy、queryXXBy、countXXBy、getXXBy 后面跟属性名称：

 <pre>User findByUserName(String userName);</pre>

 > 也可以加一些关键字 And、Or：

 <pre>User findByUserNameOrEmail(String username, String email);</pre>

 > 修改、删除、统计也是类似语法：

<pre>
Long deleteById(Long id);
Long countByUserName(String userName)
</pre>
 
 > 基本上 SQL 体系中的关键词都可以使用，如 LIKE、IgnoreCase、OrderBy。

<pre>
List<User> findByEmailLike(String email);
User findByUserNameIgnoreCase(String userName);
List<User> findByUserNameOrderByEmailDesc(String email);
</pre>

> 具体的关键字，使用方法和生产成 SQL 如下表所示：

<table style="border-collapse: collapse; border-spacing: 0;background-color: transparent;">
<thead>
<tr>
<th>Keyword</th>
<th>Sample</th>
<th>JPQL snippet</th>
</tr>
</thead>
<tbody>
<tr>
<td>And</td>
<td>findByLastnameAndFirstname</td>
<td>… where x.lastname = ?1 and x.firstname = ?2</td>
</tr>
<tr>
<td>Or</td>
<td>findByLastnameOrFirstname</td>
<td>… where x.lastname = ?1 or x.firstname = ?2</td>
</tr>
<tr>
<td>Is,Equals</td>
<td>findByFirstnameIs,findByFirstnameEquals</td>
<td>… where x.firstname = ?1</td>
</tr>
<tr>
<td>Between</td>
<td>findByStartDateBetween</td>
<td>… where x.startDate between ?1 and ?2</td>
</tr>
<tr>
<td>LessThan</td>
<td>findByAgeLessThan</td>
<td>… where x.age &lt; ?1</td>
</tr>
<tr>
<td>LessThanEqual</td>
<td>findByAgeLessThanEqual</td>
<td>… where x.age ⇐ ?1</td>
</tr>
<tr>
<td>GreaterThan</td>
<td>findByAgeGreaterThan</td>
<td>… where x.age &gt; ?1</td>
</tr>
<tr>
<td>GreaterThanEqual</td>
<td>findByAgeGreaterThanEqual</td>
<td>… where x.age &gt;= ?1</td>
</tr>
<tr>
<td>After</td>
<td>findByStartDateAfter</td>
<td>… where x.startDate &gt; ?1</td>
</tr>
<tr>
<td>Before</td>
<td>findByStartDateBefore</td>
<td>… where x.startDate &lt; ?1</td>
</tr>
<tr>
<td>IsNull</td>
<td>findByAgeIsNull</td>
<td>… where x.age is null</td>
</tr>
<tr>
<td>IsNotNull,NotNull</td>
<td>findByAge(Is)NotNull</td>
<td>… where x.age not null</td>
</tr>
<tr>
<td>Like</td>
<td>findByFirstnameLike</td>
<td>… where x.firstname like ?1</td>
</tr>
<tr>
<td>NotLike</td>
<td>findByFirstnameNotLike</td>
<td>… where x.firstname not like ?1</td>
</tr>
<tr>
<td>StartingWith</td>
<td>findByFirstnameStartingWith</td>
<td>… where x.firstname like ?1 (parameter bound with appended %)</td>
</tr>
<tr>
<td>EndingWith</td>
<td>findByFirstnameEndingWith</td>
<td>… where x.firstname like ?1 (parameter bound with prepended %)</td>
</tr>
<tr>
<td>Containing</td>
<td>findByFirstnameContaining</td>
<td>… where x.firstname like ?1 (parameter bound wrapped in %)</td>
</tr>
<tr>
<td>OrderBy</td>
<td>findByAgeOrderByLastnameDesc</td>
<td>… where x.age = ?1 order by x.lastname desc</td>
</tr>
<tr>
<td>Not</td>
<td>findByLastnameNot</td>
<td>… where x.lastname &lt;&gt; ?1</td>
</tr>
<tr>
<td>In</td>
<td>findByAgeIn(Collection&lt;Age&gt; ages)</td>
<td>… where x.age in ?1</td>
</tr>
<tr>
<td>NotIn</td>
<td>findByAgeNotIn(Collection&lt;Age&gt; age)</td>
<td>… where x.age not in ?1</td>
</tr>
<tr>
<td>TRUE</td>
<td>findByActiveTrue()</td>
<td>… where x.active = true</td>
</tr>
<tr>
<td>FALSE</td>
<td>findByActiveFalse()</td>
<td>… where x.active = false</td>
</tr>
<tr>
<td>IgnoreCase</td>
<td>findByFirstnameIgnoreCase</td>
<td>… where UPPER(x.firstame) = UPPER(?1)</td>
</tr>
</tbody>
</table>

## 复杂查询

在实际的开发中需要用到分页、筛选、连表等查询的时候就需要特殊的方法或者自定义 SQL。

### 1.分页查询

 > 分页查询在实际使用中非常普遍了，Spring Data JPA 已经帮我们实现了分页的功能，在查询的方法中，需要传入参数 Pageable，当查询中有多个参数的时候 Pageable 建议做为最后一个参数传入：


 在AccountRepository类中添加
<pre>
Page&lt;Account&gt; findAllByAddressLike(String address, Pageable pageable);
</pre>

 测试代码
<pre>
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
</pre>

 输出结果
<pre>
一共有2页
一共有6条记录
本页的数据是有
张五丰	山西忻州
张三丰	山西太原
张四丰	山西临汾
张八丰	山西xinzhou
</pre>

### 2.限制查询

 > 有时候我们只需要查询前 N 个元素，或者只取前一个实体。

<pre>
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
</pre>

### 3.自定义 SQL 查询

 > 用 Spring Data 大部分的 SQL 都可以根据方法名定义的方式来实现，但是由于某些原因我们想使用自定义的 SQL 来查询，Spring Data 也可以完美支持；在 SQL 的查询方法上面使用 @Query 注解，如涉及到删除和修改需要加上 @Modifying，也可以根据需要添加 @Transactional 对事物的支持，查询超时的设置等。