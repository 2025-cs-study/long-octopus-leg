# 14. 컬렉션과 부가 기능

## 14.1 컬렉션
- **JPA와 컬렉션**: JPA는 엔티티에서 `Collection`, `List`, `Set`, `Map`과 같은 자바의 표준 컬렉션 인터페이스들을 지원함.

    |      컬렉션 인터페이스      |     내장 컬렉션     | 중복 허용 | 순서 보관 |
    |:-------------------:|:--------------:|:-----:|:-----:|
    |  Collection, List   | PersistentBag  |   O   |   X   |
    |         Set         | PersistentSet  |   X   |   X   |
    | List + @OrderColumn | PersistentList |   O   |   O   |
- **Collection, List**: `List` 인터페이스는 순서가 있는 컬렉션이며, 객체의 중복 저장을 허용함.
- **Set**: `Set` 인터페이스는 순서가 없고 중복을 허용하지 않는 컬렉션임. 하이버네이트는 `Set`을 구현하기 위해 내부적으로 `HashSet`을 주로 사용함.
- **List + @OrderColumn**: `List`의 순서를 데이터베이스 테이블에 저장하기 위해 사용하는 어노테이션임.
  - 순서 값을 저장할 별도의 컬럼을 매핑하며, 이로 인해 예상치 못한 UPDATE 쿼리가 발생할 수 있어 관리가 까다로움.
  - 중간에 한 요소를 삭제하고, 다른 요소들의 순서 값을 수정하지 않으면 해당 위치에 null이 보관되어 `NullPointerException`이 발생함.
- **@OrderBy**: `@OrderColumn`이 DB에 순서를 저장하는 방식이라면, `@OrderBy`는 DB의 `ORDER BY` 절을 사용하여 컬렉션을 **조회할 때** 정렬하는 방식임. 이는 DB에 순서를 저장하지 않으며 훨씬 실용적임.

## 14.2 @Converter

- 엔티티의 특정 필드 타입을 데이터베이스의 컬럼 타입과 다르게 매핑해야 할 때 사용하는 기능임.
- `AttributeConverter` 인터페이스를 구현하여, 자바 객체(ex.`Boolean`)와 DB 데이터(ex. `Y`, `N` 문자) 간의 변환 로직을 직접 정의할 수 있음.
- `@Converter` 어노테이션을 구현 클래스에, `@Convert` 어노테이션을 엔티티 필드에 적용하여 사용함.
- **글로벌 설정**: `@Converter(autoApply = true)` 옵션을 사용하면, 해당 컨버터가 적용될 수 있는 모든 타입의 필드에 대해 자동으로 컨버터를 적용함.

## 14.3 리스너

- JPA는 엔티티의 생명주기(Lifecycle)에 따라 특정 로직을 수행할 수 있는 이벤트 리스너 기능을 제공함.
- **이벤트 종류**: `PostLoad`(조회 직후), `PrePersist`(저장 전), `PreUpdate`(업데이트 전), `PreRemove`(삭제 전), `PostPersist`(저장 후), `PostUpdate`(업데이트 후), `PostRemove`(삭제 후) 등 다양한 이벤트 콜백이 존재함.
- **이벤트 적용 위치**: 엔티티 클래스 내부에 어노테이션(ex.`@PrePersist`)이 붙은 메소드를 직접 정의하거나, `@EntityListeners` 어노테이션을 통해 외부의 별도 리스너 클래스를 지정하여 적용할 수 있음.

## 14.4 엔티티 그래프

- JPA 2.1부터 도입된 기능으로, 엔티티 조회 시 연관된 엔티티의 페치(Fetch) 전략을 동적으로 결정할 수 있게 함.
- 이는 `FetchType.LAZY`로 설정된 엔티티를 JPQL의 페치 조인(Fetch Join) 없이도 함께 조회할 수 있도록 도와줌.
- **Named 엔티티 그래프**: `@NamedEntityGraph` 어노테이션을 엔티티 클래스에 선언하여, '그래프'의 이름과 함께 조회할 속성(필드)들을 미리 정의해두는 정적 방식임.
- **em.find( )에서 엔티티 그래프 사용**: `em.find()` 메소드 호출 시 `javax.persistence.fetchgraph` 또는 `javax.persistence.loadgraph` 힌트(hint)를 제공하여 정의된 엔티티 그래프를 적용할 수 있음.
    > fetchgraph는 엔티티 그래프에 선택한 속성만 함께 조회하고, loadgraph는 선택한 속성뿐만 아니라 글로벌 fetch 모드가 FetchType.EAGER로 설정된 연관관계도 포함해서 조회함.
- **subgraph**: 엔티티 그래프 내에서 연관된 엔티티의 연관 엔티티까지(ex. `Order` -> `OrderItem` -> `Item`) 정의해야 할 때 `subgraph`를 사용하여 중첩된 그래프를 정의할 수 있음.
- **JPQL에서 엔티티 그래프 사용**: JPQL 쿼리 실행 시에도 `em.find()`와 마찬가지로 힌트를 설정하여 엔티티 그래프를 적용할 수 있음.
- **동적 엔티티 그래프**: `em.createEntityGraph()` API를 사용하여 런타임에 프로그래밍 방식으로 엔티티 그래프를 동적으로 생성하고 적용할 수 있음.
- **엔티티 그래프 정리**: 엔티티 그래프는 `fetchgraph`와 `loadgraph` 두 가지 모드를 제공하며, 페치 조인의 대안으로 사용하여 N+1 문제를 해결하고 특정 조회 로직에 최적화된 페치 전략을 구사할 수 있게 함. 
영속성 컨텍스트에 해당 엔티티가 이미 로딩되어 있으면 엔티티 그래프가 적용되지 않는데, 이는 캐시에 이미 있으므로 쿼리를 따로 진행하지 않기 때문임.