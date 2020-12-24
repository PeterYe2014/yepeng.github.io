# Spring 技术笔记

## Day 1 预热知识

## 一、 基本术语

Blob类型，二进制对象

Object Graph：对象图，软件工程

AOP：

## 一、 常识

json 对象和 json字符串转换

JSON.parse(jsonstr);

JSON.stringify(jsonobj);

obj = eval(“(”+jsonStr+”)”);

## 1.2 classpath理解 

classpath指定java运行时需要的class字节码文件所在的位置。

eclipse中.classpath文件有三个作用：

1.源文件的具体位置（kind="src"）

2.运行的系统环境（kind="con"）

3.工程的library的具体位置信息(kind="lib")

4.项目的输出目录(kind="output")

同时Eclipse中Java项目的build path是依赖于.classpath文件的,build path中指定的位置就是classpath，两个任意一个变化会影响另一个。同时，Java Web项目中，有Java Resource这个选项，它会显示项目中的classpath指定的目录，但是它并非目录，它只是一个便于浏览源文件的一个选项。

问：spring boot 项目 classpath的设置？

 

## 二、  简单探索

### 2.1 Spring 容器

2.2.1 简介

Spring容器是Spring的核心组件，容器会创建对象，并且将对象组装在一起，使用容器来配置对象，管理对象的生命周期。Spring容器使用依赖注入（DI）来管理它的组件。容器会通过扫描metadata来实例化、配置以及组装哪些对象，metadata的类型可以是注解、XML文件或者Java代码。

Spring 提供了两个不同类型的容器，Spring Bean Factory Container 和 Spring ApplicationContext Container，后者包含前者所有的功能，但前者更轻量化。

​       2.2.2 组成

​       Core: 提供Spring框架的基础，包括IoC和 依赖注入特性

​       Bean：提供BeanFactory, 工厂模式的实现

​       Context：基于Core和Bean模块，访问已经配置和定义的对象的中间层。

​       SpEL：提供表达式语言用于查询和操作对象。

 

### 2.2 依赖注入（控制反转的一种）

## 2.2.1 概念

​       一个依赖就是可以被使用的对象（看成一个服务对象），注入就是将依赖传入给一个独立的对象（作为客户对象），只将服务对象传入客户端，而不新建该服务对象（不能new，不能有静态方法），是依赖注入核心。为何需要依赖注入？因为客户对象的依赖不是不变的，是会经常变换的（包括创建的方法），所有客户对象只要知道去哪里去拿服务对象，而不需要知道服务的具体创建方法。（减少程序的耦合）

​       同时客户对象有责任告诉injector（注入代码）自己的依赖是什么，因为，客户对象不能直接调用injector，而是injector创建服务后，通知客户对象注入服务。可以总结下，客户对象：

（1）       不知道注入代码

（2）       不知道服务的创建过程

（3）       不知道自己使用的是服务的哪个实例

（4）       只知道服务提供的接口即可

（5）       需要告诉injector自己需要注入的依赖

（6）       injector能够通知客户对象注入依赖。

## 2.2.2 客户对象接收依赖的方法[例子]

（0）没有使用依赖注入：

// An Example without dependency injection

​        *public class Client{*

​        

​        *private ServiceExample service;*

​        

​        *public Client(){*

​               *service = new ServiceExample();*

​        *}*

​        

​        *public String get(){*

​               

​               *return service.getName();*

​        *}*

*}*

（1）       构造函数来接收依赖，

​        *public Client(ServiceExample service){*

​                       *this.service = service;*

​        *}*       

（2）       setter方法接收依赖：

​        *public setService(ServiceExample service){*

​               *this.service = service;*

​        *}*

 

（3）       基于接口的接收方法：让依赖控制自己的注入

*// Service setter interface.*

**public** **interface** **ServiceSetter** {

​    **public** void setService(Service service);

}

 

*// Client class*

**public** **class** **Client** **implements** ServiceSetter {

​    *// Internal reference to the service used by this client.*

​    **private** Service service;

 

​    *// Set the service that this client is to use.*

​    @Override

​    **public** void setService(Service service) {

​        **this**.service = service;

​    }

}                  

服务端需要实现一个含有setService方法的接口。

## 2.2.2 三种方法对比

（1）构造函数，可以始终保持客户端是有效状态，但是缺少改变依赖的方法灵活性。由于服务是不可以变的所以是线程安全的。

（2）Setter，可以操作服务的状态，但是当依赖多了之后，就很难保证所有依赖已经注入。

​       由于这些注入可能有失败的情况（传入参数为null），所以还应该有一个检测注入是否完成的方法validateInjection

（3）接口方法，这个的有优点是，服务本身可以不清楚客户的构成，但是也可以接收一个客户参数，同时调用该客户实现的依赖接口方法，从而注入依赖。相当于依赖自己注入到客户中。

## 2.2.2 Spring Boot 配置的认识

通过@Configuration 注解类来表示该类为配置类，用于配置Spring Boot不能配置的一些Bean。如下例子：

**Spring****配置Bean****方法：**

<bean id="securityManager" class="org.apache.shiro.mgt.DefaultSecurityManager">

​    <!-- Single realm app.  If you have multiple realms, use the 'realms' property instead. -->

​    <property name="realm" ref="myRealm"/>

</bean>

**Spring Boot** **配置Bean****方法：（在Configuration****类中设置下面方法）**

@Bean(name="securityManager") // 注册返回值为Bean

public SecurityManager securityManager(@Qualifier("authRealm") AuthRealm authRealm) 

{

​      System.err.println("--------------shiro已经加载----------------");

​      DefaultWebSecurityManager manager=new DefaultWebSecurityManager();

​      manager.setRealm(authRealm);

​      return manager;

}

 

## 2.2.3  Spring boot 依赖注入

Spring 通过@ComponentScan来扫描当前注解类目录中的所有组件，并且为这些组件注册Bean。

A 使用标准的Spring 注解来注入

（1）@Autowired 自动从spring 上下文找bean来注入

（2）@ Resource用来指定名称注入（有name属性，默认为字段名）

（3）@Qualifier和@Autowired配合使用，指定bean的名称

同时@Qualifier可以用来注入接口时来区别到底绑定哪一个实现（通过实现类的名字来绑定）

@Resource和@Autowired区别？

byName 通过参数名 自动装配，如果一个bean的name 和另外一个bean的 property 相同，就自动装配。

byType 通过参数的数据类型自动自动装配，如果一个bean的数据类型和另外一个bean的property属性的数据类型兼容，就自动装配

作者：wuxinliulei

链接：https://www.zhihu.com/question/39356740/answer/80926247

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

（4）@Service，@Controller，@Repository分别标记类是Service层类，Controller层类，数据存储层的类

（5）@Component是一种泛指，标记类是组件，spring扫描注解配置时，会标记这些类要生成bean。

 

B. 使用Spring Boot的自动注入

​       （1）声明注入的对象

​       （2）创建以该对象为参数的构造函数

private final RiskAssessor riskAssessor;

​    public DatabaseAccountService(RiskAssessor riskAssessor) {

​        this.riskAssessor = riskAssessor;

​    }

 

 

## Day 2  Spring Boot Basic

一、        pom.xml配置

一个简单的Spring Boot项目继承 spring-boot-starter-parent，同时依赖spring-boot-starter-web(支持MVC等web特性)。（含有starter组件是一批jar的打包）同时我们要设置build插件 org.springframework.boot，这个插件特别方便：

（1）       将所有的classpath下的jar合并成一个über-jar

（2）       搜索public static void main方法，来启动应用

（3）       内置依赖解析器，设置和Spring Boot相应的的依赖版本号。

同时，依靠spring-boot-maven-plugin可以创建可执行的jar或者war包。可以在build标签中添加该插件。

*<plugin>*

​                                     *<groupId>org.springframework.boot</groupId>*

​                                     *<artifactId>spring-boot-maven-plugin</artifactId>*

​                                     *<configuration>*

​                                               *<executable>true</executable>*

​                                     *</configuration>*

*</plugin>*

我们在pom.xml有如下关键配置：

*<parent>*

​         *<groupId>org.springframework.boot</groupId>*

​         *<artifactId>spring-boot-starter-parent</artifactId>*

​         *<version>1.5.2.RELEASE</version>*

*</parent>*

*<dependencies>*

​         *<dependency>*

​                  *<groupId>org.springframework.boot </groupId>*

​                   *<artifactId>* *spring-boot-starter </artifactId>*

​         *</dependency>*

*</dependencies>*

二、 注解设置

​         省略

三、启动入口

​         // 配置入口

​         新建一个含有main函数的类Application，函数内调用SpringApplication.run(Application.class,args);同时用注解@SpringBootApplication标记类Application。

@SpringBootApplication该注解等效于三个注解：

​         @Configuration, 标记某个类是bean定义的来源，该类起配置作用（Spring Boot推荐使用Java来配置而不是XML来配置）

@EnableAutoConfiguration 根据你添加的jar依赖来自动配置Spring的bean，也可以自己添加需要的

@ComponentScan 在这个类所在的目录里面搜索组件（用于获取注解或者配置）

通过 mvn spring-boot:run 启动该类，会启动程序，直接可以访问localhost:8080。

启动加上参数 - - debug 可以输出细节的启动日志。

Application类的位置有一定要求，最好如下设置为项目根目录里面：

com

 +- example

​     +- myproject

​         +- Application.java

​         |

​         +- domain

​         |   +- Customer.java

​         |   +- CustomerRepository.java

​         |

​         +- service

​         |   +- CustomerService.java

​         |

​         +- web

​             +- CustomerController.java

## Day 3  Spring MVC

一、          基本注解

@Controller 标记某个内类为控制类，该类的方法可以用@RequestMapping编辑请求映射,同时可以返回View以及数据。（ModelAndView）

@RestController 与@Controller不同一点是，更方便返回JSON数据。

二、    静态文件处理

Spring Boot的默认静态文件目录为/static（/public or /resources）；同时 Spring Boot自动在静态文件中寻找favicon.ico文件作为网站的favicon，同时在application.properties文件也默认在静态目录下，该文件用来配置一些常见的参数，该配置文件会在例如：搜索的jsp在哪里

​       spring.mvc.view.prefix: /WEB-INF/jsp/

spring.mvc.view.suffix: .jsp

application.message: Hello Phil

​         SpringApplication 会加载application.properties文件，自动在下面的目录进行搜索。

（1）         当前目录下/config 子目录下进行搜索

（2）         当前目录

（3）         classpath下的/config目录

（4）         classpath的根目录

默认的classpath有：src/main/resources；src/main/java;

[?      为何不能放在自己定义的classpath中]

{？ 如何拦截jsp页面}

## Day 3  Spring With JDBC

一、        核心操作

包含spring-boot-starter-jdbc依赖以及相应的数据库组件。

   // 自动注入 JdbcTemplate 组件

*@Autowired*

*JdbcTemplate jdbcTemplate;*

​         *JdbcTemplate.excute(sql)* *执行**sql**语句*

​         *JdbcTemplate.batchUpdate(sql**，**list)**（**list**的单个元素要和**sql**的参数类型数目对应）*

​         *（批量插入）*

​         *JdbcTemplate.insert(sql,objarray)* *插入一个元素*

​         *JdbcTemplate.query(sql**，**interface(rs, rowNum));**查询**SQL**，同时返回**rs**对象和记录数目*

*Updating**包括* *(INSERT/UPDATE/DELETE)*

*execute()* *可以创建一个表*

*https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html*

 

二、高级特性



JdbcTemplate是通过 ？占位符来实现参数传递的，同时还有NamedParameterJdbcTemplate，通过参数名来指定参数的位置。

同时我们可以实现一些常用的回调函数来自定义查询：

PreparedStatementCreator： 创建自己的pr statement

new PreparedStatementCreator() {

​                            @Override

​            public PreparedStatement createPreparedStatement(

​                                               Connection connection){

}

 

ResultSetExtractor： 自己解析rs数据

new ResultSetExtractor<Object>() {

​                      @Override

​            public Object extractData(ResultSet rs) throws SQLException, DataAccessException {

​                                     if (rs.next()) {

​                                               Object object = rs.getObject(1);

​                                               return object;

​                                     }

​                                     return null;

}

三、JdbcTemplate DataSource的获取

jdbcTemplate 有一个构造函数 new JdbcTemplate(dataSource);

（1）bean.xml 中定义 dataSource，通过getBean注入

（2）通过注解：@Repository 以及   @Autowired 来注入

（3）继承JdbcDaoSupport，，就可以通过this. getJdbcTemplate()来获取相应的jdbcTemplate，bean.xml 中定义 dataSource 

 

## Day 4  Mybatis入门

一、        配置和依赖

依赖：

​              <dependency>

​                    <groupId>org.mybatis</groupId>

​                    <artifactId>mybatis</artifactId>

​                    <version>x.x.x</version>

</dependency>

Mybatis可以通过config.xml来配置（文件放在classpath下面），该文件结构如下：

<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE configuration

  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"

  "http://mybatis.org/dtd/mybatis-3-config.dtd"> // dtd声明

<configuration> // 根目录

​         <enviroments default=”development”> // 配置多个环境

​                  <environment id=”development”>

​                            <transactionManager type=”JDBC”/>

​                            <dataSource type=”POOLED”> // 数据库连接池

​                                     4个property标签配置:driver;url;username;password

​                            </dataSource>

​                   </environment>

​         </enviroments>

</configuration>

二、SqlSessionFactory的创建

​         通过config.xml文件来创建：

​         String resource = <xml文件包含包的路径>;

​         InputStream inputStream = Resource.getResourceAsStream(resource);

​         SqlSessionFactory sqlSessionFactory = SqlSessionFactoryBuilder.build(inputStream);

​         // 主要关键Resource获取文件输入流

​         // SqlSessionFactoryBuilder.通过输入流创建SqlSessionFactory

三、sql映射文件

​         3.1 XML映射

​         Mybatis的映射文件也可以是XML文件。该文件结构如下：

​         <?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper

​       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"

​       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

​         <mapper namespace=”org.mybatis.example.BlogMapper”> // root;namespace要唯一

​                            <select id=”selectBlog” resultType=”Blog”> // 定义sql statement

​                                     select * from Blog where id = #{id}

​                            </select>

​         </mapper>

​         我们可以通过sqlSessionFactory.openSession()来获取一个SqlSession，依次操作数据库。

同时namespace + id的方法获取到响应的的sql语句，并且传入session的方法中去执行。

Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);

3.2 注解映射

// 创建 接口，并且定义方法。同时使用@Select注解来标识该方法对应的sql语句

// 映射器方法签名应该匹配相关联的 SqlSession 方法

@Mapper // 必须注册

public interface BlogMapper {

  @Select("SELECT * FROM blog WHERE id = #{id}")

  Blog selectBlog(int id);

}

// 获取映射，同时执行方法。

BlogMapper mapper = session.getMapper(BlogMapper.class);

Blog blog = mapper.selectBlog(101);

 

​         3.3 注册映射

​         无论是上述哪种方法都需要在config.xml注册xml或者类，需要包括包名。

​         批量注册interface mapper：

​         <mappers>

​                   <resource>classpath下面的路径</resource>

​         <mappers>

​         <mappers>

​                   <package name="org.mybatis.builder"/>

</mappers>

四、XML映射详细

​         4.1 顶级元素

​                   select，insert，delete，update，resultMap，cache，

​                   sql(定义可以重用的sql片段)

​         4.2 简单增加，更新，删除（关键对象属性名和关系名映射）

​                   <insert id="insertAuthor">

​                            insert into Author (id,username,password,email,bio)

​                            values (#{id},#{username},#{password},#{email},#{bio})

​                  </insert>

​                 <update id="updateAuthor">

​                          update Author set

​                          username = #{username},

​                       password = #{password},

​                      email = #{email},

​                      bio = #{bio}

​                      where id = #{id}

</update>

<delete id="deleteAuthor">

​                 delete from Author where id = #{id}

</delete>

4.3 参数

每个句子中可以设置 parameterType 以及resultType （这样能够将对象的属性直接和sql参数对应）resultType可以将结果映射到JavaBean（如果对象名和列名不同，可以如下使用别名

select

user_id             as "id",

这里相当于Mybatis在幕后创建一个ResultMap来进行映射了，也可以创建自己的ResultMap，因为很多时候会出现需要的数据没有地方存的时候，存在对象里面又不是很恰当，就可以自定义resultMap。如下:

 

*<resultMap id="userResultMap" type="User">*

  *<id property="id" column="user_id" />*

  *<result property="username" column="user_name"/>*

  *<result property="password" column="hashed_password"/>*

*</resultMap>*

*使用**resultMap**时，设置查询**<select>**的的返回类型为**resultMap**，而不是**resultType**（查找类）*

   4.4 sql 元素

​         <sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>（复用的sql模式片段）

​         // 使用include标签，同时定义参数来引用

​         select

<include refid="userColumns"><property name="alias" value="t1"/></include>,

<typeAlias type="com.someapp.model.User" alias="User"/> // 类的别名，引用类型，不用写全名。

五、Java API（最重要的）

​         5.1 SqlSession接口创建

​         （1）通过SqlSessionFactoryBuilder来创建SqlSessionFactory接口。

​         （2）SqlSessionFactory 可以创建多种SqlSession实例：

​                   A. 使用事务(事务等级设置)和自动提交

​                   B. 来自配置的连接还是自己提供连接

​                   C. 预处理语句复用以及批量执行（ExecutorType{ SIMPLE(新建语句), REUSE, BATCH}）

​                   openSession()返回下面特性的SqlSession:

\1.        开启事务

\2.        从配置中获取连接

\3.        事务隔离级别将会使用驱动或数据源的默认设置。

\4.        预处理语句不会被复用，也不会批量更新

​         5.2 SqlSession 重要方法

​         （1）语句执行方法

​                   <T> T selectOne(String statement, Object parameter)

<E> List<E> selectList(String statement, Object parameter)

<K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey)

int insert(String statement, Object parameter)

int update(String statement, Object parameter)

int delete(String statement, Object parameter)

// 参数是原生类型，JavaBean,POJO 或 Map。

​                   // statement是XML的映射中的namespace.<语句id>

​                   // 这些方法也有重载，可以不需要参数

​                   // selectOne必须返回一个对象，否则会保存

​                   

​                   高级查询：

​                   <E> List<E> selectList (String statement, Object parameter, RowBounds rowBounds)

​                   // rowBounds 设置略过（跳过）指定数量的记录，限制返回结果数量（用于分页）

​                   RowBounds rowBounds = new RowBounds(offset, limit);

（4）         事务控制

void commit(); void rollback();

默认情况下 MyBatis 不会自动提交事务, 除非它侦测到有插入, 更新或删除操作改变了 数据库。可以往上述两个方法传入 true，强制执行。（注意,你不能在自动提交模式下强制 session,或者使用 了外部事务管理器时，MyBatis-Spring和MyBatis-Guice提供了声明事务处理）

确保每次的操作模式是这样：

SqlSession session = sqlSessionFactory.openSession();

try {

​    // following 3 lines pseudocod for "doing some work"

​    session.insert(...);

​    session.delete(...);

​    session.commit();

} finally {

​    session.close();

}

（5）         使用映射器

<T> T getMapper(Class<T> type)

映射器类是一个接口，同时里面的每个方法都要匹配于 SqlSession 方法。返回类型必须匹配期望的结果类型。

如何才算匹配到了SqlSession中的方法？

 

// (Author) selectOne("selectAuthor",5);

  Author selectAuthor(int id);

每个映射器方法签名（方法名和形参数据类型）要匹配关联的SqlSession方法，同时方法名必须匹配XML映射语句的ID。（方法名的select，insert。。。等要匹配）

你可以传递多个参数给一个映射器方法, 默认情况下它们将会以它们，在参数列表中的位置来命名,比如:#{param1},#{param2}（传入参数命名有对应关系），可以在参数上添加注解：可以在参数上使用@Param(“paramName”)注解，这个时候传入的参数名，可以任意。

（6）         注解的使用

@Insert，@Update，@Delete，@Select，属性为value，表示执行的SQL语句，使用字符串数组或者单独字符串。其中参数用#{param}格式定义。

六、Mybatis-Spring的特点

​         重点：（1）Spring中需要加载Mybatis的哪些类？

​                   （2）如何注入Mybatis的数据映射器？

​                   （3）如何处理事务？

​         6.1 依赖

​                   <dependency>

​                          <groupId>org.mybatis</groupId>

​                            <artifactId>mybatis-spring</artifactId>

  <version>1.3.2-SNAPSHOT</version>

</dependency>

​         6.2 SqlSessionFactoryBean

​                   mybatis-spring使用SqlSessionFactoryBean方法来创建SqlSessionFactory ；

​                   该SqlSessionFactory 需要在bean xml文件中定义，同时此bean可以设置property有dataSource（必须）和mapperLocations等（再续后面在详细了解）

## 七、mybatis-spring-boot

7.1 依赖（https://github.com/mybatis/spring-boot-starter）

​         <dependency>

​       <groupId>org.mybatis.spring.boot</groupId>

​       <artifactId>mybatis-spring-boot-starter</artifactId>

​       <version>1.3.1-SNAPSHOT</version>

</dependency>

数据源配置：application.properties文件里面

spring.datasource.url=jdbc:mysql://localhost:3306/wage_sys

spring.datasource.username=root

spring.datasource.password=ypyzfd2014

spring.datasource.driver-class-name=com.mysql.jdbc.Driver

7.2.1 在Spring中使用Mybatis至少需要一个SqlSessionFactory和一个mapper接口。

​         MyBatis-Spring-Boot-Starter：会做以下事情：

（1）         自动检测存在的DataSource

（2）         需要注册一个SqlSessionFactory通过传入DataSource到SqlSessionFactoryBean

（3）         需要注册一个SqlSessionTemplate(来自SqlSessionFactory)

（4）         自动扫描Mappers，让映射器和SqlSessionTemplate相连接，同时在Spring上下文中注册这些映射器，以便直接注入。

7.2.2 Mapper扫描和注入

默认MyBatis-Spring-Boot-Starter会搜索带有@Mapper注解的类作为映射器。注入映射器可以：

private final CityMapper cityMapper;

​    public SampleMybatisApplication(CityMapper cityMapper) {

​        this.cityMapper = cityMapper;

​    }

// 构造函数方法来自动注入

同理注入一个sqlSession 如下：

@Component

public class CityDao {

​         private final SqlSession sqlSession;

​         public CityDao(SqlSession sqlSession) {

​                   this.sqlSession = sqlSession;

​         }

​         public City selectCityById(long id) {

​                   return this.sqlSession.selectOne("selectCityById", id);

​         }

}

mybatis.config-location=classpath:mybatis-config.xml 配置Mybatis的XML文件在哪里。

##      7.3 mybatis-spring-boot-starter-test

​         7.3.1 通过添加该依赖可以更方便的进行Mybatis类的测试

​         <dependency>

​       <groupId>org.mybatis.spring.boot</groupId>

​    <artifactId>mybatis-spring-boot-starter-test</artifactId>

​    <version>2.0.0-SNAPSHOT</version>

</dependency>

7.3.2 使用@MybatisTest

通过使用该注解注解测试类，我们可以方便的测试Mybatis的Mapper类以及一些Dao类。默认情况下，该注解会自动配置Mybatis的组件(SqlSessionFactory和SqlSessionTemplate)、以及Mybatis的Mapper类和一个内置数据库。并且Mybatis的测试类在默认情况下会自动回滚。

配置使用真实数据库：

@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)

屏蔽真实的@SpringBootApplication，为了避免加载不需要的组件以及防止一些错误。在测试的包里面新建一个用@SpringBootApplication注解的空类。

给内置数据库创建结构以及添加初始数据，在application.properties中配置spring.datasource.schema=classpath:import.sql；同时在classpath下放入初始化数据库的sql脚本。

测试Mapper类：只需使用下面两个类注解测试类即可。

@RunWith(SpringRunner.class)

@MybatisTest

启动： 目前只支持maven来启动并不支持java来启动。

八、MyBatis Generator（MBG）

​         8.1 自动生成：

（1）数据表的Java POJOs（可能会包含：匹配表主键的类；启用动态USD的类等）

（2）映射文件，插入，依靠主键更新、删除；使用where语句的USD操作

（3）符合Spring 的Daos；DAOs 只使用iBATIS映射Api的Daos；符合iBATIS DAO框架的Daos
          8.2 Eclipse中安装该Feature（不推荐）

​         （1）Help>Eclipse Marketplace 打开；

（2）打开http://marketplace.eclipse.org/marketplace-client-intro?mpc_install=2947754，拖动install按钮到刚打开的窗口，则Eclipse会自动Resolve该Feature【不推荐，主要是】

​         8.3 使用Maven来运行MBG

​                   依赖：（省略数据库）

​                            <plugin>

​                                     <groupId>org.mybatis.generator</groupId>  

​                    <artifactId>mybatis-generator-maven-plugin</artifactId>  

​                    <version>1.3.2</version>  

​                            </plugin>

​                            <dependency>

​                  <groupId>org.mybatis.generator</groupId>

​             <artifactId>mybatis-generator-core</artifactId>

​          <version>1.3.2</version> 

​             </dependency>    

​                   配置文件generatorConfig.xml:（注意XML标签顺序以及标签嵌套关系）；主要配置以下属性：

（1）         数据库jar路径：<classPathEntry>

（2）         数据库连接驱动，URL，密码，用户名<jdbcConnection>

（3）         java模型导出的路径（targetProject）和包设置（可以是目录也可以是包）

a)          targetProjet路径比较特殊，（xml文件放在main\resources下）设置为 ..\WageSystem\src\<项目目录>

b)         xml文件放在src\main根目录，则为\WageSystem\src\<项目目录>

（九）常见错误

​         ApplicationContext错误 

（1）         数据库连接配置错误，需要配置在application.properties里面

（2）         映射文件语法（找不到类等）出错，id重复等

（3）         Mapper类没有注册@Mapper或者config文件里面没有注册xml映射文件

## Day 5  Spring 单元测试

一、        基本概念和配置

依赖：

<dependency>

​            <groupId>org.springframework.boot</groupId>

​            <artifactId>spring-boot-starter-test</artifactId>

​            <scope>test</scope>

</dependency>

通过添加这个依赖，可以方便实现Spring Boot的Junit单元测试；

二、测试实例

​       2.1 编写Service类，通过@Service注解

​       2.2 新建测试类，添加下面注解：

（1）@RunWith(SpringJUnit4ClassRunner.class) SpringUnit支持

（2）@SpringApplicationConfiguration(classes = App.class) 指定配置类【过时】

​       SpringBootTest(classes=<>)

（3）@WebAppConfiguration（？）

2.3 @Resource 注解引入的Service对象。

2.4 编写测试方法

三、Junit基础

​       3.1 注解

​       （1）@BeforeClass 所有测试方法执行一次

​       （2）@AfterClass 所有测试方法结束后执行一次

​       （3）@Before 和@After是每个方法都会执行一次，整个测试中会有多次

​       （4）@Test 标明方法为测试参数，有下面属性：

​                     timeout<int> 测试方法最多时间

​                     expected<Exception.class> 要求抛出的异常

​       （5）@Ignore(“not ready yet”)  忽略该方法

​       （6）@RunWith 设置运行测试代码的Runner 

​       3.2 验证结果

​              使用：org.junit.Assert类（其他包下的类可能不适用了）

​       3.3 The method assertEquals(Object, Object) is ambiguous for the type Assert

​             方法调用有歧义，保证调用参数一致。short 和Short都是不同类型

## Day 8  Mybatis高级技术

### 一、spring-boot分页实现

1.1 使用插件来实现使用（PageHelper）

（1）依赖

<dependency>

​    <groupId>com.github.pagehelper</groupId>

​    <artifactId>pagehelper-spring-boot-starter</artifactId>

​    <version>1.1.2</version>

</dependency>

​         （2）配置，默认不需要，需要的话，需要的在application.properties配置：

​              pagehelper.propertyName=propertyValue

（4）       多种使用方法

PageHelper.startPage(1, 10);// 下面第一个会被分页

List<Entity> list = mapper.select(参数),

 

### 二、Mybatis多参数映射

​       2.1 索引

```
       public List<XXXBean> getXXXBeanList(String xxId, String xxCode);  
```

 

​          <select id="getXXXBeanList" resultType="XXBean">

 

　　                 select t.* from tableName where id = #{0} and name = #{1}  

 

​          </select>  

​    2.2 注解 

​    public List<XXXBean> getXXXBeanList(@Param(“id”)String xxId, 

​            @Param(“code”)String xxCode);  

​       select t.* from tableName where id = #{id} and name = #{code}  // 省略输入参数

​       2.3 Map封装

​       public List<XXXBean> getXXXBeanList(HashMap map);  

​       // parameteType设置为java.util.Map通过key来引用值

​       select 字段... from XXX where id=#{xxId} code = #{xxCode}  

 

## 三、Mybatis高级映射

​       一般情况下，我们都将查询的结果映射到基本的类型，但是在对象模型中不仅仅含有基本类型，对象可能还包含**其它对象**或者**列表**，高级映射就是解决这两个映射问题。

现在有这样需求，用户表 t_user 和订单表 t_order(id,user_id)，一个用户有很多个订单，

一个订单对应一个用户。

User.java 基本数据（id,name,address,sex,tel），列表（orderList）

Oder.java 类如下： 含有基本类型数据（id,user_id,note），对象数据（User user）

sql 语句：

SELECT t1.*,

​        t2.username,

​        t2.sex,

​        t2.address

​    FROM

​        t_order t1,

​        t_user t2

​    WHERE t1.user_id=t2.id

现在需求：（1）将查询的结果映射到Oder类中所有的属性（包括User对象）

​               （2）查询用户的所有订单，并且映射到User类（包括里面的List）

3.1 映射查询的属性到一个POJO对象（一对一）

​       （1）定义ResultMap，type（映射到的主对象：包名.Order）和id（orderMap）要设置。

​       （2）配置基本属性映射：column是查询结果属性，property：映射对象的属性

​       <id column="id" property="id"/>

  <result column="user_id" property="userid"/>

​    （3）对象映射，使用association来映射，该元素有property（映射的对象属性：user，该对象的类型javaType：包名.User）同理标签内部有属性映射：

​    <id column="user_id" property="id"/>

  <result column="username " property="name"/> 等

3.2 映射查询的属性到一个列表里面（一对多）

​     collection元素，也有设置property（映射到对象的内部列表名orders）;

​    列表的条目类型javaType（包名Order），映射Order内部属性：

​    <id column="user_id" property="id"/>

<result column="username" property="username"/>

​       延伸：collection可以使用嵌套来实现多对多，就是说collection元素的javaType设置的对象里面可能还会有List属性（继续使用collection来映射）。

## 四、Mybatis动态sql

4.1 解决like问题

​       （1） <bind name=”key” value=”’%’ + name + ‘%’”> 传递 #{key}，绑定一个key

​       （2）CONCAT语句，有两种不同。

​              CONCAT(‘%’,’${name}’,’%’) // 将name值引入sql语句中

​              CONCAT(‘%’,#{name},’%’) // name作为参数，sql 语句会预编译（推荐）

4.2  sql语句的构建过程

​       （1）先生成带有？的预编译sql语句

​       （2）参数传递给根据sql语句生成statement

 

## 五、Mybatis日志

​       在Application.properties 里面配置：

logging.level.com.scsdzchy.wage.dao=DEBUG

## Day 9 Spring 简单Session和Cookie

一、Session 两个注解掌握就OK

1.1 ModelAttribute("user") UserModel user

​       （1） 使用在方法参数上，表名参数需要来自model，如果model中没有该参数，则会先被实例化然后再放入到model中，一旦出现在模型中，该参数的属性会用有相同名字的请求参数来填充，就是绑定数据到对象。

​       即被修饰的参数：

A.     使用默认构造函数初始化

B.      会有相同名字的请求参数来绑定它

C.      已经存在model中

1.2 结合@SessionAttributes实现变量的方法

​       主要表示model中哪些属性、哪些类型属性需要被存在Session中。

@SessionAttribute注释参数，主要用来之前已经存在的全局Seesion变量

二、Cookie掌握

读取Cookie的值，@CookieValue("test") 使用@CookieValue注解来标识函数参数，以便识别是哪个Cookie（注解参数即为Cookie的key）

 

## Day 9 Spring RestFul

#### 一、 关键要点

注解使用@RestController 或者 @Controller注解类，@ResponseBody注解方法。

同时映射方法返回对象、列表、Map等，Spring MVC会直接返回json对象到前端。（返回的就是对象，ajax中不需要eval 或者 JSON.parse转换）

 

## Day 9 Vue基础入门

一、      Vue实体基础属性：el,data,methods

el :指明Vue需要渲染的区域的元素的id选择。

data：用于创建模板的变量集合，Vue对象代理data变量内容（vue.name == data.name）

methods: 预定义的方法

var vue = new Vue({

​    el: “#container”,

​    data:{

​           name: “vue”

​    },

​    methods: {

​           info:function(){}

​    }

});

二、模板

2.1 <span> {{name}} </span> 绑定内容，{{ expression }} 只能是表达式

2.2 <span v-bind:title=”name”></span> 绑定属性

2.3 <input v-model=”id” /> 双向绑定到变量和view

2.4 事件<a href=”javascript:” v-on:click=”info”></a> 调用vue对象中的info方法

2.5 v-bind:title 可以缩写为 :title ; v-on:click 也可以 @click=””

三、指令

​       3.1 <div v-if=”param”> you can see this, if param is true</div> // 里面不能嵌套，且v-if所在元素不输出

​       3.2 <ul v-for=”(index, item) in items”>{{item}}</ul>  列表循环数据

​       3.3 使用<template v-if=” ”| v-for=”” ></template> 渲染多元素模块

​       3.4 循环渲染对象：

              <div
v-for="(value, key) in object">

​                     {{ key }} : {{ value }}

</div>

<div v-for="(value, key, index) in object">

  {{ index }}. {{ key }} : {{ value }}

</div>

 

## Day 10 Spring 外部配置文件

Spring Boot主应用可以在application.properties文件中配置很多属性以及key-value变量，可以通过@Value注解将这些配置变量注入应用如下：

*@Value("${name}")*

*private String name;*

​       同时可以通过：@TestPropertySource 以及@SpringBootTest#properties 注解来为测试类指定配置文件。

​       随机变量：

my.secret=${random.value}

my.number=${random.int}

my.bignumber=${random.long}

my.uuid=${random.uuid}

my.number.less.than.ten=${random.int(10)}

my.number.in.range=${random.int[1024,65536]}

 

## Day 11 Spring 后端校验

Controller中传递参数方法：

（1）@PathVariable（2）@RequestParam （3）@RequestBody （4）@CookieValue或者直接传递对象，那么如何来验证这些参数的有效性呢？

 

对于对象：通过一系列注解来标识，其属性需要满足的条件。例如：

​       @Max(value = 999999,message = "超过最大数值")

​       在Controller类中的方法中通过 @Valid注解来标识传入的对象，同时参数中加入BindingResult来获取验证错误信息。BindingResult 对象有hasErrors()方法，返回布尔值。同时bindingResult.getFieldError().getDefaultMessage();可以获取错误信息；

​       这种验证可以使用在任何函数的参数中。

上面所述是在Controller层进行校验参数，如果想基于方法来验证的话，要采用下面一种方法。

（1）       首先在需要面向方法的验证类上加上注解：@Validated

（2）       方法中参数也加上@Valid或者其他限制注解

（3）       这种方法验证，不会将错误放在BindingResult里面，而是直接抛出异常：

ConstraintViolationException，我们只需要捕捉该异常来将错误消息返回给用户即可。（这种方法是JSR-303标准，同时使用MethodValidationPostProcessor去检索@Validated注解）

 

## Day 12 Spring Boot错误处理

一、错误捕捉

Spring Boot默认将所有的错误映射到/error路径，在该页面里面简单的展示错误的信息和错误码，如果想要替换默认的处理可以实现ErrorController接口，并且注册一个实现的Bean。或者添加一个ErrorAttributes类型的Bean

**ErrorController****接口**

​       getErrorPath(): String // 返回错误页面的路径

实现类：

@Controller

@RequestMapping(value="${server.error.path:${error.path:/error}}")

BasicErrorController 

二、处理异常

2.1 HandlerExceptionResolver

该接口的实现用来处理Controller层出现的异常。有点类似在web.xml中的异常映射。单是却又更高级；https://docs.spring.io/spring/docs/5.0.0.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers

2.2 @ExceptionHandler

在Controller中，具有该注解的方法，可以处理该Controller类中或者其子类中抛出的特定异常；例子如下：

@Controller

public class SimpleController {

 

​        // @RequestMapping methods omitted ...

 

​        @ExceptionHandler(IOException.class)

​        public ResponseEntity<String> handleIOException(IOException ex) {

​                // prepare responseEntity

​                return responseEntity;

​        }

}

同时方法的返回值，可以是String，ModelAndView,对象等。也可以使用@ResponseBody来注解方法，返回JSON数据。

 

也可以使用@ControllerAdvice注解为一个Controller或者特定异常返回自定义的JSON信息

 

参考API：

public class ResponseEntity<T>

extends HttpEntity<T> ，在HttpEntity的基础上扩展，添加了状态码；

 

HttpEntity 表示一个Http请求或者响应实体，包括请求/响应头和体；

 

HttpHeaders也是一个对象，表示请求头或者体