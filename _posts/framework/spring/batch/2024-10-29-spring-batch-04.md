---
title: Spring Batch - FlatFileItemReader/FlatFileItemWriter 구현
date: 2024-10-29 00:00:00 +0900
categories: [ Framework, Spring-Batch ]
tags: [ 스프링 배치, Spring-Batch, FlatFileItemReader, FlatFileItemWriter ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

# Spring Batch Study 4

*참고: [DEVOCEAN KIDO님 SpringBatch 연재 04](https://devocean.sk.com/blog/techBoardDetail.do?ID=166828)*

---

Spring Batch의 Chunk Model에 사용되는 `FlatFileItemReader`와 `FlatFileItemWriter`에 대해 알아보고 구현해보자.

## 1. FlatFileItemReader/FlatFileItemWriter 개요

### FlatFileItemReader 개요

- Spring Batch에서 제공하는 **기본 ItemReader**이다.,
- ItemReader 인터페이스를 **구현**하며 텍스트 파일로부터 **데이터를 읽는다.**
- 고정 길이, 구분자 기반, 멀티라인 등 **다양한 형식의 텍스트 파일을 지원**한다.
- 설정 및 사용이 간편하며, **대규모 데이터 처리**에 효율적이다.
- **Tokenizer**, **Filter** 등을 통해 기능을 확장할 수 있다.

### FlatFileItemReader 장단점

- **장점**
  - 간단하고 효율적인 구현이 가능하다.
  - 대용량 데이터 처리에 효율적이다.
  - 다양한 형식의 텍스트 파일을 지원한다.
- **단점**
  - 복잡한 데이터 구조 처리에는 적합하지 않다.

### FlatFileItemWriter 개요

- Spring Batch에서 제공하는 **기본 ItemWriter**이다.
- ItemWriter 인터페이스를 **구현**하며 텍스트 파일로 **데이터를 출력한다.**
- CSV, 고정 길이 파일 등 다양한 텍스트 파일 형식으로 데이터를 출력할 수 있다.

### FlatFileItemWriter 장단점

- **장점**
  - **간편성**: 텍스트 파일로 데이터를 출력하는 간편한 방법을 제공한다.
  - **유연성**: 다양한 설정을 통해 원하는 형식으로 출력 파일을 만들 수 있다.
  - **성능**: 대량의 데이터를 빠르게 출력할 수 있다.
- **단점**
  - **형식 제약**: 텍스트 파일 형식만 지원한다.
  - **복잡한 구조**: 복잡한 구조의 데이터를 출력할 경우 설정이 복잡해질 수 있다.
  - **오류 가능성**: 설정 오류 시 출력 파일이 손상될 수 있다.

<br>

---

## 2. FlatFileItemReader 구현

읽어들인 정보를 매핑할 객체를 정의한다.

```java

@Setter
@Getter
public class Customer {

  private String name;
  private int age;
  private String gender;
}
```

`FlatFileItemReader` **빈**을 정의하고, 매핑할 객체(Customer)를 설정한다.

```java

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
```

- `name`: `FlatFileItemReader` 빈의 이름을 지정한다.
- `resource`: 읽어들일 파일의 위치를 지정한다.
- `encoding`: 파일의 인코딩을 지정한다.
- `delimited().delimiter()`: 구분자를 지정한다.
- `names()`: 구분자로 구분된 데이터의 이름을 지정한다.
- `targetType()`: 매핑할 객체의 타입을 지정한다.

<br>

---

## 3. FlatFileItemWriter 구현

`FlatFileItemWriter` **빈**을 정의한다.

```java

@Bean
public FlatFileItemWriter<Customer> flatFileItemWriter() {

  return new FlatFileItemWriterBuilder<Customer>()
    .name("flatFileItemWriter")
    .resource(new FileSystemResource("./output/customer_new.csv"))
    .encoding(ENCODING)
    .delimited().delimiter("\t")
    .names("Name", "Age", "Gender")
    .append(false)
    .lineAggregator(new CustomerLineAggregator())
    .headerCallback(new CustomerHeader())
    .footerCallback(new CustomerFooter(aggregateInfos))
    .build();
}
```

- `name`: `FlatFileItemWriter` 빈의 이름을 지정한다.
- `resource`: 저장할 파일의 위치를 지정한다.
- `encoding`: 저장할 파일의 인코딩을 타입을 지정한다.
- `delimited().delimiter()`: 구분자를 지정한다.
- `names()`: 구분자로 구분된 데이터의 이름을 지정한다.
- `append()`: 기존 파일에 추가할지 여부를 지정한다.
- `lineAggregator()`: 라인 구분자를 지정한다.
- `headerCallback()`: 파일의 헤더를 지정한다.
- `footerCallback()`: 파일의 푸터를 지정한다.

```java
public class CustomerLineAggregator implements LineAggregator<Customer> {
  @Override
  public String aggregate(Customer item) {
    return item.getName() + "," + item.getAge();
  }
}
```

- `LineAggregator`은 **FlatFile**에 라인 별로 저장할 아이템 정보를 문자열로 변환한다.
- `aggregate()`를 구현하여 아이템(데이터)을 문자열 라인으로 변환한다.

```java
public class CustomerHeader implements FlatFileHeaderCallback {
  @Override
  public void writeHeader(Writer writer) throws IOException {
    writer.write("ID,AGE");
  }
}

```

- `FlatFileHeaderCallback`은 출력될 파일의 헤더를 달아주는 역할을 한다.

```java

@Slf4j
public class CustomerFooter implements FlatFileFooterCallback {
  ConcurrentHashMap<String, Integer> aggregateCustomers;

  public CustomerFooter(ConcurrentHashMap<String, Integer> aggregateCustomers) {
    this.aggregateCustomers = aggregateCustomers;
  }

  @Override
  public void writeFooter(Writer writer) throws IOException {
    writer.write("총 고객 수: " + aggregateCustomers.get("TOTAL_CUSTOMERS"));
    writer.write(System.lineSeparator());
    writer.write("총 나이: " + aggregateCustomers.get("TOTAL_AGES"));
  }
}
```

- `FlatFileFooterCallback`은 출력될 파일의 푸터를 달아주는 역할을 한다.

```java

@Slf4j
public class AggregateCustomerProcessor implements ItemProcessor<Customer, Customer> {

  ConcurrentHashMap<String, Integer> aggregateCustomers;

  public AggregateCustomerProcessor(ConcurrentHashMap<String, Integer> aggregateCustomers) {
    this.aggregateCustomers = aggregateCustomers;
  }

  @Override
  public Customer process(Customer item) throws Exception {
    aggregateCustomers.putIfAbsent("TOTAL_CUSTOMERS", 0);
    aggregateCustomers.putIfAbsent("TOTAL_AGES", 0);

    aggregateCustomers.put("TOTAL_CUSTOMERS", aggregateCustomers.get("TOTAL_CUSTOMERS") + 1);
    aggregateCustomers.put("TOTAL_AGES", aggregateCustomers.get("TOTAL_AGES") + item.getAge());
    return item;
  }
}
```

- `CustomerFooter`에서 사용할 집계 정보를 가공한다.
- `ItemProcessor`를 구현하고 **process()** 를 통해 아이템을 하나씩 읽고 아이템의 내용을 집계한다.

<br>

---

## 4. FlatFileItemReader/FlatFileItemWriter 전체 Config 설정

```java

@Slf4j
@Configuration
public class FlatFileItemJobConfig {

  /**
   * CHUNK 크기를 지정한다.
   */
  public static final int CHUNK_SIZE = 100;
  public static final String ENCODING = "UTF-8";
  public static final String FLAT_FILE_WRITER_CHUNK_JOB = "FLAT_FILE_WRITER_CHUNK_JOB";

  private ConcurrentHashMap<String, Integer> aggregateInfos = new ConcurrentHashMap<>();

  private final ItemProcessor<Customer, Customer> itemProcessor = new AggregateCustomerProcessor(aggregateInfos);

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
  public FlatFileItemWriter<Customer> flatFileItemWriter() {

    return new FlatFileItemWriterBuilder<Customer>()
      .name("flatFileItemWriter")
      .resource(new FileSystemResource("./output/customer_new.csv"))
      .encoding(ENCODING)
      .delimited().delimiter("\t")
      .names("Name", "Age", "Gender")
      .append(false)
      .lineAggregator(new CustomerLineAggregator())
      .headerCallback(new CustomerHeader())
      .footerCallback(new CustomerFooter(aggregateInfos))
      .build();
  }

  @Bean
  public Step flatFileStep(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
    log.info("------------------ Init flatFileStep -----------------");

    return new StepBuilder("flatFileStep", jobRepository)
      .<Customer, Customer>chunk(CHUNK_SIZE, transactionManager)
      .reader(flatFileItemReader())
      .processor(itemProcessor)
      .writer(flatFileItemWriter())
      .build();
  }

  @Bean
  public Job flatFileJob(Step flatFileStep, JobRepository jobRepository) {
    log.info("------------------ Init flatFileJob -----------------");
    return new JobBuilder(FLAT_FILE_WRITER_CHUNK_JOB, jobRepository)
      .incrementer(new RunIdIncrementer())
      .start(flatFileStep)
      .build();
  }
}
```

- `flatFileItemReader` : 텍스트 파일로 부터 데이터를 읽어들인다.
- `flatFileItemWriter` : 객체로 매핑된 데이터를 텍스트 파일로 출력한다.
- `flatFileStep` : `flatFileItemReader`, `itemProcessor`, `flatFileItemWriter`를 사용하여 **Step**을 생성한다.
- `flatFileJob` : `flatFileStep`을 사용하여 **Job**을 생성한다.
