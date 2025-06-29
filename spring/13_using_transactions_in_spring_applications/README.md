# 13장 | 스프링 앱에서 트랜잭션 사용

## 목차

- [x] [트랜잭션](https://github.com/2025-cs-study/long-octopus-leg/blob/main/spring/13_using_transactions_in_spring_applications/13.1_transaction.md)
- [x] [스프링에서 트랜잭션의 작동 방식](https://github.com/2025-cs-study/long-octopus-leg/blob/main/spring/13_using_transactions_in_spring_applications/13.2_how_transactions_work_in_spring.md)
- [x] [스프링 앱에서 트랜잭션 사용](https://github.com/2025-cs-study/long-octopus-leg/blob/main/spring/13_using_transactions_in_spring_applications/13.3_using_transactions.md)

## 면접 예상 질문

- 트랜잭션이란 무엇이고, ACID 속성에 대해 설명해주세요.
- 트랜잭션 전파 속성에 대해 설명해주세요.
- 트랜잭션 격리 수준에 대해 설명해주세요.
- 트랜잭션 동작 원리에 대해 설명해주세요.
- 체크 예외에 대해 설명하고, 트랜잭션에서 체크 예외가 롤백이 되려면 어떻게 해야하는지 설명해주세요.
- 트랜잭션 메서드에서 예외를 직접 처리하면 어떻게 되는지 설명해주세요.
- 읽기 전용 작업에는 @Transactional(readOnly = true)를 붙이는 것이 좋은지 설명해주세요.