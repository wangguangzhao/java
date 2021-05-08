1、创建maven工程，pom.xml

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.1.6.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
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

``` sql
create table student(
id int primary key auto_increment,
name varchar(11),
score double,
birthday date
);
```

3、创建对应的实体类

```java
package jdbc.entity;

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
package jdbc.repository;

import jdbc.entity.Student;

import java.util.List;

public interface StudentRepository {

    public List<Student> findAll();

    public Student findById(Long id);

    public void save(Student student);

    public void update(Student student);

    public void delete (Long id);
}
```

5、在application.yml中配置数据源

```xml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mytest?useUnicode=true&characterEncoding=UTF-8
    username: guangzhao
    password: guangzhao
    driver-class-name: com.mysql.cj.jdbc.Driver
```

6、实现StudentRepository 接口

```java
package jdbc.repository.Impl;

import jdbc.entity.Student;
import jdbc.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;
@Repository
public class StudentRepositoryImpl implements StudentRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;


    public List<Student> findAll() {

       return  jdbcTemplate.query("select * from student",new BeanPropertyRowMapper(Student.class));
    }

    public Student findById(Long id) {

        return (Student) jdbcTemplate.queryForObject("select * from student where id = ?",new Object[]{id},new BeanPropertyRowMapper(Student.class));
    }

    public void save(Student student) {
        jdbcTemplate.update("insert into student(name,score,birthday) values (?,?,?)",student.getName(),student.getScore(),student.getBirthday());
    }

    public void update(Student student) {
        jdbcTemplate.update("update student set name=?,score=?,birthday=? where id = ?",student.getName(),student.getScore(),student.getBirthday(),student.getId());
    }

    public void delete(Long id) {
        jdbcTemplate.update("delete from student where id=?",id);
    }
}

```

7、创建控制器 StudentHandler

```java
package jdbc.controller;

import jdbc.entity.Student;
import jdbc.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/Student")
public class StudentHandler {

    @Autowired
    private StudentRepository studentRepository;
    @GetMapping("/findAll")
    public List<Student> findAll(){

       return studentRepository.findAll();

    }
    @GetMapping("/findById/{id}")
    public Student findById(@PathVariable Long id){

        return  studentRepository.findById(id);
    }

    @PostMapping("/save")
    public void save (@RequestBody Student student){

          studentRepository.save (student);
    }
    @PutMapping("/update")
    public void update (@RequestBody Student student){

        studentRepository.update (student);
    }

    @GetMapping("/delete/{id}")
    public void delete(@PathVariable Long id){
          studentRepository.delete(id);
    }




}

```

8、创建启动类

```java
package jdbc;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args){
        SpringApplication.run(Application.class,args);
    }
}
```

