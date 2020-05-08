Spring Boot整合Spring Data JPA理解（一）
=========================
> 官方文档: [Spring Data JPA 官方文档](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)  

一、 Sping Data JPA 简介
--------------------

   **Spring Data JPA** 是 Spring 生态中基于Hibernate 的 JPA框架，Spring Data JPA开发者旨在让使用者用非常简洁的代码就可以实现对数据的访问和操作，并且它还提供了包括增删改查等在内的常用功能，扩展性非常强！

二、 将Spring Data JPA集成到Spring Boot
---------------------------------

第一步：首先我们需要所需要的引入maven依赖包，包括Spring Data JPA和Mysql的驱动

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
     <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    

第二步：修改application.yml，配置好数据库连接和jpa的相关配置(也可以修改properties，本人习惯yml)

    spring:
      datasource:
        url: jdbc:mysql://192.168.1.91:3306/blog?useUnicode=true&characterEncoding=utf-8&useSSL=false
        username: root
        password: fabiha
        driver-class-name: com.mysql.jdbc.Driver
      jpa:
        hibernate:
          ddl-auto: update
        database: mysql
        show-sql: true
	logging:
      level:
        root: info	
    

`spring.jpa.properties.hibernate.hbm2ddl.auto`可以配置属性，自动根据实体类的定义创建、更新、验证数据库表结构。参数配置如下：

*   `create`：每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，就算数据库表前后两次没有任何改变也会这样执行，这就可能会导致数据库表数据丢失。
*   `create-drop`：每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
*   `update`：最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等应用第一次运行起来后才会。
*   `validate`：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。

三、 JPA的核心用法
---------

示例如下：

### 3.1.实体Model类

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    @Entity
    @Table(name="blog")
    public class Blog {
    
        @Id
        @GeneratedValue
        private Long id;
        @Column(nullable = false)
        private String content;
        @Column(nullable = false,length = 32)
        private String blog;
    
        @Column(nullable = false, unique = true,length = 32)
        private String title;
        @Column(nullable = false, unique = true,length = 32)
        private Type type;
        @Column(length = 512)
        private String content;
        @Temporal(TemporalType.TIMESTAMP)
        private Date createTime;
        @Temporal(TemporalType.TIMESTAMP)
        private Date updateTime;
    }
    
*   @Id 指定这个字段为表的主键
*   @Entity 表示这个类是一个实体类，接受JPA控制管理，对应数据库中的一个表
*    @Temporal(TemporalType.TIMESTAMP)如果在某类中有Date类型的属性，数据库中存储可能是'yyyy-MM-dd hh:MM:ss'要在查询时获得年月日，在该属性上标注@Temporal(TemporalType.DATE) 会得到形如'yyyy-MM-dd' 格式的日期。TIMESTAMP是取日期和时间，例如2020-01-01  12:52:32
*   @Table 指定这个类对应数据库中的表名。如果这个类名和数据库表名符合驼峰及下划线规则，可以省略这个注解。例如BlogType类名对应表名blog_type。

*   @GeneratedValue(strategy=GenerationType.IDENTITY) 指定主键的生成方式，一般主键为自增的话，就采用GenerationType.IDENTITY的生成方式
*   @Column 注解针对一个字段，对应表中的一列。nullable = false表示数据库字段不能为空, unique = true表示数据库字段不能有重复值,length = 32表示数据库字段最大程度为32.

### 3.2.dao层数据库操作的接口

    public interface BlogRepository extends JpaRepository<Blog,Long> {
    }
    

对于单表操作来说JPA就是神器，通常XxxRepository继承 JpaRepository<T,ID>就可以为我们提供了各种针对单表的数据操作方法：增删改查。通过调用接口的方法名称就能知道方法是做什么操作的，当然我们也可以自定义方法，直接在接口里面定义它就自动为我们实现了，通过type字段查找blog表的所有数据。也就是说，我们使用了find(查找)关键字，JPA就自动识别方法名为我们解析成数据库操作。

        //jPA会根据方法名自动生成SQL执行，有兴趣的可以看一看底层实现
      Blog findByType(Type type);
    


### 3.3.service层接口:

    
    public interface BlogService {
    
    Blog getBlog(Long id);

    Blog getAndConvert(Long id);

    Page<Blog> listBlog(Pageable pageable,BlogQuery blog);

    Page<Blog> listBlog(Pageable pageable);

    Page<Blog> listBlog(Long tagId,Pageable pageable);

    Page<Blog> listBlog(String query,Pageable pageable);

    List<Blog> listRecommendBlogTop(Integer size);

    Map<String,List<Blog>> archiveBlog();

    Long countBlog();

    Blog saveBlog(Blog blog);

    Blog updateBlog(Long id,Blog blog);

    void deleteBlog(Long id);
    }
    
    

### 3.4.service层接口实现

    @Service
    public class BlogServiceImpl implements BlogService  {
    
        //注入jpa对象
        @Autowired
        private BlogRepository blogRepository;
        //根据id从数据库中找博客
        @Override
        public Blog getBlog(Long id) {
            return blogRepository.findOne(id);
    }
         //拿到分页信息
         @Override
        public Page<Blog> listBlog(Pageable pageable) {
        return blogRepository.findAll(pageable);
    }
         @Override
        public Long countBlog() {
          return blogRepository.count();
    }
         @Override
        public void deleteBlog(Long id) {
          blogRepository.delete(id);//通过id删除
    } 
    //实现类没有全部贴出来
    }
    

##### 注意：Pageable 是Spring Data库中定义的一个接口，该接口是所有分页相关信息的一个抽象，通过该接口，我们可以得到和分页相关所有信息（例如pageNumber、pageSize等）Jpa就能够通过pageable参数来得到一个带分页信息的Sql语句。
 ##### Page类也是Spring Data提供的一个接口，该接口表示一部分数据的集合以及其相关的下一部分数据、数据总数等相关信息，通过该接口，我们可以得到数据的总体信息（数据总数、总页数...）以及当前数据的信息（当前数据的集合、当前页数等）。
四、关键字查询接口
---------
还有很多关键字方法都是**通过解析方法名创建查询**。JPA针对单表的数据查询非常简单。
<table class="tableblock frame-all grid-all fit-content">
<caption class="title"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">表3.方法名称中受支持的关键字</font></font></caption>
<colgroup>
<col>
<col>
<col>
</colgroup>
<thead>
<tr>
<th class="tableblock halign-left valign-top"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Keyword（关键字）		</font></font></th>
<th class="tableblock halign-left valign-top"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Sample（方法）</font></font></th>
<th class="tableblock halign-left valign-top"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">JPQL （SQL语句）</font></font></th>
</tr>
</thead>
<tbody>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>And</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByLastnameAndFirstname</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.lastname = ?1 and x.firstname = ?2</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>Or</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByLastnameOrFirstname</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.lastname = ?1 or x.firstname = ?2</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>Is</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">， </font></font><code>Equals</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByFirstname</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">，</font></font><code>findByFirstnameIs</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">，</font></font><code>findByFirstnameEquals</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.firstname = ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>Between</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByStartDateBetween</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.startDate between ?1 and ?2</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>LessThan</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAgeLessThan</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age &lt; ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>LessThanEqual</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAgeLessThanEqual</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age &lt;= ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>GreaterThan</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAgeGreaterThan</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age &gt; ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>GreaterThanEqual</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAgeGreaterThanEqual</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age &gt;= ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>After</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByStartDateAfter</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.startDate &gt; ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>Before</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByStartDateBefore</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.startDate &lt; ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>IsNull</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">， </font></font><code>Null</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAge(Is)Null</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age is null</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>IsNotNull</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">， </font></font><code>NotNull</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAge(Is)NotNull</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age not null</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>Like</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByFirstnameLike</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.firstname like ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>NotLike</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByFirstnameNotLike</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.firstname not like ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>StartingWith</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByFirstnameStartingWith</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.firstname like ?1</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">（参数附后</font></font><code>%</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">）</font></font></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>EndingWith</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByFirstnameEndingWith</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.firstname like ?1</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">（参数加前缀</font></font><code>%</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">）</font></font></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>Containing</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByFirstnameContaining</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.firstname like ?1</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">（参数绑定在中</font></font><code>%</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">）</font></font></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>OrderBy</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAgeOrderByLastnameDesc</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age = ?1 order by x.lastname desc</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>Not</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByLastnameNot</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.lastname &lt;&gt; ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>In</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAgeIn(Collection&lt;Age&gt; ages)</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age in ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>NotIn</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByAgeNotIn(Collection&lt;Age&gt; ages)</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.age not in ?1</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>True</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByActiveTrue()</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.active = true</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>False</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByActiveFalse()</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where x.active = false</code></p></td>
</tr>
<tr>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>IgnoreCase</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>findByFirstnameIgnoreCase</code></p></td>
<td class="tableblock halign-left valign-top"><p class="tableblock"><code>… where UPPER(x.firstame) = UPPER(?1)</code></p></td>
</tr>
</tbody>
</table>



五、其他
----
其他细节以及后续会更新







