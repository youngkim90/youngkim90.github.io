---
title: Spring Batch - JpaPagingItemReader/JpaItemWriter 구현
date: 2024-11-12 00:00:00 +0900
categories: [ Framework, Spring-Batch ]
tags: [ 스프링 배치, Spring-Batch, JpaPagingItemReader, JpaItemWriter ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

# Spring Batch Study 6

*참고: [DEVOCEAN KIDO님 SpringBatch 연재 06](https://devocean.sk.com/blog/techBoardDetail.do?ID=166902)*

---

Spring Batch의 `JpaPagingItemReader`와 `JpaItemWriter`로 DB 데이터를 읽고 쓰는 방법을 알아보자.

## 1. JpaPagingItemReader/JpaItemWriter 개요

### 1-1. JpaPagingItemReader 개요

- Spring Batch에서 제공하는 `ItemReader` 인터페이스를 구현하는 클래스이다
- JPA를 사용하여 데이터베이스로부터 데이터를 페이지 단위로 읽어온다.
- **JPA 기능 활용**: JPA 엔티티 기반 데이터 처리, 객체 매핑 자동화 등 JPA의 다양한 기능을 활용할 수 있다.
- **쿼리 최적화**: JPA 쿼리 기능을 사용하여 최적화된 데이터 읽기가 가능하다.
- **커서 제어**: JPA Criteria API를 사용하여 데이터 순회를 제어할 수 있다.

### 1-2. JpaPagingItemReader 주요 구성 요소

- **EntityManagerFactory**: JPA 엔티티 매니저 팩토리를 설정한다.
- **JpaQueryProvider**: 데이터를 읽을 JPA 쿼리를 제공한다.
- **PageSize**: 페이지 크기를 설정한다.
- **SkippableItemReader**: 오류 발생 시 해당 Item을 건너뛸 수 있도록 한다.
- **ReadListener**: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다.
- **SaveStateCallback**: 잡시 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 한다.

### 1-3. JpaItemWriter 개요

- Spring Batch에서 제공하는 `ItemWriter` 인터페이스를 구현하는 클래스이다.
- 데이터를 **JPA**를 통해 데이터베이스에 저장하는 데 사용된다.

### 1-4. JpaItemWriter 주요 구성 요소

- **EntityManagerFactory**: JPA EntityManager 생성을 위한 팩토리 객체
- **JpaQueryProvider**: 저장할 엔터티를 위한 JPA 쿼리를 생성하는 역할
- 장점
  - **ORM 연동**: JPA를 통해 다양한 데이터베이스에 데이터를 저장할 수 있다.
  - **객체 매핑**: 엔터티 객체를 직접 저장하여 코드 간결성을 높일 수 있다.
  - **유연성**: 다양한 설정을 통해 원하는 방식으로 데이터를 저장할 수 있다.
- 단점
  - **설정 복잡성**: JPA 설정 및 쿼리 작성이 복잡할 수 있다.
  - **데이터베이스 종속**: 특정 데이터베이스에 종속적이다.
  - **오류 가능성**: 설정 오류 시 데이터 손상 가능성이 있다.-

<br>

---

## 2. JpaPagingItemReader 구현

`JpaPagingItemReader`를 활용하여 db의 **customer** 테이블로 부터 데이터를 읽어들이고 **flatfile**(csv)로 저장하는 로직을 구현해보자.

![img.png](img.png)

현재 **customer** 테이블에 등록된 데이터 정보이다. 이 age가 20보다 큰 row를 찾아와 csv에 저장해보자.

```java

@Slf4j
@Configuration
public class JpaPagingReaderJobConfig {

  /**
   * CHUNK 크기를 지정한다.
   */
  public static final int CHUNK_SIZE = 2;
  public static final String ENCODING = "UTF-8";
  public static final String JPA_PAGING_CHUNK_JOB = "JPA_PAGING_CHUNK_JOB";

  @Autowired
  EntityManagerFactory entityManagerFactory;

  @Bean
  public JpaPagingItemReader<Customer> customerJpaPagingItemReader() {

    return new JpaPagingItemReaderBuilder<Customer>()
      .name("customerJpaPagingItemReader")
      .queryString("SELECT c FROM Customer c WHERE c.age > :age order by id desc")
      .pageSize(CHUNK_SIZE)
      .entityManagerFactory(entityManagerFactory)
      .parameterValues(Collections.singletonMap("age", 20))
      .build();
  }

  @Bean
  public FlatFileItemWriter<Customer> customerJpaFlatFileItemWriter() {

    return new FlatFileItemWriterBuilder<Customer>()
      .name("customerJpaFlatFileItemWriter")
      .resource(new FileSystemResource("./output/customer_new_v2.csv"))
      .encoding(ENCODING)
      .delimited().delimiter("\t")
      .names("Name", "Age", "Gender")
      .build();
  }

  @Bean
  public Step customerJpaPagingStep(JobRepository jobRepository, PlatformTransactionManager transactionManager) throws
    Exception {
    log.info("------------------ Init customerJpaPagingStep -----------------");

    return new StepBuilder("customerJpaPagingStep", jobRepository)
      .<Customer, Customer>chunk(CHUNK_SIZE, transactionManager)
      .reader(customerJpaPagingItemReader())
      .processor(new CustomerItemProcessor())
      .writer(customerJpaFlatFileItemWriter())
      .build();
  }

  @Bean
  public Job customerJpaPagingJob(Step customerJpaPagingStep, JobRepository jobRepository) {
    log.info("------------------ Init customerJpaPagingJob -----------------");
    return new JobBuilder(JPA_PAGING_CHUNK_JOB, jobRepository)
      .incrementer(new RunIdIncrementer())
      .start(customerJpaPagingStep)
      .build();
  }
}
```

- `JpaPagingItemReaderBuilder`를 이용하여 `JpaPagingItemReader` 빈을 생성한다.
  - **queryString**: 데이터 조회 쿼리문
  - **pageSize**: CHUNK 크기
  - **entityManagerFactory**: JPA 연동을 위한 EntityManagerFactory
  - **parameterValues**: 쿼리문 변수에 입력할 값
- `CustomerItemProcessor`에서는 `Customer` 객체를 그대로 반환한다.
- Step 정의시에 **Reader**, **Processor**, **Writer** 를 지정했다.

### 2-1. 실행 결과

![img_1.png](img_1.png)

정상적으로 csv 파일에 저장된 데이터를 확인할 수 있다.

<br>

---

## 3. JpaItemWriter 구현

`JpaItemWriter`를 활용하여 **flatfile**에서 데이터를 읽어들인 후 **customer** 테이블에 저장하는 로직을 구현해보자.

![img_2.png](img_2.png)

일단 customer.csv 파일에 등록할 데이터를 추가해주었다.

```java

@Slf4j
@Configuration
public class JpaItemJobConfig {

  /**
   * CHUNK 크기를 지정한다.
   */
  public static final int CHUNK_SIZE = 100;
  public static final String ENCODING = "UTF-8";
  public static final String JPA_ITEM_WRITER_JOB = "JPA_ITEM_WRITER_JOB";

  @Autowired
  EntityManagerFactory entityManagerFactory;

  @Bean
  public FlatFileItemReader<Customer> flatFileItemReader() {

    return new FlatFileItemReaderBuilder<Customer>()
      .name("FlatFileItemReader")
      .resource(new ClassPathResource("./customer.csv"))
      .encoding(ENCODING)
      .delimited().delimiter(",")
      .names("name", "age", "gender")
      .targetType(Customer.class)
      .build();
  }

  @Bean
  public JpaItemWriter<Customer> jpaItemWriter() {
    return new JpaItemWriterBuilder<Customer>()
      .entityManagerFactory(entityManagerFactory)
      .usePersist(true)
      .build();
  }

  @Bean
  public Step flatFileStep(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
    log.info("------------------ Init flatFileStep -----------------");

    return new StepBuilder("flatFileStep", jobRepository)
      .<Customer, Customer>chunk(CHUNK_SIZE, transactionManager)
      .reader(flatFileItemReader())
      .writer(jpaItemWriter())
      .build();
  }

  @Bean
  public Job flatFileJob(Step flatFileStep, JobRepository jobRepository) {
    log.info("------------------ Init flatFileJob -----------------");
    return new JobBuilder(JPA_ITEM_WRITER_JOB, jobRepository)
      .incrementer(new RunIdIncrementer())
      .start(flatFileStep)
      .build();
  }
}
```

- `FlatFileItemReader`, `JpaItemWriter` 빈을 생성하고 Step과 Job을 등록해준다.
- `JpaItemWriter는` JPA를 통해 데이터를 ORM 방식으로 저장하기 때문에 별도의 insert 쿼리문을 작성하지 않아도 된다.
- `JpaItemWriterBuilder` 의 usePersist를 true로 설정하면 자동으로 데이터를 저장한다.

### 3-1. 실행 결과

![img_3.png](img_3.png)

정상적으로 **customer** 테이블에 데이터가 insert 된 것을 확인할 수 있다.
