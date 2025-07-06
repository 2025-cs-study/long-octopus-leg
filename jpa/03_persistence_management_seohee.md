# 3. 영속성 관리

## 3.1 엔티티 매니저 팩토리와 엔티티 매니저

### 엔티티 매니저 팩토리 (EntityManagerFactory)
- 엔티티 매니저를 만드는 공장
- **생성 비용이 크므로 애플리케이션 전체에서 하나만 생성하여 공유**
- **스레드 안전 (Thread-Safe)** - 여러 스레드가 동시 접근 가능

### 엔티티 매니저 (EntityManager)
- 생성 비용이 거의 들지 않음
- **스레드 안전하지 않음** - 스레드 간 공유 금지
- 데이터베이스 커넥션이 필요한 시점까지 커넥션을 얻지 않음
- **보통 트랜잭션 시작 시 커넥션 획득**

```java
// 엔티티 매니저 팩토리는 하나만 생성하여 공유
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");

// 엔티티 매니저는 요청마다 생성
EntityManager em = emf.createEntityManager();
```

## 3.2 영속성 컨텍스트 (Persistence Context)

### 정의
- **엔티티를 영구 저장하는 환경**
- 엔티티 매니저로 엔티티를 저장하거나 조회하면 영속성 컨텍스트에 엔티티를 보관하고 관리
- `persist()` 메소드는 엔티티를 영속성 컨텍스트에 저장

```java
em.persist(member); // 엔티티를 영속성 컨텍스트에 저장
```

## 3.3 엔티티의 생명주기

### 4가지 상태
1. **비영속 (New/Transient)**: 영속성 컨텍스트와 전혀 관계가 없는 상태
2. **영속 (Managed)**: 영속성 컨텍스트에 저장된 상태
3. **준영속 (Detached)**: 영속성 컨텍스트에 저장되었다가 분리된 상태
4. **삭제 (Removed)**: 삭제된 상태

### 비영속 (New/Transient)
```java
// 객체를 생성한 상태 (비영속)
Member member = new Member("member1", "회원1");
```

### 영속 (Managed)
```java
// 엔티티 매니저를 통해 영속성 컨텍스트에 저장 (영속)
em.persist(member);

// 조회한 엔티티도 영속 상태
Member findMember = em.find(Member.class, "member1");
```

### 준영속 (Detached)
```java
// 특정 엔티티를 준영속 상태로 전환
em.detach(member);

// 영속성 컨텍스트 초기화
em.clear();

// 영속성 컨텍스트 종료
em.close();
```

### 삭제 (Removed)
```java
// 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제
em.remove(member);
```

## 3.4 영속성 컨텍스트의 특징

### 기본 특징
- 영속성 컨텍스트는 엔티티를 **식별자 값으로 구분**
- 영속 상태는 **식별자 값이 반드시 있어야 함**
- JPA는 **트랜잭션 커밋 순간** 영속성 컨텍스트의 새로 저장된 엔티티를 데이터베이스에 반영 (**플러시**)

### 영속성 컨텍스트 사용의 5가지 장점
1. **1차 캐시**
2. **동일성 보장**
3. **트랜잭션을 지원하는 쓰기 지연**
4. **변경 감지 (Dirty Checking)**
5. **지연 로딩**

## 3.4.1 엔티티 조회 - 1차 캐시

### 1차 캐시 동작 방식
- 영속성 컨텍스트 내부에 **Map 구조의 1차 캐시** 존재
- **키**: @Id로 매핑한 식별자, **값**: 엔티티 인스턴스

```java
// 1차 캐시에서 조회
Member member = em.find(Member.class, "member1");

// 같은 엔티티를 반복 조회 - 1차 캐시에서 반환
Member member2 = em.find(Member.class, "member1");

// member == member2 (동일성 보장)
System.out.println(member == member2); // true
```

### 조회 순서
1. **1차 캐시에서 엔티티 찾기**
2. 없으면 **데이터베이스에서 조회**
3. 조회한 엔티티를 **1차 캐시에 저장**
4. **영속 상태의 엔티티 반환**

## 3.4.2 엔티티 등록 - 쓰기 지연

### 쓰기 지연 (Write-Behind) 동작 방식
```java
EntityTransaction tx = em.getTransaction();
tx.begin();

em.persist(memberA);  // INSERT SQL을 쓰기 지연 SQL 저장소에 보관
em.persist(memberB);  // INSERT SQL을 쓰기 지연 SQL 저장소에 보관

// 트랜잭션 커밋 순간 데이터베이스에 INSERT SQL 전송
tx.commit();
```

### 쓰기 지연 과정
1. **persist() 호출** → 엔티티를 1차 캐시에 저장
2. **INSERT SQL 생성** → 쓰기 지연 SQL 저장소에 보관
3. **트랜잭션 커밋** → 플러시 호출
4. **쓰기 지연 SQL 저장소의 쿼리들을 데이터베이스에 전송**
5. **실제 데이터베이스 트랜잭션 커밋**

## 3.4.3 엔티티 수정 - 변경 감지 (Dirty Checking)

### 변경 감지 동작 방식
```java
EntityTransaction tx = em.getTransaction();
tx.begin();

// 영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

// 영속 엔티티 데이터 수정
memberA.setUsername("김수정");
memberA.setAge(30);

// em.update(member) 같은 코드가 필요 없음
tx.commit(); // 트랜잭션 커밋
```

### 변경 감지 과정
1. **트랜잭션 커밋** → 엔티티 매니저 내부에서 플러시 호출
2. **엔티티와 스냅샷 비교** → 변경된 엔티티 감지
3. **UPDATE SQL 생성** → 쓰기 지연 SQL 저장소에 전송
4. **쓰기 지연 저장소의 SQL을 데이터베이스에 전송**
5. **데이터베이스 트랜잭션 커밋**

### JPA 수정 전략
- **기본 전략**: 엔티티의 **모든 필드를 업데이트**
- **장점**:
    - 수정 쿼리가 항상 같아 미리 생성하여 재사용 가능
    - 데이터베이스에서 이전에 파싱된 쿼리 재사용 가능
- **동적 업데이트**: `@DynamicUpdate` 어노테이션 사용 시 수정된 필드만 업데이트

```java
@Entity
@DynamicUpdate  // 수정된 필드만 UPDATE SQL에 포함
public class Member {
    // ...
}
```

## 3.4.4 엔티티 삭제

### 삭제 과정
```java
Member memberA = em.find(Member.class, "memberA");  // 삭제 대상 엔티티 조회
em.remove(memberA);  // 엔티티 삭제
```

1. **삭제 대상 엔티티 조회**
2. **DELETE SQL을 쓰기 지연 SQL 저장소에 등록**
3. **트랜잭션 커밋** → 플러시 호출 → 실제 데이터베이스에 DELETE SQL 전달
4. **remove() 호출 즉시 영속성 컨텍스트에서 제거**

## 3.5 플러시 (Flush)

### 플러시란?
- **영속성 컨텍스트의 변경 내용을 데이터베이스에 반영**
- 영속성 컨텍스트를 비우는 것이 아니라 **동기화하는 것**

### 플러시 발생 시점 (3가지)
```java
// 1. 직접 호출 (거의 사용하지 않음)
em.flush();

// 2. 트랜잭션 커밋 시 자동 호출
tx.commit();

// 3. JPQL 쿼리 실행 시 자동 호출
List<Member> members = em.createQuery("select m from Member m", Member.class)
                        .getResultList();
```

### JPQL 실행 시 플러시가 필요한 이유
```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

// 아직 데이터베이스에 반영되지 않은 엔티티들을 조회하기 위해 플러시 자동 호출
List<Member> members = em.createQuery("select m from Member m", Member.class)
                        .getResultList();
```

## 3.6 준영속 (Detached)

### 준영속 상태란?
- **영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태**
- 영속성 컨텍스트가 제공하는 기능을 사용할 수 없음

### 준영속으로 만드는 3가지 방법
```java
// 1. 특정 엔티티만 준영속 상태로 전환
em.detach(entity);

// 2. 영속성 컨텍스트를 완전히 초기화
em.clear();

// 3. 영속성 컨텍스트를 종료
em.close();
```

### 준영속 상태의 특징
1. **거의 비영속 상태에 가까움**
2. **식별자 값을 가지고 있음** (이미 한번 영속 상태였으므로)
3. **지연 로딩을 할 수 없음** (영속성 컨텍스트가 관리하지 않으므로)

### 병합 (Merge) - 준영속 → 영속
```java
// 준영속 상태의 엔티티
Member detachedMember = em.find(Member.class, "member1");
em.detach(detachedMember);

// 데이터 수정
detachedMember.setUsername("김병합");

// 병합 - 새로운 영속 상태의 엔티티를 반환
Member mergedMember = em.merge(detachedMember);

// detachedMember는 여전히 준영속 상태
// mergedMember는 영속 상태
```

### 병합 동작 방식
- **식별자 값으로 엔티티 조회**
    - 조회할 수 있으면 → **기존 엔티티에 병합 (UPDATE)**
    - 조회할 수 없으면 → **새로운 엔티티 생성 후 병합 (INSERT)**
- **Save or Update**
  - 준영속, 비영속을 따지지 않음.