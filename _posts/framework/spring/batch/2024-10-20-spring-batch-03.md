---
title: Spring Batch - Chunk Model과 Tasklet Model
date: 2024-10-20 00:00:00 +0900
categories: [ Framework, Spring-Batch ]
tags: [ 스프링 배치, Spring-Batch, Chunk Model, Tasklet Model ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

# Spring Batch Study 3

*참고: [DEVOCEAN KIDO님 SpringBatch 연재 03](https://devocean.sk.com/blog/techBoardDetail.do?ID=166694)*

---

Spring Batch의 데이터 처리 방식으로 사용되는 **Chunk Model**과 **Tasklet Model**에 대해 알아보자.

## 1. Chunk 모델

Chunk Model이란 처리할 데이터를 **일정 단위(청크)**로 처리하는 방식이다.

### Chunk 모델 알아보기

- 대량의 데이터를 효율적으로 처리할 수 있도록 도와주며, **트랜잭션 관리**와 **성능 최적화**에 유리하다.
- `ChunkOrientedTasklet`은 청크 처리를 지원하는 **Tasklet**의 구체적인 클래스 역할을 수행한다.
  - 본 클래스의 `commit-interval`이라는 설정값을 이용하여 청크에 포함될 데이터의 최대 레코드 수(청크 Size)를 조정 가능하다.
- `ItemReader`, `ItemProcessor`, `ItemWriter` 는 청크 단위를 처리하기 위한 인터페이스이다.
  ![Chunk 모델](https://devocean.sk.com/editorImg/2024/8/19/59f6679fe7386daa27c694c126c6440439f2932006e57ea74116b765bafa5d86)

  *참고: [DEVOCEAN KIDO님 SpringBatch 연재 03](https://devocean.sk.com/blog/techBoardDetail.do?ID=166694)*
  - 위 시퀀스 다이어그램과 같이 `ChunkOrientedTasklet`은 `ItemReader`,`ItemProcessor`,`ItemWriter` 구현체를 각각 호출한다.
  - 이때 `ChunkOrientedTasklet`은 **청크 단위**에 따라 `ItemReader`,`ItemProcessor`,`ItemWriter` 를 **반복실행**한다.
  - 청크 크기만큼 `ItemReader`가 데이터를 읽어 들인다.
  - 읽어들인 `ItemProcessor`로 전달하고, 데이터를 처리한다.
  - `ItemProcessor`를 처리하고난 청크 단위 데이터가 `ItemWriter`로 전달되어 **저장**하거나, **파일처리**를 수행한다.

### Chunk-oriented processing

![Chunk-oriented processing](https://docs.spring.io/spring-batch/reference/_images/chunk-oriented-processing-with-item-processor.png)

*참고: [SpringBatch 공식 문서](https://docs.spring.io/spring-batch/reference/step/chunk-oriented-processing.html)*

- Spring Batch는 가장 일반적인 구현 방식으로 "Chunk-oriented(청크 지향)" 처리 스타일을 사용한다.
- **한 번에 하나**씩 데이터를 읽고 트랜잭션 경계 내에서 쓰여지는 **'청크'**를 만드는 것을 말한다.

<br>

---

## 2. ItemReader

`ItemReader`는 청크 기반의 데이터 처리에서 데이터를 읽어오는 역할을 담당하는 인터페이스다.  
데이터 청크 단위를 하나씩 읽어들여 `ItemProcessor`와 `ItemWriter`로 전달한다.  
`ItemReader`는 직접 커스텀 구현을 할 수 있지만 Spring Batch에서는 이미 구현된 다양한 `ItemReader 구현체`를 제공한다.

### FlatFileItemReader

- 파일로부터 데이터를 읽어오는 구현체이다.
- **플랫파일**(CSV 파일, 텍스트 파일 등)을 읽어 들인다.
- 읽어들인 데이터를 객체로 매핑하기 위해서 **delimeter**를 기준으로 매핑 룰을 이용하여 **객체로 매핑**한다.
- 혹은 입력에 대해서 **Resource object**를 이용하여 커스텀하게 매핑할 수도 있다.

### JdbcCursorItemReader / JdbcPagingItemReader는

- `JdbcCursorItemReader`는 데이터베이스에서 **커서 방식**으로 데이터를 읽어오는 구현체이다.
- SQL 쿼리를 통해 데이터를 **한 행씩** 읽어 올 수 있다.
- JDBC를 사용하여 SQL을 실행하고 데이터베이스의 레코드를 읽는다.
- `JdbcPagingItemReader`는 `JdbcTemplate`을 이용하여 각 페이지에 대한 SELECT SQL을 나누어 처리하는 방식으로 구현된다.

### MyBatisCursorItemReader / MyBatisPagingItemReader

- `MyBatisCursorItemReader`는 **MyBatis**를 사용하여 데이터베이스로부터 **커서 방식**으로 데이터를 읽어오는 구현체이다.
- **Paging**과 **Cursor**의 차이점은 **MyBatis** 구현 방법이 다를뿐이지 **JdbcXXXItemReader**와 동일하다
- `ItemReader` JPA구현이나 Hibernate와 연동하여 데이터베이스의 레코드를 읽어오는 `JpaPagingItemReader`, `HibernatePagingItemReader`,
  `HibernateCursor`를 제공한다.

### JmsItemReader / AmqpItemReader

- `JmsItemReader`는 **JMS(Java Message Service)**를 사용하여 **메시지 큐(Message Queue)**로부터 데이터를 읽어온다.
- **AMQP(Advanced Message Queuing Protocol)**를 사용하여 메시지 브로커로부터 메시지를 읽어온다.

<br>

---

## 3. ItemProcessor

`ItemProcessor`는 Spring Batch에서 데이터 변환과 가공을 담당하는 핵심 인터페이스이다.  
`ItemReader`에서 읽어들인 데이터를 `ItemWriter`로 전달하기 전에 가공하거나 필터링할 수 있는 역할을 한다.  
**chunk model**에서 없어도 되는 옵션이다.

### PassThroughItemProcessor

- 입력된 데이터를 **그대로** 반환하는 `ItemProcessor`로, 아무런 변환도 수행하지 않고 데이터를 그대로 전달한다.

### CompositeItemProcessor

- 여러 개의 `ItemProcessor`를 **순차적**으로 실행할 수 있는 구현체이다.
- `ValidatingItemProcessor`를 사용하여 **입력 확인**을 수행한 후 **비즈니스 로직을 실행**하려는 경우 활성화 된다.

### ValidatingItemProcessor

- 데이터를 처리하기 전에 **유효성 검사**를 수행하는 `ItemProcessor`이다.
- Spring Batch의 `Validator` 인터페이스를 구현하여 유효성 검사 로직을 정의할 수 있다.
- 일반적인 Spring `Validator` 의 어댑터인 `SpringValidator`와 `org.springframework.validation`의 규칙을 제공한다.

### ClassifierCompositeItemProcessor

- 각 입력 항목에 따라 여러 ItemProcessor중 **하나를** 선택하여 처리할 수 있도록 하는 ItemProcessor이다.
- 입력 데이터의 **타입**이나 **조건**에 따라 서로 다른 데이터 처리 로직을 적용할 수 있다.

<br>

---

## 4. ItemWriter

`ItemWriter`는 Spring Batch에서 데이터 **저장 및 출력**을 담당하는 인터페이스이다.  
배치 처리의 마지막 단계에서 데이터를 **파일, 데이터베이스, 메시지 큐 등**의 최종 저장소로 **쓰는** 역할을 한다.

### FlatFileItemWriter

- 처리된 Java 객체를 **플랫파일**(CSV 파일, 텍스트 파일 등)로 데이터를 쓰는 구현체이다.
- 파일에 쓰기 위한 형식을 **직접** 지정할 수 있으며, CSV 파일의 경우 **구분자**(Delimiter) 등을 설정할 수 있다.

### StaxEventItemWriter

- **XML 파일**로 데이터를 쓰는 구현체이다.
- JAXB를 통해 객체를 XML로 변환할 수 있다.

### JdbcBatchItemWriter

- **JDBC**를 사용하여 SQL을 수행하고 자바 객체를 데이터베이스에 저장한다.
- 내부적으로 JdbcTemplate를 사용하게 된다.

### JpaItemWriter

- **JPA**를 사용하여 데이터베이스에 저장하는 구현체이다.

### MyBatisBatchItemWriter

- **MyBatis**를 사용하여 자바 객체를 데이터베이스에 저장하는 구현체이다.

### JmsItemWriter / AmqpItemWriter

- **JMS(Java Message Service)**를 사용하여 **메시지 큐(Message Queue)**로 데이터를 쓰는 구현체이다.
- **AMQP(Advanced Message Queuing Protocol)**를 사용하여 메시지 브로커로 데이터를 쓴다.

<br>

---

## 5. Tasklet 모델

**한번에 하나**의 레코드만 읽어서 쓰기해야하는 경우 **Tasklet Model**이 적합하다.  
청크 단위의 처리가 딱 맞지 않을 경우 **Tasklet Model**이 유용하다.  
사용자는 Tasklet 모델을 사용하면서 Spring Batch에서 제공하는 `Tasklet 인터페이스`를 구현해야한다.

### SystemCommandTasklet

- **시스템 명령어**를 **비동기적**으로 실행하는 Tasklet이다.
- 명령 속성에 수행해야할 **명령어**를 직접 지정하여 사용할 수 있다.
- **비동기**로 실행되기 때문에 프로세스 도중 **타임아웃**을 설정하고, 시스템 명령의 실행 스레드를 취소할 수 있다.

### MethodInvokingTaskletAdapter

- **메서드 호출**을 수행하는 Tasklet이다.
- **POJO** 클래스의 특정 메소드를 실행하기 위한 Tasklet이다.
- **targetObject** 속성에 대상 클래스의 빈을 지정하고, **targetMethod** 속성에 실행할 메소드 이름을 지정한다.
