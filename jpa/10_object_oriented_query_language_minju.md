# 10. 객체지향 쿼리 언어

## 10.1 객체지향 쿼리 소개

- JPA는 복잡한 검색 조건을 사용해서 엔티티 객체를 조회할 수 있는 다양한 쿼리 기술을 지원함.

- `EntityManager.find()` 메소드만으로는 애플리케이션에서 필요한 모든 검색 기능을 구현하기는 현실적으로 어려움.

- **ORM을 사용하면 데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 개발**하므로 검색도 테이블이 아닌 엔티티 객체를 대상으로 하는 방법이 필요함.

**JPA가 공식 지원하는 쿼리 방법**

- **JPQL (Java Persistence Query Language)**: SQL과 가장 비슷하며 가장 일반적으로 사용됨.
- **JPA Criteria**: JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스들을 제공함.
- **네이티브 SQL**: JPA에서 직접 SQL을 사용할 수 있게 해주는 기능임.

**JPA가 공식 지원하지 않는 기능**

- **QueryDSL**: Criteria 쿼리처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음, 비표준 오픈소스 프레임워크임.
- **JDBC 직접 사용, 마이바티스 같은 SQL 매퍼 프레임워크**

## 10.2 JPQL

- JPQL은 엔티티 객체를 조회하는 객체지향 쿼리 언어임.
- SQL을 추상화해서 특정 데이터베이스에 의존하지 않음.
- JPQL은 결국 SQL로 변환됨.

### 10.2.1 기본 문법과 쿼리 API

JPQL의 기본 문법은 SQL과 매우 유사함.

```java
select_문 ::= 
    select_절
    from_절
    [where_절]
    [groupby_절]
    [having_절]
    [orderby_절]

update_문 ::= update_절 [where_절]
delete_문 ::= delete_절 [where_절]
```

**JPQL의 특징**

- 엔티티와 속성은 대소문자를 구분함 (Member, username)
- JPQL 키워드는 대소문자를 구분하지 않음 (SELECT, from)
- 테이블 이름이 아닌 엔티티 이름을 사용함
- 별칭은 필수임 (as는 생략 가능)

```java
// TypedQuery: 반환 타입이 명확할 때 사용
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
List<Member> resultList = query.getResultList();

// Query: 반환 타입이 명확하지 않을 때 사용
Query query = em.createQuery("SELECT m.username, m.age FROM Member m");
List resultList = query.getResultList();
```

### 10.2.2 파라미터 바인딩

JPQL은 이름 기준 파라미터 바인딩과 위치 기준 파라미터 바인딩을 지원함.

```java
// 이름 기준 파라미터 바인딩 (권장)
em.createQuery("SELECT m FROM Member m WHERE m.username = :name", Member.class)
  .setParameter("name", "회원1")
  .getResultList();

// 위치 기준 파라미터 바인딩
em.createQuery("SELECT m FROM Member m WHERE m.username = ?1", Member.class)
  .setParameter(1, "회원1")
  .getResultList();
```

### 10.2.3 프로젝션

SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라 함.

- **엔티티 프로젝션**: `SELECT m FROM Member m`
- **임베디드 타입 프로젝션**: `SELECT m.address FROM Member m`
- **스칼라 타입 프로젝션**: `SELECT m.username, m.age FROM Member m`

```java
// 여러 값 조회 - Object[] 타입으로 조회
List<Object[]> resultList = em.createQuery("SELECT m.username, m.age FROM Member m")
                               .getResultList();

// 여러 값 조회 - new 명령어로 DTO 조회
List<MemberDTO> result = em.createQuery(
    "SELECT new jpabook.dto.MemberDTO(m.username, m.age) FROM Member m", 
    MemberDTO.class)
    .getResultList();
```

### 10.2.4 페이징 API

JPA는 페이징을 다음 두 API로 추상화함.

- **setFirstResult(int startPosition)**: 조회 시작 위치 (0부터 시작)
- **setMaxResults(int maxResult)**: 조회할 데이터 수

```java
List<Member> resultList = em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class)
                           .setFirstResult(10)
                           .setMaxResults(20)
                           .getResultList();
```

### 10.2.5 집합과 정렬

JPQL은 다음과 같은 집합 함수를 제공함: COUNT, SUM, AVG, MAX, MIN

```java
SELECT 
    COUNT(m),        // 회원 수
    SUM(m.age),      // 나이 합
    AVG(m.age),      // 평균 나이
    MAX(m.age),      // 최대 나이
    MIN(m.age)       // 최소 나이
FROM Member m
```

### 10.2.6 JPQL 조인

JPQL에서 조인은 연관 필드를 사용함.

```java
// 내부 조인
SELECT m FROM Member m INNER JOIN m.team t WHERE t.name = '팀A'

// 외부 조인
SELECT m FROM Member m LEFT JOIN m.team t

// 세타 조인
SELECT count(m) FROM Member m, Team t WHERE m.username = t.name
```

### 10.2.7 페치 조인

페치 조인은 JPQL에서 성능 최적화를 위해 제공하는 기능으로, 연관된 엔티티나 컬렉션을 한 번에 같이 조회함.

```java
// 엔티티 페치 조인
SELECT m FROM Member m JOIN FETCH m.team

// 컬렉션 페치 조인
SELECT t FROM Team t JOIN FETCH t.members WHERE t.name = '팀A'
```

- 페치 조인은 SQL 한 번으로 연관된 엔티티들을 함께 조회할 수 있어 성능을 최적화할 수 있음.
- 페치 조인은 글로벌 로딩 전략보다 우선함.
- 연관된 엔티티를 쿼리 시점에 조회하므로 지연 로딩이 발생하지 않음.

### 10.2.8 경로 표현식

경로 표현식은 .(점)을 찍어 객체 그래프를 탐색하는 것임.

- **상태 필드**: 단순히 값을 저장하기 위한 필드 (m.username, m.age)
- **연관 필드**: 연관관계를 위한 필드

### 10.2.9 서브 쿼리

JPQL도 SQL처럼 서브 쿼리를 지원함.

```java
// 나이가 평균보다 많은 회원
SELECT m FROM Member m 
WHERE m.age > (SELECT AVG(m2.age) FROM Member m2)

// 한 건이라도 주문한 고객
SELECT m FROM Member m
WHERE (SELECT COUNT(o) FROM Order o WHERE m = o.member) > 0
```

### 10.2.15 Named 쿼리: 정적 쿼리

Named 쿼리는 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용할 수 있는 정적 쿼리임.

```java
@Entity
@NamedQuery(
    name = "Member.findByUsername",
    query = "SELECT m FROM Member m WHERE m.username = :username"
)
public class Member {
    //...
}

// Named 쿼리 사용
List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                           .setParameter("username", "회원1")
                           .getResultList();
```

- Named 쿼리는 애플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱해둠.
- 오류를 빨리 확인할 수 있고 파싱된 결과를 재사용하여 성능상 이점이 있음.
- 변하지 않는 정적 SQL이 생성되어 데이터베이스 조회 성능 최적화에도 도움됨.

## 10.3 Criteria

- Criteria는 JPQL을 자바 코드로 작성하도록 도와주는 빌더 클래스 API임.
- 문법 오류를 컴파일 단계에서 잡을 수 있고 동적 쿼리를 안전하게 생성할 수 있음.
- 대신 코드가 복잡하고 장황해서 직관적 이해가 어려움.

### 10.3.1 Criteria 기초

```java
// JPQL: SELECT m FROM Member m WHERE m.username='member1' ORDER BY m.age DESC

// Criteria로 작성한 코드
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class);
cq.select(m)
  .where(cb.equal(m.get("username"), "member1"))
  .orderBy(cb.desc(m.get("age")));

List<Member> resultList = em.createQuery(cq).getResultList();
```

### 10.3.12 동적 쿼리

Criteria의 장점은 동적 쿼리를 편리하게 작성할 수 있다는 점임.

```java
// 검색 조건에 따른 동적 쿼리
public List<Member> findMembers(String name, Integer age) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Member> cq = cb.createQuery(Member.class);
    Root<Member> m = cq.from(Member.class);
    
    List<Predicate> criteria = new ArrayList<>();
    
    if (name != null) {
        criteria.add(cb.equal(m.get("username"), name));
    }
    if (age != null) {
        criteria.add(cb.equal(m.get("age"), age));
    }
    
    cq.where(criteria.toArray(new Predicate[0]));
    return em.createQuery(cq).getResultList();
}
```

## 10.4 QueryDSL

- QueryDSL은 JPQL을 코드로 작성할 수 있도록 도와주는 빌더 API임.
- 코드 기반이면서 단순하고 사용하기 쉬움.
- 작성한 코드도 JPQL과 비슷해서 가독성이 향상됨.
- **실무에서는 복잡한 동적 쿼리를 작성할 때 QueryDSL을 많이 사용함.**

### 10.4.1 QueryDSL 설정

QueryDSL을 사용하려면 엔티티를 기반으로 쿼리 타입(Q클래스)을 생성해야 함.

### 10.4.2 시작

```java
// JPQL: SELECT m FROM Member m WHERE m.username = 'member1'

// QueryDSL
QMember member = QMember.member;

List<Member> members = queryFactory
    .select(member)
    .from(member)
    .where(member.username.eq("member1"))
    .fetch();
```

### 10.4.11 동적 쿼리

BooleanBuilder를 사용하면 특정 조건에 따른 동적 쿼리를 생성할 수 있음.

```java
SearchParam param = new SearchParam();
param.setName("itemA");
param.setPrice(10000);

QItem item = QItem.item;
BooleanBuilder builder = new BooleanBuilder();

if (StringUtils.hasText(param.getName())) {
    builder.and(item.name.contains(param.getName()));
}
if (param.getPrice() != null) {
    builder.and(item.price.gt(param.getPrice()));
}

List<Item> result = queryFactory
    .selectFrom(item)
    .where(builder)
    .fetch();
```

## 10.5 네이티브 SQL

- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 사용해야 할 때는 네이티브 SQL을 사용함.
- 네이티브 SQL은 SQL을 직접 사용할 수 있게 해주는 기능임.
- 네이티브 SQL을 사용하면 엔티티를 조회할 수 있고 JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있음.

### 10.5.1 네이티브 SQL 사용

```java
// 네이티브 SQL로 엔티티 조회
String sql = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?";

Query nativeQuery = em.createNativeQuery(sql, Member.class)
                     .setParameter(1, 20);

List<Member> resultList = nativeQuery.getResultList();
```

- 네이티브 SQL은 위치 기반 파라미터만 지원함 (하이버네이트는 이름 기반도 지원)
- 반환 타입을 지정해도 TypedQuery가 아닌 Query를 반환함

### 10.5.2 Named 네이티브 SQL

네이티브 SQL도 Named 쿼리를 사용할 수 있음.

```java
@Entity
@NamedNativeQuery(
    name = "Member.memberSQL",
    query = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?",
    resultClass = Member.class
)
public class Member { ... }
```

## 10.6 객체지향 쿼리 심화

### 10.6.1 벌크 연산

엔티티를 수정하려면 영속성 컨텍스트의 변경 감지 기능이나 병합을 사용하고, 삭제하려면 `EntityManager.remove()` 메소드를 사용함. 하지만 이 방법으로 수백 개 이상의 엔티티를 하나씩 처리하기에는 시간이 너무 오래 걸림.

```java
// 재고가 10개 미만인 모든 상품의 가격을 10% 상승
String qlString = 
    "UPDATE Product p " +
    "SET p.price = p.price * 1.1 " +
    "WHERE p.stockAmount < :stockAmount";

int resultCount = em.createQuery(qlString)
                   .setParameter("stockAmount", 10)
                   .executeUpdate();
```

- 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리함.
- 벌크 연산을 먼저 실행하거나, 벌크 연산 수행 후 영속성 컨텍스트를 초기화하는 것이 안전함.

### 10.6.2 영속성 컨텍스트와 JPQL

JPQL로 엔티티를 조회하면 영속성 컨텍스트에서 관리되지만, 임베디드 타입이나 값 타입은 영속성 컨텍스트에서 관리되지 않음.

- JPQL로 조회한 엔티티는 영속 상태임.
- 영속성 컨텍스트에 이미 존재하는 엔티티가 있으면 기존 엔티티를 반환함.