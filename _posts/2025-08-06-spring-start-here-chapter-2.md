---
layout: single
title: "Spring Start Here - Chapter 2: The Spring Context: Defining Beans"
date: 2025-08-06
categories: [Spring, Java]
tags: [spring, java, context, bean]
---

# Spring Start Here - Chapter 2: The Spring Context: Defining Beans

##  챕터 개요

이 장에서는 Spring 프레임워크의 핵심 요소인 **Context**(또는 Application Context)와 함께 작업하는 방법을 학습한다. Context는 애플리케이션 메모리에서 프레임워크가 관리하고자 하는 모든 객체 인스턴스를 추가하는 공간으로 생각할 수 있다.

##  학습 목표

- Spring Context의 필요성 이해
- Spring Context에 새로운 객체 인스턴스 추가하기

##  Spring Context란?

Spring은 기본적으로 애플리케이션에서 정의한 어떤 객체도 알지 못한다. Spring이 객체를 "볼" 수 있도록 하려면 해당 객체들을 Context에 추가해야 한다. 이렇게 Context에 추가된 객체 인스턴스를 **Bean**이라고 부른다.

> ** 핵심 개념**: Spring Context는 Spring이 관리해야 하는 인스턴스들을 담는 "버킷"과 같다. Spring은 Context에 추가된 인스턴스만 볼 수 있다.

##  Maven 프로젝트 생성

### 기본 설정

먼저 Maven 프로젝트를 생성하고 Spring Context 의존성을 추가해야 한다.

**pom.xml 설정:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>spring-context-example</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

### 기본 클래스 정의

예제를 위해 간단한 `Parrot` 클래스를 생성한다:

```java
public class Parrot {
    private String name;
    
    // getter와 setter 생략
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
```

##  Bean을 Spring Context에 추가하는 3가지 방법

### 1. @Bean 어노테이션 사용

**단계별 진행:**

#### 1단계: Configuration 클래스 생성

```java
public class ProjectConfig {
}
```

#### 2단계: @Bean 메서드 정의

```java
public class ProjectConfig {
    
    @Bean
    Parrot parrot() {
        var p = new Parrot();
        p.setName("Koko");
        return p;
    }
}
```

#### 3단계: Spring Context 초기화

```java
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        
        Parrot p = context.getBean(Parrot.class);
        System.out.println(p.getName()); // 출력: Koko
    }
}
```

#### 다양한 타입의 Bean 추가

```java
public class ProjectConfig {
    
    @Bean
    Parrot parrot() {
        var p = new Parrot();
        p.setName("Koko");
        return p;
    }
    
    @Bean
    String hello() {
        return "Hello";
    }
    
    @Bean
    Integer ten() {
        return 10;
    }
}
```

#### 동일한 타입의 여러 Bean 추가

```java
public class ProjectConfig {
    
    @Bean
    Parrot parrot1() {
        var p = new Parrot();
        p.setName("Koko");
        return p;
    }
    
    @Bean
    Parrot parrot2() {
        var p = new Parrot();
        p.setName("Miki");
        return p;
    }
    
    @Bean
    Parrot parrot3() {
        var p = new Parrot();
        p.setName("Riki");
        return p;
    }
}
```

**Bean 이름으로 참조하기:**
```java
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        
        Parrot p = context.getBean("parrot2", Parrot.class);
        System.out.println(p.getName()); // 출력: Miki
    }
}
```

#### Primary Bean 설정

```java
 @Bean @Primary
Parrot parrot2() {
    var p = new Parrot();
    p.setName("Miki");
    return p;
}
```

#### Bean 이름 커스터마이징

```java
// 다음 세 가지 방법 모두 동일한 결과
 @Bean(name = "miki")
 @Bean(value = "miki")  
 @Bean("miki")
Parrot parrot2() {
    var p = new Parrot();
    p.setName("Miki");
    return p;
}
```

### 2. Stereotype 어노테이션 사용 ( @Component)

#### 1단계: 클래스에 @Component 추가

```java
 @Component
public class Parrot {
    private String name;
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
```

#### 2단계: @ComponentScan 설정

```java
@ComponentScan(basePackages = "main")
public class ProjectConfig {
}
```

#### 3단계: 테스트

```java
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        
        Parrot p = context.getBean(Parrot.class);
        System.out.println(p); // 인스턴스의 기본 String 표현
        System.out.println(p.getName()); // null (이름을 설정하지 않았음)
    }
}
```

#### @PostConstruct를 이용한 초기화

**의존성 추가 (Java 11+):**
```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

** @PostConstruct 사용:**
```java
 @Component
public class Parrot {
    private String name;
    
    @PostConstruct
    public void init() {
        this.name = "Kiki";
    }
    
    // getter, setter 생략
}
```

### 3. 프로그래밍 방식 (registerBean)

동적으로 조건에 따라 Bean을 등록해야 하는 경우 사용한다.

```java
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        
        Parrot x = new Parrot();
        x.setName("Kiki");
        
        Supplier<Parrot> parrotSupplier = () -> x;
        
        context.registerBean("parrot1", Parrot.class, parrotSupplier);
        
        Parrot p = context.getBean(Parrot.class);
        System.out.println(p.getName()); // 출력: Kiki
    }
}
```

#### registerBean() 메서드 파라미터

```java
<T> void registerBean(
    String beanName,                    // Bean 이름
    Class<T> beanClass,                // Bean 클래스
    Supplier<T> supplier,              // Bean 인스턴스를 반환하는 Supplier
    BeanDefinitionCustomizer... customizers  // Bean 특성 설정 (선택사항)
);
```

#### Primary Bean으로 설정:
```java
context.registerBean("parrot1", 
    Parrot.class, 
    parrotSupplier, 
    bc -> bc.setPrimary(true));
```

##  방법별 비교표

| 특징 | @Bean | @Component | registerBean |
|------|-------|------------|--------------|
| **인스턴스 제어** | 완전 제어 | 생성 후 제어만 가능 | 완전 제어 |
| **다중 인스턴스** | ✅ | ❌ (클래스당 하나) | ✅ |
| **외부 라이브러리** | ✅ | ❌ | ✅ |
| **코드 간결성** | ❌ (메서드 필요) | ✅ | 보통 |
| **동적 등록** | ❌ | ❌ | ✅ |
| **Spring 버전** | 모든 버전 | 모든 버전 | Spring 5+ |

## ⚡ 장단점 상세 분석

### @Bean 어노테이션
**장점:**
- 인스턴스 생성을 완전히 제어 가능
- 동일한 타입의 여러 인스턴스를 Context에 추가 가능
- 모든 종류의 객체 인스턴스를 추가 가능

**단점:**
- Bean마다 별도의 메서드가 필요하여 보일러플레이트 코드 증가

### @Component (Stereotype 어노테이션)
**장점:**
- 적은 코드로 Bean 등록 가능
- 보일러플레이트 코드가 없어 설정이 깔끔함

**단점:**
- 프레임워크가 인스턴스를 생성한 후에만 제어 가능
- 클래스당 하나의 인스턴스만 추가 가능
- 애플리케이션이 소유한 클래스에만 사용 가능

### registerBean() (프로그래밍 방식)
**장점:**
- 조건부 Bean 등록 가능
- 커스텀 로직을 구현한 Bean 등록 가능

**단점:**
- Spring 5 이상에서만 사용 가능
- 복잡한 설정이 필요할 수 있음

##  실무 가이드라인

### 언제 @Bean을 사용할까?
- 라이브러리 클래스의 Bean을 등록할 때
- 동일한 타입의 여러 Bean이 필요할 때
- Bean 생성 과정을 완전히 제어해야 할 때
- String, Integer 같은 기본 타입을 Bean으로 등록할 때

### 언제 @Component를 사용할까?
- 애플리케이션에서 직접 작성한 클래스일 때
- 간단한 Bean 등록이 필요할 때
- 코드를 간결하게 유지하고 싶을 때
- 클래스당 하나의 인스턴스만 필요할 때

### 언제 registerBean을 사용할까?
- 런타임 조건에 따라 Bean을 동적으로 등록해야 할 때
- 복잡한 Bean 등록 로직이 필요할 때
- 특정 설정에 따라 다른 Bean을 등록해야 할 때

##  실제 사용 예시

**실무에서는 주로 이렇게 조합해서 사용한다:**

```java
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    
    // 외부 라이브러리 Bean
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
    
    // 설정이 필요한 Bean
    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("user");
        dataSource.setPassword("password");
        return dataSource;
    }
}

// 애플리케이션 클래스는 @Component 사용
 @Component
public class UserService {
    // 비즈니스 로직
}

 @Component
public class UserController {
    // 컨트롤러 로직
}
```

##  핵심 요약

1. **Spring Context는 Bean 컨테이너**: Spring이 관리하는 모든 객체의 저장소
2. **3가지 Bean 등록 방법**: 각각 고유한 사용 사례가 있음
3. **실무에서는 @Component 우선**: 간결함 때문에 주로 사용, 필요시 @Bean으로 보완
4. **동적 Bean 등록은 registerBean**: 복잡한 조건부 로직이 필요한 경우
5. **모듈러 설계**: Spring은 필요한 부분만 추가하는 모듈러 구조

##  다음 단계

이 장에서 Bean을 Spring Context에 추가하는 방법을 배웠다. 3장에서는 Context에 추가된 Bean들을 참조하고 Bean들 간의 관계를 설정하는 방법을 학습하게 된다.

---

*Bean 정의는 Spring 학습의 첫 번째 단계이자 가장 중요한 기초이다. 이 개념을 확실히 이해하고 넘어가자!*