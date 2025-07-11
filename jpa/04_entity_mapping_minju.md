# 4. 엔티티 매핑

- **객체와 테이블 매핑**: `@Entity`, `@Table`
- **기본키 매핑**: `@Id`
- **필드와 컬럼 매핑**: `@Column`
- **연관관계 매핑**: `@ManyToOne`, `@JoinColumn`

<img width="2912" height="1796" alt="mermaid-diagram-2025-07-11-142228" src="https://github.com/user-attachments/assets/e07279a0-d5ed-43de-b058-4c9e10af841e" />

## 4.1 @Entity

- JPA를 사용해서 테이블과 매핑할 클래스는 **`@Entity` 어노테이션**을 필수로 붙여야 함. `@Entity`가 붙은 클래스를 **엔티티**라고 부름.
- **파라미터가 없는 기본 생성자**는 필수임. (public 또는 protected)
- **final 클래스, enum, interface, inner 클래스**에는 사용할 수 없음. 저장할 필드에 final을 사용하면 안 됨.

| **속성** | **기능** | **기본값** |
|----------|----------|------------|
| name | JPA에서 사용할 엔티티 이름을 지정한다. | 클래스 이름 |

## 4.2 @Table

- **@Table**은 엔티티와 매핑할 테이블을 지정함.

| **속성** | **기능** | **기본값** |
|----------|----------|------------|
| name | 매핑할 테이블 이름 | 엔티티 이름 |
| catalog | catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다. |  |
| schema | schema 기능이 있는 데이터베이스에서 schema를 매핑한다. |  |

## 4.3 데이터베이스 스키마 자동 생성

- JPA는 **데이터베이스 스키마를 자동으로 생성**하는 기능을 지원함.
- JPA는 클래스의 매핑 정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 생성함.
- `hibernate.show_sql = true`로 설정하면 콘솔에 실행되는 테이블 생성 **DDL(Data Definition Language)**을 출력할 수 있음.
- DDL은 운영 환경에서 사용할 정도로 완벽하지 않으므로 개발 환경에서 사용하거나 매핑을 어떻게 해야 하는지 참고하는 정도로 사용하는 것이 좋음.

```xml
<!-- 애플리케이션 실행 시점에 데이터베이스 테이블 자동 생성 -->
<property name="hibernate.hbm2ddl.auto" value="create" />
```

| **옵션** | **설명** |
|----------|----------|
| create | 기존 테이블을 삭제하고 새로 생성한다. |
| create-drop | create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL을 제거한다. |
| update | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정한다. |
| validate | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 예외를 발생시키고 애플리케이션을 실행하지 않는다. DDL을 수정하지 않는다. |
| none | 자동 생성 기능을 사용하지 않는다. hibernate.hbm2ddl.auto 속성 자체를 삭제하는 방법도 있다. |

## 4.4 DDL 생성 기능

- **@Column** 매핑정보에 `nullable = false`을 지정하면 자동 생성되는 DDL에 **not null 제약조건**을 추가할 수 있음.
- **@Column** 매핑정보에 `length` 속성 값을 사용하면 자동 생성되는 DDL에 **문자의 크기**를 지정할 수 있음.
- **@Table**의 `uniqueConstraints` 속성은 **유니크 제약조건**을 추가할 수 있음.
- 이 기능들은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않음. 따라서 직접 DDL을 만든다면 사용할 필요가 없음.

## 4.5 기본키 매핑

- 데이터베이스마다 **기본키(Primary Key)**를 생성하는 방식이 다르고, JPA는 이 문제를 다음과 같이 해결함:
    - **직접 할당**: 기본키를 애플리케이션에서 직접 할당함.
    - **자동 생성**:
        - **IDENTITY**: 기본키 생성을 데이터베이스에 위임함.
        - **SEQUENCE**: 데이터베이스 시퀀스를 사용해서 기본키를 할당함.
        - **TABLE**: 키 생성 테이블을 사용함.
- 기본키를 직접 할당하려면 **@Id**만 사용하고, 자동 생성 전략을 사용하려면 **@Id에 @GeneratedValue**를 추가하고 원하는 키 생성 전략을 선택함.

<img width="2912" height="1796" alt="mermaid-diagram-2025-07-11-142141" src="https://github.com/user-attachments/assets/050c6204-774c-46e9-8f25-43a344cd5fac" />

### 4.5.1 기본키 직접 할당 전략

```java
@Id
@Column(name = "id")
private String id;
```

- **기본키 직접 할당 전략**: `em.persist()`로 엔티티를 저장하기 전에 애플리케이션에서 기본키를 직접 할당하는 방법
- **@Id 적용 가능 자바 타입**:
    - 자바 기본형
    - 자바 래퍼형
    - String
    - java.util.Date
    - java.sql.Date
    - java.math.BigDecimal
    - java.math.BigInteger

### 4.5.2 IDENTITY 전략

- **IDENTITY**: 기본키 생성을 데이터베이스에 위임하는 전략
- 주로 **MySQL, PostgreSQL, SQL Server, DB2**에서 사용함.
- 데이터베이스에 값을 저장하고 나서야 기본키 값을 구할 수 있을 때 사용함. 따라서 `em.persist()`가 호출되는 즉시 **INSERT SQL**이 데이터베이스에 전달됨.
- 이 전략을 사용하려면 **@GeneratedValue에 strategy = GenerationType.IDENTITY**로 지정하면 됨.

### 4.5.3 SEQUENCE 전략

- **SEQUENCE**: 데이터베이스 시퀀스(유일한 값을 순서대로 생성하는 데이터베이스 오브젝트)를 사용해서 기본키를 생성하는 전략
- **오라클, PostgreSQL, DB2, H2** 데이터베이스에서 사용함.
- **@SequenceGenerator**를 사용해서 시퀀스 생성기를 등록하고, `sequenceName` 속성의 이름을 지정하면 JPA가 이 시퀀스 생성기를 실제 데이터베이스의 시퀀스와 매핑함.
- 키 생성 전략을 **GenerationType.SEQUENCE**로 설정하고 `generator` 속성으로 시퀀스 생성기를 선택함.

| **속성** | **기능** | **기본값** |
|----------|----------|------------|
| name | 식별자 생성기 이름 | 필수 |
| sequenceName | 데이터베이스에 등록되어 있는 시퀀스 이름 | hibernate_sequence |
| initialValue | DDL 생성 시에만 사용되며, 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다. | 1 |
| allocationSize | 시퀀스 한 번 호출에 증가하는 수 | 50 |
| catalog, schema | 데이터베이스 catalog, schema 이름 |  |

### 4.5.4 TABLE 전략

- **TABLE**: 키 생성 전용 테이블을 하나 만들고 이름과 값으로 사용할 칼럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략
- 테이블을 사용하므로 **모든 데이터베이스에 적용**할 수 있음.
- **@TableGenerator**를 사용해서 테이블 키 생성기를 등록함. **GenerationType.TABLE**을 선택하고 `@GeneratedValue.generator`에 테이블 키 생성기를 지정함.
- 시퀀스 대신 테이블을 사용한다는 것만 제외하면 **SEQUENCE 전략과 내부 동작방식**이 같음.

| **속성** | **기능** | **기본값** |
|----------|----------|------------|
| name | 식별자 생성기 이름 | 필수 |
| table | 키 생성 테이블명 | hibernate_sequences |
| pkColumnName | 시퀀스 컬럼명 | sequence_name |
| valueColumnName | 시퀀스 값 컬럼명 | next_val |
| pkColumnValue | 키로 사용할 값 이름 | 엔티티 이름 |
| initialValue | 초기 값 | 0 |
| allocationSize | 시퀀스 한 번 호출에 증가하는 수 | 50 |
| catalog, schema | 데이터베이스 catalog, schema |  |
| uniqueConstraints(DDL) | 유니크 제약 조건을 지정할 수 있다. |  |

### 4.5.5 AUTO 전략

- **GenerationType.AUTO**는 선택한 데이터베이스 방언에 따라 **IDENTITY, SEQUENCE, TABLE 전략** 중 하나를 자동으로 선택함.
- 예) 오라클 - SEQUENCE, MySQL - IDENTITY 등
- **@GeneratedValue.strategy**의 기본값은 AUTO임. 데이터베이스를 변경해도 코드를 수정할 필요가 없음.

## 4.6 필드와 컬럼 매핑

| **분류** | **매핑 어노테이션** | **설명** |
|----------|-------------------|----------|
| 필드와 컬럼 매핑 | @Column | 컬럼을 매핑한다. |
|  | @Enumerated | 자바의 enum 타입을 매핑한다. |
|  | @Temporal | 날짜 타입을 매핑한다. |
|  | @Lob | BLOB, CLOB 타입을 매핑한다. |
|  | @Transient | 특정 필드를 데이터베이스에 매핑하지 않는다. |
| 기타 | @Access | JPA가 엔티티에 접근하는 방식을 지정한다. |