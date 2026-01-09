//Lms-project-java
//The LMS enables:  Course creation and content management  Student enrollment &amp; assignment workflows  Performance tracking for learners  System monitoring and analytics for administrators  Admin → Manages users, system, and analytics Instructor → Builds courses &amp; evaluates students Student → Learns, submits work, tracks progress
//pom.xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lms</groupId>
    <artifactId>lms</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>jakarta.validation</groupId>
            <artifactId>jakarta.validation-api</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

//application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/lms_db
spring.datasource.username=root
spring.datasource.password=yourpassword

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect

//LmsApplication.java
package com.lms;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class LmsApplication {

    public static void main(String[] args) {
        SpringApplication.run(LmsApplication.class, args);
    }
}

//User.java
package com.lms.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;

@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Name is required")
    private String name;

    @Email(message = "Invalid email")
    @Column(unique = true)
    private String email;

    @NotBlank(message = "Password is required")
    private String password;

    @NotBlank(message = "Role is required")
    private String role; // ADMIN, STUDENT, INSTRUCTOR

    // Getters and Setters
}

//Course.java
package com.lms.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;

@Entity
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Course title required")
    private String title;

    private String description;

    // Getters and Setters
}

//UserRepository.java
package com.lms.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import com.lms.model.User;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);
}

//CourseRepository.java
package com.lms.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import com.lms.model.Course;

public interface CourseRepository extends JpaRepository<Course, Long> {
}

//UserService.java
package com.lms.service;

import org.springframework.stereotype.Service;
import com.lms.model.User;
import com.lms.repository.UserRepository;

@Service
public class UserService {

    private final UserRepository repo;

    public UserService(UserRepository repo) {
        this.repo = repo;
    }

    public User register(User user) {
        return repo.save(user);
    }
}

//CourseService.java
package com.lms.service;

import org.springframework.stereotype.Service;
import java.util.List;
import com.lms.model.Course;
import com.lms.repository.CourseRepository;

@Service
public class CourseService {

    private final CourseRepository repo;

    public CourseService(CourseRepository repo) {
        this.repo = repo;
    }

    public Course addCourse(Course course) {
        return repo.save(course);
    }

    public List<Course> getAllCourses() {
        return repo.findAll();
    }
}

//AuthController.java
package com.lms.controller;

import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.*;
import com.lms.model.User;
import com.lms.service.UserService;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final UserService service;

    public AuthController(UserService service) {
        this.service = service;
    }

    @PostMapping("/register")
    public User register(@Valid @RequestBody User user) {
        return service.register(user);
    }
}

//CourseController.java
package com.lms.controller;

import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.*;
import java.util.List;
import com.lms.model.Course;
import com.lms.service.CourseService;

@RestController
@RequestMapping("/api/courses")
public class CourseController {

    private final CourseService service;

    public CourseController(CourseService service) {
        this.service = service;
    }

    @PostMapping
    public Course addCourse(@Valid @RequestBody Course course) {
        return service.addCourse(course);
    }

    @GetMapping
    public List<Course> getCourses() {
        return service.getAllCourses();
    }
}

//GlobalExceptionHandler.java
package com.lms.exception;

import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public String handleValidation(MethodArgumentNotValidException ex) {
        return ex.getBindingResult().getFieldError().getDefaultMessage();
    }

    @ExceptionHandler(Exception.class)
    public String handleAll(Exception ex) {
        return "Error: " + ex.getMessage();
    }
}




