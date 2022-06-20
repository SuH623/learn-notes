## mybatis多表操作【方式一】

### 数据库准备

```sql
CREATE DATABASE if not exists mybatis_duobiao;

USE db2;
-- 一对一
CREATE TABLE person(
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20),
  age INT
);
INSERT INTO person VALUES (NULL,'张三',23);
INSERT INTO person VALUES (NULL,'李四',24);
INSERT INTO person VALUES (NULL,'王五',25);

CREATE TABLE card(
  id INT PRIMARY KEY AUTO_INCREMENT,
  number VARCHAR(30),
  pid INT,
  CONSTRAINT cp_fk FOREIGN KEY (pid) REFERENCES person(id)
);

INSERT INTO card VALUES (NULL,'12345',1);
INSERT INTO card VALUES (NULL,'23456',2);
INSERT INTO card VALUES (NULL,'34567',3);


-- 一对多
CREATE TABLE classes( -- 班级表
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20)
);
INSERT INTO classes VALUES (NULL,'黑马一班');
INSERT INTO classes VALUES (NULL,'黑马二班');


CREATE TABLE student( -- 学生表
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(30),
  age INT,
  cid INT,
  CONSTRAINT cs_fk FOREIGN KEY (cid) REFERENCES classes(id)
);
INSERT INTO student VALUES (NULL,'张三',23,1);
INSERT INTO student VALUES (NULL,'李四',24,1);
INSERT INTO student VALUES (NULL,'王五',25,2);
INSERT INTO student VALUES (NULL,'赵六',26,2);

-- 多对多
CREATE TABLE course(-- 课程
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20)
);
INSERT INTO course VALUES (NULL,'语文');
INSERT INTO course VALUES (NULL,'数学');


CREATE TABLE stu_cr(
  id INT PRIMARY KEY AUTO_INCREMENT,
  sid INT,
  cid INT,
  CONSTRAINT sc_fk1 FOREIGN KEY (sid) REFERENCES student(id),
  CONSTRAINT sc_fk2 FOREIGN KEY (cid) REFERENCES course(id)
);
INSERT INTO stu_cr VALUES (NULL,1,1);
INSERT INTO stu_cr VALUES (NULL,1,2);
INSERT INTO stu_cr VALUES (NULL,2,1);
INSERT INTO stu_cr VALUES (NULL,2,2);
```

```yaml
# 配置连接数据库参数
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root

#mybatis配置
mybatis:
  configuration:
    map-underscore-to-camel-case: true #开启驼峰命名自动映射【掌握】

# 设置日志级别
logging:
  level:
    com.itheima.mapper: debug  #mapper包中的所有日志输出级别为debug级别。可以看到sql语句
```

```java
@SpringBootApplication
@MapperScan("com.itheima.mapper")
public class MybatisDuobiaoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisDuobiaoApplication.class, args);
    }
}
```

### 一对一关系

#### 1 javabean类

```java
@Data
public class Person {
    private Integer id;     //主键id
    private String name;    //人的姓名
    private Integer age;    //人的年龄
}
@Data
public class Card {
    private Integer id;     //主键id
    private String number;  //身份证号
    private Person p;       //所属人的对象
}
```

#### 2 接口类

```java
public interface OneToOneMapper {
    //查询全部
    public abstract List<Card> findAll();
}
```

#### 3 映射文件配置OneToOneMapper.xml

resultMap标签的作用：手动定义查询结果和bean对象的映射关系，一般用来处理查询结果和bean的属性不一致情况。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.OneToOneMapper">
  <!--配置一对一映射关系
        id="oneToOne"：表示resultMap的唯一标识，再select标签中使用
        type="com.itheima.pojo.Card"：表示最终要封装的数据类型
        autoMapping="true" ：自动映射-->
  <resultMap id="oneToOne" type="com.itheima.pojo.Card" autoMapping="true">
      <!--映射主键-->
      <id column="id" property="id"/>
      <!--映射普通字段-->
      <!--<result column="number" property="number"/>-->
      <!--配置一对一：也就是给Card中的Person对象封装数据
                property="p" : Card中的属性名
                javaType="com.itheima.pojo.Person"  :属性的类型，可以省略不写-->
     <association property="p" javaType="com.itheima.pojo.Person">
            <id column="pid" property="id"/>
            <result column="pname" property="name"/>
            <result column="page" property="age"/>
      </association>
  </resultMap>

  <!--一对一：查询所有card信息，包括对应的person信息
        resultType：表示封装查询结果的类型，会自动根据列名和对象的属性名自动封装，但是一对一关系中的person对象属性无法封装，那么需要我们手动映射封装查询结果，此时就不能再使用resultType，改用resultMap-->
  <select id="findAll" resultMap="oneToOne">
    select
      c.*,
      p.id pid,
      p.name pname,
      p.age page
    from card c,person p where p.id=c.pid
  </select>
</mapper>
```

**==注意：封装结果集不再使用resultType属性，而是使用resultMap属性。==**

#### 4 测试类

```java
@SpringBootTest
@Slf4j
class MybatisDuobiaoApplicationTests {

    @Autowired
    private OneToOneMapper oneToOneMapper;

    /*
        测试一对一关系：查询所有card的信息，包括每个card对应的person信息
     */
    @Test
    void testOneToOne() {
        List<Card> list = oneToOneMapper.findAll();
        list.forEach(card->log.info("{}",card));
    }
}

```

### 一对多关系

#### 1 javabean类

```java
@Data
public class Classes {
  private Integer id;     //主键id
  private String name;    //班级名称
  //一个班级中有多名学生，所以用集合存储
  private List<Student> students; 
}

@Data
public class Student {
    private Integer id;     //主键id
    private String name;    //学生姓名
    private Integer age;    //学生年龄
}
```

#### 2 接口类

```java
public interface OneToManyMapper {
  //查询全部
  public abstract List<Classes> findAll();
}
```

#### 3 映射文件配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.OneToManyMapper">

    <resultMap id="oneToManyMap" type="com.itheima.pojo.Classes" autoMapping="true">
        <id column="cid" property="id"/>
        <result column="cname" property="name"/>
        <!--配置一对多
            collection标签：配置一对多关系，给Bean中的集合封装数据
            ofType="com.itheima.pojo.Course"：集合中的泛型类型
        -->
        <collection property="students" ofType="com.itheima.pojo.Student">
            <id column="sid" property="id"/>
            <result column="sname" property="name"/>
            <result column="sage" property="age"/>
        </collection>
    </resultMap>
    <!--
        //查询全部
        public abstract List<Card> findAll();
    -->
    <select id="findAll" resultMap="oneToManyMap">
        select
               c.*,
               s.id sid,
               s.NAME sname,
               s.age sage
        from classes c,student s where c.id=s.cid;
    </select>
</mapper>
```

#### 4 测试类

```java
/*
        测试一对多关系：查询所有班级以及该班级的所有学生信息
     */
@Autowired
private OneToManyMapper oneToManyMapper;
@Test
void testOneToMany() {
    List<Classes> list = oneToManyMapper.findAll();
    list.forEach(classes->log.info("{}",classes));
}
```

#### ==注意：javaType属性和ofType属性的区别？==

**javaType的值用来表示bean对象的属性类型，ofType的值用来表示集合中元素的类型。**

### 多对多关系

#### 1 javabean类

```java
@Data
public class Student {
    private Integer id;     //主键id
    private String name;    //学生姓名
    private Integer age;    //学生年龄
    private List<Course> courses;   // 学生所选择的课程集合
}
@Data
public class Course {
    private Integer id;     //主键id
    private String name;    //课程名称
}
```

#### 2 接口类

```java
public interface ManyToManyMapper {
  //查询全部
  public abstract List<Student> findAll();
}
```

#### 3 映射文件配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.ManyToManyMapper">

    <resultMap id="manyToManyMap" type="com.itheima.pojo.Student" autoMapping="true">
        <id column="id" property="id"/>
        <!--配置一对多
            collection标签：配置一对多关系，给Bean中的集合封装数据
            ofType="com.itheima.pojo.Course"：集合中的泛型类型
        -->
        <collection property="courses" ofType="com.itheima.pojo.Course">
            <id column="cid" property="id"/>
            <result column="cname" property="name"/>
        </collection>
    </resultMap>
    <!--
        //查询全部
        public abstract List<Card> findAll();
    -->
    <select id="findAll" resultMap="manyToManyMap">
       select s.id,
       s.NAME,
       s.age,
       c.id   cid,
       c.NAME cname
from student s
         left outer join stu_cr sc on s.id = sc.sid
         left outer join course c on c.id = sc.cid
    </select>
</mapper>
```

#### 4 测试类

```java
@Autowired
private ManyToManyMapper manyToManyMapper;
@Test
void testManyToMany() {
    List<Student> list = manyToManyMapper.findAll();
    list.forEach(student->log.info("{}",student));
}
```



## mybatis多表操作【方式二】

> 思想：将多表查询拆成多次单表查询执行

### 一对一关系

#### 1 映射文件配置OneToOneMapper.xml

resultMap标签的作用：手动定义查询结果和bean对象的映射关系，一般用来处理查询结果和bean的属性不一致情况。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.OneToOneMapper">

    <!--
        autoMapping="true" :自动映射，如果列名和bean的属性名一样，就能自动封装，不一样要手动封装
    -->
    <resultMap id="oneToOneMap" type="com.itheima.pojo.Card" autoMapping="true">
        <!--配置一对一,也就是给card中的 private Person p封装数据
            association标签：配置一对一关系
            property="p" ：表示要封装到的属性，往p对象身上封装
            javaType="com.itheima.pojo.Person" ：表示p对象的类型
            select="com.itheima.mapper.PersonMapper.findById":要执行的下一条SQL语句唯一标识
            column="pid" ：执行下一条SQL语句要传递的参数值
        -->
        <association property="p"
                     column="pid"
                     select="com.itheima.mapper.PersonMapper.findById">
        </association>
    </resultMap>
    <!--
        //查询全部
        public abstract List<Card> findAll();
    -->
    <select id="findAll" resultMap="oneToOneMap">
        select * from card
    </select>
</mapper>
```

#### 2 PersonMapper接口

```java
public interface PersonMapper {

    @Select("select * from person where id=#{id}")
    Person findById(int id);
}
```

### 一对多关系

#### 1 映射文件配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.OneToManyMapper">
    <resultMap id="oneToManyMap" type="com.itheima.pojo.Classes" autoMapping="true">
        <!--注主键id无法自动映射，只能手动映射了-->
        <id column="id" property="id"/>
        <!--配置一对多
            collection标签：配置一对多关系，给Bean中的集合封装数据
            ofType="com.itheima.pojo.Student"：集合中的泛型类型
        -->
        <collection property="students"
                    ofType="com.itheima.pojo.Student"
                    column="id"
                    select="com.itheima.mapper.StudentMapper.findByCid">
        </collection>
    </resultMap>
    <!--
        //查询全部
        public abstract List<Card> findAll();
    -->
    <select id="findAll" resultMap="oneToManyMap">
        select * from classes;
    </select>
</mapper>
```

#### 2 StudentMapper接口

```java
public interface StudentMapper {

    @Select("select * from student where cid=#{cid}")
    List<Student> findByCid(int cid);
}
```

### 多对多关系

#### 1 映射文件配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.ManyToManyMapper">

    <resultMap id="manyToManyMap" type="com.itheima.pojo.Student" autoMapping="true">
        <!--注主键id无法自动映射，只能手动映射了-->
        <id column="id" property="id"/>
        <!--配置一对多
            collection标签：配置一对多关系，给Bean中的集合封装数据
            ofType="com.itheima.pojo.Course"：集合中的泛型类型
        -->
        <collection property="courses"
                    ofType="com.itheima.pojo.Course"
                    column="id"
                    select="com.itheima.mapper.CourseMapper.findBySid">
        </collection>
    </resultMap>
    <!--
        //查询全部
        public abstract List<Card> findAll();
    -->
    <select id="findAll" resultMap="manyToManyMap">
        select * from student;
    </select>
</mapper>
```

#### 2 CourseMapper接口

```java
public interface CourseMapper {

    @Select("select c.* from stu_cr sc,course c where sc.cid=c.id and sc.sid=#{sid}")
    List<Course> findBySid(int sid);
}
```



