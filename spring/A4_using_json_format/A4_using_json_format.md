## A.4 JSON 형식

- JSON: REST 엔드포인트에서 HTTP 요청/응답 데이터 형식으로 사용되는 방법
- REST 엔드포인트: 앱 간 통신에서 가장 많이 사용하는 방법

### JSON 표현 규칙

1. **객체 정의**: 중괄호 `{}` 사용
2. **속성-값 쌍**: 쉼표 `,`로 구분
3. **속성 이름**: 쌍따옴표 안에 작성
4. **문자열 값**: 쌍따옴표 안에 작성
5. **숫자 값**: 따옴표 없이 작성
6. **속성과 값**: 콜론 `:`으로 구분

![image](https://github.com/user-attachments/assets/cedd1a45-da05-4fc9-a566-f11297b65ddd)

### JSON 예시

**단순 객체**
```json
{
  "name": "chocolate",
  "price": 5
}
```

**중첩 객체**
```json
{
  "name": "chocolate",
  "price": 5,
  "pack": {
    "color": "blue"
  }
}
```

**객체 컬렉션**
```json
[
  {
    "name": "chocolate",
    "price": 5
  },
  {
    "name": "candy",
    "price": 3
  }
]
```


## ⁉️예상 질문

1. JSON이란 무엇이고 왜 사용하는지 설명해주세요.
2. JSON과 XML의 차이점을 설명해주세요.

&nbsp;

※ 자세한 내용은 [블로그](https://mandusitstudy.tistory.com/450)를 참고해주세요.