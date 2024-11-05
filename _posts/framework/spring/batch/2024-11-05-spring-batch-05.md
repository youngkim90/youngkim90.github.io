---
title: Spring Batch - JdbcPagingItemReader/JdbcBatchItemWriter 구현
date: 2024-11-05 00:00:00 +0900
categories: [ Framework, Spring-Batch ]
tags: [ 스프링 배치, Spring-Batch, JdbcPagingItemReader, JdbcBatchItemWriter ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

# Spring Batch Study 5

*참고: [DEVOCEAN KIDO님 SpringBatch 연재 05](https://devocean.sk.com/blog/techBoardDetail.do?ID=166867)*

---

Spring Batch의 `JdbcPagingItemReader`와 `JdbcBatchItemWriter`로 DB 데이터를 읽고 쓰는 방법을 알아보자.

## 1. JdbcPagingItemReader/JdbcBatchItemWriter 개요

### JdbcPagingItemReader 개요

- Spring Batch에서 제공하는 `ItemReader` 인터페이스를 구현하는 클래스이다.
- **페이지 단위 데이터 읽기**: 데이터베이스에서 **페이지 단위**로 데이터를 읽어오는 기능을 제공한다.
- **대규모 데이터 처리 효율성**: 지정된 크기 만큼만 읽어와 메모리 사용량을 최소화하고, Chunk 단위로 커밋 간격을 설정하여 대규모 데이터를 **효율적**으로 처리할 수 있다.
- **페이지네이션 방식과 성능 최적화**: 각 페이지를 독립적으로 조회하는 페이지네이션 방식을 사용하여 커서 유지 부담을 줄이고 대용량 데이터를 **안정적**으로 처리할 수 있다.
- **SQL 쿼리 정의**: SQL 쿼리를 **직접 작성**하여 최적화된 데이터 읽기가 가능하다.
- **커서 제어**: 데이터베이스 **커서를 사용**하여 데이터 순회를 제어할 수 있다.

### JdbcPagingItemReader 주요 구성 요소

- **DataSource**: 데이터베이스 **연결 정보**를 설정한다.
- **SqlQuery**: **SelectClause**, **FromClause**, **WhereClause**, **SortKeys** 구성요소를 통해 데이터를 읽을 SQL 쿼리를 설정한다.
- **RowMapper**: SQL 쿼리의 각 결과 행을 **Item으로 변환**하는 역할을 한다.
- **PageSize**: Chunk와는 별도로 **페이지 크기**를 설정한다.
- **SaveState**: 기본적으로 **상태 저장**을 지원하며, 배치 잡이 중단되었을 때 마지막으로 읽은 위치를 저장하여 재시작 시 이어서 처리할 수 있도록 한다.
- **Exception Handling**: 읽기 과정에서 발생하는 예외를 처리하기 위해 다양한 리스너(**SkipListener**, **ReadListener**)를 지원한다.

### JdbcBatchItemWriter 개요

- Spring Batch에서 제공하는 `ItemWriter` 인터페이스를 구현하는 클래스이다.
- **JDBC를 통한 대량 데이터 저장**: JDBC를 통해 데이터를 저장하며, 대량 데이터 저장에 최적화되어 있다.
- **SQL 쿼리 정의**: SQL 쿼리를 **직접 작성**하여 원하는 방식으로 데이터베이스에 저장할 수 있다.
- **대용량 데이터 처리에 적합**: 데이터를 저장할 때 Chunk 기반으로 데이터를 처리하며, 커밋 간격에 따라 데이터베이스에 저장하여 안정적이고 효율적으로 데이터를 처리 힌다.

### JdbcBatchItemWriter 주요 구성 요소

- **DataSource**: 데이터베이스 **연결 정보**를 설정한다.
- **SqlStatementCreator**: INSERT, UPDATE, 또는 MERGE와 같은 **쿼리를 생성**하는 역할을 한다.
- **PreparedStatementSetter**: INSERT 쿼리의 **파라미터 값을 설정**하는 역할을 한다.
- **ItemSqlParameterSourceProvider**: Item 객체의 필드를 PreparedStatementSetter에 전달할 **쿼리 파라미터 값을 생성**하는 역할을 한다.

<br>

---

## 2. JdbcPagingItemReader 구현

`JdbcPagingItemReader`를 활용하여 db의 **customer** 테이블로 부터 데이터를 읽어들이고 **flatfile**(csv)로 저장하는 로직을 구현해보자.

```java

@Slf4j
@Configuration
public class JdbcPagingReaderJobConfig {

  /**
   * CHUNK 크기를 지정한다.
   */
  public static final int CHUNK_SIZE = 2;
  public static final String ENCODING = "EUC-KR";
  public static final String JDBC_PAGING_CHUNK_JOB = "JDBC_PAGING_CHUNK_JOB";

  @Autowired
  DataSource dataSource;

  @Bean
  public JdbcPagingItemReader<Customer> jdbcPagingItemReader() throws Exception {

    Map<String, Object> parameterValue = new HashMap<>();
    parameterValue.put("age", 20);

    return new JdbcPagingItemReaderBuilder<Customer>()
      .name("jdbcPagingItemReader")
      .fetchSize(CHUNK_SIZE)
      .dataSource(dataSource)
      .rowMapper(new BeanPropertyRowMapper<>(Customer.class))
      .queryProvider(queryProvider())
      .parameterValues(parameterValue)
      .build();
  }

  @Bean
  public PagingQueryProvider queryProvider() throws Exception {
    SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
    queryProvider.setDataSource(dataSource);  // DB 에 맞는 PagingQueryProvider 를 선택하기 위함
    queryProvider.setSelectClause("id, name, age, gender");
    queryProvider.setFromClause("from customer");
    queryProvider.setWhereClause("where age >= :age");

    Map<String, Order> sortKeys = new HashMap<>(1);
    sortKeys.put("id", Order.DESCENDING);

    queryProvider.setSortKeys(sortKeys);

    return queryProvider.getObject();
  }

  @Bean
  public FlatFileItemWriter<Customer> customerFlatFileItemWriter() {
    return new FlatFileItemWriterBuilder<Customer>()
      .name("customerFlatFileItemWriter")
      .resource(new FileSystemResource("./output/customer_new_v1.csv"))
      .encoding(ENCODING)
      .delimited().delimiter("\t")
      .names("Name", "Age", "Gender")
      .build();
  }

  @Bean
  public Step customerJdbcPagingStep(JobRepository jobRepository,
                                     PlatformTransactionManager transactionManager) throws Exception {
    log.info("------------------ Init customerJdbcPagingStep -----------------");

    return new StepBuilder("customerJdbcPagingStep", jobRepository)
      .<Customer, Customer>chunk(CHUNK_SIZE, transactionManager)
      .reader(jdbcPagingItemReader())
      .writer(customerFlatFileItemWriter())
      .build();
  }

  @Bean
  public Job customerJdbcPagingJob(Step customerJdbcPagingStep, JobRepository jobRepository) {
    log.info("------------------ Init customerJdbcPagingJob -----------------");
    return new JobBuilder(JDBC_PAGING_CHUNK_JOB, jobRepository)
      .incrementer(new RunIdIncrementer())
      .start(customerJdbcPagingStep)
      .build();
  }
}
```

- `JdbcPagingItemReader` 빈을 생성하고 **dataSource**, **rowMapper**, **queryProvider** 등을 설정한다.
- `SqlPagingQueryProviderFactoryBean`로 데이터를 조회하기 위한 쿼리 정보를 설정할 수 있다.
  - **SelectClause** : select에서 프로젝션할 필드 이름을 지정한다.
  - **FromClause** : 조회할 테이블 이름을 지정한다.
  - **WhereClause** : 조건절을 지정한다.
  - **SortKeys** : 정렬 기준을 지정한다.
- **parameterValue**에는 queryProvider에서 사용할 변수(**:age**) 값을 설정한다.
- `FlatFileItemWriter` 빈을 등록하여 **output** 디렉토리에 **customer_new_v1.csv** 파일을 생성한다.
- `Job`, `Step` 빈을 생성하고 **chunk**, **reader**, **writer**를 설정한다.

### 실행 결과

![img_1.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/5_week/img_1.png)

일단 customer 테이블을 생성하고 위와 같이 데이터를 추가한 후에 배치를 실행하였다.

![img.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/5_week/img.png)

로그를 통해 **JDBC_PAGING_CHUNK_JOB**이 실행되어 `customerJdbcPagingStep`이 실행되는 것을 확인할 수 있었다.  
읽어들인 데이터가 csv로 잘 저장되었는지 확인해보자.

![img_2.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/5_week/img_2.png)

**customer_new_v1.csv** 파일이 생성되었고, 데이터가 잘 저장된 것을 확인할 수 있다.

<br>

---

## 3. JdbcBatchItemWriter 구현

`JdbcBatchItemWriter`를 활용하여 **flatfile**에서 데이터를 읽어들인 후 **customer2** 테이블에 저장하는 로직을 구현해보자.

![img_3.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/5_week/img_3.png)

읽어들일 **customer.csv** 파일을 생성해준다.

```sql
create table study.customer2
(
  id     int auto_increment primary key,
  name   varchar(100) null,
  age    int          null,
  gender varchar(10)  null
);
```

읽어들인 데이터를 저장할 테이블도 생성한다.

```java

@Slf4j
@Configuration
public class JdbcBatchItemJobConfig {

  /**
   * CHUNK 크기를 지정한다.
   */
  public static final int CHUNK_SIZE = 100;
  public static final String ENCODING = "UTF-8";
  public static final String JDBC_BATCH_WRITER_CHUNK_JOB = "JDBC_BATCH_WRITER_CHUNK_JOB";

  @Autowired
  DataSource dataSource;

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
  public JdbcBatchItemWriter<Customer> flatFileItemWriter() {

    return new JdbcBatchItemWriterBuilder<Customer>()
      .dataSource(dataSource)
      .sql("INSERT INTO customer2 (name, age, gender) VALUES (:name, :age, :gender)")
      .itemSqlParameterSourceProvider(new CustomerItemSqlParameterSourceProvider())
      .build();
  }

  @Bean
  public Step flatFileStep(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
    log.info("------------------ Init flatFileStep -----------------");

    return new StepBuilder("flatFileStep", jobRepository)
      .<Customer, Customer>chunk(CHUNK_SIZE, transactionManager)
      .reader(flatFileItemReader())
      .writer(flatFileItemWriter())
      .build();
  }

  @Bean
  public Job flatFileJob(Step flatFileStep, JobRepository jobRepository) {
    log.info("------------------ Init flatFileJob -----------------");
    return new JobBuilder(JDBC_BATCH_WRITER_CHUNK_JOB, jobRepository)
      .incrementer(new RunIdIncrementer())
      .start(flatFileStep)
      .build();
  }
}

public class CustomerItemSqlParameterSourceProvider implements ItemSqlParameterSourceProvider<Customer> {
  @Override
  public SqlParameterSource createSqlParameterSource(Customer item) {
    return new BeanPropertySqlParameterSource(item);
  }
}
```

- `FlatFileItemReader` 빈을 생성하여 읽어들일 파일 정보를 저장한다.
- `FlatFileItemWriter` 빈을 생성하고 **dataSource**, **sql**, **itemSqlParameterSourceProvider**를 설정한다.
  - **sql** : 데이터를 저장할 쿼리를 작성한다.
  - **itemSqlParameterSourceProvider** : item으로 부터 sql에 전달할 파라미터 정보를 생성한다.
- `Job`, `Step` 빈을 생성하고 **chunk**, **reader**, **writer**를 설정한다.
- `CustomerItemSqlParameterSourceProvider` 클래스를 생성하여 **itemSqlParameterSourceProvider**를 구현한다.

### 실행 결과

![img_4.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/5_week/img_4.png)

로그를 통해 **JDBC_BATCH_WRITER_CHUNK_JOB**이 실행되어 `flatFileStep`이 실행되는 것을 확인할 수 있었다.

```sql
select *
from customer2;
```

customer2 테이블에 데이터가 잘 들어갔는지 조회해보자.

![img_5.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/5_week/img_5.png)

정상적으로 데이터가 저장된 것을 확인할 수 있다.
