# 14. 컬렉션과 부가 기능

## 14.1 컬렉션

- JPA는 자바에서 기본으로 제공하는 Collection, List, Set, Map 컬렉션을 지원함.
- 컬렉션은 다음 경우에 사용할 수 있음.
    - @OneToMany, @ManyToMany를 사용해서 일대다나 다대다 엔티티 관계를 매핑할 때
    - @ElementCollection을 사용해서 값 타입을 하나 이상 보관할 때

**자바 컬렉션 인터페이스의 특징**
- **Collection**: 자바가 제공하는 최상위 컬렉션 (하이버네이트는 순서X, 중복O로 가정)
- **Set**: 중복을 허용하지 않는 컬렉션 (순서X, 중복X)
- **List**: 순서가 있는 컬렉션 (순서O, 중복O)
- **Map**: Key, Value 구조로 되어 있는 특수한 컬렉션

### 14.1.1 JPA와 컬렉션

- 하이버네이트는 엔티티를 영속 상태로 만들 때 컬렉션 필드를 하이버네이트에서 준비한 컬렉션으로 감싸서 사용함.
- 하이버네이트는 효율적인 컬렉션 관리를 위해 엔티티를 영속 상태로 만들 때 원본 컬렉션을 감싸고 있는 내장 컬렉션을 생성해서 이 내장 컬렉션을 사용하도록 참조를 변경함.
- 하이버네이트가 제공하는 내장 컬렉션은 원본 컬렉션을 감싸고 있어서 **래퍼 컬렉션**으로 부름.

### 14.1.2 Collection, List

- Collection, List는 엔티티를 추가할 때 단순히 저장만 하면 됨. (중복O)
- 따라서 엔티티를 추가해도 지연 로딩된 컬렉션을 초기화하지 않음.

### 14.1.3 Set

- HashSet은 add() 메소드로 객체를 추가할 때마다 equals() 메소드로 같은 객체가 있는지 비교함. (중복X)
- HashSet은 해시 알고리즘을 사용하므로 hashcode()도 함께 사용해서 비교함.
- Set은 엔티티를 추가할 때 중복된 엔티티가 있는지 비교해야 함. 따라서 엔티티를 추가할 때 지연 로딩된 컬렉션을 초기화함.

### 14.1.4 List + @OrderColumn

- 순서가 있다는 의미는 데이터베이스에 순서 값을 저장해서 조회할 때 사용한다는 의미임.
- 하이버네이트는 내부 컬렉션인 PersistentList를 사용함.
- 순서가 있는 컬렉션은 데이터베이스에 순서 값도 함께 관리함.
```java
@Entity
public class Board {
    @Id @GeneratedValue
    private Long id;
    
    @OneToMany(mappedBy = "board")
    @OrderColumn(name = "POSITION")
    private List<Comment> comments = new ArrayList<Comment>();
    
    ...
}
```

**@OrderColumn의 단점**
- @OrderColumn을 Board 엔티티에서 매핑하므로 Comment는 POSITION의 값을 알 수 없음. 그래서 Comment를 INSERT할 때는 POSITION 값이 저장되지 않고, Board.comments의 위치 값을 사용해서 POSITION의 값을 UPDATE하는 SQL이 추가로 발생함.
- List를 변경하면 연관된 많은 위치 값을 변경해야 함.
- 중간에 POSITION 값이 없으면 조회한 List에는 null이 보관됨. 컬렉션을 순회할 때 NullPointerException이 발생할 수 있음.
- @OrderColumn을 매핑하지 말고 개발자가 직접 POSITION 값을 관리하거나 @OrderBy를 사용하는 것을 권장함.

### 14.1.5 @OrderBy

- @OrderColumn이 데이터베이스에 순서용 컬럼을 매핑해서 관리하면, @OrderBy는 데이터베이스의 ORDER BY절을 사용해서 컬렉션을 정렬함.
- 따라서 순서용 컬럼을 매핑하지 않아도 되고, **모든 컬렉션에 사용할 수 있음**.
```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "team")
    @OrderBy("username desc, id asc")
    private Set<Member> members = new HashSet<Member>();
    ...
}
```

- 하이버네이트는 Set에 @OrderBy를 적용해서 결과를 조회하면 순서를 유지하기 위해 HashSet 대신 LinkedHashSet을 내부에서 사용함.

## 14.2 @Converter

컨버터를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장할 수 있음.
```java
@Entity
public class Member {

    @Id
    private String id;
    private String username;
    
    @Convert(converter = BooleanToYNConverter.class)
    private boolean vip;
    
    // Getter, Setter
    ...
}

@Converter
public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {
    
    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return (attribute != null && attribute) ? "Y" : "N";
    }
    
    @Override
    public Boolean convertToEntityAttribute(String dbData) {
        return "Y".equals(dbData);
    }
}
```

**AttributeConverter 인터페이스**
- 컨버터 클래스는 @Converter 어노테이션을 사용하고 AttributeConverter 인터페이스를 구현해야 함.
- **convertToDatabaseColumn()**: 엔티티의 데이터를 데이터베이스 컬럼에 저장할 데이터로 변환함.
- **convertToEntityAttribute()**: 데이터베이스에서 조회한 컬럼 데이터를 엔티티의 데이터로 변환함.

### 14.2.1 글로벌 설정

모든 Boolean 타입에 컨버터를 적용하려면 @Converter(autoApply = true) 옵션을 적용하면 됨.
```java
@Converter(autoApply = true)
public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {
    
    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return (attribute != null && attribute) ? "Y" : "N";
    }
    
    @Override
    public Boolean convertToEntityAttribute(String dbData) {
        return "Y".equals(dbData);
    }
}
```

## 14.3 리스너

JPA 리스너 기능을 사용하면 엔티티의 생명주기에 따른 이벤트를 처리할 수 있음.

### 14.3.1 이벤트 종류

- **PostLoad**: 엔티티가 영속성 컨텍스트에 조회된 직후 또는 refresh를 호출한 후에 호출됨. (2차 캐시에 저장되어 있어도 호출됨)
- **PrePersist**: persist() 메소드를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출됨.
- **PreUpdate**: flush나 commit을 호출해서 엔티티를 데이터베이스에 수정하기 직전에 호출됨.
- **PreRemove**: remove() 메소드를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출됨. 삭제 명령어로 영속성 전이가 일어날 때도 호출됨. orphanRemoval에 대해서는 flush나 commit 시에 호출됨.
- **PostPersist**: flush나 commit을 호출해서 엔티티를 데이터베이스에 저장한 직후에 호출됨. 식별자가 항상 존재함.
- **PostUpdate**: flush나 commit을 호출해서 엔티티를 데이터베이스에 수정한 직후에 호출됨.
- **PostRemove**: flush나 commit을 호출해서 엔티티를 데이터베이스에 삭제한 직후에 호출됨.

### 14.3.2 이벤트 적용 위치

이벤트는 엔티티에서 직접 받거나 별도의 리스너를 등록해서 받을 수 있음.

1. **엔티티에 직접 적용**
2. **별도의 리스너 등록**
3. **기본 리스너 사용**

## 14.4 엔티티 그래프

- 엔티티를 조회할 때 연관된 엔티티들을 함께 조회하려면 글로벌 fetch 옵션을 FetchType.EAGER로 설정함.
- 글로벌 fetch 옵션은 애플리케이션 전체에 영향을 주고 변경할 수 없는 단점이 있음. 따라서 글로벌 fetch 옵션은 FetchType.LAZY를 사용하고, 엔티티를 조회할 때 연관된 엔티티를 함께 조회할 필요가 있으면 JPQL의 페치 조인을 사용함.
- 하지만 페치 조인을 사용하면 같은 JPQL을 중복해서 작성하는 경우가 많음. 엔티티 그래프를 사용하면 이 문제를 해결할 수 있음.

### 14.4.1 Named 엔티티 그래프
```java
@NamedEntityGraph(name = "Order.withMember", attributeNodes = {
    @NamedAttributeNode("member")
})
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    
    ...
}
```

### 14.4.2 em.find()에서 엔티티 그래프 사용

- Named 엔티티 그래프를 사용하려면 정의한 엔티티 그래프를 em.getEntityGraph()를 통해서 찾아오면 됨.
- 엔티티 그래프는 JPA의 힌트 기능을 사용해서 동작함.
```java
EntityGraph graph = em.getEntityGraph("Order.withMember");

Map hints = new HashMap();
hints.put("jakarta.persistence.fetchgraph", graph); // javax.persistence에서 jakarta.persistence로 변경

Order order = em.find(Order.class, orderId, hints);
```

### 14.4.3 subgraph

- Order → OrderItem → Item까지 함께 조회하려면 subgraph를 사용하면 됨.
- Order → OrderItem은 Order가 관리하는 필드지만 OrderItem → Item은 Order가 관리하는 필드가 아님.
```java
@NamedEntityGraph(name = "Order.withAll", attributeNodes = {
    @NamedAttributeNode("member"),
    @NamedAttributeNode(value = "orderItems", subgraph = "orderItems")
    },
    subgraphs = @NamedSubgraph(name = "orderItems", attributeNodes = {
        @NamedAttributeNode("item")
    })
)
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<OrderItem>();
    ...
}
```

### 14.4.4 JPQL에서 엔티티 그래프 사용

JPQL에서 엔티티 그래프를 사용하는 방법은 em.find()와 동일하게 힌트만 추가하면 됨.

### 14.4.5 동적 엔티티 그래프

엔티티 그래프를 동적으로 구성하려면 createEntityGraph() 메소드를 사용하면 됨.

### 14.4.6 엔티티 그래프 정리

**이미 로딩된 엔티티**
- 영속성 컨텍스트에 해당 엔티티가 이미 로딩되어 있으면 엔티티 그래프가 적용되지 않음.
- 아직 초기화되지 않은 프록시에는 엔티티 그래프가 적용됨.

**fetchgraph, loadgraph의 차이**
- **fetchgraph**: 엔티티 그래프에 선택한 속성만 함께 조회함.
- **loadgraph**: 엔티티 그래프에 선택한 속성뿐만 아니라 글로벌 fetch 모드가 FetchType.EAGER로 설정된 연관관계도 포함해서 함께 조회함.

## 14.5 정리

- 컨버터를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장할 수 있음.
- 리스너를 사용하면 엔티티에서 발생한 이벤트를 받아서 처리할 수 있음.
- 페치 조인은 객체지향 쿼리를 사용해야 하지만 엔티티 그래프를 사용하면 객체지향 쿼리를 사용하지 않아도 원하는 객체 그래프를 한 번에 조회할 수 있음.