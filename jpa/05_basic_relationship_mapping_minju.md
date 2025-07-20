# 5. 연관관계 매핑 기초

- **방향(Direction)**: 단방향, 양방향
- **다중성(Multiplicity)**: 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)
- **연관관계의 주인(owner)**: 객체를 양방향 연관관계로 만들면 정해야 함.

## 5.1 단방향 연관관계

### 객체 연관관계와 테이블 연관관계 차이점

- **객체**의 연관관계는 **참조를 통한 단방향**임. 양방향으로 만들려면 반대쪽에도 필드를 추가해서 참조를 보관해야 함. 정확히는 서로 다른 단방향 관계 2개임.

- **테이블**의 연관관계는 **외래키를 사용한 양방향**임.

- **객체**는 **참조**(주소)로 연관관계를 맺고 **테이블**은 **외래키**로 연관관계를 맺음. 연관된 데이터를 조회할 때 객체는 참조를 사용하지만 테이블은 조인을 사용함.

```java
// 단방향 연관관계
class A {
    B b;
}
class B {}

// 양방향 연관관계
class A {
    B b;
}
class B {
    A a;
}
```

### 5.1.1 순수한 객체 연관관계

- **객체 그래프 탐색**: 객체는 참조를 탐색해서 연관관계를 탐색할 수 있음.

```java
Team findTeam = member1.getTeam();
```

### 5.1.2 테이블 연관관계

- **조인**: 데이터베이스는 외래키를 사용해서 연관관계를 탐색할 수 있음.

```sql
SELECT T.*
FROM MEMBER M
    JOIN TEAM T ON M.TEAM_ID = T.ID
WHERE M.MEMBER_ID = 'member1'
```

### 5.1.3 객체 관계 매핑

- **객체 연관관계**: 회원 객체의 Member.team 필드 사용
- **테이블 연관관계**: 회원 테이블의 MEMBER.TEAM_ID 외래키 칼럼 사용

```java
// 연관관계 매핑
@ManyToOne	// 다대일 관계
@JoinColumn(name="TEAM_ID")	// 외래키를 매핑할 때 사용, 생략 가능
private Team team;
```

> **참고**: 다대일(@ManyToOne)과 비슷한 일대일(@OneToOne) 관계도 있음. 반대편이 일대다 관계면 다대일을 사용하고, 반대편이 일대일 관계면 일대일을 사용함.

## 5.2 연관관계 사용

### 5.2.1 저장

JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 함.

```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1);	// 회원1 -> 팀1 참조
em.persist(member1);	// 저장
```

### 5.2.2 조회

1. 객체 그래프 탐색(객체 연관관계를 사용한 조회)
2. 객체지향 쿼리 사용(JPQL)

```java
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();	// 객체 그래프 탐색
```

```java
String jpql = "select m from Member m join m.team t where " + "t.name=:teamName";

List<Member> resultList = em.createQuery(jpql, Member.class)
    .setParameter("teamName", "팀1");
    .getResultList();
```

### 5.2.3 수정

수정은 `em.update()` 같은 메소드가 없음. 엔티티의 값을 변경하면 트랜잭션을 커밋할 때 플러시가 일어나면서 변경 감지 기능이 작동하고, 변경사항을 데이터베이스에 자동 반영함.

```java
Team team2 = new Team("team2", "팀2");
em.persist(team2);

// 회원1에 새로운 팀2 설정
Member member = em.find(Member.class, "member1");
member.setTeam(team2);
```

### 5.2.4 연관관계 제거

```java
Member member1 = em.find(Member.class, "member1");
member1.setTeam(null);	// 연관관계 제거
```

### 5.2.5 연관된 엔티티 삭제

연관된 엔티티를 삭제하려면 기존에 있던 연관관계를 먼저 제거하고 삭제해야 함. 그렇지 않으면 외래키 제약조건으로 데이터베이스에서 오류가 발생함.

```java
member1.setTeam(null);	// 회원1 연관관계 제거
member2.setTeam(null);	// 회원2 연관관계 제거
em.remove(team);	// 팀 삭제
```

## 5.3 양방향 연관관계

- **객체 연관관계**: 일대다 관계는 여러 건과 연관관계를 맺을 수 있으므로 **컬렉션**을 사용해야 함.
- **테이블 연관관계**: 데이터베이스 테이블은 외래키 하나로 양방향으로 조회할 수 있음.

### 5.3.1 양방향 연관관계 매핑

```java
@Entity
public class Member {
    
    // ...
    
    // 연관관계 설정
    public void setTeam(Team team) {
        this.team = team;
    }
    
    // Getter, Setter...
}

@Entity
public class Team {
    
    // ...
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
    
    // Getter, Setter...
}
```

### 5.3.2 일대다 컬렉션 조회

```java
Team team = em.find(Team.class, "team1");
List<Member> members = team.getMembers();	// (팀 -> 회원) 객체 그래프 탐색
```

## 5.4 연관관계의 주인

- 객체에는 양방향 연관관계가 없고, 서로 다른 **단방향 연관관계 2개**를 양방향인 것처럼 보이게 하는 것임. 테이블은 외래키 하나로 두 테이블의 연관관계를 관리함.

- 엔티티를 단방향으로 매핑하면 참조를 하나만 사용하므로 이 참조로 외래키를 관리함. 엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래키는 하나이므로 차이가 발생함. 따라서 JPA는 **연관관계의 주인**을 정해서 테이블의 외래키를 관리함.

### 5.4.1 양방향 매핑의 규칙: 연관관계의 주인

- 연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래키를 **관리**(등록, 수정, 삭제)할 수 있음. 주인이 아닌 쪽은 **읽기**만 할 수 있음.
    - 주인은 `mappedBy` 속성을 사용하지 않음.
    - 주인이 아니면 `mappedBy` 속성을 사용해서 속성의 값으로 연관관계의 주인을 지정해야 함.

### 5.4.2 연관관계의 주인은 외래키가 있는 곳

- 연관관계의 주인은 테이블에 외래키가 있는 곳으로 정함. 예를 들어 회원 테이블이 외래키를 가지고 있으므로 `Member.team`이 주인이 되고, `Team.members`에는 `mappedBy="team"` 속성을 사용해서 주인이 아님을 설정함.

- 데이터베이스 테이블의 다대일, 일대다 관계에서는 항상 다 쪽이 외래키를 가짐. 따라서 `@ManyToOne`에는 `mappedBy` 속성이 없음.

## 5.5 양방향 연관관계 저장

```java
// 팀1 저장
Team team1 = new Team("team1", "팀1");
em.persist(team1);

// 회원1 저장
Member member1 = new Member("member1", "회원1");
member1.setTeam(team1);	// 연관관계 설정
em.persist(member1);

// 회원2 저장
Member member2 = new Member("member2", "회원2");
member2.setTeam(team1);	// 연관관계 설정
em.persist(member2);
```

TEAM_ID 외래키에 팀의 기본키 값이 저장되어 있음. 양방향 연관관계는 연관관계의 주인이 외래키를 관리함.

```java
team1.getMembers().add(member1);	// 무시(연관관계의 주인X)
team1.getMembers().add(member2);	// 무시(연관관계의 주인X)

member1.setTeam(team1);		// 연관관계 설정(연관관계의 주인O)
member2.setTeam(team1);		// 연관관계 설정(연관관계의 주인O)
```

## 5.6 양방향 연관관계의 주의점

**연관관계의 주인에는 값을 입력하지 않고, 주인이 아닌 곳에 값을 입력**하는 실수를 조심해야 함. `team1.getMembers().add(member1);`처럼 하면 TEAM_ID에 null 값이 입력될 수 있음.

### 5.6.1 순수한 객체까지 고려한 양방향 연관관계

객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전함. JPA를 사용하지 않는 순수한 객체 상태에서 문제가 발생할 수 있기 때문임.

### 5.6.2 연관관계 편의 메소드

양방향 관계에서 두 코드는 하나인 것처럼 사용하는 것이 안전함.

```java
public void setTeam(Team team) {	// 메소드 하나로 양방향 관계 모두 설정
    this.team = team;
    team.getMembers().add(this);
}
```

### 5.6.3 연관관계 편의 메소드 작성 시 주의사항

연관관계를 변경할 때는 기존 팀이 있으면 기존 팀과 회원의 연관관계를 삭제하는 코드를 추가해야 함.

```java
public void setTeam(Team team) {

    // 기존 팀과 관계 제거
    if (this.team != null) {
        this.team.getMembers().remove(this);
    }
    this.team = team;
    team.getMembers().add(this);
}
```

## 5.7 정리

- **단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료**되었음.
- **단방향을 양방향으로 만들면 반대 방향으로 객체 그래프 탐색 기능이 추가**됨.
- **양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리**해야 함.