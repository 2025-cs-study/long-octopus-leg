# 10. 객체지향 쿼리 언어

## JPQL
- 엔티티 객체를 조회하는 객체지향 쿼리
- SQL을 추상화해서 특정 데이터베이스에 의존하지 않고, SQL보다 간결함
- `SELECT m FROM MEMBER AS m where m.username = 'Hello'` ➡️ 별칭 필수(여기서는 m이 별칭임)
- 파라미터 바인딩 시, 이름 기준 파라미터를 `:` 사용하는 것이 더 정확함, 위치 기준 파라미터는 `?` 사용
- `setFirstResult()`, `setMaxResults()` API를 통해 페이징 쿼리를 쉽게 처리할 수 있음
- 경로 표현식을 사용하여 연관된 엔티티의 속성까지 탐색할 수 있음 (예: m.team.name)
- **페치 조인의 특징과 한계**
  - 페치 조인 대상에는 별칭을 줄 수 없음
  - 둘 이상의 컬렉션을 페치할 수 없음
  - 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없음

## Criteria
- JPQL을 자바 코드로 동적으로 생성할 수 있도록 도와주는 빌더 API임
- 문법적으로 안전하여, 컴파일 시점에 쿼리 오류를 잡을 수 있다는 장점이 있으나 코드가 복잡하다는 단점이 있음

## QueryDSL
- Criteria의 복잡성 때문에 등장하게 된 QueryDSL
- 엔티티 클래스를 기반으로 **Q-Class(Query Type)** 가 생성되어야 함
- JPQL 문법과 유사하고 직관적이어서 학습하기 쉬움
- 수정, 삭제 배치 쿼리(Bulk)도 지원함
```java
// ⭐️ JPAQuery는 구버전이고, 현재는 JPAQueryFactory를 사용함 ⭐️
// JPAQuery는 상태를 가져서 늘 새로운 인스턴스를 생성해야 했지만, JPAQueryFactory는 상태를 비저장해서 빈으로 등록해 재사용할 수 있음
JPAQueryFactory queryFactory = new JPAQueryFactory(em);

return queryFactory
        .selectFrom(member)
        .where(member.age.gt(age))
        .orderBy(member.username.asc())
        .fetch();
```

## 네이티브 SQL
- JPQL이 자동 생성하는 SQL을 수동으로 직접 정의하는 것 ➡️ JPA가 제공하는 기능 대부분을 그대로 사용할 수 있음
- 네이티브 SQL은 관리하기 쉽지 않고 자주 사용하면 특정 데이터베이스에 종속적인 쿼리가 증가해서 이식성이 떨어지게 됨
- 될 수 있으면 표준 SQL을 사용하고 기능이 부족하면 차선책으로 하이버네이트 같은 JPA 구현체가 제공하는 기능을 사용하고 마지막 방법으로 네이티브 SQL을 사용
하는 것이 바람직함

## 객체지향 쿼리 심화
- 벌크 연산 : `executeUpdate()`를 사용 ➡️ 데이터베이스에 직접 쿼리해서 이로 인해 영속성 컨텍스트의 1차 캐시에 있는 엔티티와 DB 데이터 간의 불일치가 발생할 수 있음
- 따라서 벌크 연산 수행 후에는 em.clear() 등으로 영속성 컨텍스트를 초기화하는 것을 권장함