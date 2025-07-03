# 2. JPA 시작

## 2.4 객체 매핑 시작

JPA를 사용하려면 회원 클래스와 회원 테이블을 매핑해야 함.

| **매핑 정보** | **회원 객체** | **회원 테이블** |
|---------------|---------------|-----------------|
| **클래스와 테이블** | Member | MEMBER |
| **기본 키** | id | ID |
| **필드와 컬럼** | username | NAME |
| **필드와 컬럼** | age | AGE |

### 매핑 어노테이션

- **@Entity**: 이 클래스(**엔티티 클래스**)를 테이블과 매핑한다고 JPA에게 알려줌.
- **@Table**: 엔티티 클래스에 매핑할 테이블 정보를 알려줌. 생략하면 클래스(엔티티) 이름을 테이블 이름으로 매핑함.
- **@Id**: 엔티티 클래스의 필드(**식별자 필드**)를 테이블의 기본키(Primary key)에 매핑함.
- **@Column**: 필드를 컬럼에 매핑함.
- **매핑 정보가 없는 필드**: 매핑 어노테이션을 생략하면 필드명을 사용해서 컬럼명으로 매핑함. 대소문자를 구분하는 데이터베이스를 사용하면 `@Column(name="AGE")`처럼 명시적으로 매핑해야 함.

```java
import jakarta.persistence.*; // javax -> jakarta

@Entity
@Table(name = "MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private Long id;

    @Column(name = "NAME")
    private String username;

    // 매핑 정보가 없는 필드
    private Integer age;
    ...
}
```

## 2.5 persistence.xml 설정

### JPA 표준 속성

- **javax.persistence.jdbc.driver**: JDBC 드라이버
- **javax.persistence.jdbc.user**: 데이터베이스 접속 아이디
- **javax.persistence.jdbc.password**: 데이터베이스 접속 비밀번호
- **javax.persistence.jdbc.url**: 데이터베이스 접속 URL

> **참고**: 현재 기준으로는 `javax.persistence.*` 패키지가 `jakarta.persistence.*`로 변경됨.

### 하이버네이트 속성

- **hibernate.dialect**: 데이터베이스 방언(Dialect) 설정

### 하이버네이트가 제공하는 데이터베이스 방언

- **H2**: org.hibernate.dialect.H2Dialect
- **오라클 10g**: org.hibernate.dialect.Oracle
- **MySQL**: org.hibernate.dialect.MySQL5InnoDBDialect

> **참고**: 각 데이터베이스마다 SQL 문법과 함수가 조금씩 다름. 방언을 설정하면 JPA가 특정 데이터베이스에 맞는 SQL을 자동 생성함. 현재는 방언을 명시적으로 설정하지 않아도 하이버네이트에서 JDBC 메타 데이터를 기반으로 적절한 방언을 선택해줌.

## 2.6 애플리케이션 개발

JPA 애플리케이션 코드는 크게 3부분으로 나뉘어 있음.

### 2.6.1 엔티티 매니저 설정

1. **엔티티 매니저 팩토리 생성**
    - persistence.xml의 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고 JPA 구현체에 따라 데이터베이스 커넥션 풀도 생성하므로, 엔티티 매니저 팩토리를 생성하는 비용은 아주 큼.
    - 엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 함.

2. **엔티티 매니저 생성**
    - 엔티티 매니저 팩토리에서 엔티티 매니저를 생성함.
    - JPA의 기능 대부분(엔티티를 데이터베이스에 등록/수정/삭제/조회하는 등)은 엔티티 매니저가 제공함.
    - 엔티티 매니저는 내부에 데이터소스(데이터베이스 커넥션)를 유지하면서 데이터베이스와 통신함. 따라서 엔티티 매니저를 스레드간 공유하거나 재사용하면 안 됨.

3. **엔티티 매니저, 엔티티 매니저 팩토리 종료**

### 2.6.2 트랜잭션 관리

- JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 함.
- 트랜잭션을 시작하려면 엔티티 매니저에서 트랜잭션 API를 받아와야 함.
- 트랜잭션 API를 사용해서 비즈니스 로직이 정상 동작하면 트랜잭션을 커밋(commit)하고 예외가 발생하면 트랜잭션을 롤백(rollback)함.

> **참고**: [카카오페이 기술 블로그](https://tech.kakaopay.com/post/jpa-transactional-bri/#%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A0%91%EA%B7%BC%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9B%90%EC%9E%90%EC%84%B1-%EB%B3%B4%EC%9E%A5%EC%9D%84-%EC%9C%84%ED%95%9C-jpa-transaction-%EC%97%AD%ED%95%A0)에 따르면, 여러 데이터에 대한 업데이트가 필요한 경우에 JPA Transactional을 사용한다고 함. `@Transactional(readOnly=true)`는 신중하게 사용하자.

### 2.6.3 비즈니스 로직

비즈니스 로직을 보면 등록(`persist()` 메소드), 수정(엔티티 값 변경, 더티 체킹), 삭제(`remove()` 메소드), 조회(`find()` 메소드) 작업이 엔티티 매니저를 통해서 수행되는 것을 알 수 있음.

### 2.6.4 JPQL

- **JPQL(Java Persistence Query Language)**: 객체지향 쿼리 언어
- JPQL은 SQL과 문법이 거의 유사해서 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등을 사용할 수 있음.

**JPQL vs. SQL**
- **JPQL**: 엔티티 객체(클래스와 필드)를 대상으로 쿼리함.
- **SQL**: 데이터베이스 테이블을 대상으로 쿼리함.

## 참고: JPA vs Spring Data JPA

JPA와 Spring Data JPA는 다름.

### JPA (Java Persistence API)
- 자바 진영의 ORM 기술 표준 명세
- 인터페이스만 제공함. (Hibernate, EclipseLink, OpenJPA 등 구현체 필요)
- EntityManager를 직접 사용함.

### Spring Data JPA
- Spring에서 제공하는 모듈로, JPA를 더 쉽고 편하게 사용할 수 있도록 도와주는 추상화 레이어
- JPA 위에 Repository 패턴을 구현함.
- JPA를 추상화시킨 Repository 인터페이스를 제공하여 EntityManager를 직접 다루지 않아도 됨.

| | **JPA** | **Spring Data JPA** |
|------|---------|---------------------|
| **등록** | entityManager.persist() | repository.save() |
| **조회** | entityManager.find() | repository.findById() |
| **수정** | 더티체킹 또는 merge() | 더티체킹 또는 save() |
| **삭제** | entityManager.remove() | repository.delete() |