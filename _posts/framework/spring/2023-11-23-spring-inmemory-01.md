---
title: 스프링부트 인메모리 DB 테스트하기
date: 2022-12-15 00:00:00 +0900
categories: [ Framework, SpringBoot ]
tags: [ 스프링부트, 인메모리, 데이터베이스, inmemory database, h2 ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

스프링부트 환경에서 **인메모리 데이터베이스**는 **테스트** 목적으로 종종 사용된다.  
데이터베이스 커넥션이 필요하거나 데이터 확인이 필요한 테스트의 경우 로컬DB or 테스트DB를 사용할 수도 있지만,
스프링부트에서 지원하는 인메모리 데이터베이스를 사용하면 직접 DB 커넥션을 하는 것에 비해 다른 이점을 가질 수 있다.

## 인메모리 데이터베이스 특징

- 디스크가 아닌 주 메모리에 모든 데이터를 보유하고 있는 데이터베이스
- 디스크 검색보다 자료 접근이 훨씬 빠른 것이 가장 큰 장점
- ram이 휘발성이라 서버가 종료되면 데이터는 유실된다.

## 스프링부트가 지원하는 인메모리 DB

- HSQLDB
- Apache Derby
- **H2**
  - 임베디드 및 독립형 데이터베이스 모두에 대해 표준 SQL을 지원하는 Java로 작성된 오픈 소스 데이터베이스이다.
  - 매우 빠르며 약 1.5MB의 JAR에 포함
  - [설정 및 적용방법](https://www.baeldung.com/spring-boot-h2-database)

스프링부트 환경에서 위와 같은 인메모리 데이터베이스들을 활용할 수 있는데 그 중에서도 **H2 Database**는 가볍고 빠르며,
인터페이스가 **Java**로 구현되어 있어서 스프링부트에서 가장 많이 사용되는 인메모리DB이다.

## 스프링부트로 H2Database 테스트 시 장단점

### 장점

- 테스트를 위해 로컬이나 개발에 불필요한 작업을 하지 않아도 된다.
- 메모리상에서 DB 작업이 진행되기 때문에 속도가 매우 빠르다.
- 테스트 데이터는 휘발성이기 때문에 안전하다.
- Hibernate가 H2를 지원한다.

### 단점

- H2와 일반 DB(Mysql, Oracle)들의 sql이 완벽히 호환되진 않는다.
- 단위테스트 용으로는 괜찮지만, 통합테스트용으로 사용하기엔 다소 무리가 있다.

## H2 Database 테스트 환경 구축

#### POM.xml 설정

H2 Database 의존성을 추가한다.

``` xml
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
  <version>2.1.214</version>
</dependency>
```

#### .properties 파일 설정

스프링부트 테스트용 설정 파일에 인메모리 DB **접속정보**를 추가한다.  
기본적으로 H2용 query를 사용하기 떄문에 MySql과 같은 DB query를 직접 사용하면 에러가 날 수 있다.  
`spring.datasource.url` 값 뒤에 `MODE=MySQL` 옵션을 추가해주면 MySql query도 호환이 가능해진다.

``` properties
# test.properties
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:~/test;MODE=MySQL;
spring.datasource.username=sa
spring.datasource.password=
```

## 스프링부트 H2 Database 테스트

#### JPA Repository Unit Test

간단히 H2 DB `insert`, `select` 유닛테스트를 통해서 인메모리DB 테스트가 잘 동작하는지 확인해보자.

``` java
@ActiveProfiles("test")
@Sql(scripts = "classpath:db/member.sql") // 테스트 전 실행할 sql파일 경로 지정
@DataJpaTest
public class H2InMemoryRepositoryTest {

  @Autowired private MemberRepository memberRepository;

  @BeforeEach
  void before() {
    // 테스트 전에 이름이 '코난'인 100명의 멤버를 인메모리DB에 저장한다.
    for (int i = 0; i < 100; i++) {
      Member member = Member.builder()
        .name("코난")
        .email("test@test.com")
        .phoneNumber("01012345678")
        .build();

      memberRepository.save(member);
    }
  }

  @Test
  void test() {
    List<Member> allMember = memberRepository.findAllByName("코난");
    // 테스트 전에 저장한 이름이 '코난'인 100명의 멤버가 조회되는지 확인한다.
    assertThat(allMember).hasSize(100);
  }
}
```

위 코드 처럼 JPA를 활용하면 H2 ORM 커넥션이 가능하기 때문에 간단하게 테스트 진행 가능하다.
사전에 등록이 필요한 DB 스키마가 있다면 `@Sql` 어노테이션을 통해 테스트 전에 실행할 **sql** 파일을 지정할 수 있다.  
무론 스프링부트 설정에 따라서 기본적으로 **entity**로 관리되는 클래스들은 테스트 전에 테이블로 자동 등록도 가능하다.
