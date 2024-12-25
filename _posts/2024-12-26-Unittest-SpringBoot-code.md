---
layout: post
title:  "Unit test Spring Boot application code"
date:   2024-12-24 19:41:42 +0530
categories: General
---

**JUnit and Spring Boot Test**

JUnit is the underlying testing framework when using Spring Boot Test. Spring Boot Test uses JUnit to define and execute test cases. 

The annotation @SpringBootTest - Loads the full Spring application context for testing.

**Types of tests..**

1. Unit test 
2. Integration test 
3. Hybrid test

**Unit test**

Used while testing individual method or class.  
This can be used in the inital stage of development when we write a class or method. We may face chllenges that one of the method we want to call has private visbility or class has some other dependencies. There are solutions for it. We will look at the latter case where one of the depency is mocked.

Snippet of code when using Spring Boot test with mocking
```
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @MockBean
    private UserRepository userRepository;

    @Test
    public void testFindUser() {
        when(userRepository.findById(1L)).thenReturn(Optional.of(new User("John")));

        User user = userService.findUserById(1L);

        assertNotNull(user);
        assertEquals("John", user.getName());
    }
}
```

In the above, @MockBean is a spring annotation and can register mock objects in the spring application context

**Integration test**

Used in context where

- Ensuring data flows correctly between modules.
- Testing actual service-to-service or service-to-database interactions.

For integration tests, one can connect to an in memory database like H2 where we can have dummy data required for tests. We need to populate this database on startup of Junit test cases.

Below is a view of how we place files in Spring Boot application
```
src/test/resources/
│
├── application-test.properties
├── schema.sql  # Your DDL statements
└── data.sql    # Your DML statements
```

Contents of application-test.properties
```
# Use H2 as the in-memory database
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.datasource.platform=h2

# Automatically initialize the schema and data
spring.sql.init.mode=always
spring.sql.init.schema-locations=classpath:schema.sql
spring.sql.init.data-locations=classpath:data.sql

# H2 console (optional for debugging)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```
schema.sql contents
```
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL
);
```
data.sql contents
```
INSERT INTO users (name, email) VALUES ('John Doe', 'john.doe@example.com');
INSERT INTO users (name, email) VALUES ('Joe Martin', 'joe.martin@example.com');
```

Test case that uses H2

```
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;

import javax.persistence.EntityManager;
import javax.persistence.Query;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
@ActiveProfiles("test")  // Loads application-test.properties
public class H2DatabaseTest {

    @Autowired
    private EntityManager entityManager;

    @Test
    public void testDatabaseInitialization() {
        Query query = entityManager.createQuery("SELECT u.name FROM User u");
        List<String> names = query.getResultList();

        assertEquals(2, names.size());
        assertEquals("John Doe", names.get(0));
        assertEquals("Jeo Martin", names.get(1));
    }
}
```

**Hybrid approach**

Here we employ both unit testing and integrations test approach. In practical use cases this may be ideal so that we can leverage on the benefits of both.

**Final notes..**

**Quality gate**

For Continous Integration/Continous Deployement (CI/CD) cycle, its quite often that Quality gate plugins (for tools like SonarQube) are configured to detect code coverage, code smells , duplication of code etc.

This will help to enforce the code developed meets certain criteria.









