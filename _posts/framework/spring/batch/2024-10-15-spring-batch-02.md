---
title: Spring Batch 시작하기 (with DEVOCEAN)
date: 2024-10-15 00:00:00 +0900
categories: [ Framework, Spring-Batch ]
tags: [ 스프링 배치, Spring-Batch, 스프링 ]
image:
  path: /assets/img/logo/spring_logo.png
  content: false
---

# Spring Batch Study 2

*참고: [DEVOCEAN KIDO님 SpringBatch 연재 02](https://devocean.sk.com/blog/techBoardDetail.do?ID=166690)*

---

## 1. Spring Batch Architecture

스프링배치를 구현하기에 앞서 스프링배치가 어떤 아키텍처로 구성되어 있고, 각 구성 요소들은 어떤 역할을 수행하는지 알아보자.

![Spring Batch 아키텍처](https://github.com/schooldevops/spring-batch-tutorials/blob/03.SpringBatchArchitecture/imgs/spring-batch-architecture.png?raw=true)

### Job

- Spring Batch에서 일괄 적용을 위한 일련의 프로세스를 요약하는 **단일 실행 단위**가 된다.
- 여러 개의 Step을 정의하고, 각 Step 간의 실행 순서를 결정한다.

### JobLauncher

- Job을 수행하기 위한 인터페이스이다.
- Spring Batch에서 배치 작업을 시작하는 역할을 담당한다.
- 자바 커맨드를 통해서 `CommandLineJobRunner`를 실행하여 단순하게 배치 프로세스가 수행될 수 있다.
- **Job**과 **JobParameters**를 인자로 받아 실행한다.

### Step

- Job을 구성하는 **처리 단위**이다.
- 하나의 Step은 하나의 처리 흐름을 의미하며, 일반적으로 **읽기**, **처리**, **쓰기** 단계를 포함한다.
- 하나의 Job에서 여러 Step이 재사용, 병렬화, 조건분기 등을 수행할 수 있다.
- `ItemReader`, `ItemProcessor`, `ItemWriter`와 같은 배치 처리를 담당하는 컴포넌트로 구성된다.

### JobRepository

- Job과 Step의 상태를 관리하는 시스템이다.
- 배치 메타데이터를 저장하고 관리하는 역할을 담당한다.
- 배치 작업의 **실행 이력**, **상태**, **성공/실패 여부**를 저장한다.
- 배치의 **재시작** 기능을 지원하기 위해 필요한 정보를 제공한다.

### ItemReader

- **청크단위** 모델에서 사용하며, 소스 데이터를 읽어 들이는 역할을 수행한다.
- 파일, 데이터베이스, API 등 다양한 데이터 소스에서 데이터를 읽어 배치 처리의 입력으로 제공한다.

### ItemProcessor

- 읽어들인 청크 데이터를 **처리**한다.
- 읽어온 데이터를 **가공**하는 역할을 수행한다.
- **데이터 변환, 유효성 검증, 필터링** 등을 수행할 수 있다.

### ItemWriter

- 청크 데이터를 읽어들였거나, 처리된 데이터를 실제 **쓰기작업**을 담당한다.
- 처리된 데이터를 최종적으로 **저장**하는 역할을 수행한다.
- **데이터베이스, 파일, 메시지 큐** 등 다양한 데이터 저장소로 데이터를 쓸 수 있다.

## 2. Spring Batch Process

이번에는 앞서 스프링 배치의 기본 아키텍처를 바탕으로 배치 프로세스가 진행되는 흐름을 살펴보자.

![Spring Batch 프로세스](https://github.com/schooldevops/spring-batch-tutorials/blob/03.SpringBatchArchitecture/imgs/springbatch-flow.png?raw=true)

### 처리 흐름

1. `JobScheduler`가 배치를 트리거링 하면 `JobLauncher`를 실행한다.
2. `JobLauncher`가 `Job`을 실행한다.
3. `Job`이 실행되면 `JobExecution`이 실행되어 배치 작업의 **실행 상태, 시작 시간, 종료 시간 등**을 기록한다.
4. `Job`은 자신에게 정의된 `Step`을 실행한다. Step의 **상태**는 `StepExecution`이 실행되어 기록한다.
5. 각 `Step`은 `Chunk` 모델(Chunk-Oriented Processing)로 구성되며, **ItemReader/ItemProcessor/ItemWriter**로 Chunk 단위의 데이터를 처리한다.
6. `ItemReader`가 데이터를 **읽고**, `ItemProcessor`가 **처리 및 가공**하며, `ItemWriter`가 데이터를 **기록**한다.
7. `Step`이 완료되면 상태가 `StepExecution`에 실행이력과 상태가 기록된다.
8. `Job`의 모든 `Step`이 완료되면 최종적으로 `JobExecution`의 상태가 `JobRepository`를 통해 DB에 저장된다.

### Job 정보의 흐름

- JobLauncher는 JobRepository를 통해서 **JobInstance정보**를 데이터베이스에 등록한다.
- JobExecution을 통해서 **Job실행 정보**를 데이터베이스에 저장한다.
- Step은 JobRepository를 통해서 **I/O 레코드와 상태정보**를 저장한다.
- Job이 완료되면 JobRepository를 통해서 데이터베이스에 **완료 정보**를 저장한다.

### 스프링 배치 저장 정보

- **JobInstance**
    - JobInstance는 Job **이름과 전달 파라미터**를 정의한다.
    - Job이 중단되는 경우 다음 실행할때 중단 이후부터 실행하도록 지원한다.
- **JobExecution**
    - Job의 물리적인 실행을 나타낸다.
    - JobInstance와 달리 동일한 Job이 여러번 수행될 수 있다.(1:N)
    - Job의 실행 이력 및 상태 정보(성공/실패, 시작 시간, 종료 시간 등)를 기록한다.
    - 특정 Step에서 실패했다면, 실패한 Step 이후부터 다시 실행할 수 있도록 Job의 상태를 기록한다.
- **Job-ExecutionContext**
    - 각각의 JobExecution에서 처리 단계와 같은 메타 정보들을 공유하는 영역이다.
    - Job내의 모든 Step에서 공유된다. 즉, Step 간에 데이터를 공유해야 할 경우, Job-ExecutionContext를 사용하여 데이터를 주고받을 수 있다.
    - 스프링 배치가 **프레임워크 상태**를 기록하는데 사용하며, 또한 애플리케이션에서 ExecutionContext에 액세스 하는 수단도 제공된다.
    - Job이 중단되거나 실패했을 때, 재시작 시 필요한 상태를 저장한다.
    - 데이터는 바이트 스트림 형태로 저장되기 때문에 반드시 java.io.Serializable를 구현해야한다.
- **StepExecution**
    - Step의 물리적인 실행을 나타낸다.
    - Job은 여러 Step을 수행하므로 1:N 관계가 된다.
    - 실행 이력 및 상태 정보(Chunk별 처리된 레코드 수, 성공/실패 상태 등)를 기록한다.
    - Step이 중단되었을 때 다음에 실행할 Step 등을 결정한다.
- **Step-ExecutionContext**
    - Step내에서만 사용하는 데이터를 공유해야하는 영역이다.
    - Step의 실행 중에 발생하는 상태 정보를 저장하고 재시작 시 필요한 상태를 저장한다.
    - 데이터는 바이트 스트림 형태로 저장되기 때문에 반드시 java.io.Serializable를 구현해야한다.
- **JobRepository**
    - 배치 실행정보나 상태, 결과정보들이 데이터베이스에 저장될 필요가 있으며 이를 처리하는 것이 JobRepository이다.
    - 저장된 정보를 활용하여 스프링배치는 Job을 재실행 하거나, 정지된 상태 후부터 수행할 수 있는 수단을 제공한다.

| **항목**        | **Execution**                          | **ExecutionContext**                             |
|---------------|----------------------------------------|--------------------------------------------------|
| **역할**        | Job 또는 Step의 실행 상태를 추적하고 기록            | 배치 실행 중의 **중간 데이터**나 **상태 정보**를 저장               |
| **저장 정보**     | 실행 시간, 처리된 데이터 수, 성공/실패 상태, 트랜잭션 상태 등  | 중간에 처리된 데이터, 재시작 시 필요한 상태 정보, Step 간 공유 데이터      |
| **주요 사용 시점**  | Job 또는 Step이 실행될 때마다 자동으로 생성 및 업데이트    | Step 실행 중 중간 상태를 기록하거나 재시작할 때 사용                 |
| **재시작 기능 지원** | Job 또는 Step이 중단되었을 때 재시작 지점에서 상태 복원 지원 | 세부적인 중간 상태 데이터를 저장하여, 보다 정교하게 재시작을 지원            |
| **데이터 구조**    | 내부적으로 관리되는 **상태 정보 객체**                | Key-Value 방식의 **저장소**, 직렬화 가능한 데이터 저장            |
| **직렬화 필요 여부** | 직렬화 필요 없음 (Spring Batch가 관리)           | **직렬화(`Serializable`)**가 필요, DB에 저장하기 위해 바이트로 변환 |

## 3. Spring Batch Tasklet 방식 구현 및 실행

**Tasklet? Chunk?**

스프링 배치를 Tasklet 방식으로 실행하기에 앞서, Chunk 방식과는 어떤 차이가 있는지 간단히 아래 표에 정리하였다.

| **항목**      | **Tasklet 방식**             | **Chunk 방식**                              |
|-------------|----------------------------|-------------------------------------------|
| **처리 방식**   | 단일 작업(단일 태스크) 처리           | 대량 데이터를 일정 크기(Chunk)로 나누어 처리              |
| **반복 처리**   | 반복적이지 않음, 한 번에 한 작업만 처리    | Chunk 단위로 데이터를 반복적으로 처리                   |
| **트랜잭션 관리** | 트랜잭션 수동 관리                 | Chunk 단위로 자동으로 트랜잭션 커밋 및 롤백 관리            |
| **주요 사용**   | 단일 작업(파일 삭제, 데이터 초기화 등)    | 대용량 데이터 처리 시 사용 (파일 처리, DB 연동 등)          |
| **구성 요소**   | Tasklet 인터페이스              | ItemReader, ItemProcessor, ItemWriter로 구성 |
| **유연성**     | 단순 작업 처리에 적합, 복잡한 처리에는 부적합 | 대용량 데이터 처리에 적합, 유연한 트랜잭션 관리               |

**Tasklet 구현체 생성**

구현체가 어느시점에 실행되는지는 아직 모르나 일단 KIDO님의 구현 순서를 따라서 작성하였다.

```java

@Slf4j
public class GreetingTask implements Tasklet, InitializingBean {
	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) {
		log.info("------------------ Task Execute -----------------");
		log.info("GreetingTask: {}, {}", contribution, chunkContext);

		return RepeatStatus.FINISHED;
	}

	@Override
	public void afterPropertiesSet() {
		log.info("----------------- After Properites Sets() --------------");
	}
}
```

**Job, Step 빈 구성**

```java

@Slf4j
@Configuration
public class BasicTaskJobConfiguration {
	@Autowired
	PlatformTransactionManager transactionManager;

	@Bean
	public Tasklet greetingTasklet() {
		return new GreetingTask();
	}

	@Bean
	public Step step(JobRepository jobRepository) {
		log.info("------------------ Init myStep -----------------");

		return new StepBuilder("myStep", jobRepository)
			.tasklet(greetingTasklet(), transactionManager)
			.build();
	}

	@Bean
	public Job myJob(JobRepository jobRepository) {
		log.info("------------------ Init myJob -----------------");
		return new JobBuilder("myJob", jobRepository)
			.incrementer(new RunIdIncrementer())
			.start(step(jobRepository))
			.build();
	}
}
```

- 빈은 Tasklet, Step, Job 순으로 등록되는 것을 확인할 수 있다.
- Job Bean에서는 진행할 step과 jobRepository를 주입받아 JobBuilder를 통해 Job을 생성한다.
- Step Bean에서는 실행할 tasklet과 jobRepository를 주입받아 StepBuilder를 통해 Step을 생성한다.

### 스프링 배치 어플리케이션 실행

![img.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img.png)

로그를 자세히 확인해보면 Job -> Step -> Tasklet 순으로 로그가 노출되는 것을 확인할 수 있다.

![img_2.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img_2.png)

Job에 대한 파라미터 정보, 실행 Step 정보, Tasklet 실행 정보 등을 확인할 수 있다.
이 정보들 중에서 execute 메서드 실행시 들어오는 파라미터 정보들은 다음과 같았다.

**contribution**

```json
StepContribution: read=0, written=0, filtered=0, readSkips=0, writeSkips=0, processSkips=0, exitStatus=EXECUTING
```

**chunkContext**

```json
ChunkContext: attributes=[], complete=false, stepContext=SynchronizedAttributeAccessor: [], stepExecutionContext={batch.version=5.1.0, batch.taskletType=com.schooldevops.springbatch.batchstudy.jobs.task01.GreetingTask, batch.stepType=org.springframework.batch.core.step.tasklet.TaskletStep}, jobExecutionContext={batch.version=5.1.0}, jobParameters={run.id=2}
```

StepContribution에는 read, write count 같은 stepExecution 정보가 포함되어 있는 것을 확인할 수 있고,  
ChunkContext에는 기본적인 Job,Step ExcutionContext 정보가 포함되어있는 것을 확인할 수 있다.

### 스프링 배치 DB 메타데이터

스프링 배치가 실행되면서 기록된 MySQL DB 스프링배치 메타데이터도 확인할 수 있다.

**BATCH_JOB_INSTANCE**

![img_3.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img_3.png)

**BATCH_JOB_EXECUTION**

![img_4.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img_4.png)

**BATCH_JOB_EXECUTION_CONTEXT**

![img_5.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img_5.png)

**BATCH_JOB_EXECUTION_PARAMS**

![img_6.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img_6.png)

**BATCH_STEP_EXECUTION**

![img_7.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img_7.png)

**BATCH_STEP_EXECUTION_CONTEXT**

![img_8.png](https://github.com/youngkim90/spring-batch-study/raw/main/study/2_week/img_8.png)
