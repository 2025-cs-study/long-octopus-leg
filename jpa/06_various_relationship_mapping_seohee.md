# 6. 다양한 연관관계 매핑

## 연관관계의 주인
- 데이터베이스 외래키를 관리하는 연관관계를 연관관계의 주인이라고 함
- 연관관계의 주인은 mappedBy 속성을 사용하지 않음
- 연관관계의 주인이 아니면 mappedBy 속성을 사용하고 연관관계의 주인 필드 이름 값을 입력해야 함


## 6.1 다대일 관계

### 기본 개념
- 객체 양방향 관계에서 연관관계의 주인은 항상 다 쪽임
- 예를 들어 회원(N)과 팀(1)이 있으면 회원 쪽이 연관관계의 주인임

### 6.1.1 다대일 단방향
```java
// 회원 엔티티
@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;
```
- `@JoinColumn(name = "TEAM_ID")`을 사용해서 Member.team 필드를 TEAM_ID 외래키와 매핑함

### 6.1.2 다대일 양방향
- 양방향은 외래키가 있는 쪽이 연관관계의 주인임
- 일대다와 다대일 연관관계는 항상 다에 외래키가 있음
- 양방향 연관관계는 항상 서로를 참조해야 함
- 항상 서로 참조하게 하려면 연관관계 편의 메소드를 작성하는 것이 좋은데 편의 메소드를 양쪽에 다 작성하면 무한루프에 빠지므로 주의해야 함


## 6.2 일대다 관계

### 기본 개념
- 일대다 관계는 엔티티를 하나 이상 참조할 수 있으므로 자바 컬렉션인 Collection, List, Set, Map 중에 하나를 써야 함

### 6.2.1 일대다 단방향
- 하나의 팀은 여러 회원을 참조할 수 있음
- 팀은 회원들을 참조하지만 회원은 팀을 참조하지 않으면 단방향임
- 일대다 단방향 관계는 특이하게도 외래키는 항상 다쪽 테이블에 있지만, 다 쪽인 Member 엔티티에는 외래키를 매핑할 수 있는 참조 필드가 없음
- 대신 반대쪽인 Team 엔티티에만 참조 필드인 members가 있음. 따라서 반대편 테이블의 외래키를 관리하는 특이한 모습이 나타남
- 일대다 단방향 관계를 매핑할 때 @JoinColumn을 명시하지 않으면 JPA는 연결 테이블을 중간에 두고 연관관계를 관리하는 조인 테이블 전략을 기본으로 사용해서 매핑함

```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID") // MEMBER 테이블의 TEAM_ID
    private List<Member> members = new ArrayList<>();
}
```

#### 일대다 단방향 매핑의 단점
- 매핑한 객체가 관리하는 외래키가 다른 테이블에 있음
- 본인 테이블에 외래키가 있으면 엔티티의 저장과 연관관계 처리를 INSERT SQL 한번으로 끝낼 수 있음
- 연관관계를 처리를 위한 UPDATE SQL을 추가로 실행해야 함

#### 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자
- 관리도 부담스럽고, 성능에 문제가 있음
- 다대일 양방향 매핑은 관리해야 하는 외래키가 본인 테이블에 있음

### 6.2.2 일대다 양방향
- 일대다 양방향 매핑은 존재하지 않음. 대신 다대일 양방향 매핑을 사용해야 함
- @OneToMany는 연관관계의 주인이 될 수 없다는 말임
- 일대다, 다대일 관계는 항상 다 쪽에 외래키가 있음. 그래서 연관관계의 주인은 항상 다 쪽은 @ManyToOne을 사용한 곳임
- 그렇다고 일대다 양방향 매핑이 완전히 불가능한 것은 아님
- 일대다 단방향 매핑 반대편에 같은 외래 키를 사용하는 다대일 단방향 매핑을 읽기 전용으로 하나 추가하면 됨
- 아래처럼 insertable = false, updateable = false로 하면 읽기만 가능함

```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID") // MEMBER 테이블의 TEAM_ID
    private List<Member> members = new ArrayList<>();
}
```

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "Member_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updateable = false)
    private Team team;
}
```

## 6.3 일대일 관계

### 기본 개념
- 일대일 관계는 그 반대도 일대일임
- 일대일 관계는 주 테이블과 대상 테이블 둘 중 어느 곳이나 외래키를 가질 수 있음

### 외래키 위치에 따른 특징
#### 주 테이블에 외래키
- 외래키를 객체 참조와 비슷하게 사용할 수 있어서 객체지향 개발자들이 선호함
- 주 테이블이 외래키를 가지고 있으므로 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있음

#### 대상 테이블에 외래키
- 전통적인 데이터베이스 개발자들은 보통 대상 테이블에 외래키를 두는 것을 선호함
- 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다는 장점이 있음

### 6.3.1 주 테이블에 외래키
- 객체지향 개발자들은 주 테이블에 외래키가 있는 것을 선호함
- MEMBER가 주테이블이고 LOCKER가 대상 테이블일 때

```java
// 단방향
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {
    @Id 
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
}
```

```java
// 양방향
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {
    @Id 
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "locker")
    private Member member;
}
```

### 6.3.2 대상 테이블에 외래키
#### 단방향
- 지원하지 않음

#### 양방향
- 일대일 매핑에서 대상 테이블에 외래키를 두고 싶으면 이렇게 양방향으로 매핑해야 함

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne(mappedBy = "member")
    private Locker locker;
}

@Entity
public class Locker {
    @Id 
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```


## 6.4 다대다 관계

### 기본 개념
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
- 따라서 다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용함
- 하지만 객체는 테이블과 다르게 객체 2개를 다대다 관계로 만들 수 있음

### 6.4.1 다대다: 단방향

```java
@Entity
public class Member {
    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
            joinColumns = @JoinColumn(name = "MEMBER_ID"),
            inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<>();
}
```
- @JoinTable을 사용해서 연결 테이블을 바로 매핑함
- 회원과 상품을 연결하는 회원_상품 엔티티 없이 매핑을 완료할 수 있음

### 6.4.2 다대다: 양방향
```java
@Entity
public class Product {
    @Id
    private String id;
    
    @ManyToMany(mappedBy = "products")
    private List<Member> members;
}
```

### 6.4.3 다대다: 매핑 한계와 극복, 연결 엔티티 사용
- 실무에서는 다대다 매핑을 사용하기에는 한계가 있음
- 연결 테이블에 회원 아이디와 상품 아이디만 담고 끝나지 않고, 상품의 수량, 주문할 날짜와 같은 칼럼이 더 필요하기 때문임
- 따라서 연결 테이블을 매핑하는 연결 엔티티를 만들어서 일대다, 다대일 관계로 풀어야 함

```java
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {
    @Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    
    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
    
    private int orderAmount;
}
```

#### 복합 기본키 사용 시 요구사항
- @Id와 @JoinColumn을 동시에 사용해서 기본키와 외래키를 한번에 매핑함
- @IdClass를 사용해서 복합 기본키를 매핑함
- 복합 키는 별도의 식별자 클래스로 만들어야 함
- Serializable을 구현해야 함
- equals와 hashCode 메소드를 구현해야 함
- 기본 생성자가 있어야 함
- 식별자 클래스는 public이어야 함
- @IdClass를 사용하는 방법 외에 @EmbeddedId를 사용하는 방법도 있음
- 회원상품은 회원과 상품의 기본키를 받아서 자신의 기본키로 사용함. 이러한 관계를 식별 관계라고 함
- 복합키를 사용하는 방법은 복잡해서 새로운 기본키를 사용하는 것이 더 간단함

### 6.4.4 다대다: 새로운 기본키 사용
```java
@Entity
public class Order {
    
    @Id 
    @GeneratedValue
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
- 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 값으로 사용하는 것임
- 대리키를 사용함으로써 매핑이 단순하고 이해하기 쉬워짐

### 6.4.5 다대다 연관관계 정리
- 식별 관계: 받아온 식별자를 기본키 + 외래키로 사용함
- 비식별 관계: 받아온 식별자는 외래키로만 사용하고 새로운 식별자를 추가함
- 비식별 관계를 추천함