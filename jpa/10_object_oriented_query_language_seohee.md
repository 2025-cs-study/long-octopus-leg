# 10. 객체지향 쿼리 언어

## JPQL
- 엔티티 객체를 조회하는 객체지향 쿼리
- SQL을 추상화해서 특정 데이터베이스에 의존하지 않고, SQL보다 간결함
- `SELECT m FROM MEMBER AS m where m.username = 'Hello'` ➡️ 별칭 필수
- 이름 기준 파라미터에는 `:` 사용(이게 더 정확), 위치 기준 파라미터는 `?` 사용
- **페치 조인의 특징과 한계**
  - 페치 조인 대상에는 별칭을 줄 수 없음
  - 둘 이상의 컬렉션을 페치할 수 없음
  - 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없음

## QueryDSL
- Criteria의 복잡성 때문에 등장하게 된 QueryDSL
- 엔티티 클래스를 기반으로 **Q-Class(Query Type)** 가 생성되어야 함
```java
JPAQueryFactory queryFactory = new JPAQueryFactory(em);

return queryFactory
        .selectFrom(member)
        .where(member.age.gt(age))
        .orderBy(member.username.asc())
        .fetch();
```

## 네이티브 SQL 소개
- 특정 데이터베이스만 사용하는 함수

## JDBC 직접 사용, 마이바티스 같은 매퍼 프레임워크 사용