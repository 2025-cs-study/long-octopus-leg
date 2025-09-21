# 8. 프록시와 연관관계 관리

## 8.1 프록시
- 지연 로딩을 위한 가짜 객체
- `EntityManager.getReference()` 사용 시 프록시 객체 반환
- 실제 사용 시점에 초기화 (한번만)
- 영속성 컨텍스트에 엔티티가 있으면 실제 엔티티 반환
- 준영속 상태에서 초기화 시 예외 발생

### 8.1.1 프록시 기초
- 프록시는 실제 객체 참조 보관
- 초기화해도 프록시 객체가 실제 엔티티로 바뀌지 않음

### 8.1.2 프록시와 식별자
- 식별자 값은 프록시 객체가 보관
- `@Access(AccessType.PROPERTY)` 설정 시 getId() 호출해도 초기화 안됨
- 연관관계 설정 시에는 초기화하지 않음

### 8.1.3 프록시 확인
- `PersistenceUnitUtil.isLoaded()` - 초기화 여부 확인
- 클래스명에 `..javassist` 붙음

## 8.2 즉시 로딩과 지연 로딩
- **즉시 로딩**: `@ManyToOne(fetch = FetchType.EAGER)`
    - 연관 엔티티 함께 조회, 조인 쿼리 사용
    - `nullable=false` 설정 시 내부 조인 사용 (성능 최적화)
- **지연 로딩**: `@ManyToOne(fetch = FetchType.LAZY)`
    - 실제 사용 시점에 조회, 프록시 객체 반환

## 8.3 지연 로딩 활용
- 자주 함께 사용되면 즉시 로딩, 가끔 사용되면 지연 로딩

### 8.3.1 프록시와 컬렉션 래퍼
- 컬렉션은 하이버네이트 내장 컬렉션으로 변경
- `getOrders()` 호출해도 초기화 안됨
- `getOrders().get(0)` 같은 실제 데이터 접근 시 초기화

### 8.3.2 JPA 기본 페치 전략
- 단일 엔티티: 즉시 로딩 / 컬렉션: 지연 로딩
- **권장**: 모든 연관관계를 지연 로딩으로 설정 후 필요시 즉시 로딩 최적화

### 8.3.3 컬렉션에 FetchType.EAGER 사용 시 주의점
- 컬렉션을 하나 이상 즉시 로딩하는 것은 권장하지 않음
- 컬렉션 즉시 로딩은 항상 외부 조인 사용

## 8.4 영속성 전이: CASCADE
- 부모 엔티티 영속화 시 자식도 함께 영속화
- JPA에서 연관된 모든 엔티티는 영속 상태여야 함

### 8.4.1 영속성 전이: 저장
- `cascade = CascadeType.PERSIST` - 부모 영속화 시 자식도 함께 영속화

### 8.4.2 영속성 전이: 삭제
- `cascade = CascadeType.REMOVE` - 부모 삭제 시 자식도 함께 삭제

### 8.4.3 CASCADE의 종류
- ALL, PERSIST, MERGE, REMOVE, REFRESH, DETACH
- 여러 속성 동시 사용 가능

## 8.5 고아 객체
- 부모와 연관관계가 끊어진 자식 엔티티 자동 삭제
- `orphanRemoval = true` 설정
- `@OneToOne`, `@OneToMany`에만 사용 가능

## 8.6 영속성 전이 + 고아 객체, 생명주기
- `CascadeType.ALL + orphanRemoval = true`
- 부모를 통해 자식의 생명주기 완전 관리 가능