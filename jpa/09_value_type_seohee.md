# 9. 값 타입

## 9.1 기본값 타입
```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;
}
```
- 위의 Member에서 String, int가 값 타입임.
- name, age 속성은 회원 엔티티에 의존함.
- 값 타입은 공유하면 안됨.

## 9.2 임베디드 타입(복합 값 타입)
```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    @Embedded Period workPeriod;    // 근무 기간
    @Embedded Address homeAddress;  // 집 주소
    
}
```
```java
@Embeddable
    public class Period {
    @Temporal (Tempora1Type. DATE) java.util.Date startDate;
    @Temporal (TemporalType. DATE) java.util.Date endDate;
    
    public boolean isWork(Date date) {
    }
}
```
```java
@Embeddable
public class Address {
    
    @Column(name = "city")
    private String city;
    private String street;
    private String zipcode;
}
```
- 새로운 값 타입을 직접 정의해서 사용하는 것임.
- 여기서 근무 기간, 집 주소가 임베디드 타입임.
- 이렇게 새로 정의한 값 타입들은 재사용할 수 있고 응집도도 아주 높음.
- 임베디드 타입은 기본 생성자가 필수임.
- 임베디드 타입을 사용하려면 다음 2가지 어노테이션이 필요하고, 둘 중 하나는 생략해도 됨.
  - @Embeddable : 값 타입을 정의하는 곳에 표시함.
  - @Embedded : 값 타입을 사용하는 곳에 표시함.
- 임베디드 타입은 값 타임을 포함하거나 엔티티를 참조할 수 있음.
- 임베디드 타입을 @AttributeOverride로 재정의할 수 있음.


## 9.3 값 타입과 불변 객체
### 9.3.1 값 타입 공유 참조
- 값을 공유하면 의도하지 않게 각 엔티티의 값이 같이 변경될 수 있음.

### 9.3.2 값 타입 복사
- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험해서 값을 복사해서 사용해야 함.
- 자바는 기본 타입에 값을 대입하면 값을 복사해서 전달함.
- 하지만 객체는 항상 참조값을 전달하기 때문에 인스턴스를 복사해서 대입해야 함.
- 원본의 참조 값을 직접 넘기는 것을 막을 방법이 없어서 setCity()와 같은 수정자 메소드를 전부 제거하는 것이 좋음.

### 9.3.3 불변 객체
- 객체를 불변하면 만들면 값을 수정할 수 없어서 부작용을 차단할 수 있음.
- 값 타입은 될 수 있으면 불변 객체로 설계해야 함.
- 불변 객체를 구현하는 간단한 방법은 생성자로만 값을 설정하고 수정자를 만들지 않으면 됨.
```java
@Embeddable
public class Address {
    
    private String city;

    protected Address() { } // JPA에서 기본 생성자는 필수

    public Address(String city) {
        this.city = city;
    }

    public String getCity() {
        return city;
    }
    // 수정자(Setter)는 만들지 않음.
}
```
- 위와 같이 만들면 값을 수정할 수 없으므로 부작용이 발생하지 않음.
- String, Integer는 대표적인 불변 객체임.

## 9.4 값 타입 비교
- 동일성 비교 : 인스턴스의 참조 값을 비교, == 사용함.
- 동등성 비교 : 인스턴스의 값을 비교, equals() 사용함.

## 9.5 값 타입 컬렉션
- 값 타입을 하나 이상 저장하려면 `@ElementCollection`, `@CollectionTable` 어노테이션을 사용함.
