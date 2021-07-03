---
layout: page
title: /spring-boot&JPA
permalink: /spring-boot&JPA
---
### Spring Boot & JPA - Documentation (frequent update)
`12.05.2021-`
@yb park 

---

CONTENTS
[1. Spring Boot basic Annotations](#1-spring-boot-basic-annotations)
  [1-0. Reference](#1-0-reference)
  [1-1. Introduction](#1-1-introduction)
  [1-2. Explanation](#1-2-explanation)
    [1-2-1. Spring Boot basic annotations](#1-2-1-spring-boot-basic-annotations)

---


# 1. Spring Boot basic Annotations
![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2b8a7759-d6b7-4e9c-89ff-cf5cb2c8acf6/Untitled.png](/uploads/Untitled%20(11).png)

## 1-0. Reference
[Spring Boot basic annotations](https://zetcode.com/springboot/annotations/)

## 1-1. Introduction
Spring Boot basic annotations tutorial shows how to use basic Spring Boot annotations including @Bean, @Service, @Configuration, @Controller, @RequestMapping, @Repository, @Autowired also @SpringBootApplication.

*Spring* is a popular Java application framework for creating enterprise applications. *Spring Boot* is the next step in evolution of Spring framework. It help create stand-alone, production-grade Spring based applications with minimal effort. It does not use XML configurations anymore and implements the [*convention over configuration*¹](https://en.wikipedia.org/wiki/Convention_over_configuration)  principle.

*Annotation* is a form of [metadata](https://whatis.techtarget.com/definition/metadata#:~:text=Metadata%20is%20data%20that%20describes,particular%20instances%20of%20data%20easier.)² which provides data about a program that is not part of the program it self. Annotations do not have direct effect on the operation of the code they annotate.

*Spring Boot basic annotations tutorial*은 @Bean, @Service, @Configuration, @Controller, @Requestmapping, @Repository, @Autowired 및 @SpringBootApplication을 포함하여 기본적인 Spring Boot annotations를 어떻게 사용할 지 보여준다.

*Spring*은 기업의 application들을 제작하기 위한 대중적인 java application framework이다. *Spring Boot*는 Spring framework 진화의 다음 단계이다. 이는 최소한의 노력으로 독립적이고, production-grade(상품으로의 수준)인 Spring 기반 application 제작에 도움을 준다. 이는 더 이상 XML 설정들과 *[관습에 의한 설정*¹](https://en.wikipedia.org/wiki/Convention_over_configuration) 원칙을 구현하지 않는다.

*Annotation*은 해당 프로그램의 일부가 아닌 프로그램에 관한 데이터를 제공하는 [메타데이터](https://whatis.techtarget.com/definition/metadata#:~:text=Metadata%20is%20data%20that%20describes,particular%20instances%20of%20data%20easier.)²의 양식이다. Annotations는 그들이 주석을 단 코드의 운영에는 직접적인 효과를 가지지 않는다.

## 1-2. Explanation
### 1-2-1. Spring Boot basic annotations
In the example application, we have these Spring Boot annotations:

- `@Bean` -  indicates that a method produces a bean to be managed by Spring.
- `@Service` - indicates that an annotated class a service class.
- `@Repository` - indicates that an annotated class is a repository, which is an abstraction of data access and storage.
- `@Configuration` - indicates that a class is a configuration class that may contain bean definitions.
- `@Controller` - marks the class as web controller, capable of handling the requests.
- `@RequestMapping` - maps HTTP request with a path to a controller method.
- `@Autowired` - marks a constructor, field or setter method to be auto-wired by Spring dependency Injection.
- `@SpringBootApplication` - enable Spring Boot autoconfiguration and component scanning.

`@Component` is a generic stereotype for a Spring managed component. It turns the class into a Spring bean at the auto-scan time. Classes decorated with this annotation are considered as candidates for auto-detection when using annotation-based configuration and class-path scanning. `@Repository`, `@Service`, and `@Controller` are specializations of `@Component` for more specific use cases.

There are also Hibernate³ `@Entity`, `@Table`, `@Id` and `@GeneratedValue` annotations in the example.
예시 application에서, 우리는 다음과 같은 Spring Boot annotations를 포함한다:

- `@Bean` -  Spring에 의해 관리되는 bean을 산출하는 method임을 시사한다.
- `@Service` - 주석이 달린 class가 service class임을 시사한다.
- `@Repository` - 주석이 달린 class가 repository, 즉 data 접근과 저장의 추상화임을 시사한다.
- `@Configuration` - class가 bean의 정의 등을 포함할 설정 class임을 시사한다.
- `@Controller` - class가 web controller, 즉 요청들의 handling이 가능함을 시사한다.
- `@RequestMapping` - HTTP 요청들과 경로를 controller method로 값을 대응한다.
- `@Autowired` - Spring 의존성 주입에 의해 constructor, field, 또는 setter method가 자동 주입 됨을 표시한다.
- `@SpringBootApplication` - Spring Boot autoconfiguration(자동 설정)과 component scanning을 가능하게 한다.

---