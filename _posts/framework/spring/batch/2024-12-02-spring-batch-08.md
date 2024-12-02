---
title: Spring Batch - CompositeItemProcessor로 여러 processor를 순차적으로 실행하기
date: 2024-12-02 00:00:00 +0900
categories: [ Framework, Spring-Batch ]
tags: [ 스프링 배치, Spring-Batch, CompositeItemProcessor ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

# Spring Batch Study 8

*참고: [DEVOCEAN KIDO님 SpringBatch 연재 08](https://devocean.sk.com/blog/techBoardDetail.do?ID=166950)*

---

Spring Batch의 `CompositeItemProcessor` 으로 여러 단계에 걸쳐 데이터를 Transform 하는 과정을 진행해본다.

## 1. CompositeItemProcessor

### 1-1. 개요

- `CompositeItemProcessor`는 Spring Batch에서 제공하는 `ItemProcessor` 인터페이스를 구현하는 클래스이다.
- 여러 개의 ItemProcessor를 **하나의 Processor**로 연결하여, 데이터를 여러 단계의 처리를 수행할 수 있도록 한다.

### 1-2. 주요 구성 요소

- **Delegates**: 처리에 사용할 ItemProcessor들의 목록이다. CompositeItemProcessor는 이 리스트에 포함된 각 Processor를 순차적으로 호출한다.
- **TransactionAttribute**: 트랜잭션 속성을 설정하여 각 단계의 데이터 처리가 하나의 트랜잭션에서 수행될지 여부를 결정한다.

### 1-3. CompositeItemProcessor 동작 원리

- **초기 입력**: ItemReader에서 데이터를 읽어 첫 번째 ItemProcessor로 전한다.
- **단계별 처리**: 각 ItemProcessor는 데이터를 변환하거나 가공하며, 변환된 데이터를 다음 ItemProcessor로 넘긴다.
- **최종 출력**: 마지막 ItemProcessor에서 처리된 결과는 ItemWriter로 전달된다.

### 1-4. 장점

- **단계별 처리**
  - 여러 단계로 나누어 처리를 수행하여 코드를 명확하고 이해하기 쉽게 만들 수 있다.
  - 단계별로 테스트 및 디버깅이 용이하다.
- **재사용 가능성**
  - 각 Processor는 독립적으로 설계되므로, 다른 Job이나 Step에서도 재사용할 수 있다.
  - 비즈니스 로직의 모듈화(Modularity)가 가능해진다.
- **유연성**
  - 다양한 ItemProcessor를 조합하여 원하는 처리 과정을 구현할 수 있다.

### 1-5. 단점

- **설정 복잡성**
  - 여러 개의 Processor를 설정하고 관리해야 하기 때문에 설정이 복잡해질 수 있다.
  - Processor의 순서를 잘못 설정하거나 의존 관계를 잘못 설계하면 의도한 결과를 얻지 못할 수 있다.
- **성능 저하**
  - 각 Processor를 순차적으로 호출하기 때문에, 단계 수가 많아질수록 성능에 영향을 줄 수 있다.

<br>

---

## 2. CompositeItemProcessor 구현

### 2-1. 코드 구현

일단 순서대로 진행할 ItemProcessor를 구현하여 step에 적용시킨다.

```java
public class LowerCaseItemProcessor implements ItemProcessor<Customer, Customer> {
  /**
   * 이름, 성별을 소문자로 변경하는 ItemProcessor
   */
  @Override
  public Customer process(Customer item) {
    System.out.println("### LowerCaseItemProcessor execute");
    item.setName(item.getName().toLowerCase());
    item.setGender(item.getGender().toLowerCase());
    return item;
  }
}

public class After20YearsItemProcessor implements ItemProcessor<Customer, Customer> {
  /**
   * 나이에 20년을 더하는 ItemProcessor
   */
  @Override
  public Customer process(Customer item) {
    System.out.println("### After20YearsItemProcessor execute");
    item.setAge(item.getAge() + 20);
    return item;
  }
}
```

- 이름,성별을 소문자로 변환하는 ItemProcessor와 나이에 20년을 더하는 ItemProcessor 구현체를 생성한다.
- 각 proccesor에 실행 순서를 기록할 로그를 추가하였다.

MyBatisReaderJobConfig에 `CompositeItemProcessor` 빈 설정을 추가해준다.

```java

@Slf4j
@Configuration
public class MyBatisReaderJobConfig {
  ...

  @Bean
  public CompositeItemProcessor<Customer, Customer> compositeItemProcessor() {
    return new CompositeItemProcessorBuilder<Customer, Customer>()
      // delegates를 통해서 ItemProcessor가 수행할 순서대로 배열을 만들어 전달
      .delegates(List.of(
        new LowerCaseItemProcessor(),
        new After20YearsItemProcessor()))
      .build();
  }

  @Bean
  public Step CustomerJdbcCursorStep(JobRepository jobRepository,
                                     PlatformTransactionManager transactionManager) throws Exception {
    log.info("------------------ Init CustomerJdbcCursorStep -----------------");

    return new StepBuilder("CustomerJdbcCursorStep", jobRepository)
      .<Customer, Customer>chunk(CHUNK_SIZE, transactionManager)
      .reader(myBatisItemReader())
      .processor(compositeItemProcessor())
      .writer(CustomerCursorFlatFileItemWriter())
      .build();
  }
  
  ...

}
```

- **CompositeItemProcessorBuilder**의 **delegates()** 메서드에 LowerCaseItemProcessor, After20YearsItemProcessor 순으로 리스트를
  전달한다.
- step의 processor 설정에 **compositeItemProcessor** 빈을 전달한다.

### 2-1. 실행 및 결과

먼저 customer 테이블의 데이터를 확인해보자.

![img.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/8_week/img.png)

이제 스프링배치를 실행하여 DB에서 읽어온 데이터를 ItemProcessor 목록을 순차적으로 거쳐서 진행되는지 확인해보자.

![img_1.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/8_week/img_1.png)

실행 로그를 보면 step이 실행되고 ItemProceesor가 LowerCaseItemProcessor, After20YearsItemProcessor 순으로 순차적으로 실행된 로그를 확인할 수 있다.  
customer 테이블에서 네 개의 row를 읽어서 처리하였기 때문에 processor가 각각 **네 번** 실행되었다.

이제 생성된 csv 파일을 확인해보자.

![img_2.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/8_week/img_2.png)

정상적으로 대문자였던 이름과 성별이 소문자로 변환되었고, 나이가 20씩 증가하여 csv 파일에 저장된 것을 확인할 수 있다.
