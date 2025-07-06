## A.3 HTTP 기초

### A.3.1 HTTP란 무엇인가?

- 웹 앱에서 클라이언트가 서버와 통신하는 방식
- 클라이언트-서버 컴퓨팅 모델 사용
- 무상태(stateless) 텍스트 기반 요청-응답 프로토콜

![image](https://github.com/user-attachments/assets/0d8a36b9-a533-4751-a39b-3e970537845a)

### A.3.2 클라이언트와 서버 간 HTTP 요청

**HTTP 요청 구성요소**

1. **요청 URI(request URI)**: 클라이언트가 요청하는 자원의 경로

   ```
   http://<server_location>:<application_port>/<resource_path>
   ```

![image](https://github.com/user-attachments/assets/4159c11c-8191-45ea-9175-211d4bf88469)

2. **요청 메서드(request method)**: 자원에 대해 수행할 행동
   - **GET**: 데이터 조회
   - **POST**: 데이터 추가
   - **PUT**: 데이터 변경
   - **DELETE**: 데이터 제거
   - **OPTIONS**: 지원 매개변수 목록 반환 (CORS)
   - **PATCH**: 데이터 일부만 변경

3. **요청 매개변수(request parameter)** (선택): 소량 데이터 (10~50자)

4. **요청 헤더(request header)** (선택): URI에 보이지 않는 소량 데이터

5. **요청 본문(request body)** (선택): 대량 데이터 (수백 문자)

### A.3.3 HTTP 응답: 서버가 응답하는 방식

**응답 상태(response status)**: 요청 결과를 간단하게 정의하는 100~599 정수
- **2xx**: 성공
   - 200 OK: 서버가 요청을 처리하는 데 아무 문제도 없음.
   - 201 Created: 서버가 요청된 자원을 추가했다는 것을 성공적으로 클라이언트에 알림.
   - 204 No Content: 클라이언트가 이 응답에 대해 응답 본문을 기대하지 않아도 됨.
- **4xx**: 클라이언트 오류
   - 400 Bad Request: HTTP 요청에 대한 문제를 나타냄.
   - 401 Unauthorized: 일반적으로 클라이언트에 요청 인증이 필요함.
   - 403 Forbidden: 서버가 클라이언트에게 해당 요청을 실행할 권한이 없음.
   - 404 Not Found: 요청된 자원이 없다는 것을 클라이언트에 알림.
- **5xx**: 서버 오류
   - 500 Internal Server Error: 백엔드가 클라이언트 요청을 처리하는 동안 문제가 발생함.

### A.3.4 HTTP 세션

- 서버가 동일한 클라이언트와 여러 요청-응답 간 데이터 저장 메커니즘
- HTTP에서 각 요청은 다른 요청과 서로 독립적이므로 한 요청은 이전 요청, 이후 요청, 동시에 오는 다른 요청에 대해 전혀 알지 못함.

**동작 방식**
- 서버가 클라이언트에 '세션 ID' 할당
- 클라이언트는 각 요청의 HTTP 헤더에 세션 ID 포함
- 일정 시간 후 세션 종료 (보통 1시간 미만)

![image](https://github.com/user-attachments/assets/c768a82f-e394-4e01-8c51-f4456538d9f3)


## ⁉️예상 질문

1. HTTP에 대해 설명해주세요.
2. HTTP 메서드에 대해 설명해주세요. (GET, POST, PUT, DELETE)
3. HTTP 상태 코드에 대해 아는대로 설명해주세요.
4. POST와 PUT의 차이점을 설명해주세요.

&nbsp;

※ 자세한 내용은 [블로그](https://mandusitstudy.tistory.com/450)를 참고해주세요.