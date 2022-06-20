## MyBatis-Plus
参考资料
* 官方指南 - https://baomidou.com/guide/
* 官方配置 - https://baomidou.com/config/
* github - https://github.com/baomidou/mybatis-plus

### 1. MyBatis-Plus 快速上手

#### MyBatis-Plus 是什么
MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生

#### 入门案例
加入 starter
```xml
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.4.2</version>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

实体类
```java
@Data
public class User {

    private Long id;
    private String name;
    private Integer sex;
    private LocalDate birthday;
    private Date created;
    private Date modified;

}
```

注意：id属性的类型为**Long**类型。

数据库表

```sql
use db1;
drop table if exists tb_user;
create table tb_user
(
    id       bigint primary key auto_increment,
    name     varchar(8) not null,
    sex      tinyint,
    birthday datetime,
    created  datetime,
    modified datetime
);
insert into tb_user
values (1, '赵一伤', 0, '2000-1-1', now(), now()),
       (2, '钱二败', 0, '2000-1-1', now(), now()),
       (3, '孙三毁', 0, '2000-1-1', now(), now()),
       (4, '李四摧', 0, '2000-1-1', now(), now()),
       (5, '周五输', 0, '2000-1-1', now(), now()),
       (6, '吴六破', 0, '2000-1-1', now(), now()),
       (7, '郑七灭', 0, '2000-1-1', now(), now()),
       (8, '王八衰', 0, '2000-1-1', now(), now()),
       (9, '张无忌', 0, '2010-3-1', now(), now()),
       (10, '赵敏', 1, '2010-4-1', now(), now());
```
注意：实体类名和表名之间是一致的，以及id的数据类型为**bigint**类型。

添加 mapper 接口
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```
* @Mapper（加在 mapper 接口上） 与 @MapperScan（加在引导类上）二选一即可
* 集成 BaseMapper 之后会让自定义的 mapper 拥有最基本的增删改查能力
* BaseMapper 接口的泛型对应实体类类型

测试用例
```java
@SpringBootTest
class MybatisPlusCaseApplicationTests {

    @Autowired
    private UserMapper UserMapper;

    @Test
    void findAll() {
        List<User> list = UserMapper.selectList(null);
        Assertions.assertEquals(10, list.size());
    }
}
```

### 2. 基本使用

#### 按 id 查询
```java
@Test
void findById() {
	User User = UserMapper.selectById(9);
	Assertions.assertNotNull(User);
	Assertions.assertEquals("张无忌", User.getName());
}
```

#### 新增
```java
@Test
void insert() {
	User User = new User();
	User.setName("黛绮丝");
	User.setSex(1);
	User.setBirthday(LocalDate.of(1998, 10, 23));
	Date now = new Date();
	User.setCreated(now);
	User.setModified(now);
	UserMapper.insert(User);
	Assertions.assertTrue(User.getId() > 0);
}
```

新增时要注意：
1. 测试代码如果提供 id，那么主键会采用此 id
2. 测试代码没有提供 id，虽然我们建表时采用了自增主键，但实际会发现其 id 是一个很长的数字：1359322588994756610，这是因为 Mybatis-plus 默认的主键生成策略有五种，默认是其中的 ASSIGN_ID

```java
public enum IdType {
    /**
     * 数据库ID自增
     * <p>该类型请确保数据库设置了 ID自增 否则无效</p>
     */
    AUTO(0),
    /**
     * 该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT)
     */
    NONE(1),
    /**
     * 用户输入ID
     * <p>该类型可以通过自己注册自动填充插件进行填充</p>
     */
    INPUT(2),

    /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
    /**
     * 分配ID (主键类型为number或string）,
     * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(雪花算法)
     *
     * @since 3.3.0
     */
    ASSIGN_ID(3),
    /**
     * 分配UUID (主键类型为 string)
     * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(UUID.replace("-",""))
     */
    ASSIGN_UUID(4),
	
	...
}
```

* AUTO - 采用数据库自增主键
* NONE - 没啥用
* INPUT - 用户自行指定
* ASSIGN_ID - 用于分布式场景，采用雪花算法（默认）
* ASSIGN_UUID - 用于分布式场景，UUID算法

ASSIGN_ID 和 ASSIGN_UUID 都是分布式场景下才用的方法，单机 MySQL 有点大材小用。如果要改变默认生成策略，有两种办法：

方法1 - 注解（局部做法）
```java
@Data
public class User {

    @TableId(type = IdType.AUTO)
    private Long id;
    
	// ...
}
```

方法2 - spring 配置（全局做法）
```properties
mybatis-plus.global-config.db-config.id-type=auto
```

#### 修改
```java
@Test
void update() {
	User User = new User();
	User.setId(9L);
	User.setName("明教教主 张无忌");
	UserMapper.updateById(User);

	User dbUser = UserMapper.selectById(9);
	Assertions.assertNotNull(dbUser);
	Assertions.assertEquals("明教教主 张无忌", dbUser.getName());
	Assertions.assertEquals(LocalDate.of(2010, 3, 1), dbUser.getBirthday());
}
```
修改时会根据实体对象的属性值来动态生成 sql，例如上例中 User 的 id，name 不为空，其它值 null 的属性不会出现在最终 sql 中：
```sql
UPDATE user_info SET name=? WHERE id=?
```


#### 删除
```java
@Test
void delete() {
	UserMapper.deleteById(10);
	User dbUser = UserMapper.selectById(10);
	Assertions.assertNull(dbUser);
}
```

#### 分页查询
分页需要首先加入分页插件，新版方式如下，旧版用的 `PaginationInterceptor` 已不推荐使用
```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
	MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
	interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
	return interceptor;
}

@Bean
public ConfigurationCustomizer configurationCustomizer() {
	return configuration -> configuration.setUseDeprecatedExecutor(false);
}
```

其次，分页条件中指定当前页、和每页记录数，返回结果中包含了总页数，当前页记录列表等信息
```java
@Test
@DisplayName("06-测试分页查询")
void findPage() {
	Page<User> page = UserMapper.selectPage(new Page<>(2, 4), null);
	Assertions.assertEquals(3, page.getPages());
	Assertions.assertEquals(4, page.getRecords().size());
	String[] names = {"周五输", "吴六破", "郑七灭", "王八衰"};
	Assertions.assertArrayEquals(names, page.getRecords().stream().map(User::getName).toArray());
}
```

异常信息

```java
java.lang.NoSuchMethodError: 'net.sf.jsqlparser.statement.select.GroupByElement net.sf.jsqlparser.statement.select.PlainSelect.getGroupBy()'
```

原因：在使用mybatis分页插件时，需要依赖 jsqlparser。这个错误表示分页插件版本与依赖版本不匹配。

pom.xml中添加

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.1.10</version>
</dependency>
<dependency>
    <groupId>com.github.jsqlparser</groupId>
    <artifactId>jsqlparser</artifactId>
    <version>3.1</version>
</dependency>
```

#### 带条件查询

mybatis-plus 支持基于 java 代码的多条件组合查询，好处是可以避免 xml mapper 中的 `<if>` 标签的使用
```java
@Test
@DisplayName("07-测试带条件查询-case1")
void findByConditionCase1() {
	String name = "周";
	Integer sex = null;
	LocalDate birthday = null;
	List<User> list = query(name, sex, birthday);
	Assertions.assertEquals(2, list.size());
}

@Test
@DisplayName("08-测试带条件查询-case2")
void findByConditionCase2() {
	String name = "周";
	Integer sex = 1;
	LocalDate birthday = null;
	List<User> list = query(name, sex, birthday);
	Assertions.assertEquals(1, list.size());
}

@Test
@DisplayName("09-测试带条件查询-case3")
void findByConditionCase3() {
	String name = "周";
	Integer sex = 0;
	LocalDate birthday = LocalDate.of(2000, 1, 1);
	List<User> list = query(name, sex, birthday);
	Assertions.assertEquals(1, list.size());
}

private List<User> query(String name, Integer sex, LocalDate birthday) {
	QueryWrapper<User> qw = new QueryWrapper<>();
	// 根据姓名模糊查询
	if (StringUtils.hasLength(name)) {
		qw.like("name", name);
	}
	// 根据性别查询
	if (sex != null) {
		qw.eq("sex", sex);
	}
	// 根据生日小于等于查询
	if (birthday != null) {
		qw.le("birthday", birthday);
	}
	return UserMapper.selectList(qw);
}
```

也可以避免 `<foreach>` 标签的使用，例如
```java
@Test
@DisplayName("10-测试带条件查询-case4")
void findByConditionCase4() {
	QueryWrapper<User> qw = new QueryWrapper<>();
	qw.in("id", Arrays.asList(1, 2, 3, 18));
	List<User> list = UserMapper.selectList(qw);
	Assertions.assertEquals(3, list.size());
}
```

如果你喜欢 lambda，还可以将构造 QueryWrapper 的语法做如下修改：
```java
private List<User> queryLambda(String name, Integer sex, LocalDate birthday) {
	QueryWrapper<User> qw = new QueryWrapper<>();
	qw.lambda()
			.like(StringUtils.hasLength(name), User::getName, name)
			.eq(sex != null, User::getSex, sex)
			.le(birthday != null, User::getBirthday, birthday);
	return UserMapper.selectList(qw);
}
```
这些条件中，都有三个参数
* 参数1 是布尔值，条件成立，才会将列名与值拼接入 sql，相当于未改写前的 if 判断
* 参数2 是列名，使用方法引用，目的是根据方法引用找到真正的列名
	* 例如：方法名是 getFirstName，而列名实际是 first_name，直接写列名缺点是，将列名硬编码进入了 java 代码
* 参数3 是值

#### 带条件更新
例如，将所有男性变为女性🤢
```java
@Test
@DisplayName("11-测试带条件更新")
void updateByCondition() {
	User User = new User();
	User.setSex(1);

	UpdateWrapper<User> uw = new UpdateWrapper<>();
	uw.eq("sex", 0);
	int affects = UserMapper.update(User, uw);
	Assertions.assertEquals(9, affects);
}
```

删除所有 id 小于 3 的
```java
@Test
@DisplayName("12-测试带条件更新")
void deleteByCondition() {
	QueryWrapper<User> qw = new QueryWrapper<>();
	qw.lt("id", 3);
	int affects = UserMapper.delete(qw);
	Assertions.assertEquals(2, affects);
}
```

注意：
* 查询和删除用 QueryWrapper
* 更新用 UpdateWrapper



### 3. 进阶使用
#### 当映射不一致时
若数据库表不变，实体类和 mapper 变化为：
```java
@Data
public class User2 {

    private Long id;
    private String myName;
    private Integer sex;
    private LocalDate birthday;
    private Date created;
    private Date modified;
}
```

```java
@Mapper
public interface User2Mapper extends BaseMapper<User2> {
}
```

再运行下面的测试用例
```java
@Test
void select() {
	User2 user2 = user2Mapper.selectById(3);
	System.out.println(user2);
}
```

直接报错，原因是表名与实体类名对应不上
```
Cause: java.sql.SQLSyntaxErrorException: Table 'test.user2' doesn't exist
```

表名与实体类名的映射：
* 方式一：同名
* 方式二：满足 `驼峰 <-> 下划线` 映射规则
* 方式三：用 `@TableName("对应表名")` 标注实体类

改进如下
```java
@TableName("user_info")
@Data
public class User2 {

    private Long id;
    private String myName;
    private Integer sex;
    private LocalDate birthday;
    private Date created;
    private Date modified;
}
```

重新运行测试用例，依然报错
```
Cause: java.sql.SQLSyntaxErrorException: Unknown column 'my_name' in 'field list'
```

不用说是表字段名与实体属性名对应不上，继续修改
```java
@TableName("user_info")
@Data
public class User2 {

    private Long id;
    @TableField("name")
    private String myName;
    private Integer sex;
    private LocalDate birthday;
    private Date created;
    private Date modified;
}
```

表字段名与实体属性名的映射：
* 方式一：同名
* 方式二：满足 `驼峰 <-> 下划线` 映射规则
* 方式三：用 `@TableField("对应字段名")` 标注实体属性

#### 需要统一填充时
每个实体对象都有一些通用的属性需要填充值，例如
* 新增时，需要填充 created，modified 这两个时间
* 修改时，需要填充 modified 这个时间

首先需要标注哪些属性需要填充
```java
@Data
@TableName("user_info")
public class User3 {

    private Long id;
    private String name;
    private Sex sex;
    private LocalDate birthday;
    
	// 新增时需要填充
    @TableField(fill = FieldFill.INSERT)
    private Date created;
	
	// 新增及修改时需要填充
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date modified;

}
```

其次设置填充处理器
```java
@Bean
public MetaObjectHandler metaObjectHandler() {
	return new MetaObjectHandler() {
		
		// 新增时提供的值
		public void insertFill(MetaObject metaObject) {
			Date now = new Date();
			this.strictInsertFill(metaObject, "created", Date.class, now);
			this.strictInsertFill(metaObject, "modified", Date.class, now);
		}

		// 修改时提供的值
		public void updateFill(MetaObject metaObject) {
			Date now = new Date();
			this.strictInsertFill(metaObject, "modified", Date.class, now);
		}
	};
}
```
MetaObjectHandler 提供的默认方法的策略均为：如果属性有值则不覆盖

以下两个测试用例应当通过
```java
@Test
@DisplayName("16-测试自动填充-插入")
void fillInsert() {
	User3 user = new User3();
	user.setName("殷天正");
	user.setSex(Sex.MALE);
	user.setBirthday(LocalDate.of(1990, 1, 1));
	user3Mapper.insert(user);
	Assertions.assertNotNull(user.getCreated());
	Assertions.assertNotNull(user.getModified());
	Assertions.assertEquals(user.getCreated(), user.getModified());
}

@Test
@DisplayName("17-测试自动填充-更新")
void fillUpdate() {
	User3 user = new User3();
	user.setId(11L);
	user.setName("峨眉派 周芷若");
	user3Mapper.updateById(user);
	Assertions.assertNull(user.getCreated());
	Assertions.assertNotNull(user.getModified());
}
```
#### 需要复杂查询时
虽然 MyBatis-plus 提供了很方便的方法，但如果是需要连表查询，仍然会用到 MyBatis 原始方式

例如：
```java
@Data
@TableName("user_info")
public class User4 {

    private Long id;
    private String name;
    private Sex sex;
    private LocalDate birthday;
    private Date created;
    private Date modified;

	// 表达一对多关系
    private Set<SkillInfo> skillInfos;

}

@Data
public class SkillInfo {
    private Long id;
    private String name;
}
```

对应的数据库表
```sql
create table user_info
(
    id       bigint primary key auto_increment,
    name     varchar(8) not null,
    sex      tinyint,
    birthday date,
    created  datetime,
    modified datetime
);

create table skill_info
(
    id   bigint primary key auto_increment,
    name varchar(8)
);

create table user_skill_info
(
    user_id  bigint not null,
    skill_id bigint not null,
    foreign key (user_id) references user_info (id),
    foreign key (skill_id) references skill_info (id)
);
```

现在要查询某个用户，以及用户的武功，需要连表查询，此时就需要扩展 mapper 接口的方法
```java
@Mapper
public interface User4Mapper extends BaseMapper<User4> {
    User4 selectUserWithSkill(Long userId);
}
```

并增加 xml mapper 文件 `src/main/resources/com/itheima/mapper/User4Mapper.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.User4Mapper">
    <select id="selectUserWithSkill" resultMap="userMap">
        select ui.*, si.id sid, si.name sname from user_info ui
            left join user_skill_info usi on ui.id = usi.user_id
            left join skill_info si on si.id = usi.skill_id
        where ui.id=#{userId}
    </select>
    <resultMap id="userMap" type="com.itheima.domain.User4">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="sex" property="sex" typeHandler="com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler"/>
        <result column="birthday" property="birthday"/>
        <result column="created" property="created"/>
        <result column="modified" property="modified"/>
        <collection property="skillInfos" ofType="com.itheima.domain.SkillInfo">
            <id column="sid" property="id"/>
            <result column="sname" property="name"/>
        </collection>
    </resultMap>
</mapper>
```
#### 代码生成器
有时候，需要自动生成代码，减少编码量

首先引入依赖
```xml
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-generator</artifactId>
	<version>3.4.1</version>
</dependency>
<dependency>
	<groupId>org.freemarker</groupId>
	<artifactId>freemarker</artifactId>
</dependency>
```
截至写作时，阿里云 maven 仓库中没有 3.4.2 版本的 mybatis-plus-generator

编写如下代码
```java
public class CodeGenerator {
    
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入" + tip + "：");
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        AutoGenerator mpg = new AutoGenerator();
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("jobob");
        gc.setOpen(false);
        mpg.setGlobalConfig(gc);

        // 修改1: 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/test?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=Asia/Shanghai");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("root");
        mpg.setDataSource(dsc);

        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("模块名"));
        // 修改2: 父包配置
        pc.setParent("org.test");
        mpg.setPackageInfo(pc);
        TemplateConfig templateConfig = new TemplateConfig();
        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
            }
        };
        List<FileOutConfig> focList = new ArrayList<>();
        focList.add(new FileOutConfig("/templates/mapper.xml.ftl") {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 修改3: xml mapper 的路径, 请与父包保持一致
                return projectPath + "/src/main/resources/org/test/" + pc.getModuleName() + "/mapper/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        StrategyConfig strategy = new StrategyConfig();
        // 修改4: 表名前缀
        strategy.setTablePrefix("ss_");
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}
```


