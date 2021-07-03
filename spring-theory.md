---
title: /spring-theory
layout: page
permalink: /spring-theory
---

### Spring Boot Theory (frequent update)

`12.05.2021-`
@yb park 

---

CONTENTS
[1. Aspect Oriented Programming with Spring](#1-aspect-oriented-programming-with-spring)
    [0. Theory : Aspect-Oriented-Programming](#0-theory--aspect-oriented-programming)
        [0-0. Reference](#0-0-reference)
        [0-1. Introduction](#0-1-introduction)
        [0-2. AOP Concepts](#0-2-aop-concepts)
    [1. Spring AOP AspectJ pointcut expression examples](#1-spring-aop-aspectj-pointcut-expression-examples)
        [1-0. Reference](#1-0-reference)
        [1-1. How to math method signature patterns](#1-1-how-to-math-method-signature-patterns)
        [1-2. How to match class type signature patterns](#1-2-how-to-match-class-type-signature-patterns)
        [1-3. How to match class name patterns](#1-3-how-to-match-class-name-patterns)
        [1-4. How to combine pointcut expressions](#1-4-how-to-combine-pointcut-expressions)
    [2. Types of Advice](#2-types-of-advice)
        [2-0. Reference](#2-0-reference)
        [2-1. Syntax & Explanation](#2-1-syntax--explanation)
[2. Spring Boot Architecture & Flow Architecture](#2-spring-boot-architecture--flow-architecture)
        [2-0. Reference](#2-0-reference-1)
        [2-1. Introduction](#2-1-introduction)
        [2-2. Explanation](#2-2-explanation)
            [2-2-1. Spring Boot Architecture](#2-2-1-spring-boot-architecture)
            [2-2-2. Spring Boot Flow Architecture](#2-2-2-spring-boot-flow-architecture)
        [2-4. Remark](#2-4-remark)
[3. Java Persistence API(JPA)](#3-java-persistence-apijpa)
    [3-1. Java persistence API(JPA) architecture](#3-1-java-persistence-apijpa-architecture)
        [3-1-0. Reference](#3-1-0-reference)
        [3-1-1. Introduction](#3-1-1-introduction)
        [3-1-2. Explanation](#3-1-2-explanation)
        [3-1-3. Remark](#3-1-3-remark)

---

# 1. Aspect Oriented Programming with Spring
## 0. Theory : Aspect-Oriented-Programming
### 0-0. Reference

[[Spring] Spring AOP - 기본 이론편](https://sabarada.tistory.com/94)
[[Spring] 스프링 AOP (Spring AOP) 총정리 : 개념, 프록시 기반 AOP, @AOP](https://engkimbs.tistory.com/746)
[Chapter 6. Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/2.5.x/reference/aop.html)
[Spring AOP Tutorial - HowToDoInJava](https://howtodoinjava.com/spring-aop-tutorial/)
[AOP(3) - Dynamic Proxy](https://dahye-jeong.gitbook.io/spring/spring/2020-04-10-aop-dynamicproxy)

### 0-1. Introduction
![img2](/uploads/Untitled%20(4).png)
Aspect Oriented Programming, 즉 관점 지향 프로그래밍이란 어떠한 로직을 기준으로 핵심적인 관점, 부가적인 관점을 구분하여 관점 기준으로 각각 모듈화(공통된 로직 또는 기능 등을 하나의 단위로 묶어내는 것)하는 것을 말한다. 예를 들어 핵심적인 관점이란 핵심 비즈니스 로직을 의미하며, 부가적인 관점이란 핵심 로직을 실행하기 위해서 행해지는 데이터베이스 연결, 로깅, 파일 입출력 등을 말한다. AOP에서 각 관점을 기준으로 로직을 모듈화 한다는 것은 곧 코드들을 부분적으로 나누어서 모듈화 하겠다는 의미이며, 이 때 소스 코드 상에서 여러 부분에 반복 적으로 사용하는 코드들을 Crosscutting-concern이라고 명칭한다. 이러한 **Crosscutting-concern을 Aspect로 모듈화 하고, 핵심적인 비즈니스 로직에서 분리하여 재사용하는 것이 곧 AOP의 취지이다.**

Spring AOP enables Aspect-Oriented Programming in spring applications. In AOP, aspects enable the modularization of concerns such as transaction management, logging or security that cut across multiple types and objects (often termed **crosscutting concerns**).

AOP provides the way to dynamically add the cross-cutting concern before, after or around the actual logic using simple pluggable configurations. it make easy to maintain code in present and future as well. You can add/remove concerns without recompiling complete source code simply by changing configuration files.

### 0-2. AOP Concepts

1. Advice
    Advice represents taken by an aspect at a particular join point. 
    There are different types of advices:

    1-1. Before Advice: executes before a join point
    1-2. After Returning Advice: executes after a join point completes normally.
    1-3. After Throwing Advice: executes if method exits by throwing an exception.
    1-4. After (finally) Advice: executes after a join point regardless of join point exit whether normally of exceptional return.
    1-5. Around Advice: executes before and after a join point.

    기능적인 역할을 하는 모듈. 즉, 실제 비즈니스 로직을 제외한 기능적인 역할을 하는 코드들을 분리, 재사용할 수 있도록 가공한 것. Aspect는 Advice와 Point Cut으로 이루어져있다.

2. Pointcut
    It is an expression language of AOP that matches join points.
    Advice가 적용될 JoinPoint를 선정하기 위한 표현식, 또는 그 방법. 일반적으로 모든 Join Point에 Advice를 적용하는 것이 아니라, 특정 Join Point에 Advice를 적용하며, 이를 위해 Join Point를 선정하는 방법이다.

3. Introduction
    It means introduction of additional method and fields for a type. 
    It allows you to introduce new interface to any advised object.
    AOP를 적용할 때 내부적으로 코드를 생성(e.g. wrapping classes)하는 것.

4. Target Object
    ![img1](/uploads/Untitled%20(5).png)
    It is the object i.e. being advised by one or more aspects. It is also known as proxied object in spring because Spring AOP is implemented using runtime proxies.
    Advice가 적용되어지는 기존 메소드, 클래스 등을 의미한다.

5. Weaving
    ![img2](/uploads/Untitled%20(4).png)

    It is process of linking aspect with other application types or objects to create an advised object. Weaving can be done at compile time, load time or runtime. Spring AOP performs weaving at runtime.

    Aspect가 target에 적용되는 전체적인 과정을 의미. 즉, PointCut으로 지정된 Join Point에 Advice가 적용되어 Target을 호출 시 AOP Proxy가 만들어지는 과정.

---

## 1. Spring AOP AspectJ pointcut expression examples
### 1-0. Reference
[Spring aop aspectJ pointcut expression examples - HowToDoInJava](https://howtodoinjava.com/spring-aop/aspectj-pointcut-expressions/)

### 1-1. How to math method signature patterns
The most typical pointcut expressions are used to match a number of methods by their signatures.
For example, the following pointcut expression matches all of the methods declared in the **EmploeeManager** interface. The preceding wildcard matches methods with any modifier(public, protected and private) and any return type. The two dots in the argument list match any number of arguments.

*so the syntax :*
`execution(MODIFIER(omittable) RETURN-TYPE(ommitable) * PATH OF PACAKGE*(PARAMETER1, DEFINITE-PARAMETER1(e.g.Integer)));`

*also wildcards :*
- **asterisk** : matches with any return type also any modifier.
- **two dots** : matches with any number of arguments.

1. Match all methods within a class in another package
{% highlight java %}
execution(*com.howtodoinjava.EmployeeManager.*(..))
{% endhighlight %}

1. Match all methods within a class within same package
 {% highlight java %}
execution(* EmployeeManager.*(..))
{% endhighlight %}

3. Match methods in EmployeeManager with specific modifier
 {% highlight java %}
execution(MODIFIER * EmployeeManager.*(..))
{% endhighlight %}

4. Match all methods in EmployeeManager with specific modifier also return type
 {% highlight java %}
execution(MODIFIER RETURN-TYPE EmployeeManager.*(..))
{% endhighlight %}

5. Match methods in EmployeeManager with specific modifier also return type and parameter
 {% highlight java %}
execution(MODIFIER RETURN-TYPE EmployeeManager.*(PARAMETER1, ..))
{% endhighlight %}

6. Math methods in EmployeeManager with specific modifier also return type and parameter, definite parameters
 {% highlight java %}
execution(MODIFIER RETURN-TYPE.*(PARAMETER1, DEFINITE, PARAMETER1)) 
{% endhighlight %}

### 1-2. How to match class type signature patterns
when applied Spring AOP, the scope of these pointcuts will be narrowed to matching all method executions within the certain types only.

1. Match all methods defined in classes inside specific package
 {% highlight java %}
within(PACKAGE-PATH.*)
{% endhighlight %}

2. Match all methods defined in classes inside specific package also classes inside all sub-packages as well
 {% highlight java %}
within(PACKAGE-PATH..*)
{% endhighlight %}

3. Match all methods with a class in another package
 {% highlight java %}
within(PACKAGE-PATH.CLASS-NAME)
{% endhighlight %}

4. Match all methods with a class in same package
 {% highlight java %}
within(CLASS-NAME)
{% endhighlight %}

5. Match all methods within all implementing classes of specific interface
 {% highlight java %}
within(INTERFACE-NAME+)
{% endhighlight %}

*so the syntax :*
`within(PACKAGE-PATH(ommitable).CLASS-OR-INTERFACE-NAME(ommitable) WILDCARD(ommitable))`

*also wildcard :*
`+` - this plus sign match all implementations of an interface.

### 1-3. How to match class name patterns
We can match all beans as well having common naming pattern.
1. Match all methods defined in beans whose name ends with "Specific-keyword". It's quite easy one. Use an * to match anything preceding in bean name and then matching word.
 {% highlight java %}
bean(*SPECIFIC-KEYWORD)
{% endhighlight %}

### 1-4. How to combine pointcut expressions
In AspectJ, pointcut expressions can be combined with the operators e.g. `&& (and)`, `|| (or)`, and `! (not)`.

1. Match all methods with names ending with Manager and DAO
 {% highlight java %}
		bean(*SPECIFIC-KEYWORD) || bean(*SPECIFIC-KEYWORD)
{% endhighlight %}
---

## 2. Types of Advice
### 2-0. Reference
[[Spring] 스프링 AOP (Spring AOP) 총정리 : 개념, 프록시 기반 AOP, @AOP](https://engkimbs.tistory.com/746)

### 2-1. Syntax & Explanation
 {% highlight java %}
@Component // AOP를 사용하기 위해 해당 클래스를 Spring Bean으로 선언
@Aspect // Aspect를 나타내는 클래스라는 것을 명시
@Before // TargetMethod가 호출되기 전에 Advice 기능을 수행
@After // TargetMethod의 결과와 관계없이 해당 메소드가 완료되면 Advice 기능을 수행
@AfterReturning // Target이 성공적으로 결과값을 변환하면 Advice 기능을 수행
@AfterThrowing // Target이 예외를 던지는 경우 Advice 기능을 수행
@Around // Target의 호출 전후에 Advice 기능을 수행
@PerLogging // 경로 지정을 대체하여 이 Annotation이 붙은 포인트에 Advice 기능을 수행
{% endhighlight %}

---

# 2. Spring Boot Architecture & Flow Architecture
![img3](/uploads/Untitled%20(7).png)

## 2-0. Reference
[스프링과 DAO, DTO, Repository, Entity](https://velog.io/@agugu95/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%8C%A8%ED%84%B4%EA%B3%BC-DAO-DTO-Repository)
[[Java] DAO, DTO, Repository을 이해하자](https://azderica.github.io/00-java-repositorys/)
[Spring Boot Architecture - javatpoint](https://www.javatpoint.com/spring-boot-architecture#:~:text=It%20is%20used%20to%20create,above%20(hierarchical%20structure)%20it)

## 2-1. Introduction
Spring Boot is a module of the Spring Framework. It is used to create stand-alone, production-grade Spring based Applications with minimum efforts. It is developed on top of the core Spring Framework. Spring Boot follows a layered architecture in which each layer communicates with the layer directly below or above (hierarchical structure) it.

## 2-2. Explanation
### 2-2-1. Spring Boot Architecture
![img3](/uploads/Untitled%20(8).png)
Before understanding the Spring Boot Architecture, we must know the different layers and classes present it. There are four layers in Spring Boot are as follows:

**Presentation Layer :**
    The presentation layer handles the HTTP requests, translate the JSON parameter to object, and authenticates the request and transfer it to business layer. In short, it consists of **views**. i.e. frontend part.

**Business Layer :**
    The business layer handles all the business logic. it consists of service classes and uses services provided by data access layers. It also performs authorization and validation.

**Persistence Layer :**
    The persistence layer contains all the storage logic and translates business objects from and to database rows.

**Database Layer :**
    In the database layer, **CRUD(create, retrieve, update, delete)** operations are performed. ****

### 2-2-2. Spring Boot Flow Architecture
![img4](/uploads/Untitled%20(9).png)

Now we have validator classes, view classes, and utility classes. Spring Boot uses all the modules of Spring-like Spring MVC, Spring Data, etc. The architecture of Spring boot is the same as the architecture of Spring MVC, except one thing : *there is no need for **DAO** and **DAOImpl** classes in Spring boot.* 

1. Creates a data access layer and performs CRUD operation.
2. The client makes the HTTP requests (PUT or GET).
3. The request goes to the controller, and the controller maps that request and handles it. 
   After that, it calls the service logic if required.
4. In the service layer, all the business logic performs. It performs the logic on the data that is mapped to JPA with model classes. 
5. A JSP page is returned to the user if no error occurred.
<br>
<br>

## 2-4. Remark

---

# 3. Java Persistence API(JPA)
## 3-1. Java persistence API(JPA) architecture
![img3](/uploads/Untitled%20(10).png)
### 3-1-0. Reference
[Java persistence API - Tutorial](https://www.vogella.com/tutorials/JavaPersistenceAPI/article.html#jpaintro)
[[Spring] 영속성이란 (persistence)](https://hyeooona825.tistory.com/87)
[(JPA - 2) 영속성(Persistence) 관리](https://kihoonkim.github.io/2017/01/27/JPA(Java%20ORM)/2.%20JPA-%EC%98%81%EC%86%8D%EC%84%B1%20%EA%B4%80%EB%A6%AC/)
[JPA 기본 키 전략](https://www.icatpark.com/entry/JPA-%EA%B8%B0%EB%B3%B8-%ED%82%A4-%EC%A0%84%EB%9E%B5?category=858321)

### 3-1-1. Introduction
Data persistence is the ability to maintain data between application executions. Persistence is vital [enterprise applications](https://en.wikipedia.org/wiki/Enterprise_software)¹ because of the required access to relational databases. Applications that are developed for this environment must manage persistence themselves or use third-party solutions to handle database updates and retrievals with persistence. The Java Persistence API (JPA) provides a mechanism for managing persistence and object-relational mapping and functions for the [EJB(Enterprise JavaBeans) specifications](https://www.ibm.com/docs/en/was/9.0.5?topic=beans-enterprise-javabeans-ejb-30-specification)².

The JPA specification defines the object-relational mapping internally, rather than relying on vendor-specific mapping implementations. JPA is based on the Java Programming model that applies to Java EE environments, but JPA can function within a Java SE environment for testing application functions.

JPA represents a simplification of the persistence programming model. The JPA specification explicitly defines the object-relational mapping, rather than relying on vendor-specific mapping implementations. JPA standardizes the important task of object-relational mapping by using annotations or XML to map objects into one or more tables of a database. To further simplify the persistence programming model :

- The EntityManager API can persist, update, retrieve, or remove objects from a database.
- The EntityManager API and object-relational mapping metadata handle most of the database operations without requiring you to write JDBC or SQL code to maintain persistence.
- JPA provides a query language, extending the independent EJB(Enterprise JavaBeans) querying language (also known as JPQL), that you can to retreive objects without writing SQL queries specific to the database you are working with.
<br>
<br>
데이터 영속성은 어플리케이션 실행 사이에 데이터를 유지하는 능력이다. 영속성은 관계 지향 데이터 베이스에 접근하는 것이 요구되기 때문에 필수적인 전사적 어플리케이션이다. 이런 환경을 위해 개발된 어플리케이션들은 영속성과 함께 데이터베이스 업데이트와 회수를 다루기 위해 반드시 그들 스스로 영속성을 관리하거나 third-party 솔루션을 사용해야 한다. JPA는 영속성을 관리하는 메커니즘과 객체-관계형 매핑 및 EJB를 위한 함수들을 제공한다.

JPA 사양은 업체-특정적 매핑 구현법들에 의존하는 대신 객체-관계형 매핑을 내부적으로 정의한다. JPA는 Java EE 환경을 적용한 java 프로그래밍 모델에 기반하고 있으나 JPA는 어플리케이션 기능들을 테스트하기 위해 java SE 환경에서도 기능할 수 있다.

JPA는 영속성 프로그래밍 모델의 간소화를 대표한다. JPA사양은 업체-특정적인 매핑 구현법들에 의존하는 대신 객체-관계형 매핑을 명쾌히 정의한다. JPA는 객체를 하나 또는 그 이상의 데이터베이스 테이블에 매핑하기 위해 주석 또는 XML을 사용하는 객체-관계형 매핑의 중요한 과업을 표준화한다. 더 나아가 영속성 프로그래밍 모델을 간략화하기 위해 :

- 객체관리자 API는 지속, 업데이트, 회수 그리고 데이터베이스로부터 객체를 제거할 수 있다.
- 객체관리자 API와 객체-관계형 매핑 메타데이터는 영속성을 유지하고자 JDBC 또는 SQL 코드를 작성하는 것을 필수로 하지 않고 대부분의 데이터베이스 operation들을 다룬다.
- JPA는 현재 작업 중인 데이터베이스에 특정하지 않고 SQL 질의문 작성없이 객체들을 회수할 수 있는 독립적인 EJP 질의 언어들(JPQL로도 알려진)를 포함한 질의 언어를 제공한다.
<br>
<br>
### 3-1-2. Explanation

### 3-1-3. Remark