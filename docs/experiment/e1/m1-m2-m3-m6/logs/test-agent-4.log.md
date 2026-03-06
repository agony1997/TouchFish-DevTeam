[LOG] task=T4 | event=test-design-start | strategy=red-green,boundary,error,contract
[LOG] task=T4 | event=readonly-analysis | files-read=10 | entities=Task,User,TaskStatus | dtos=CreateTaskRequest,TaskResponse | security=JwtTokenProvider,JwtAuthFilter,SecurityConfig | repos=TaskRepository,UserRepository
[LOG] task=T4 | event=test-design-complete | test-files=TaskControllerIntegrationTest.java,TaskServiceTest.java,UpdateTaskRequestValidationTest.java,PageResponseTest.java,GlobalExceptionHandlerTest.java | test-count=74
