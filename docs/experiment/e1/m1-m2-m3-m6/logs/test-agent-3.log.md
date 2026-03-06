[LOG] task=T3 | event=test-design-start | strategy=red-green,boundary,error,contract
[LOG] task=T3 | event=test-file-created | file=entity/TaskStatusTest.java | tests=18 | type=pure-unit | covers=TaskStatus-enum-values,getAllowedTransitions,canTransitionTo-valid,canTransitionTo-invalid,null-edge-case
[LOG] task=T3 | event=test-file-created | file=entity/TaskEntityTest.java | tests=9 | type=DataJpaTest | covers=defaults,persistence,auto-id,createdAt,updatedAt,Priority-enum,soft-delete,status-change,contract-fields
[LOG] task=T3 | event=test-file-created | file=repository/TaskRepositoryTest.java | tests=12 | type=DataJpaTest | covers=soft-delete-exclusion,keyword-search,priority-filter,pagination,combined-filters
[LOG] task=T3 | event=test-file-created | file=dto/CreateTaskRequestValidationTest.java | tests=12 | type=pure-unit-validation | covers=title-null,title-empty,title-blank,title-200,title-201,title-1,desc-null,desc-empty,desc-5000,desc-5001,priority-default,contract-fields
[LOG] task=T3 | event=test-file-created | file=dto/TaskResponseTest.java | tests=3 | type=pure-unit | covers=contract-fields,getter-setter-roundtrip,nullable-updatedAt
[LOG] task=T3 | event=test-design-complete | test-files=entity/TaskStatusTest.java,entity/TaskEntityTest.java,repository/TaskRepositoryTest.java,dto/CreateTaskRequestValidationTest.java,dto/TaskResponseTest.java | test-count=54
