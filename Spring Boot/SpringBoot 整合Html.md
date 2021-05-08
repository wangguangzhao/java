SpringBoot可以结合Thymeleaf模版来整合HTML，使用原生的Html作为视图

Thymlef模版是面向web和独立环境的Java模版引擎，能够处理html、xml、javascript、css

```html
<p th:text="${message}"></p>
```

1、pom.xml添加相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2、application.yml配置

```xml
thymeleaf:
  prefix: classpath:/templates
  suffix: .html
  mode: HTML5
  encoding: UTF-8
```

3、添加控制器类

```java
package mybatis.Controller;

import mybatis.entity.Student;
import mybatis.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
@RequestMapping("/index")
public class IndexHandler {

    @Autowired
    private StudentRepository studentRepository;
    @GetMapping("/index")
    public String index(Model model){

        List<Student> list = studentRepository.findAll();
        System.out.println(list);
        model.addAttribute("list",list);
        return "index";
    }
}

```

4、添加模版文件

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"></html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index测试</title>
</head>
<body>
    <table>
        <tr>
            <th>学生id</th>
            <th>学生姓名</th>
            <th>学生成绩</th>
            <th>学生生日</th>
        </tr>
        <tr th:each="student:${list}">
            <td th:text="${student.id}"></td>
            <td th:text="${student.name}"></td>
            <td th:text="${student.score}"></td>
            <td th:text="${student.birthday}"></td>
        </tr>
    </table>
</body>
</html>
```

如果希望客户端直接访问HTML资源，将这些资源放置到static路径下即可。否则必须通过handler的后台映射才可以访问静态资源

### Thymeleaf常用语法

- 赋值拼接
- 

