---
title: 스프링부트 Quartz 스케줄링 활용하기
date: 2025-01-06 00:00:00 +0900
categories: [ Framework, SpringBoot ]
tags: [ 스프링부트, QUARTZ, 스케줄링 ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

## 1. Quartz란?

`Quartz`는 Java 기반 스케줄링 라이브러리로, 복잡한 작업 **예약** 및 **실행**을 지원한다.  
다음과 같은 특징들을 갖고 있다.

- 작업(Job)과 트리거(Trigger)를 분리한 구조
- Cron 표현식을 사용한 정교한 스케줄링 지원
- 분산 환경에서 클러스터링 지원
- 저장소(예: JDBC, RAM)를 활용한 작업 상태 저장

### Quartz 주요 구성 요소

1. **Scheduler**: 작업 및 트리거 관리
2. **Job**: 수행할 작업 정의
3. **JobDetail**: 작업의 세부정보 포함
4. **Trigger**: 작업 실행 조건 정의(CronTrigger, SimpleTrigger 등)
5. **JobStore**: 작업과 트리거의 저장소(JDBC, RAM 등)

## 2. Spring Quartz

SpringBoot에서는 Quartz를 쉽게 사용할 수 있도록 `spring-boot-starter-quartz`라는 라이브러리를 제공한다.
해당 라이브러리를 사용하면 다음과 같은 장점을 갖는다.

- **Spring Boot Integration**: Quartz와의 통합을 통해 손쉬운 설정 및 관리
- **Bean 관리**: Quartz 작업 및 트리거를 Spring Bean으로 관리
- **자동 설정 지원**:
  - Spring Boot Starter를 사용해 최소 설정으로 시작 가능
  - @Scheduled와 Quartz를 조합해 복잡한 스케줄링도 처리 가능
- **유연한 데이터베이스 관리**:
  - H2, MySQL 등 다양한 데이터베이스와 호환
  - 클러스터링 지원
- **트랜잭션 통합**:
  - Spring의 트랜잭션 관리와 자연스럽게 통합 가능

### Spring Quartz vs. 기본 Quartz

- **Quartz**: 독립적으로 설정이 필요하며 XML 또는 Java 코드로 세부 설정 관리
- **Spring Quartz**: Spring 컨텍스트와 통합되어 간결한 설정 가능

## 3. Spring Quartz 활용 방식

먼저 `spring-boot-starter-quartz` 라이브러리를 의존성을 추가한다.

```xml

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

application.yml 또는 application.properties에 기본 설정 추가한다.

```yaml
spring:
  quartz:
    job-store-type: jdbc # 작업 및 트리거를 DB에 저장
    jdbc:
      initialize-schema: always # 항상 스키마 초기화
```

Quartz에 사용될 Job 클래스를 작성한다. execute 메서드에 Job이 실행될 내용을 작성한다.

```java
import org.quartz.Job;
import org.quartz.JobExecutionContext;

public class SampleJob implements Job {
  @Override
  public void execute(JobExecutionContext context) {
    System.out.println("Quartz Job is executed!");
  }
}
```

Quartz Job에 대한 detail과 trigger 정보를 설정한다. DB에도 해당 정보들이 저장된다.

```java
import org.quartz.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class QuartzConfig {

  @Bean
  public JobDetail sampleJobDetail() {
    return JobBuilder.newJob(SampleJob.class)
      .withIdentity("sampleJob")
      .storeDurably() // DB에 저장
      .build();
  }

  @Bean
  public Trigger sampleJobTrigger() {
    return TriggerBuilder.newTrigger()
      .forJob(sampleJobDetail())
      .withIdentity("sampleTrigger")
      .withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")) // 5초 간격
      .build();
  }
}
```

이후 Spring Boot 애플리케이션을 실행하면 DB에 저장된 Quartz Job이 실행되며, Job에 구현한 execute 메서드가 호출된다.  
물론 DB에 저장된 Job 정보를 사용하지 않고 메모리 기반으로 사용할 수도 있지만 데이터 유지 및 세밀한 작업 관리 등의 이유로 DB를 사용하는 것이 일반적이다.




