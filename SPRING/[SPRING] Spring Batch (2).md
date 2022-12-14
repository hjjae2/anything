## 애플리케이션 실행 순서

```
SpringApplication 
    ---> JobLauncherApplicationRunner (ApplicationRunner)
    ---> 
```

<br>

### JobLauncherApplicationRunner

SpringApplication 에 의해 `run()` 메서드가 호출되고, `execute()` 메서드를 실행한다.

```java
    protected void execute(Job job, JobParameters jobParameters)
            throws JobExecutionAlreadyRunningException, 
            JobRestartException, 
            JobInstanceAlreadyCompleteException,
            JobParametersInvalidException, 
            JobParametersNotFoundException {
        JobParameters parameters = getNextJobParameters(job, jobParameters); // (1)
        JobExecution execution = this.jobLauncher.run(job, parameters); // (2)
        if (this.publisher != null) {
            this.publisher.publishEvent(new JobExecutionEvent(execution)); // (3)
        }
    }
```

(1) `jobParameter` 를 가져온다. <br>
(2) `jobLauncher.run()` 을 통해 job 을 실행한다. `jobLauncher.run()`은 `JobExecution` 을 반환한다. <br>
(3) `JobExecutionEvent` 발행한다. <br>

<br>

### SimpleJobLauncher (JobLauncher)

```java
    // Run the provided job with the given JobParameters. 
    // The JobParameters will be used to determine if this is an execution of an existing job instance, or if a new one should be created.
    // JobParameter 는 새 잡 인스턴스를 생성할 지, 기존 잡 인스턴스를 실행할 지 판단하기 위해 사용된다.
    public JobExecution run(final Job job, final JobParameters jobParameters)
			throws JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException,
			JobParametersInvalidException {
        ...
		
        // LastJobExecution 을 가져온다.
        // LastJobExecution 은 JobInstance 기준으로 마지막에 생성된 JobExecution 을 의미한다.
        JobExecution lastExecution = jobRepository.getLastJobExecution(job.getName(), jobParameters);

		// 이전 jobExecution 이 있다면 재시작 가능한 상태인지 확인합니다.
		if (lastExecution != null) {
			if (!job.isRestartable()) {
				throw new JobRestartException("JobInstance already exists and is not restartable");
			}
            
			for (StepExecution execution : lastExecution.getStepExecutions()) {
				BatchStatus status = execution.getStatus();
				if (status.isRunning() || status == BatchStatus.STOPPING) {
					throw new JobExecutionAlreadyRunningException(...);
				} else if (status == BatchStatus.UNKNOWN) {
					throw new JobRestartException(...);
				}
			}
		}

		// JobParameter 유효성 검증합니다.
		job.getJobParametersValidator().validate(jobParameters);

		// 기존 JobInstance, JobExecution 이 있는 경우 : (LastJobExecution의) ExecutionContext 사용합니다.
		// 기존 JobInstance, JobExecution 이 없는 경우 : 새 JobInstnace, ExecutionContext 를 생성합니다.
		// [참고] JobExecution 은 항상 새롭게 생성된다. (JobInstnace, ExecutionContext 내용이 달라지는 것이다.)
		jobExecution = jobRepository.createJobExecution(job.getName(), jobParameters);

		try {
			taskExecutor.execute(new Runnable() {
				@Override
				public void run() {
					try {
						...

						job.execute(jobExecution);

						...
					}
					catch (Throwable t) {
						...
					}
				}
			});
		}
		catch (TaskRejectedException e) {
			// 에러가 발생한 경우, JobExecution 의 상태를 업데이트 합니다.
			// 에러가 발생하지 않은 경우, 위 job.execute() 호출 내에서 업데이트 됩니다.
			jobExecution.upgradeStatus(BatchStatus.FAILED);
			if (jobExecution.getExitStatus().equals(ExitStatus.UNKNOWN)) {
				jobExecution.setExitStatus(ExitStatus.FAILED.addExitDescription(e));
			}
			jobRepository.update(jobExecution);
		}

		return jobExecution;
	}
```

(1) 이미 존재하고 있는 `JobInstance`, `JobExecution` 내용을 확인하고 검증합니다. (`JobParameter` 유호성 검증도 포함합니다.) <br>
(2) `JobExecution` 을 생성합니다. (ExecutionContext 는 새 것이거나, 기존 내용이 됩니다.) <br>
(3) `Job` 을 실행합니다. (`AbstractJob` -> `SimpleJob`)<br>

<br>

### Job (AbstractJob, SimpleJob)

...