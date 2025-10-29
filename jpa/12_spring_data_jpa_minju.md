# 12. 스프링 데이터 JPA

- **스프링 데이터 JPA**는 스프링 프레임워크에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트임.
- 데이터 접근 계층(Data Access Layer)을 개발할 때 구현 클래스 없이 **인터페이스만 작성**해도 개발을 완료할 수 있음.
- CRUD 처리를 위한 공통 메소드를 제공하고, 리포지토리를 개발할 때 인터페이스만 작성하면 애플리케이션 실행 시점에 구현체를 생성해서 주입해줌.

## 12.1 스프링 데이터 JPA 소개

### 12.1.1 스프링 데이터 프로젝트

- **스프링 데이터 JPA**: 스프링 데이터 프로젝트의 하위 프로젝트로, JPA, MongoDB, Neo4J, Redis, Hadoop 등 다양한 데이터 저장소에 대한 접근을 추상화함.

### 12.2 스프링 데이터 JPA 설정

- XML 또는 JavaConfig를 사용한 설정이 가능함.
  - JavaConfig: @EnableJpaRepositories 어노테이션을 사용하여 basePackages에 리포지토리를 검색할 패키지 위치를 지정함.
  - XML: <jpa:repositories>를 사용하여 리포지토리를 검색할 base-package를 지정함.

## 12.3 공통 인터페이스 기능

### 12.3.1 JpaRepository 인터페이스

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

- 제네릭에는 엔티티 타입과 식별자 타입을 지정함.

### 12.3.2 주요 메소드

**JpaRepository가 제공하는 주요 메소드**
- **save(S)**: 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합함.
- **delete(T)**: 엔티티 하나를 삭제함.
- **findById(ID)**: Optional<T>를 반환함. (Spring Data 2.0부터)
- **findAll()**: 모든 엔티티를 조회함. 정렬(Sort)이나 페이징(Pageable) 조건을 파라미터로 제공할 수 있음.
- **getReferenceById(ID)**: getOne() 대신 사용 권장함. (Spring Data JPA 2.7부터 deprecated)

## 12.4 쿼리 메소드 기능

스프링 데이터 JPA가 제공하는 쿼리 메소드 기능은 크게 3가지가 있음.

### 12.4.1 메소드 이름으로 쿼리 생성

- 메소드 이름을 분석해서 JPQL 쿼리를 생성함.
- 정해진 규칙에 따라 메소드 이름을 지어야 함.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByEmailAndName(String email, String name);
}
```

**쿼리 메소드 생성 기능 키워드**
- **findBy...**: 조회
- **countBy...**: 카운트 조회
- **existsBy...**: 존재 여부 확인
- **deleteBy... / remove...By...**: 삭제
- **distinct**: 중복 제거
- **First / Top**: 결과 제한

### 12.4.2 JPA NamedQuery

- 스프링 데이터 JPA는 메소드 이름으로 NamedQuery를 찾아서 실행함.
- 실행할 NamedQuery가 없으면 메소드 이름으로 쿼리를 생성함.

### 12.4.3 @Query, 리포지토리 메소드에 쿼리 정의

- 실행할 메소드에 정적 쿼리를 직접 작성하므로 이름 없는 NamedQuery라 할 수 있음.
- 애플리케이션 실행 시점에 문법 오류를 발견할 수 있음.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username = ?1")
    Member findByUsername(String username);
}
```

## 12.4.7 페이징과 정렬

스프링 데이터 JPA는 쿼리 메소드에 페이징과 정렬 기능을 사용할 수 있도록 2가지 파라미터를 제공함.

- **Sort**: 정렬 기능
- **Pageable**: 페이징 기능 (내부에 Sort 포함)

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    Page<Member> findByName(String name, Pageable pageable);
    List<Member> findByName(String name, Sort sort);
}
```

## 12.6 사용자 정의 리포지토리 구현

- 스프링 데이터 JPA로 리포지토리를 개발하면 인터페이스만 정의하고 구현체는 만들지 않음.
- 다양한 이유로 메소드를 직접 구현해야 할 때가 있음.
- 사용자 정의 인터페이스를 작성하고 구현 클래스를 작성한 다음, 리포지토리 인터페이스에서 사용자 정의 인터페이스를 상속받으면 됨.
- 구현 클래스 이름은 리포지토리 인터페이스 이름 + Impl 이어야 함. (기본값)

```java
// 사용자 정의 인터페이스
public interface MemberRepositoryCustom {
    public List<Member> findMemberCustom();
}

// 사용자 정의 구현 클래스
public class MemberRepositoryImpl implements MemberRepositoryCustom {
    @Override
    public List<Member> findMemberCustom() {
        // 사용자 정의 구현
    }
}

// 리포지토리 인터페이스
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```

## 12.7 Web 확장

### 12.7.2 도메인 클래스 컨버터 기능

- HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩함.
- 도메인 클래스 컨버터는 해당 엔티티와 관련된 리포지토리를 사용해서 엔티티를 찾음.

### 12.7.3 페이징과 정렬 기능

- 스프링 MVC에서 페이징과 정렬 기능을 사용할 수 있음.

## 12.10 스프링 데이터 JPA와 QueryDSL 통합

- 스프링 데이터 JPA는 2가지 방법(QuerydslPredicateExecutor, QuerydslRepositorySupport)으로 QueryDSL을 지원함.
- 최근에는 설정 파일(Configuration)에 JPAQueryFactory를 Bean으로 등록하여 사용자 정의 리포지토리 구현체(MemberRepositoryImpl)에서 JPAQueryFactory를 직접 주입받아 사용함.

### 12.10.1 QuerydslPredicateExecutor

- 리포지토리에서 QuerydslPredicateExecutor를 상속받으면 됨.
- join, fetch를 사용할 수 없다는 한계가 있음.

### 12.10.2 QuerydslRepositorySupport

- QueryDSL의 모든 기능을 사용하려면 JPAQuery 객체를 직접 생성해서 사용하면 됨. 이때 QuerydslRepositorySupport을 상속받으면 편리하게 사용할 수 있음.