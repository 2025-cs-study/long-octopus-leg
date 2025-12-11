# 16. 트랜잭션과 락, 2차 캐시

## 16.1 트랜잭션과 락

### 16.1.1 트랜잭션과 격리 수준

트랜잭션은 **ACID**라 하는 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)을 보장해야 함.

- **원자성(Atomicity)**: 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공하든가 모두 실패해야 함.
- **일관성(Consistency)**: 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 함.
- **격리성(Isolation)**: 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리함.
- **지속성(Durability)**: 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 함.

문제는 격리성인데 트랜잭션 간에 격리성을 완벽히 보장하려면 트랜잭션을 거의 차례대로 실행해야 함. 이런 문제로 인해 ANSI 표준은 트랜잭션의 격리 수준을 4단계로 나누어 정의함.

**트랜잭션 격리 수준**

- **READ UNCOMMITTED(커밋되지 않은 읽기)**: 커밋하지 않은 데이터를 읽을 수 있음. 예를 들어 트랜잭션1이 데이터를 수정하고 있는데 커밋하지 않아도 트랜잭션2가 수정 중인 데이터를 조회할 수 있음. 이것을 DIRTY READ라 함.
- **READ COMMITTED(커밋된 읽기)**: 커밋한 데이터만 읽을 수 있음. 따라서 DIRTY READ가 발생하지 않음. 하지만 NON-REPEATABLE READ는 발생할 수 있음.
- **REPEATABLE READ(반복 가능한 읽기)**: 한 번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회됨. 하지만 PHANTOM READ는 발생할 수 있음.
- **SERIALIZABLE(직렬화 가능)**: 가장 엄격한 트랜잭션 격리 수준임. 여기서는 PHANTOM READ가 발생하지 않음. 하지만 동시성 처리 성능이 급격히 떨어질 수 있음.

**격리 수준에 따른 문제점**

- **DIRTY READ**: 커밋하지 않은 데이터를 읽을 수 있음.
- **NON-REPEATABLE READ**: 반복해서 같은 데이터를 읽을 수 없음.
- **PHANTOM READ**: 반복 조회 시 결과 집합이 달라짐.

### 16.1.2 낙관적 락과 비관적 락 기초

JPA는 데이터베이스 트랜잭션 격리 수준을 READ COMMITTED 정도로 가정함. 만약 일부 로직에 더 높은 격리 수준이 필요하면 낙관적 락과 비관적 락 중 하나를 사용하면 됨.

**낙관적 락(Optimistic Lock)**

- 트랜잭션 대부분은 충돌이 발생하지 않는다고 낙관적으로 가정하는 방법
- 데이터베이스가 제공하는 락 기능을 사용하는 것이 아니라 **JPA가 제공하는 버전 관리 기능**을 사용함 (애플리케이션이 제공하는 락)
- 트랜잭션을 커밋하기 전까지는 트랜잭션의 충돌을 알 수 없음

**비관적 락(Pessimistic Lock)**

- 트랜잭션의 충돌이 발생한다고 가정하고 우선 락을 걸고 보는 방법
- 데이터베이스가 제공하는 락 기능을 사용함
- 대표적으로 **select for update** 구문이 있음

**두 번의 갱신 분실 문제(Second Lost Updates Problem)**

- 사용자 A와 사용자 B가 동시에 같은 데이터를 수정하면 마지막에 수정한 내용만 남고 처음 수정한 내용은 사라지는 문제임.
- 3가지 해결 방법:
    - **마지막 커밋만 인정하기**: 사용자 A의 내용은 무시하고 마지막에 커밋한 사용자 B의 내용만 인정함.
    - **최초 커밋만 인정하기**: 사용자 A가 이미 수정을 완료했으므로 사용자 B가 수정을 완료할 때 오류가 발생함.
    - **충돌하는 갱신 내용 병합하기**: 사용자 A와 사용자 B의 수정사항을 병합함.

### 16.1.3 @Version

JPA가 제공하는 낙관적 락을 사용하려면 **@Version** 어노테이션을 사용해서 버전 관리 기능을 추가해야 함.
```java
@Entity
public class Board {

    @Id
    private String id;
    private String title;
    
    @Version
    private Integer version;
}
```

- **@Version** 적용 가능 타입: Long(long), Integer(int), Short(short), Timestamp
- 엔티티를 수정할 때마다 버전이 하나씩 자동으로 증가함.
- 엔티티를 수정할 때 조회 시점의 버전과 수정 시점의 버전이 다르면 예외가 발생함.
- 버전 정보를 사용하면 최초 커밋만 인정하기가 적용됨.

### 16.1.4 JPA 락 사용

JPA를 사용할 때 추천하는 전략은 READ COMMITTED 트랜잭션 격리 수준 + 낙관적 버전 관리임.

**락을 적용할 수 있는 위치**

- EntityManager.lock(), EntityManager.find(), EntityManager.refresh()
- Query.setLockMode() (TypeQuery 포함)
- @NamedQuery

**LockModeType 종류**

- **낙관적 락**
    - OPTIMISTIC: 낙관적 락을 사용함.
    - OPTIMISTIC_FORCE_INCREMENT: 낙관적 락 + 버전정보를 강제로 증가함.
- **비관적 락**
    - PESSIMISTIC_READ: 비관적 락, 읽기 락을 사용함.
    - PESSIMISTIC_WRITE: 비관적 락, 쓰기 락을 사용함.
    - PESSIMISTIC_FORCE_INCREMENT: 비관적 락 + 버전정보를 강제로 증가함.
- **기타**
    - NONE: 락을 걸지 않음.
    - READ: JPA1.0 호환 기능임. OPTIMISTIC과 같음.
    - WRITE: JPA1.0 호환 기능임. OPTIMISTIC_FORCE_INCREMENT와 같음.

### 16.1.5 JPA 낙관적 락

낙관적 락은 **버전**을 사용함. 낙관적 락은 트랜잭션을 커밋하는 시점에 충돌을 알 수 있다는 특징이 있음.

**NONE**

- 락 옵션을 적용하지 않아도 엔티티에 @Version이 적용된 필드만 있으면 낙관적 락이 적용됨.
- 용도: 조회한 엔티티를 수정할 때 다른 트랜잭션에 의해 변경(삭제)되지 않아야 함.
- 동작: 엔티티를 수정할 때 버전을 체크하면서 버전을 증가함.
- 이점: 두 번의 갱신 분실 문제를 예방함.

**OPTIMISTIC**

- @Version만 적용했을 때는 엔티티를 수정해야 버전을 체크하지만 이 옵션을 추가하면 엔티티를 조회만 해도 버전을 체크함.
- 용도: 조회 시점부터 트랜잭션이 끝날 때까지 조회한 엔티티가 변경되지 않음을 보장함.
- 동작: 트랜잭션을 커밋할 때 버전 정보를 조회해서 현재 엔티티의 버전과 같은지 검증함.
- 이점: DIRTY READ와 NON-REPEATABLE READ를 방지함.

**OPTIMISTIC_FORCE_INCREMENT**

- 용도: 논리적인 단위의 엔티티 묶음을 관리할 수 있음.
- 동작: 엔티티를 수정하지 않아도 트랜잭션을 커밋할 때 UPDATE 쿼리를 사용해서 버전 정보를 강제로 증가시킴.
- 이점: 강제로 버전을 증가해서 논리적인 단위의 엔티티 묶음을 버전 관리할 수 있음.

### 16.1.6 JPA 비관적 락

JPA가 제공하는 비관적 락은 데이터베이스 트랜잭션 락 메커니즘에 의존하는 방법임.

- 주로 SQL 쿼리에 **select for update** 구문을 사용하면서 시작하고 버전 정보는 사용하지 않음.
- 엔티티가 아닌 스칼라 타입을 조회할 때도 사용할 수 있음.
- 데이터를 수정하는 즉시 트랜잭션 충돌을 감지할 수 있음.

**PESSIMISTIC_WRITE**

- 용도: 데이터베이스에 쓰기 락을 걸음.
- 동작: 데이터베이스 select for update를 사용해서 락을 걸음.
- 이점: NON-REPEATABLE READ를 방지함. 락이 걸린 로우는 다른 트랜잭션이 수정할 수 없음.

**PESSIMISTIC_READ**

- 데이터를 반복 읽기만 하고 수정하지 않는 용도로 락을 걸 때 사용함.
- 데이터베이스 대부분은 PESSIMISTIC_WRITE로 동작함.

**PESSIMISTIC_FORCE_INCREMENT**

- 비관적 락중 유일하게 버전 정보를 사용함.
- 비관적 락이지만 버전 정보를 강제로 증가시킴.

### 16.1.7 비관적 락과 타임아웃

비관적 락을 사용하면 락을 획득할 때까지 트랜잭션이 대기함. 무한정 기다릴 수는 없으므로 타임아웃 시간을 줄 수 있음.
```java
Map<String, Object> properties = new HashMap<String, Object>();

// 타임아웃 10초까지 대기 설정
properties.put("javax.persistence.lock.timeout", 10000);

Board board = em.find(Board.class, "boardId", LockModeType.PESSIMISTIC_WRITE, properties);
```

## 16.2 2차 캐시

### 16.2.1 1차 캐시와 2차 캐시

**1차 캐시**

- 영속성 컨텍스트 내부에 있음. 엔티티 매니저로 조회하거나 변경하는 모든 엔티티는 1차 캐시에 저장됨.
- 트랜잭션을 커밋하거나 플러시를 호출하면 1차 캐시에 있는 엔티티의 변경 내역을 데이터베이스에 동기화함.
- 1차 캐시는 끄고 켤 수 있는 옵션이 아님. 영속성 컨텍스트 자체가 사실상 1차 캐시임.
- 1차 캐시는 같은 엔티티가 있으면 해당 엔티티를 그대로 반환함. 따라서 1차 캐시는 객체 동일성(a==b)을 보장함.

**2차 캐시(공유 캐시)**

- 하이버네이트를 포함한 대부분의 JPA 구현체들은 애플리케이션 범위의 캐시를 지원하는데 이것을 공유 캐시 또는 2차 캐시라 함.
- 2차 캐시는 애플리케이션 범위의 캐시임. 따라서 애플리케이션을 종료할 때까지 캐시가 유지됨.
- 분산 캐시나 클러스터링 환경의 캐시는 애플리케이션보다 더 오래 유지될 수도 있음.
- 2차 캐시를 적절히 활용하면 데이터베이스 조회 횟수를 획기적으로 줄일 수 있음.
- 2차 캐시는 동시성을 극대화하려고 캐시한 객체를 직접 반환하지 않고 복사본을 만들어서 반환함.
- 2차 캐시는 데이터베이스 기본 키를 기준으로 캐시하지만 영속성 컨텍스트가 다르면 객체 동일성(a==b)을 보장하지 않음.

### 16.2.2 JPA 2차 캐시 기능

2차 캐시를 사용하려면 엔티티에 **@Cacheable** 어노테이션을 사용하면 됨.
```java
@Cacheable
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    ...
}
```

persistence.xml에 shared-cache-mode를 설정해서 애플리케이션 전체에 캐시를 어떻게 적용할지 옵션을 설정해야 함.

- **ALL**: 모든 엔티티를 캐시함.
- **NONE**: 캐시를 사용하지 않음.
- **ENABLE_SELECTIVE**: Cacheable(true)로 설정된 엔티티만 캐시함.
- **DISABLE_SELECTIVE**: 모든 엔티티를 캐시하는데 Cacheable(false)로 명시된 엔티티는 캐시하지 않음.
- **UNSPECIFIED**: JPA 구현체가 정의한 설정을 따름.

### 16.2.3 하이버네이트와 EHCACHE 적용

하이버네이트가 지원하는 캐시는 크게 3가지가 있음.

- **엔티티 캐시**: 엔티티 단위로 캐시함. 식별자로 엔티티를 조회하거나 컬렉션이 아닌 연관된 엔티티를 로딩할 때 사용함.
- **컬렉션 캐시**: 엔티티와 연관된 컬렉션을 캐시함. 컬렉션이 엔티티를 담고 있으면 식별자 값만 캐시함.
- **쿼리 캐시**: 쿼리와 파라미터 정보를 키로 사용해서 캐시함. 결과가 엔티티면 식별자 값만 캐시함.

하이버네이트 전용인 **@Cache** 어노테이션을 사용하면 세밀한 설정이 가능함.
```java
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    ...
}
```

**CacheConcurrencyStrategy 속성**

- **NONE**: 캐시를 설정하지 않음.
- **READ_ONLY**: 읽기 전용으로 설정함. 등록, 삭제는 가능하지만 수정은 불가능함.
- **NONSTRICT_READ_WRITE**: 엄격하지 않은 읽고 쓰기 전략임. 동시에 같은 엔티티를 수정하면 데이터 일관성이 깨질 수 있음.
- **READ_WRITE**: 읽기 쓰기가 가능하고 READ COMMITTED 정도의 격리 수준을 보장함.
- **TRANSACTIONAL**: 컨테이너 관리 환경에서 사용할 수 있음. 설정에 따라 REPEATABLE READ 정도의 격리 수준을 보장받을 수 있음.