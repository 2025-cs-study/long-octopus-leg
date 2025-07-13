# 4. 엔티티 매핑

## 4.1 @Entity

- JPA를 사용해서 테이블과 매핑할 클래스의 기본 규칙
  - **기본 생성자가 필수**
  - `@Entity`는 `final` 클래스, `enum`, `interface`, `inner` 클래스에는 사용할 수 없음.
  - **`record`에는 붙일 수 있음.**

```java
@Entity
public class User {
    // 기본 생성자 필수
    public User() {}
    
    // 다른 생성자들...
}
```

## 4.2 @Table

### 주요 속성들

- **name**: 매핑할 테이블 이름
- **catalog**: catalog 기능이 있는 데이터베이스에서 catalog를 매핑
- **schema**: schema 기능이 있는 데이터베이스에서 schema를 매핑
- **uniqueConstraints(DDL)**: DDL 생성 시에 유니크 제약을 만듦

### 유니크 제약 조건 예시

```java
@Table(name = "users", uniqueConstraints = {
    @UniqueConstraint(columnNames = {"email"}),
    @UniqueConstraint(columnNames = {"username", "domain"}),
    @UniqueConstraint(name = "uk_phone", columnNames = {"phone_number"})
})
public class User {
    // ...
}
```

## 4.4 데이터베이스 스키마 자동 생성

### 기본 설정

- `hibernate.hbm2ddl.auto = create`로 설정하면 애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성함.
- ⚠️ **주의**: 운영 환경에서 사용할 만큼 완벽하지 않으므로 개발 환경에서만 사용하거나 매핑 참고용으로만 사용하는 것이 좋음.

### hibernate.hbm2ddl.auto 속성값

| 속성값 | 설명 |
|--------|------|
| **create** | 기존 테이블 삭제 후 다시 생성 (DROP + CREATE) |
| **create-drop** | create와 같으나 종료시점에 테이블 DROP |
| **update** | 변경분만 반영 (운영DB에는 사용하면 안됨) |
| **validate** | 엔티티와 테이블이 정상 매핑되었는지만 확인 |
| **none** | 사용하지 않음 |

### 이름 매핑 전략

테이블명과 컬럼명이 생략되면 **자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 자동 매핑**함.

```java
// Java: userName → DB: user_name
private String userName;
```

## 4.5 DDL 생성 기능

### 제약 조건 설정

```java
@Column(nullable = false, length = 10)
private String name;
```

**중요한 점들**
- **JPA의 실행 로직에는 영향을 주지 않음** ➡️ 직접 DDL을 만든다면 사용할 이유 없음.
- 하지만 **엔티티만 보고도 제약 조건을 쉽게 파악할 수 있다는 장점** 있음.

## 4.6 기본 키 매핑

### JPA 기본 키 생성 전략

#### 1. 직접 할당
- 기본 키를 애플리케이션에서 직접 할당
- `@Id`만 사용

#### 2. 자동 생성 (대리 키 사용)
- `@Id`와 `@GeneratedValue` 사용
- **IDENTITY**: 기본 키 생성을 데이터베이스에 위임
- **SEQUENCE**: 데이터베이스 시퀀스를 사용해서 기본키를 할당
- **TABLE**: 키 생성 테이블을 사용

### 4.6.1 기본키 직접 할당 전략

```java
@Entity
public class Board {
    @Id
    private String id;
}

// 사용 시
Board board = new Board();
board.setId("id1");  // 기본 키 직접 할당
em.persist(board);
```

### 4.6.2 IDENTITY 전략

- **MySQL의 AUTO_INCREMENT** 같은 기능을 사용하는 전략임.

```java
@Entity
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

**특징:**
- 데이터베이스에 값을 저장하고 나서야 기본 키 값을 구할 수 있음
- JPA는 기본 키 값을 얻어오기 위해 **데이터베이스를 추가로 조회**
- 엔티티가 영속 상태가 되려면 식별자가 필요한데, 이 방식은 데이터베이스를 저장해야 식별자를 구할 수 있으므로 **쓰기 지연이 동작하지 않음**

### 4.6.3 SEQUENCE 전략

**데이터베이스 시퀀스**: 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트

#### 1. 먼저 시퀀스를 생성

```sql
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
```

#### 2. 엔티티에서 시퀀스 매핑

```java
@Entity
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR",
    sequenceName = "BOARD_SEQ", // 매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, 
    allocationSize = 1
)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```

#### 주요 속성들

- **sequenceName**: 데이터베이스에 등록되어 있는 시퀀스 이름
- **initialValue**: DDL 생성시에 사용됨. 처음으로 시작되는 수를 지정
- **allocationSize**: 시퀀스 한번 호출에 증가하는 수, 기본값은 50 (최적화 때문에)

**동작 방식**
1. `em.persist()` 호출 시 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회
2. 조회한 식별자를 엔티티에 할당한 후 엔티티를 영속성 컨텍스트에 저장
3. 트랜잭션 커밋해서 플러시가 일어나면 엔티티를 데이터베이스에 저장

### 4.6.4 TABLE 전략

**키 생성 전용 테이블**을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략임.

#### 1. 테이블 생성

```sql
CREATE TABLE MY_SEQUENCES (
    sequence_name VARCHAR(255) NOT NULL,
    next_val BIGINT,
    PRIMARY KEY (sequence_name)
)
```

#### 2. 엔티티에서 TABLE 매핑

```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ", 
    allocationSize = 1
)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```

#### 주요 속성들

- **table**: 키 생성 테이블명
- **pkColumnName**: 시퀀스 컬럼명
- **valueColumnName**: 시퀀스 값 컬럼명
- **pkColumnValue**: 키로 사용할 값 이름

#### MY_SEQUENCES 테이블 결과 예시

| sequence_name | next_val |
|---------------|----------|
| BOARD_SEQ     | 2        |
| MEMBER_SEQ    | 10       |
| PRODUCT_SEQ   | 50       |

### 4.6.5 AUTO 전략

```java
@Entity
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
}
```

- **특징**
  - 선택한 데이터베이스 방언에 따라 **IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택**
  - **장점**: 데이터베이스를 변경해도 코드를 수정할 필요가 없음
  - AUTO로 SEQUENCE나 TABLE 전략이 선택되면 시퀀스나 키 생성용 테이블을 미리 만들어 두어야 함
  - 스키마 자동 생성 기능을 사용한다면 하이버네이트가 기본값으로 자동으로 만들어 줌

## 4.7 필드와 컬럼 매핑: 레퍼런스

### 주요 애노테이션들

| 애노테이션 | 설명 | 사용 빈도                                                |
|-----------|------|------------------------------------------------------|
| **@Column** | 컬럼 매핑 |                                                      |
| **@Enumerated** | 자바의 enum 타입을 매핑 |                                                      |
| **@Lob** | BLOB, CLOB 타입을 매핑 | 거의 안씀 (`@Column(columnDefinition = "TEXT")`를 더 많이 씀) |
| **@Temporal** | 날짜 타입을 매핑 | 거의 안씀 (`LocalDateTime`, `LocalDate`를 많이 쓰면서 안쓰게 됨)   |
| **@Transient** | 특정 필드를 데이터베이스에 매핑하지 않음 | 자주 안씀                                                |
| **@Access** | JPA가 엔티티에 접근하는 방식을 지정 | 거의 안씀                                                |

### 4.7.1 @Column

```java
@Entity
public class Member {
    @Column(name = "member_name", nullable = false)
    private String username;
    
    @Column(length = 100)
    private String description;
}
```

> 💡 자바 기본 타입 `int`, `double` 등에는 DDL로 생성할 때는 `nullable = false`로 지정하는 것이 안전함.(기본 타입은 null이 될 수 없으므로)

### 4.7.6 @Access

- **접근 방식 결정:** `@Id`의 위치를 기준으로 접근 방식이 설정됨.

#### 필드 접근 방식

```java
@Entity
public class Member {
    @Id
    private Long id;  // 필드에 @Id가 있으면 필드 접근
    
    private String name;
}
```

#### 프로퍼티 접근 방식

```java
@Entity
public class Member {
    private Long id;
    private String name;
    
    @Id
    public Long getId() {  // getter에 @Id가 있으면 프로퍼티 접근
        return id;
    }
}
```

- **참고**: 프로퍼티 접근은 **getter를 사용**함.
