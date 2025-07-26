# 6. 다양한 연관관계 매핑

- **다중성**: 다대일(@ManyToOne), 일대다(@OneToMany), 일대일(@OneToOne), 다대다(@ManyToMany)

- **단방향, 양방향**: 객체 관계에서 한 쪽만 참조하는 것을 단방향 관계, 양쪽이 서로 참조하는 것을 양방향 관계라고 함.

- **연관관계의 주인**: JPA는 두 객체 연관관계 중 하나를 정해서 데이터베이스 외래키를 관리함.

<img width="1280" height="574" alt="image" src="https://github.com/user-attachments/assets/722607ef-40d2-41a7-91a5-41d24a764a9f" />

## 6.1 다대일 [N:1]

- 객체 양방향 관계에서 연관관계의 주인은 항상 **다(N)쪽**임.

### 6.1.1 다대일 단방향 [N:1]

- 회원은 `Member.team`으로 팀 엔티티를 참조할 수 있지만, 반대로 팀에는 회원을 참조하는 필드가 없음.
- 따라서 회원과 팀은 다대일 단방향 연관관계임.

### 6.1.2 다대일 양방향 [N:1, 1:N]

- 양방향은 **외래키가 있는 쪽**이 연관관계의 주인임.
- `MEMBER` 테이블이 외래키를 가지고 있으므로 `Member.team`이 연관관계의 주인임.
- 양방향 연관관계는 항상 서로를 참조해야 함.
- 연관관계 편의 메서드(`setTeam()`, `addMember()`)를 작성하는 것이 좋음.
- 양쪽에 다 작성하면 무한루프에 빠지므로 주의해야 함.

## 6.2 일대다 [1:N]

- 일대다 관계는 다대일 관계의 반대 방향임.
- 일대다 관계는 엔티티를 하나 이상 참조할 수 있으므로 자바 컬렉션인 `Collection`, `List`, `Set`, `Map` 중에 하나를 사용해야 함.

### 6.2.1 일대다 단방향 [1:N]

- 하나의 팀은 여러 회원을 참조할 수 있음. 팀은 회원들을 참조하지만 반대로 회원은 팀을 참조하지 않으면 둘의 관계는 단방향임.
- 일대다 단방향 매핑은 매핑한 객체가 관리하는 외래키가 다른 테이블에 있음.
- 다른 테이블에 외래키가 있으면 연관관계 처리를 위한 `UPDATE SQL`을 추가로 실행해야 함.

> **권장사항**
> 일대다 단방향 매핑보다는 **다대일 양방향 매핑을 사용하는 것을 권장함**.

### 6.2.2 일대다 양방향 [1:N, N:1]

- 일대다 양방향 매핑은 존재하지 않음. 대신 **다대일 양방향 매핑**을 사용해야 함.
- 관계형 데이터베이스의 특성상 일대다, 다대일 관계는 항상 다쪽에 외래키가 있음.
- `@OneToMany`, `@ManyToOne` 중에 연관관계의 주인은 항상 다쪽인 **@ManyToOne을 사용한 곳**임.

## 6.3 일대일 [1:1]

- 회원은 하나의 사물함만 사용하고 사물함도 하나의 회원에 의해서만 사용됨.
- 일대일 관계는 그 반대도 일대일 관계임.
- 테이블 관계에서 일대다, 다대일은 항상 **다(N)쪽**이 외래키를 가짐.
- 일대일 관계는 주 테이블이나 대상 테이블 중 어느 곳이나 외래키를 가질 수 있음.

### 6.3.1 주 테이블에 외래키

- 주 객체가 대상 객체를 참조하는 것처럼 주 테이블에 외래키를 두고 대상 테이블을 참조함.
- 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있음.
- 외래키를 객체 참조와 비슷하게 사용할 수 있어 객체지향 개발자들이 선호함.

#### 일대일 주 테이블에 외래키, 단방향
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;
}
```

#### 일대일 주 테이블에 외래키, 양방향
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne(mappedBy = "locker")
    private Member member;
}
```

### 6.3.2 대상 테이블에 외래키

- 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있음.
- 전통적인 데이터베이스 개발자들이 선호함.
- JPA에서는 대상 테이블에 외래키가 있는 단방향 관계를 지원하지 않음.
- 프록시를 사용할 때 외래키를 직접 관리하지 않는 일대일 관계는 지연 로딩으로 설정해도 즉시 로딩됨.

#### 일대일 대상 테이블에 외래키, 양방향
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne(mappedBy = "member")
    private Locker locker;
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```

## 6.4 다대다 [N:N]

- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음.
- 보통 다대다 관계를 일대다, 다대일 관계로 풀어내는 **연결 테이블**을 사용함.
- 회원들은 상품을 주문하고, 상품들은 회원들에 의해 주문됨. 중간에 `Member_Product` 연결 테이블을 추가하여 다대다 관계를 일대다, 다대일 관계로 풀어낼 수 있음.
- 객체는 객체 2개로 다대다 관계를 만들 수 있음.

### 6.4.1 다대다: 단방향

- 회원 엔티티와 상품 엔티티를 `@ManyToMany`와 `@JoinTable`을 사용해서 연결 테이블을 바로 매핑함.
- 회원_상품(`Member_Product`) 엔티티 없이 매핑할 수 있음.

**@JoinTable 속성**
- `@JoinTable.name`: 연결 테이블을 지정함.
- `@JoinTable.joinColumns`: 현재 방향인 엔티티와 매핑할 조인 컬럼 정보를 지정함.
- `@JoinTable.inverseJoinColumns`: 반대 방향인 엔티티와 매핑할 조인 컬럼 정보를 지정함.

```java
// 다대다 단방향 회원
@Entity
public class Member {
    @Id @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
        joinColumns = @JoinColumn(name = "MEMBER_ID"),
        inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<>();
}

// 다대다 단방향 상품
@Entity
public class Product {
    @Id @Column(name = "PRODUCT_ID")
    private String id;

    private String name;
}
```

### 6.4.2 다대다: 양방향

- 다대다 매핑은 역방향도 `@ManyToMany`를 사용함.
- 양방향 연관관계로 만들면 역방향으로 객체 그래프를 탐색할 수 있음.

```java
// 방법1: 다대다 양방향 연관관계 설정
member.getProducts().add(product);
product.getMembers().add(member);

// 방법2: 연관관계 편의 메서드 추가 후 양방향 연관관계 설정
public void addProduct(Product product) {
    products.add(product);
    product.getMembers().add(this);
}

member.addProduct(product);
```

### 6.4.3 다대다: 매핑의 한계와 극복, 연결 엔티티 사용

**매핑의 한계**
- `@ManyToMany`는 도메인 모델이 단순해지고 편리하지만 실무에서 사용하기에는 한계가 있음.
- 회원이 상품을 주문하면 연결 테이블에 단순히 주문한 회원 아이디만 담는 것이 아니라, 연결 테이블에 주문 수량 컬럼이나 주문한 날짜 같은 컬럼이 더 필요함.
- 추가한 칼럼을 매핑할 수 없기 때문에 `@ManyToMany`를 사용할 수 없음.

**해결 방안 다대다 매핑은 역방향도 `@ManyToMany`를 사용함.
- 양방향 연관관계로 만들면 역방향으로 객체 그래프를 탐색할 수 있음.

```java
// 방법1: 다대다 양방향 연관관계 설정
member.getProducts().add(product);
product.getMembers().add(member);

// 방법2: 연관관계 편의 메서드 추가 후 양방향 연관관계 설정
public void addProduct(Product product) {
    products.add(product);
    product.getMembers().add(this);
}

member.addProduct(product);
```

### 6.4.3 다대다: 매핑의 한계와 극복, 연결 엔티티 사용

**매핑의 한계**
- `@ManyToMany`는 도메인 모델이 단순해지고 편리하지만 실무에서 사용하기에는 한계가 있음.
- 회원이 상품을 주문하면 연결 테이블에 단순히 주문한 회원 아이디만 담는 것이 아니라, 연결 테이블에 주문 수량 컬럼이나 주문한 날짜 같은 컬럼이 더 필요함.
- 추가한 칼럼을 매핑할 수 없기 때문에 `@ManyToMany`를 사용할 수 없음.

**해결 방안**
- 연결 테이블을 매핑하는 **연결 엔티티**를 만들고 컬럼들을 매핑해야 함.
- 엔티티 간의 관계도 테이블 관계처럼 다대다에서 **일대다, 다대일**로 풀어야 함.

**식별 관계(Identifying Relationship)**
- 부모 테이블의 기본키를 받아서 자신의 기본키 + 외래키로 사용하는 것

```java
// 회원상품 엔티티 -> 복합 기본키
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {
    @Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member; //MemberProductId.member와 연결

    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product; //MemberProductId.product와 연결

    private int orderAmount;
}

// 회원상품 식별자 클래스
public class MemberProductId implements Serializable {
    private String member; // MemberProduct.member와 연결
    private String product; // MemberProduct.product와 연결

    @Override
    public boolean equals(Object o) { ... }

    @Override
    public int hashCode() { ... }
}
```

**복합키를 위한 식별자 클래스의 특징**
- 복합키는 별도의 식별자 클래스로 만들어야 함.
- `Serializable`을 구현해야 함.
- `equals`와 `hashCode` 메소드를 구현해야 함.
- 기본 생성자가 있어야 함.
- 식별자 클래스는 `public`이어야 함.
- `@IdClass`를 사용하는 방법 외에 `@EmbeddedId`를 사용하는 방법도 있음.

### 6.4.4 다대다: 새로운 기본키 사용

- 데이터베이스에서 자동으로 생성해주는 **대리키를 Long 값으로 사용**하여 다대다 관계를 풀어낼 수 있음.
- 간편하고 거의 영구히 사용할 수 있으며 비즈니스에 의존하지 않음.
- ORM 매핑 시 복합 키를 만들지 않아도 되므로 매핑이 간단함.

```java
// 주문
@Entity
public class Order {
    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    private int orderAmount;
}
```

### 6.4.5 다대다 연관관계 정리

**관계 유형**
- **식별 관계**: 받아온 식별자를 기본키 + 외래키로 사용함.
- **비식별 관계**: 받아온 식별자는 외래키로만 사용하고 새로운 식별자를 추가함.

> **권장사항**
> 객체 입장에서는 비식별 관계를 사용하는 것이 복합키를 위한 식별자 클래스를 만들지 않아도 되므로 단순하고 편리하게 ORM 매핑을 할 수 있어 식별 관계보다는 **비식별 관계**를 추천함.