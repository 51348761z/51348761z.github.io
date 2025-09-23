---
title: Running a simple SpringBoot project from scratch
date: 2025-09-18 16:33:48 +0800
categories: [Java, SpringBoot]
tags: [springboot, mysql, java]
mermaid: true
description: A minimal Spring Boot + Spring Data JPA + MySQL Student CRUD service implementation from scratch.
---

This guide focuses only on how to implement a minimal Spring Boot + Spring Data JPA + MySQL Student CRUD service.  
All general concepts (layered architecture, REST design, JPA annotations, ID strategies, best practices, pitfalls) are moved to [Spring Boot Core Concepts](/posts/SpringBoot-core-concepts/).

## Table of Contents
1. Project Initialization
2. Dependencies & Structure
3. Database Setup (DDL + Seed)
4. Entity
5. Repository
6. Service Layer
7. Controller Layer (REST API)
8. Common Issues & Quick Diagnostics

---

## 1. Project Initialization

Use Spring Initializr [https://start.spring.io](https://start.spring.io):
- Project: Maven / Java 17+
- Dependencies: Spring Web, Spring Data JPA, MySQL Driver
- Packaging: Jar

## 2. Dependencies & Structure

Core dependencies (managed via starters):
- spring-boot-starter-web
- spring-boot-starter-data-jpa
- mysql-connector-j

Minimal directory layout (irrelevant files omitted):
```
src/main/java/com/example/school/
  student_service/
    dao/
    service/
    controller/
src/main/resources/
  application.properties
```

application.yaml example:
```yaml
spring:
  datasource:
    username: root
    password: 1
  application:
    name: student-service
  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
```

## 3. Database Setup

Execute schema and (optional) seed data:
```sql
CREATE DATABASE IF NOT EXISTS school_db;
USE school_db;

CREATE TABLE students (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(120) NOT NULL UNIQUE,
  major VARCHAR(100),
  enrollment_date DATE,
  date_of_birth DATE,
  age INT,
  active TINYINT(1) DEFAULT 1
);

INSERT INTO students (first_name,last_name,email,major,enrollment_date,date_of_birth,age,active) VALUES
('John','Doe','john.doe1@example.com','Computer Science','2023-09-01','2003-05-14',22,1),
('Jane','Smith','jane.smith2@example.com','Information Systems','2022-09-01','2002-11-02',22,0);
```

## 4. Entity

Keep it lean: persistence-only, no business logic; avoid Lombok `@Data` to prevent `equals/hashCode` or lazy-loading side effects.  
Requires: no-arg constructor + fields + getters/setters.

```java
// src/main/java/com/example/school/student_service/dao/Student.java
package com.example.school.student_service.dao;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name="first_name", nullable=false, length=50)
    private String firstName;

    @Column(name="last_name", nullable=false, length=50)
    private String lastName;

    @Column(name="email", nullable=false, unique=true, length=120)
    private String email;

    @Column(name="major", length=100)
    private String major;

    @Column(name="enrollment_date")
    private LocalDate enrollmentDate;

    @Column(name="date_of_birth")
    private LocalDate dateOfBirth;

    private Integer age;
    private Boolean active = Boolean.TRUE;

    public Student() {}

    public Student(String firstName, String lastName, String email) {
        this.firstName = firstName; this.lastName = lastName; this.email = email;
    }

    // Getters & Setters omitted...
}
```
For annotations and ID strategies, see the [Core Concepts](https://51348761z.github.io/posts/SpringBoot-core-concepts/) article.

## 5. Repository

Use method name derivation for clarity; avoid manual SQL unless necessary.  
Conventions: `findById` / `deleteById` / `findByEmail` / `findByActiveTrue`.

```java
// src/main/java/com/example/school/student_service/dao/StudentRepository.java
package com.example.school.student_service.dao;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

import java.util.List;
import java.util.Optional;

public interface StudentRepository extends JpaRepository<Student, Long>, JpaSpecificationExecutor<Student> {

    Optional<Student> findByEmail(String email);

    List<Student> findByAgeBetween(Integer start, Integer end);

    List<Student> findByActiveTrue();

    List<Student> findByFirstNameStartingWith(String prefix);
}
```

## 6. Service Layer

Responsibilities: validation (e.g. unique email), transactional boundaries, controlled updates.  
Manual field copying prevents unintended overwrites.

```java
// src/main/java/com/example/school/student_service/service/StudentService.java
package com.example.school.student_service.service;

import com.example.school.student_service.dao.Student;
import java.util.List;
import java.util.Optional;

public interface StudentService {
    Student create(Student student);
    List<Student> findAll();
    Optional<Student> findOne(Long id);
    Student update(Long id, Student incoming);
    void delete(Long id);
    List<Student> findActive();
}
```

```java
// src/main/java/com/example/school/student_service/service/StudentServiceImpl.java
package com.example.school.student_service.service;

import com.example.school.student_service.dao.Student;
import com.example.school.student_service.dao.StudentRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional
public class StudentServiceImpl implements StudentService {

    private final StudentRepository repo;

    public StudentServiceImpl(StudentRepository repo) {
        this.repo = repo;
    }

    @Override
    public Student create(Student student) {
        repo.findByEmail(student.getEmail()).ifPresent(s ->
            { throw new IllegalArgumentException("Email already exists: " + s.getEmail()); });
        return repo.save(student);
    }

    @Override
    @Transactional(readOnly = true)
    public List<Student> findAll() {
        return repo.findAll();
    }

    @Override
    @Transactional(readOnly = true)
    public java.util.Optional<Student> findOne(Long id) {
        return repo.findById(id);
    }

    @Override
    public Student update(Long id, Student incoming) {
        return repo.findById(id).map(existing -> {
            existing.setFirstName(incoming.getFirstName());
            existing.setLastName(incoming.getLastName());
            existing.setEmail(incoming.getEmail());
            existing.setMajor(incoming.getMajor());
            existing.setEnrollmentDate(incoming.getEnrollmentDate());
            existing.setDateOfBirth(incoming.getDateOfBirth());
            existing.setAge(incoming.getAge());
            existing.setActive(incoming.getActive());
            return existing;
        }).orElseThrow(() -> new IllegalArgumentException("Student not found id=" + id));
    }

    @Override
    public void delete(Long id) {
        if (!repo.existsById(id)) {
            throw new IllegalArgumentException("Student not found id=" + id);
        }
        repo.deleteById(id);
    }

    @Override
    @Transactional(readOnly = true)
    public List<Student> findActive() {
        return repo.findByActiveTrue();
    }
}
```

## 7. Controller Layer

Adjustments:
- `@PutMapping` uses `@RequestBody` for update payload
- `DELETE` returns 204
- Simple inline error mapping; production systems should centralize via `@ControllerAdvice`

```java
// src/main/java/com/example/school/student_service/controller/StudentController.java
package com.example.school.student_service.controller;

import com.example.school.student_service.dao.Student;
import com.example.school.student_service.service.StudentService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.net.URI;
import java.util.List;

@RestController
@RequestMapping("/api/students")
public class StudentController {

    private final StudentService service;
    public StudentController(StudentService service) { this.service = service; }

    @PostMapping
    public ResponseEntity<Student> create(@RequestBody Student student) {
        Student saved = service.create(student);
        return ResponseEntity.created(URI.create("/api/students/" + saved.getId())).body(saved);
    }

    @GetMapping
    public List<Student> list() {
        return service.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Student> get(@PathVariable Long id) {
        return service.findOne(id).map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Student> update(@PathVariable Long id, @RequestBody Student payload) {
        try {
            return ResponseEntity.ok(service.update(id, payload));
        } catch (IllegalArgumentException e) {
            return ResponseEntity.notFound().build();
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        try {
            service.delete(id);
            return ResponseEntity.noContent().build();
        } catch (IllegalArgumentException e) {
            return ResponseEntity.notFound().build();
        }
    }

    @GetMapping("/active")
    public List<Student> active() {
        return service.findActive();
    }
}
```

## 8. Common Issues (Quick Reference)

| Symptom                          | Likely Cause                  | Action                            |
| -------------------------------- | ----------------------------- | --------------------------------- |
| Table 'students' doesn't exist   | DB not created / wrong schema | Verify schema & connection URL    |
| 400 / duplicate email constraint | Unique column violated        | Check existence before saving     |
| 404 on update/delete             | ID not found                  | existsById check before operation |
| LazyInitializationException      | Serializing lazy relations    | Use DTO (not applicable here)     |

Further topics (transactions, DTO mapping, exception handling, pagination, sorting, Specification queries) are in the [Core Concepts](/posts/SpringBoot-core-concepts/) article.

