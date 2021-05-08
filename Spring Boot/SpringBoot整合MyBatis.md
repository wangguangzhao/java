1、创建maven工程，pom.xml

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.1.6.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

2、创建数据库表

```sql
create table student(
id int primary key auto_increment,
name varchar(11),
score double,
birthday date
);
```
3、创建对应的实体类

```java
package mybatis.entity;

import lombok.Data;

import java.util.Date;

@Data
public class Student {

    private Long id;
    private String name;
    private Double score;
    private Date birthday;
}
```

 4、创建StudentRepository 接口

```java
package mybatis.repository;

import mybatis.entity.Student;

import java.util.List;

public interface StudentRepository {

    public List<Student> findAll();
    public Student findById(Long id);
    public void save(Student student);
    public void update(Student student);
    public void deleteById(Long id);
}
```

5、在resource/mapping路径下创建studentReposit接口对应的Mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="mybatis.repository.StudentRepository">
    <select id="findAll"   resultType="Student">
        SELECT * FROM student
    </select>
    <select id="findById" parameterType="java.lang.Long"   resultType="Student">
        SELECT * FROM student where id = #{id}
    </select>
    <insert id="save" parameterType="Student">
        insert into Student (name,score,birthday) values (#{name},#{score},#{birthday},)
    </insert>
    <update id="update" parameterType="Student">
        update Student set name=#{name},score=#{score},birth=#{birth} where id = #{id}
    </update>
    <delete id="delete" parameterType="java.lang.Long">
        delete from Student where id = #{id}
    </delete>
</mapper>
```

6、创建StudentHandler，注入StudentRepository

```java
package mybatis.Controller;

import mybatis.entity.Student;
import mybatis.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class StudentHandler {
    @Autowired
    private StudentRepository studentRepository;

    @GetMapping("/findAll")
    public List<Student> findAll(){
        return studentRepository.findAll();
    }
    @GetMapping("/findById/{id}")
    public Student findById(@PathVariable Long id){
        return studentRepository.findById(id);
    }
    @PostMapping("/save")
    public void save(@RequestBody Student student){
        studentRepository.save(student);
    }
    @PutMapping("/update")
    public void update(@RequestBody Student student){
        studentRepository.update(student);
    }
    @DeleteMapping("/delete/{id}")
    public void delete(@PathVariable Long id){
        studentRepository.deleteById(id);
    }

}
```

7、创建配置文件

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mytest?useUnicode=true&characterEncoding=UTF-8
    username: guangzhao
    password: guangzhao
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis:
  mapper-locations: classpath:/mapping/*.xml
  type-aliases-package: mybatis.entity
```

8、创建启动类

```java
package mybatis;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("mybatis.repository")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}

```

