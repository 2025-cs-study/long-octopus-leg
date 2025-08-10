# 7. 고급 매핑

## 7.1 상속 관계 매핑

- 슈퍼타입 서브타입 관계라는 모델링 기법이 객체의 상속 개념과 유사함
- 슈퍼타입 서브타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때는 3가지 방법을 선택할 수 있음
  - 각각의 테이블로 변환(조인 전략)
  - 통합 테이블로 변환(단일 테이블 전략)
  - 서브타입 테이블로 변환(구현 클래스마다 테이블 전략)

### 7.1.1 조인 전략

- 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본키를 받아서 기본키+외래키로 사용하는 전략임
- 조회할 때 조인을 자주 사용함
- 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없어서 타입을 구분하는 컬럼을 추가해야 함

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    
    private String name; // 이름
    private int price; // 가격
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {
    private String artist;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {
    private String director; // 감독
    private String actor; // 배우
}

@Entity
@DiscriminatorValue("B")
@PrimaryKeyJoinColumn(name = "BOOK_ID") // ID 재정의
public class Book extends Item {
    private String author;
    private String isbn;
}
```

- `@Inheritance(strategy = InheritanceType.JOINED)`: 상속 매핑은 부모 클래스에 @Inheritance를 사용해야 함
- `@DiscriminatorColumn(name = "DTYPE")`: 부모 클래스에 구분 컬럼을 지정함
- `@DiscriminatorValue("M")`: 엔티티를 저장할 때 구분 컬럼에 입력할 값을 지정함
- `@PrimaryKeyJoinColumn`을 쓰면 자식 테이블의 기본키 컬럼명을 변경할 수 있음

**장점**
- 테이블 정규화됨
- 외래키 참조 무결성 활용가능함
- 저장 공간을 효율적으로 사용함

**단점**
- 조인을 많이 사용해서 성능 저하됨
- 조회 쿼리가 복잡함
- 데이터를 등록할 때 INSERT SQL 두 번 실행됨

### 7.1.2 단일 테이블 전략

- 테이블을 하나만 사용하고 구분 컬럼으로 어떤 자식 데이터가 저장되었는지 구분함
- 조인을 사용하지 않아서 일반적으로 가장 빠름
- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 함

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    
    private String name; // 이름
    private int price; // 가격
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item { ... }

@Entity
@DiscriminatorValue("M")
public class Movie extends Item { ... }

@Entity
@DiscriminatorValue("B")
public class Book extends Item { ... }
```

- `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`을 사용하면 단일 테이블 전략을 사용함

**장점**
- 조인이 필요 없으므로 일반적으로 조회 성능이 빠름
- 조회 쿼리가 단순함

**단점**
- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 함
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있음 → 상황에 따라서 조회 성능이 느려짐

### 7.1.3 구현 클래스마다 테이블 전략

- 자식 엔티티마다 테이블을 만듦
- 일반적으로는 추천하지 않음

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    
    private String name; // 이름
    private int price; // 가격
}

@Entity
public class Album extends Item { ... }

@Entity
public class Movie extends Item { ... }

@Entity
public class Book extends Item { ... }
```

- `InheritanceType.TABLE_PER_CLASS`를 선택하면 구현 클래스마다 테이블 전략을 사용함

**장점**
- 서브 타입을 구분해서 처리할 때 효과적임
- not null 제약 조건을 사용할 수 있음

**단점**
- 여러 자식 테이블을 함께 조회할 때 성능이 느림
- 자식 테이블을 통합해서 쿼리하기 어려움

**특징**
- 구분 컬럼을 사용하지 않음

## 7.2 @MappedSuperclass

- 부모 클래스는 테이블과 매핑하지 않고 부모 클래스를 상속 받는 자식 클래스에게 매핑 정보만 제공하고 싶으면 사용함
- 추상 클래스와 비슷한 느낌으로 실제 테이블과 매핑되지 않고 단순히 매핑 정보를 상속할 목적으로만 사용함

```java
@MappedSuperclass
public abstract class BaseEntity {
    @Id @GeneratedValue
    private Long id;
    private String name;
}

@Entity
public class Member extends BaseEntity {
    // ID 상속
    // NAME 상속
    private String email;
}

@Entity
public class Seller extends BaseEntity {
    // ID 상속  
    // NAME 상속
    private String shopName;
}
```

- 부모로부터 물려받은 매핑 정보를 재정의하려면 `@AttributeOverrides`나 `@AttributeOverride`를 사용함
- 연관관계를 재정의하려면 `@AssociationOverrides`나 `@AssociationOverride`를 사용함

## 7.3 복합 키와 식별 관계 매핑

### 7.3.1 식별 관계 vs 비식별 관계

- 외래키가 기본키에 포함되는지 여부에 따라 식별 관계와 비식별 관계로 나뉨

**식별 관계**
- 부모 테이블의 기본키를 내려받아서 자식 테이블의 기본키 + 외래키로 사용하는 관계임

**비식별 관계**
- 부모 테이블의 기본키를 받아서 자식 테이블의 외래키로만 사용하는 관계임
- **필수적 비식별 관계**: 외래키에 NULL을 허용하지 않음, 연관관계를 필수적으로 맺어야 함
- **선택적 비식별 관계**: 외래키에 NULL을 허용함, 연관관계를 맺을지 말지 선택할 수 있음

### 7.3.2 복합키: 비식별 관계 매핑

- `@IdClass`와 `@EmbeddedId` 2가지 방법이 있음
- 복합 키를 사용하려면 별도의 식별자 클래스를 만들어야 함
- 식별자 클래스는 Serializable 인터페이스를 구현해야 함
- equals와 hashCode를 구현해야 함
- 기본 생성자가 있어야 함
- 식별자 클래스는 public이어야 함

### 7.3.3 복합 키: 식별 관계 매핑

- 부모, 자식, 손자까지 계속 기본키를 전달하는 식별 관계임

**@IdClass와 식별 관계**

```java
// 부모
@Entity
public class Parent {
    @Id @Column(name = "PARENT_ID")
    private String id;
    private String name;
}

// 자식
@Entity
@IdClass(ChildId.class)
public class Child {
    @Id
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;
    
    @Id @Column(name = "CHILD_ID")
    private String childId;
    private String name;
}

// 자식 ID
public class ChildId implements Serializable {
    private String parent; // Child.parent 매핑
    private String childId; // Child.childId 매핑
    
    // equals, hashCode
}

// 손자
@Entity
@IdClass(GrandChildId.class)
public class GrandChild {
    @Id
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID"),
        @JoinColumn(name = "CHILD_ID")
    })
    private Child child;
    
    @Id @Column(name = "GRANDCHILD_ID")
    private String id;
    private String name;
}

// 손자 ID
public class GrandChildId implements Serializable {
    private ChildId child; // GrandChild.child 매핑
    private String id; // GrandChild.id 매핑
    
    // equals, hashCode
}
```

**@EmbeddedId와 식별 관계**

```java
// 부모
@Entity
public class Parent {
    @Id @Column(name = "PARENT_ID")
    private String id;
    private String name;
}

// 자식
@Entity
public class Child {
    @EmbeddedId
    private ChildId id;
    
    @MapsId("parentId") // ChildId.parentId 매핑
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;
    
    private String name;
}

// 자식 ID
@Embeddable
public class ChildId implements Serializable {
    private String parentId; // @MapsId("parentId")로 매핑
    
    @Column(name = "CHILD_ID")
    private String id;
    
    // equals, hashCode
}

// 손자
@Entity
public class GrandChild {
    @EmbeddedId
    private GrandChildId id;
    
    @MapsId("childId") // GrandChildId.childId 매핑
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID"),
        @JoinColumn(name = "CHILD_ID")
    })
    private Child child;
    
    private String name;
}

// 손자 ID
@Embeddable
public class GrandChildId implements Serializable {
    private ChildId childId; // @MapsId("childId")로 매핑
    
    @Column(name = "GRANDCHILD_ID")
    private String id;
    
    // equals, hashCode
}
```

### 7.3.4 비식별 관계로 구현

```java
// 부모
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
}

// 자식
@Entity
public class Child {
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}

// 손자
@Entity
public class GrandChild {
    @Id @GeneratedValue
    @Column(name = "GRANDCHILD_ID")
    private Long id;
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "CHILD_ID")
    private Child child;
}
```

### 7.3.5 일대일 식별 관계

- 일대일 식별 관계는 자식 테이블의 기본키 값으로 부모 테이블의 기본키 값만 사용함
- 부모 테이블의 기본키가 복합키가 아니면 자식 테이블의 기본키는 복합키로 구성하지 않아도 됨

### 7.3.6 식별, 비식별 관계의 장단점

**식별 관계 장점**
- 기본키 인덱스를 활용하기 좋음
- 상위 테이블들의 기본키 컬럼을 자식, 손자 테이블들이 가지고 있으므로 특정 상황에 조인 없이도 하위 테이블만으로 검색을 완료할 수 있음

**식별 관계 단점**
- 부모 테이블의 기본키 컬럼들을 자식 테이블로 전파하면서 자식 테이블의 기본키 컬럼이 점점 늘어남
- 복합 기본키를 만들어야 하는 경우가 많음
- 비즈니스 의미가 있는 자연키 컬럼을 조합하는 경우가 많아서 비즈니스 요구사항이 변경되면 기본키도 변경될 수 있음

**비식별 관계 장점**
- 기본키를 비즈니스와 전혀 관계없는 대리키를 주로 사용함
- 테이블 구조가 유연함
- JPA에서 제공하는 기본키 생성 전략을 그대로 사용할 수 있음

**최신 추세**
- 최근에는 비식별 관계를 주로 사용하고 꼭 필요한 곳에만 식별 관계를 사용하는 추세임
- JPA는 식별 관계와 비식별 관계를 모두 지원함

## 7.4 조인 테이블

- 연관관계를 관리하는 방법이 두 가지 있음
  - 조인 컬럼 사용(외래키)
  - 조인 테이블 사용(별도의 테이블을 만들어서 연관관계 관리)

### 7.4.1 일대일 조인 테이블

```java
// 부모
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
    
    @OneToOne
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "PARENT_ID"),
        inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
    )
    private Child child;
}

// 자식
@Entity
public class Child {
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;
}
```

### 7.4.2 일대다 조인 테이블

```java
// 부모
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
    
    @OneToMany
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "PARENT_ID"),
        inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
    )
    private List<Child> child = new ArrayList<Child>();
}

// 자식
@Entity
public class Child {
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;
}
```

### 7.4.3 다대일 조인 테이블

```java
// 부모
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "parent")
    private List<Child> child = new ArrayList<Child>();
}

// 자식
@Entity
public class Child {
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;
    
    @ManyToOne(optional = false)
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "CHILD_ID"),
        inverseJoinColumns = @JoinColumn(name = "PARENT_ID")
    )
    private Parent parent;
}
```

### 7.4.4 다대다 조인 테이블

```java
// 부모
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
    
    @ManyToMany
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "PARENT_ID"),
        inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
    )
    private List<Child> child = new ArrayList<Child>();
}

// 자식
@Entity
public class Child {
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;
}
```

## 7.5 엔티티 하나에 여러 테이블 매핑

- `@SecondaryTable`을 사용함
- 잘 사용하지 않는 방법임

```java
@Entity
@Table(name = "BOARD")
@SecondaryTable(name = "BOARD_DETAIL",
    pkJoinColumns = @PrimaryKeyJoinColumn(name = "BOARD_DETAIL_ID"))
public class Board {
    @Id @GeneratedValue
    @Column(name = "BOARD_ID")
    private Long id;
    
    private String title;
    
    @Column(table = "BOARD_DETAIL")
    private String content;
}
```

- `@SecondaryTable.name`: 매핑할 다른 테이블의 이름을 지정함
- `@SecondaryTable.pkJoinColumns`: 매핑할 다른 테이블의 기본키 컬럼 속성을 지정함
- `@Column(table = "BOARD_DETAIL")`: 해당 컬럼이 어느 테이블에 있는지 지정함