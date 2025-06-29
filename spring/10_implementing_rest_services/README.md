# 10장 | REST 서비스 구현

## 목차

- [x] [REST 서비스를 이용한 앱 간 데이터 교환](./10.1_rest_service.md)
- [x] [REST 엔드포인트 구현](./10.2_rest_endpoint.md)
- [x] [HTTP 응답 관리](./10.3_managing_http_responses.md)
- [x] [요청 본문을 사용하여 클라이언트의 데이터 가져오기](./10.4_retrieving_client_data_using_requestbody.md)

## 면접 예상 질문
- REST API와 Spring MVC의 차이점은 무엇인지 설명해주세요.
- @Controller 애너테이션과 @RestController 애너테이션에 대해서 설명해주세요.
- @ResponseBody 애너테이션의 역할은 무엇인지 설명해주세요.
- DTO(Data Transfer Object)란 무엇이고 왜 사용하는지 설명해주세요.
- ResponseEntity를 사용하는 이유와 장점에 대해 설명해주세요.
- HTTP 응답 상태 코드 200, 400, 404, 500의 의미에 대해 설명해주세요.
- 컨트롤러에서 직접 예외 처리하는 방법과 @RestControllerAdvice를 사용하는 방법의 차이점은 무엇인지 설명해주세요.
- ResponseEntity<?> 에서 와일드카드(?)를 사용하는 이유는 무엇인지 설명해주세요.
- @RequestBody를 사용할 때 스프링이 기본적으로 가정하는 데이터 형식은 무엇인지 설명해주세요.
- 클라이언트가 잘못된 JSON 형식으로 데이터를 보내면 어떤 HTTP 상태 코드가 반환되는지 설명해주세요.
- POST 요청에서 @RequestBody를 사용하지 않고 데이터를 받을 수 있는 다른 방법은 무엇이 있을지 설명해주세요.