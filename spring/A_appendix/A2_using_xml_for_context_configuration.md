## A.2 XML 컨텍스트 구성

### 개요
- 과거: XML로 컨텍스트와 스프링 프레임워크 설정함.
- 현재: 어노테이션 사용으로 대체함. (앱의 가독성과 유지보수성 향상)

### XML 설정 방법

**config.xml 파일 생성** (resources 폴더)

```xml
<xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="parrotl" class="main.Parrot">
        <property name="name" value="Kiki" />
    </bean>
</beans>
```

**Main 클래스 구현**

```java
public class Main {
    public static void main(String[] args) {
        var context = new ClassPathXmlApplicationContext("config.xml");
        Parrot p = context.getBean(Parrot.class);
        System.out.println(p.getName()); // "Kiki" 출력
    }
}
```

### 참고: XML 설정과 어노테이션 설정의 차이점

| | XML 설정 | 어노테이션 설정 |
|------|----------|-----------------|
| 설정 위치 | 외부 XML 파일 | Java 클래스 내부 |
| 가독성 | 전체 구조 파악 용이 | 개별 클래스 단위로 명확 |
| 유지보수 | 파일 분리로 복잡 | 코드와 함께 관리 |
| IDE 지원 | 제한적 | 강력한 지원 |
| 타입 안전성 | 런타임 검증 | 컴파일 타임 검증 |
| 설정 변경 | 재시작만 필요 | 재컴파일 필요 |


## ⁉️예상 질문

1. XML 설정과 어노테이션 설정의 차이점은 무엇인지 설명해주세요.

&nbsp;

※ 자세한 내용은 [블로그](https://mandusitstudy.tistory.com/450)를 참고해주세요.