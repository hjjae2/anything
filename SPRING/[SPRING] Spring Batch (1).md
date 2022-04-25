## 개요

<img src="https://docs.spring.io/spring-batch/docs/current/reference/html/images/spring-batch-reference-model.png">


<br>

> *"  A Job has one to many steps, each of which has exactly one ItemReader, one ItemProcessor, and one ItemWriter "*

하나의 Job 은 여러 Step 을 갖는다.

하나의 Step 은 각각 1개의 ItemReader, ItemProcessor, ItemWriter 를 갖는다.


<br>

## Job

> *" A Job is an entity that encapsulates an entire batch process. "*

하나의 잡은 Batch Process 를 표현하는 수단으로 볼 수 있음

Job은 작업이 무엇이고, 어떻게 하는지 에 대해 정의

<br>

Job 은 다음과 같은 정보를 포함

- Job 이름 
- Step 순서 (Step 에 대한 정의) (Definition and ordering of Step instances.)
- 재시작할 수 있는지에 대한 여부 (Whether or not the job is restartable.)

<br>

## JobInstance

> *" A JobInstance refers to the concept of a logical job run. "*

Job 의 논리적인 실행 개념(인스턴스)

(+ 중복되지 않은 JobParameter) JobInstance 실행 가능


**JOB INSTANCE 정보**

```text
+---------------+-------+--------------------------+--------------------------------+
|JOB_INSTANCE_ID|VERSION|JOB_NAME                  |JOB_KEY                         |
+---------------+-------+--------------------------+--------------------------------+
|54             |0      |2022-02-26T18:50:40.703155|58e1898e417c22a4728898e6a35e76f0|
|55             |0      |2022-02-26T18:51:29.449640|58e1898e417c22a4728898e6a35e76f0|
+---------------+-------+--------------------------+--------------------------------+
```


<br>

## JobParameters

> *" How is one JobInstance distinguished from another?*
> 
> *The answer is: JobParameters.*
> 
> *A JobParameters object holds a set of parameters used to start a batch job.*
> 
> *They can be used for identification or even as reference data during the run "*

<br>

**JOB EXECUTION PARAMETER 정보**

> Job 실행 시 넘긴 파라미터 정보가 기록된다.

(JOB_EXECUTION_ID) JOB_EXECUTION 을 바라본다.

```text
+----------------+-------+--------+----------+-------------------+--------+----------+-----------+
|JOB_EXECUTION_ID|TYPE_CD|KEY_NAME|STRING_VAL|DATE_VAL           |LONG_VAL|DOUBLE_VAL|IDENTIFYING|
+----------------+-------+--------+----------+-------------------+--------+----------+-----------+
|42              |STRING |version |1.3       |1970-01-01 09:00:00|0       |0         |Y          |
|43              |STRING |version |1.3       |1970-01-01 09:00:00|0       |0         |Y          |
+----------------+-------+--------+----------+-------------------+--------+----------+-----------+
```

<br>

## JobExecution

> *" A JobExecution refers to the technical concept of a single attempt to run a Job. "*

Job(JobInstance)의 단일 시도

> *" An execution may end in failure or success, but the JobInstance corresponding to a given execution is not considered to be complete unless the execution completes successfully. "*

JobExecution 은 성공/실패 로 끝날 수 있음<br>
하지만 JobInstance 는 JobExecution 이 완벽하게 성공하지 않으면 완료되지 않음

실행(시도) 중 실제 발생한 것에 대해 기록한다.

<br>

**JOB INSTANCE 실행 기록**

- JOB INSTANCE ID
- 생성 시간
- 시작 시간
- 종료 시간
- 상태
- EXIT 상태
- EXIT 메시지
- ...


> 에러 로그는 상세히 기록된다. stackTrace 가 그대로 기록되는 것 같다.
> e.g.) org.springframework.dao.DataIntegrityViolationException: could not execute batch; SQL [update set ~~~ where ???] nested exception is org.hibernate.exception.DataException: could not execute batch ...
> e.g.) All steps already completed or no steps configured for this job. (NOOP 상태 일 때)


```text
+----------------+-------+---------------+-------------------+-------------------+-------------------+---------+---------+------------+-------------------+--------------------------+
|JOB_EXECUTION_ID|VERSION|JOB_INSTANCE_ID|CREATE_TIME        |START_TIME         |END_TIME           |STATUS   |EXIT_CODE|EXIT_MESSAGE|LAST_UPDATED       |JOB_CONFIGURATION_LOCATION|
+----------------+-------+---------------+-------------------+-------------------+-------------------+---------+---------+------------+-------------------+--------------------------+
|6               |2      |3              |2022-02-21 16:42:35|2022-02-21 16:42:35|2022-02-21 16:42:35|FAILED   |FAILED   |   에러로그   |2022-02-21 16:42:35|NULL                      |
|7               |2      |3              |2022-02-21 16:42:56|2022-02-21 16:42:56|2022-02-21 16:42:56|COMPLETED|NOOP     |  NOOP 로그  |2022-02-21 16:42:56|NULL                      |
|8               |2      |3              |2022-02-21 17:44:42|2022-02-21 17:44:42|2022-02-21 17:44:42|COMPLETED|NOOP     |  NOOP 로그  |2022-02-21 17:44:42|NULL                      |
|..              |.      |..             |2022-02-26 18:50:42|2022-02-26 18:50:43|2022-02-26 18:50:45|COMPLETED|COMPLETED|            |2022-02-26 18:50:45|NULL                      |
|71              |2      |54             |2022-02-26 18:50:42|2022-02-26 18:50:43|2022-02-26 18:50:45|COMPLETED|COMPLETED|            |2022-02-26 18:50:45|NULL                      |
|72              |2      |55             |2022-02-26 18:51:31|2022-02-26 18:51:31|2022-02-26 18:51:33|COMPLETED|COMPLETED|            |2022-02-26 18:51:33|NULL                      |
+----------------+-------+---------------+-------------------+-------------------+-------------------+---------+---------+------------+-------------------+--------------------------+
```

<br>

**JOB EXECUTION CONTEXT 정보**

(JOB_EXECUTION_ID) JOB_EXECUTION 을 바라본다.

```text
+----------------+------------------------------+------------------+
|JOB_EXECUTION_ID|SHORT_CONTEXT                 |SERIALIZED_CONTEXT|
+----------------+------------------------------+------------------+
|67              |{"@class":"java.util.HashMap"}|NULL              |
|68              |{"@class":"java.util.HashMap"}|NULL              |
|69              |{"@class":"java.util.HashMap"}|NULL              |
+----------------+------------------------------+------------------+
```


<br>

## Step

- Job의 '단계'를 의미
- Job은 한 개 이상의 Step 으로 구성

<img src="https://docs.spring.io/spring-batch/docs/current/reference/html/images/jobHeirarchyWithSteps.png">

<br>

## StepExecution

- **'Step' 의 '단일 시도'를 의미**
  - Step 이 동작할 때마다, StepExecution 생성되는 개념
- JobExecution, Step, Commit Count, Read Count, Filter Count, Write Count, ExecutionContext 등 포함

> *" ExecutionContext, which contains any data a developer needs to have persisted across batch runs, such as statistics or state information needed to restart.  "*

<br>

**STEP 실행 기록**

- JOB EXECUTION ID
- 이름
- 생성 시간
- 시작 시간
- 종료 시간
- 상태
- COMMIT COUNT
- FILTER COUNT
- WRITE COUNT
- ROLLBACK COUNT
- ...
- EXIT 상태
- EXIT 메시지
- ...

> 에러 로그는 상세히 기록된다. stackTrace 가 그대로 기록되는 것 같다.
> e.g.) org.springframework.dao.DataIntegrityViolationException: could not execute batch; SQL [update set ~~~ where ???] nested exception is org.hibernate.exception.DataException: could not execute batch ...
> e.g.) All steps already completed or no steps configured for this job. (NOOP 상태 일 때)

```text
+-----------------+-------+---------+----------------+-------------------+-------------------+---------+------------+----------+------------+-----------+---------------+----------------+------------------+--------------+---------+------------+-------------------+
|STEP_EXECUTION_ID|VERSION|STEP_NAME|JOB_EXECUTION_ID|START_TIME         |END_TIME           |STATUS   |COMMIT_COUNT|READ_COUNT|FILTER_COUNT|WRITE_COUNT|READ_SKIP_COUNT|WRITE_SKIP_COUNT|PROCESS_SKIP_COUNT|ROLLBACK_COUNT|EXIT_CODE|EXIT_MESSAGE|LAST_UPDATED       |
+-----------------+-------+---------+----------------+-------------------+-------------------+---------+------------+----------+------------+-----------+---------------+----------------+------------------+--------------+---------+------------+-------------------+
|5                |2      |step1    |13              |2022-02-21 18:19:31|2022-02-21 18:19:31|COMPLETED|0           |0         |0           |0          |0              |0               |0                 |1             |FAILED   |   에러로그   |2022-02-21 18:19:31|
|6                |3      |step1    |14              |2022-02-21 18:20:20|2022-02-21 18:20:21|COMPLETED|1           |0         |0           |0          |0              |0               |0                 |0             |COMPLETED|            |2022-02-21 18:20:21|
|7                |3      |step2    |14              |2022-02-21 18:20:21|2022-02-21 18:20:21|COMPLETED|1           |0         |0           |0          |0              |0               |0                 |0             |COMPLETED|            |2022-02-21 18:20:21|
+-----------------+-------+---------+----------------+-------------------+-------------------+---------+------------+----------+------------+-----------+---------------+----------------+------------------+--------------+---------+------------+-------------------+

...

+-----------------+-------+---------+----------------+-------------------+-------------------+---------+------------+----------+------------+-----------+---------------+----------------+------------------+--------------+---------+------------+-------------------+
|STEP_EXECUTION_ID|VERSION|STEP_NAME|JOB_EXECUTION_ID|START_TIME         |END_TIME           |STATUS   |COMMIT_COUNT|READ_COUNT|FILTER_COUNT|WRITE_COUNT|READ_SKIP_COUNT|WRITE_SKIP_COUNT|PROCESS_SKIP_COUNT|ROLLBACK_COUNT|EXIT_CODE|EXIT_MESSAGE|LAST_UPDATED       |
+-----------------+-------+---------+----------------+-------------------+-------------------+---------+------------+----------+------------+-----------+---------------+----------------+------------------+--------------+---------+------------+-------------------+
|60               |3      |step1    |66              |2022-02-26 18:19:52|2022-02-26 18:19:53|COMPLETED|1           |7         |0           |7          |0              |0               |0                 |0             |COMPLETED|            |2022-02-26 18:19:53|
|61               |3      |step1    |67              |2022-02-26 18:36:26|2022-02-26 18:36:27|COMPLETED|1           |7         |0           |7          |0              |0               |0                 |0             |COMPLETED|            |2022-02-26 18:36:27|
+-----------------+-------+---------+----------------+-------------------+-------------------+---------+------------+----------+------------+-----------+---------------+----------------+------------------+--------------+---------+------------+-------------------+

```

<br>

## ExecutionContext

> " An ExecutionContext represents a collection of key/value pairs that are persisted and controlled by the framework in order to allow developers a place to store persistent state that is scoped to a StepExecution object or a JobExecution object. "

<br>

**JOB EXECUTION CONTEXT 정보**

```text
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+
|STEP_EXECUTION_ID|SHORT_CONTEXT                                                                                                                                                                                                                        |SERIALIZED_CONTEXT|
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+
|65               |{"@class":"java.util.HashMap","batch.taskletType":"org.springframework.batch.core.step.item.ChunkOrientedTasklet","batch.stepType":"org.springframework.batch.core.step.tasklet.TaskletStep","AbstractPagingItemReader.read.count":8}|NULL              |
|66               |{"@class":"java.util.HashMap","batch.taskletType":"org.springframework.batch.core.step.item.ChunkOrientedTasklet","batch.stepType":"org.springframework.batch.core.step.tasklet.TaskletStep","AbstractPagingItemReader.read.count":8}|NULL              |
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------+
```

<br><br>

### References
> https://docs.spring.io/spring-batch/docs/current/reference/html/domain.html#domainLanguageOfBatch