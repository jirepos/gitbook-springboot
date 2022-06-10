# SpringBoot Batch 


* Step 
기본 구조(읽고, 처리하고, 저장)는 Step이라는 객체에서 정의된다. 하나의 Step은 읽고, 처리하고, 저장하는 구조를 가지고 있는 실질적인 배치를 처리하는 도메인 객체이다.  Step은 한 개 혹은 여러 개를 모아서 Job을 표현한다. 
* Job
한 개 혹은 여러 개의 Step을 이루어 하나의 단위로 만들어 표현한 객체이다. Job 객체는 JobBuilderFactory 클래스에서 생성할 수 있다. 




### Step 
배치 처리를 구성하는 최소 단위의 요소이다.  크게 두 종류의 모델로 분류할 수 있다. 

* Chunk  Model 
  * 읽기(ItemReader), 가공(ItemProcessor) 쓰기(ItemWriter) 로 구성되는 Step 모델 
  * 일기 → 가공 → 쓰기 순으로 Step이 구성되어 반드시 각각의 처리가 실행되어야 한다.  
* Tasklet Model 
  * Chunk 모델과 달리, 특별히 처리의 흐름이 결정되어 있지 않고, 자유롭게 처리를 할수 있다. 


## Getting Started 

### Tasklet 작성 
Tasklet 인터페이스의 excute 메소드를 구현한다. 결과값으로 RepeatStatus을 반환한다. 

```java
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;

import lombok.extern.slf4j.Slf4j;


@Slf4j
public class DemoTasklet implements Tasklet  {

  @Override
  public RepeatStatus execute(StepContribution arg0, ChunkContext arg1) throws Exception {
    log.debug(("DemoTasklet executed"));
    return RepeatStatus.FINISHED;
  }
}
```

**RepeatStatus**
* **FINISHED** 더 이상의 처리를 하지 않는다. 종료 처리. NULL을 반환해도 같은 결과
* **CONTINUE** Tasklet이 계속 호출된다. FINISHED를 반환하지 않는 한 계속 호출되므로 주의해야 한다. 

### Job Configuration 

Batch Job을 설정한 클래스를 만든다. @EnableBatchProcessing 어노테이션을 붙인다. 
```java
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableBatchProcessing
public class DemoBatchConfiguration {
  
}
```

배치 잡을 실행하기 위해서 Step과 Job을 생성해야 하는데 두 개의 객체가 필요하다. 
* Step을 생성하기 위한 StepBuilderFactory 
* JobBuilder를 생성하기 위한 JobBuilderFactory
* JobBuilder에서 Step으로 JobBuilder를 생성하고  
* Jobbuilder가 Job을 생성하기 위해서는 Step이 필요하다. 


이제 JobBuilderFactory와 StepBuilderFactory를 주입 받도록 선언한다. 
```java
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableBatchProcessing
public class DemoBatchConfiguration {
  
  @Autowired
  public JobBuilderFactory jobBuilderFactory;
  @Autowired
  public StepBuilderFactory stepBuilderFactory;
  
}  
```

#### Step 생성
Step을 생성한다. 
```java
   // DemoBatchConfiguration.java
  @Bean 
  public Step demoStep() {
    return stepBuilderFactory.get("demoStep")
               .tasklet(new DemoTasklet())
               .build();
  }//:
```

#### Job 생성
Job을 생성한다. 
```java
// DemoBatchConfiguration.java
  @Bean 
  public Job demoJob() {
    return jobBuilderFactory.get("demoJob")
            .start(demoStep())
            .build();
  }
```
다음은 전체 코드이다. 
```java

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Configuration
@EnableBatchProcessing
public class DemoBatchConfiguration {
  @Autowired
  public JobBuilderFactory jobBuilderFactory;
  @Autowired
  public StepBuilderFactory stepBuilderFactory;
  @Bean 
  public Step demoStep() {
    return stepBuilderFactory.get("demoStep")
               .tasklet(new DemoTasklet())
               .build();
  }//:
  @Bean 
  public Job demoJob() {
    return jobBuilderFactory.get("demoJob")
            .start(demoStep())
            .build();
  }
}///~

```


#### applicaion.yml 설정
스프링 배치잡이 사용하는 테이블을 생성하도록 application.yml에 spring.batch.jdbc.initialize-schema을 설정한다. @EnableBatchProcessing을 설정하면 정의된 Job이 모두 실행된다. 이 동작을 변경하려면 spring.batch.job.enabled를 false로 설정한다.
```yaml
spring:
  batch:
    # spring.batch.jdbc.initialize-schema
    # spring.batch.job.enabled
    job:
      enabled: false
    jdbc:
      initialize-schema: always  # or never 
```      
스프링은 자동으로 다음의 테이블을 생성한다. 

```shell
BATCH_JOB_EXECUTION
BATCH_JOB_EXECUTION_CONTEXT
BATCH_JOB_EXECUTION_PARAMS
BATCH_JOB_EXECUTION_SEQ
BATCH_JOB_INSTANCE
BATCH_JOB_SEQ
BATCH_STEP_EXECUTION
BATCH_STEP_EXECUTION_CONTEXT
BATCH_STEP_EXECUTION_SEQ
```


### Batch Job 실행 
Batch Job을 실행하기 위해서 DemoJobLauncher.java를 생성한다.  그리고 
* @EnableScheduling 어노테이션을 붙인다. 
* @Component 어노테이션을 붙인다. 
* Job을 실행하기 위해 JobLauncher를 주입받기 위해 @Autowired를 붙인다. 
```java

@EnableScheduling
@Component 
@Slf4j 
public class DemoJobLauncher {
  @Autowired
	private JobLauncher jobLauncher;

}  
```
이제 실행할 Job을 주입받을 수 있도록 Job 멤버를 선언한다. 
```java
  @Autowired
  private Job demoJob;
```
마지막으로 주입받은 Job을 실행할 메소드를 @Scheduled 어노테인션을 사용하여 정의한다. 
```java
  // 초 분 시간 일 월 요일 년 
  //@Scheduled(cron="0/5/10/15/20/30/40/50 * * * * * *")
  @Scheduled(fixedDelay = 5 * 1000L)
  public void jobDemo() throws Exception  {
    log.debug("Before jobDemo starts." + LocalDate.now().toString());
    jobLauncher.run(demoJob, 
                   new JobParametersBuilder()
                       .addString("datetime" , LocalDateTime.now().toString())
                       .toJobParameters());
  }
```
다음은 전체 코드이다. 
```java
import org.joda.time.LocalDate;
import org.joda.time.LocalDateTime;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;

@EnableScheduling
@Component 
@Slf4j 
public class DemoJobLauncher {
  @Autowired
	private JobLauncher jobLauncher;
  @Autowired
  private Job demoJob;
  
  // 초 분 시간 일 월 요일 년 
  //@Scheduled(cron="0/5/10/15/20/30/40/50 * * * * * *")
  @Scheduled(fixedDelay = 5 * 1000L)
  public void jobDemo() throws Exception  {
    log.debug("Before jobDemo starts." + LocalDate.now().toString());
    jobLauncher.run(demoJob, 
                   new JobParametersBuilder()
                       .addString("datetime" , LocalDateTime.now().toString())
                       .toJobParameters());
  }
}///~
```


## @EnableBatchProcessing

스프링 배치에서 @EnableBatchProcessing을 사용하면 Autowired에서 사용되는 StepScope로 정의된 빈이 생성된다.

* JobRepository - bean name "jobRepository"
* JobLauncher - bean name "jobLauncher"
* JobRegistry - bean name "jobRegistry"
* PlatformTransactionManager - bean name "transactionManager"
* JobBuilderFactory - bean name "jobBuilders"
* StepBuilderFactory - bean name "stepBuilders"





### 자동 시작 
@EnableBatchProcessing을 설정하면 정의된 Job이 모두 실행된다. 이 동작을 변경하려면 spring.batch.job.enabled를 false로 설정한다.

실행 매개변수로 spring.batch.job.names를 전달한다.
```shell
java -jar app.jar --spring.batch.job.names=demoJob
```
spring.batch.job.enabled를 false로 설정해 놓은 경우 이 값도 true로 전달해야 한다.

```shell
java -jar app.jar --spring.batch.job.enabled=true --spring.batch.job.names=demoJob
```
이때 전달된 파라미터는 모두 BATCH_JOB_EXECUTION_PARAMS에 저장된다.

### Rest 요청으로 실행 

spring.batch.job.enabled를 false로 설정하고 RequestMapping을 정의한 뒤 JobLauncher로 Job을 실행한다.

```java
@Autowired
JobLauncher jobLauncher;

@Autowired
Job job;

@RequestMapping("/run")
public void handle() throws Exception{
    JobParameters jobParameters =
                    new JobParametersBuilder()
                    .addLong("time",System.currentTimeMillis())
                    .toJobParameters();
    jobLauncher.run(job, jobParameters);
}
```


## JobExecution 
JobLaucher.run()으로 Job을 실행하면 JobExecution 객체가 반환된다. 이 객체로 부터 정보를 구할 수 있다. 
```java
 // 초 분 시간 일 월 요일 년 
  //@Scheduled(cron="0/5/10/15/20/30/40/50 * * * * * *")
  @Scheduled(fixedDelay = 5 * 1000L)
  public void jobDemo() throws Exception  {
    log.debug("Before jobDemo starts." + LocalDate.now().toString());
    JobExecution jobExecution =jobLauncher.run(demoJob, 
                   new JobParametersBuilder()
                       .addString("datetime" , LocalDateTime.now().toString())
                       .toJobParameters());
    while (jobExecution.isRunning()) {
        log.info("...");
    }                       
    log.info("Job Execution: " + jobExecution.getStatus());
		log.info("Job getJobConfigurationName: " + jobExecution.getJobConfigurationName());
		log.info("Job getJobId: " + jobExecution.getJobId());
		log.info("Job getExitStatus: " + jobExecution.getExitStatus());
		log.info("Job getJobInstance: " + jobExecution.getJobInstance());
		log.info("Job getStepExecutions: " + jobExecution.getStepExecutions());
		log.info("Job getLastUpdated: " + jobExecution.getLastUpdated());
		log.info("Job getFailureExceptions: " + jobExecution.getFailureExceptions());    
  }

```

run() 메소드의 두번째 파라미터인 JobParametersBuilder는 NULL이면 오류 난다. 다음과 같이 파라미터를 설정하지 않고 넘길 수 있다. 
```java
    JobExecution jobExecution =jobLauncher.run(demoJob, 
                   new JobParametersBuilder()
                       .toJobParameters());
```                       



## JobParameter와 Scope
Spring Batch의 경우 외부 혹은 내부에서 파라미터를 받아 여러 Batch 컴포넌트에서 사용할 수 있게 지원하고 있는데 이 파라미터를 Job Parameter라고 한다. Job Parameter를 사용하기 위해선 항상 Spring Batch 전용 Scope를 선언해야만 한다. @StepScope와 @JobScope가 있다. 

사용법은 아주 간단한다. 
```java
@Value("#{jobParameters[파라미터명]}")
```
jobParameters 외에도 jobExecutionContext, stepExecutionContext 등도 SpEL로 사용할 수 있다. 

* @JobScope에서는 stepExecutionContext는 사용할 수 없고
* jobParameters와 jobExecutionContext만 사용할 수 있다. 
* @JobScope는 Step 선언문에서만 사용이 가능하고, 
* @StepScope는 Step을 구성하는 ItemReader, ItemProcessor, ItemWriter에서 사용 가능







JobLauncher에서 파라미터를 다음과 같이 전달할 수 있다. 현재 Job Parameter의 타입으로 사용할 수 있는 것은 Double, Long, Date, String 이 있다. 

* **주의할 것은 파라미터의 값이 동일하다면 오류가 발생한다.**
* **Spring Batch는 같은 JobParameter로 같은 Job을 두 번 실행하지 않는다.**

```java
    JobExecution jobExecution =jobLauncher.run(demoJob, 
                   new JobParametersBuilder()
                       .addString("idss" , "aaa"+ LocalDateTime.now().toString())
                       .toJobParameters());
```

Tasklet을 생성할 때 @Value를 사용하여 값을 받을 수 있다. 이 메서드에서는 **@StepScope**를 사용해야 한다. 

```java
  @Bean
  @StepScope 
  public Tasklet sampleTasklet(@Value("#{jobParameters[idss]}") String idss) {
    log.debug(">>>> id:" + idss);
    return (contribution, context) -> { 
      log.debug("################# tasklet created.");
      return RepeatStatus.FINISHED;
    };
  }//:
```


Spring Batch가 Spring 컨테이너를 통해 지정된 Step의 실행시점에 해당 컴포넌트를 Spring Bean으로 생성한다. @JobScope는 Job 실행시점에 Bean이 생성 된다. 즉, Bean의 생성 시점을 지정된 Scope가 실행되는 시점으로 지연시킨다. 

* @StepScope는 Step의 실행 시점에 해당 컴포넌트를 Spring Bean으로 생성
* @JobScope는 Job의 실행 시점에 해당 컴포넌트를 Spring Bean으로 생성


> @JobScope와 @StepScope는 스프링의 기본 Scope인 싱글톤 방식과는 대치되는 역할이다. 
Bean의 생성 시점이 스프링 애플리케이션이 실행되는 시점이 아닌 @JobScope, @StepScope가 명시된 메서드가 실행될 때까지 지연시키는 것을 의미한다. 이것을 Late Binding이라 한다. 

> Step 안에 Tasklet이 있고, 이 Tasklet은 멤버 변수와 이 멤버 변수를 변경하는 로직이 있다고 가정한다. 이 경우 @StepScope 없이 Step을 병렬로 실행시키게 되면 서로 다른 Step에서 하나의 Tasklet을 두고 서로 변경하게 된다. 하지만 @StepScope가 있다면 각각의 Step에서 별도의 Tasklet을 생성하고 관리하기 때문에 서로의 상태를 침범할 일이 없다. 



## 여러 개의 step 실행 
배치를 어떤 흐름에 따라 순서대로 실행해야 하는 경우에 Step을 여러개를 만들어서 실행한다. 

처음 실행하는 Step는 start()를 사용하고 그 다음부터는 next()를 사용한다. 


```java
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.JobScope;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Configuration
@EnableBatchProcessing
public class DemoBatchConfiguration {
  
  @Autowired
  public JobBuilderFactory jobBuilderFactory;

  @Autowired
  public StepBuilderFactory stepBuilderFactory;

  @Bean 
  public Step step1() {
    return stepBuilderFactory.get("step1")
              .tasklet((contribution, context) -> {
                log.debug("task1 was executed.");
                return RepeatStatus.FINISHED;
              })
              .build();
  }//:
  
  @Bean 
  public Step step2() {
    return stepBuilderFactory.get("step2")
              .tasklet((contribution, context) -> {
                log.debug("task2 was executed.");
                return RepeatStatus.FINISHED;
              })
              .build();
  }//:

  
  @Bean 
  public Job job1(@Autowired Step step1, @Autowired Step step2) {
    return jobBuilderFactory.get("job1")
            .start(step1)  
            .next(step2)
            .build();
  }
  
}///~
```

## 흐름 제어(Flow) 

실패 시나리오와 성공 시나리오로 배치를 작성해 보자. 

* .on() 
  * 캐치할 ExitStatus 지정
  * "*"일 경우 모든 ExitStatus가 지정된다.
* to()
  * 다음으로 이동할 Step 지정
* from()
  * 일종의 이벤트 리스너 역할
  * 상태값을 보고 일치하는 상태라면 to()에 포함된 step을 호출합니다.
  * step1의 이벤트 캐치가 FAILED로 되있는 상태에서 추가로 이벤트 캐치하려면 from을 써야만 함
* end()
  * end는 FlowBuilder를 반환하는 end와 FlowBuilder를 종료하는 end 2개가 있음
  * on("*")뒤에 있는 end는 FlowBuilder를 반환하는 end
  * build() 앞에 있는 end는 FlowBuilder를 종료하는 end
  * FlowBuilder를 반환하는 end 사용시 계속해서 from을 이어갈 수 있음

> on이 캐치하는 상태값이 BatchStatus가 아닌 ExitStatus인 것에 주의해야 한다. 

```java
import org.springframework.batch.core.ExitStatus;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@EnableBatchProcessing
@Configuration
public class DemoBatchConfigurationCondition {

    
  @Autowired
  public JobBuilderFactory jobBuilderFactory;

  @Autowired
  public StepBuilderFactory stepBuilderFactory;


  @Bean 
  public Step step10() {
    log.debug("step10");
    return stepBuilderFactory.get("step10")
              .tasklet((contribution, context) -> {
                log.debug("task10 was executed.");
                contribution.setExitStatus(ExitStatus.FAILED);  // ExitStatus를 FAILED로 지정하면 실패 
                return RepeatStatus.FINISHED;
              })
              .build();
  }//:
  @Bean 
  public Step step10Alt() {
    log.debug("step10Alt");
    return stepBuilderFactory.get("step10Alt")
              .tasklet((contribution, context) -> {
                log.debug("task10Alt was executed.");
                return RepeatStatus.FINISHED;
              })
              .build();
  }//:
  @Bean 
  public Step step20() {
    log.debug("step20");
    return stepBuilderFactory.get("step20")
              .tasklet((contribution, context) -> {
                log.debug("task20 was executed.");
                return RepeatStatus.FINISHED;
              })
              .build();
  }//:
  @Bean 
  public Step step30() {
    log.debug("step20");
    return stepBuilderFactory.get("step30")
              .tasklet((contribution, context) -> {
                log.debug("task30 was executed.");
                return RepeatStatus.FINISHED;
              })
              .build();
  }//:




  @Bean
  public Job flowJob() {

    return jobBuilderFactory.get("flowJob")
            // 실패 시나리오 :  step 10 -> step10Alt
            // 성공 시나리오 :  step 10 -> step 20 -> step 30
            .start(step10())
                .on("FAILED")       // catch 할 경우 ExitStatus 지정, * 일 경우 모든 ExitStatus가 지정
                .to(step10Alt())    // 다음으로 이동할 Step 지정 
                .on("*")     // step10Alt가 ExitStatus와 관계없이 모두 해당
                .end()       // setp10Alt로 이동하면 flow 종료, 아래 from에 해당하는 flow는 시작되지 않는다. 
            .from(step10()) // step 1으로 부터 
                .on("*")    // FAILED 이외의 모든 경우 
                .to(step20())  // step 2로 이동 
                .next(step30()) // step 2가 성공하면  step 3으로 이동
                .on("*")  // step 3 결과에 관계 없이 
                .end()    // Flow 종료 
            .end() // Job 종료 
            .build();
  }//:

  
}///~

```



## 정리할 내용 
* 실행 매개 변수로 파라미터 전달하는 방법 


