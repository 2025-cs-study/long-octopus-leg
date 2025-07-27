# 5. 연관관계 매핑 기초

## 5.1 단방향 연관관계

### 객체 연관관계와 테이블 연관관계의 차이점
- 객체의 참조(주소)를 통한 연관관계는 언제나 단방향임
- 객체 간에 양방향 연관관계를 만들려면 반대쪽에도 필드를 추가해서 참조를 보관해야 함
- 이는 서로 다른 단방향 관계 2개라고 할 수 있음
- 반면 테이블은 외래키 하나로 양방향 조인이 가능함 (A JOIN B 가능하면 B JOIN A도 가능함)

### 5.1.1 순수한 객체 연관관계
- **객체 그래프 탐색**: 객체는 참조를 사용해서 연관관계를 탐색할 수 있음
- 예시: `Team findTeam = member1.getTeam();`

### 5.1.2 테이블 연관관계
- **조인**: 데이터베이스에서 외래 키를 사용해서 연관관계를 탐색할 수 있음
- 예시: `SELECT T.* FROM MEMBER M JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID WHERE M.MEMBER_ID = 'member1'`

### 5.1.3 객체 관계 매핑
```java
@Entity
public class Member {
    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name ="TEAM_ID")
    private Team team;
}

@Entity
public class Team {
    @Id
    @Column(name = "TEAM_ID")
    private String id;
    
    private String name;
}
```

- **객체 연관관계**: 회원 객체의 `Member.team` 필드 사용함
- **테이블 연관관계**: 회원 테이블의 `MEMBER.TEAM_ID` 외래 키 칼럼을 사용함
- `Member.team`과 `MEMBER.TEAM_ID`를 매핑하는 것이 연관관계 매핑임
- **@ManyToOne**: 다대일(N:1) 관계라는 매핑 정보를 나타냄
- **@JoinColumn**: 외래 키를 매핑할 때 사용함, 생략 가능함 (생략하면 `team_TEAM_ID` 외래키를 사용함)

### 5.1.4 @JoinColumn 주요 속성
- **name**: 매핑할 외래 키 이름을 지정함
- **referencedColumnName**: 외래 키가 참조하는 대상 테이블의 컬럼명을 지정함
- **foreignKey(DDL)**: 외래 키 제약 조건을 직접 지정할 수 있음 (테이블 생성 시에만 사용함)

### 5.1.5 @ManyToOne 주요 속성
- **fetch**: 글로벌 페치 전략을 설정함 (@ManyToOne 기본값은 FetchType.EAGER, @OneToMany 기본값은 FetchType.LAZY임)
- **cascade**: 영속성 전이 기능을 사용함

## 5.2 연관관계 사용

### 5.2.1 저장
```java
member1.setTeam(team1);
em.persist(member1);
```
- JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 함

### 5.2.2 조회 방법
#### 연관관계가 있는 엔티티를 조회하는 두 가지 방법
1. **객체 그래프 탐색**
    - `member.getTeam()`을 사용해서 member와 연관된 team 엔티티를 조회할 수 있음
    - 객체를 통해 연관된 엔티티를 조회함

2. **객체지향 쿼리(JPQL) 사용**
    - JPQL도 조인을 지원함
    - 예시: `select m from Member m join m.team t where t.name=:teamName`
    - `:teamName`과 같이 `:`로 시작하는 것은 파라미터를 바인딩받는 문법임

### 5.2.3 수정
- 단순히 불러온 엔티티의 값만 변경해두면 트랜잭션을 커밋할 때 플러시가 일어나면서 변경 감지 기능이 작동함
- 변경사항을 데이터베이스에 자동으로 반영함
- 연관관계를 수정할 때도 마찬가지로, 참조하는 대상만 변경하면 나머지는 JPA가 자동으로 처리함

### 5.2.4 연관관계 제거
- `member1.setTeam(null);`로 연관관계를 null로 설정함

### 5.2.5 연관된 엔티티 삭제
- 연관된 엔티티를 삭제하려면 기존에 있던 연관관계를 먼저 제거하고 삭제해야 함
- 그렇지 않으면 외래 키 제약조건으로 인해 데이터베이스에서 오류가 발생함
```java
member1.setTeam(null);
member2.setTeam(null);
em.remove(team);
```

## 5.3 양방향 연관관계

### 양방향 연관관계의 특징
- 회원에서 팀으로 접근하고, 팀에서도 회원으로 접근할 수 있도록 함
- 회원과 팀은 다대일 관계, 팀에서 회원은 일대다 관계임
- 일대다 관계는 여러 건과 연관관계를 맺을 수 있으므로 컬렉션을 사용해야 함
- **회원 → 팀**: `Member.team`
- **팀 → 회원**: `Team.members` (List 컬렉션)

### 5.3.1 양방향 연관관계 매핑
```java
@Entity
public class Member {
    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}

@Entity
public class Team {
    @Id
    @Column(name = "TEAM_ID")
    private String id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
}
```

- 팀과 회원은 일대다 관계여서 `List<Member> members`를 추가함
- 일대다 관계를 매핑하기 위해 `@OneToMany` 매핑 정보를 사용함
- `mappedBy` 속성은 양방향 매핑일 때 사용하는데 반대쪽 매핑의 필드 이름을 값으로 주면 됨
- 반대쪽 매핑이 `Member.team`이므로 `team`을 값으로 줌

### 5.3.2 컬렉션 조회
- **객체 그래프 탐색**: `List<Member> members = team.getMembers();`

## 5.4 연관관계의 주인

### 연관관계의 주인이 필요한 이유
- 객체에는 양방향 연관관계라는 것이 없음
- 서로 다른 단방향 연관관계 2개를 양방향처럼 보이게 하는 것임
- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리함
- 엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나임
- 이 차이로 JPA에서는 두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 함
- 이것을 **연관관계의 주인**이라고 함

### 5.4.1 양방향 매핑의 규칙: 연관관계의 주인
- 연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있음
- 반면에 주인이 아닌 쪽은 읽기만 할 수 있음
- 어떤 연관관계를 주인으로 정할지는 `mappedBy` 속성을 사용하면 됨
- **주인**: `mappedBy` 속성을 사용하지 않음
- **주인이 아님**: `mappedBy` 속성을 사용해서 속성의 값으로 연관관계의 주인을 지정해야 함

### 5.4.2 연관관계의 주인은 외래 키가 있는 곳
- 연관관계의 주인은 테이블에 외래 키가 있는 곳으로 정해야 함
- 회원 테이블이 외래 키를 가지고 있으면 `Member.team`이 주인이 됨
- 주인이 아닌 `Team.members`에는 `mappedBy="team"` 속성을 사용해서 주인이 아님을 설정함
- 데이터베이스 테이블의 다대일, 일대다 관계에서는 항상 다 쪽이 외래 키를 가짐
- 따라서 `@ManyToOne`에는 `mappedBy` 속성이 없음

## 5.5 양방향 연관관계 저장
- 주인이 아닌 방향은 값을 설정하지 않아도 데이터베이스에 외래 키 값이 정상 입력됨

## 5.6 양방향 연관관계의 주의점

### 잘못된 예시
```java
team1.getMembers().add(member1);
team1.getMembers().add(member2);
```
- 연관관계의 주인에는 값을 입력하지 않고, 주인이 아닌 곳에만 값을 입력하면 외래 키 값이 정상적으로 저장되지 않음
- 연관관계의 주인만이 외래 키의 값을 변경할 수 있음

### 5.6.1 순수한 객체까지 고려한 양방향 연관관계
- 연관관계의 주인에만 값을 저장하고 주인이 아닌 곳에는 값을 저장하지 않아도 될까?
- 객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전함
- 객체까지 고려하면 양쪽 다 관계를 맺어줘야 함

```java
public void test() {
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);
    
    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1);
    team1.getMembers().add(member1);
    em.persist(member1);

    Member member2 = new Member("member2", "회원2");
    member2.setTeam(team1);
    team1.getMembers().add(member2);
    em.persist(member2);
}
```

- 순수한 객체 상태에서도 동작하고, 테이블의 외래 키도 정상 입력됨
- 객체의 양방향 연관관계는 양쪽 모두 관계를 맺어주자

### 5.6.2 연관관계 편의 메소드
- `member.setTeam(team)`과 `team.getMembers().add(member)`를 각각 호출하다 보면 실수로 둘 중 하나만 호출해서 양방향이 깨질 수 있음
- 그래서 두 코드를 하나인 것처럼 사용하는 것이 안전함

```java
// 기존 코드
public void setTeam(Team team) {
    this.team = team;
}

// 개선된 코드
public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

- 위와 같이 `setTeam()` 메소드 하나로 양방향 관계를 모두 설정하도록 변경함
- 이렇게 한번에 양방향 관계를 설정하는 메소드를 **연관관계 편의 메소드**라고 함

### 5.6.3 연관관계 편의 메소드 작성 시 주의사항
```java
member1.setTeam(teamA);
member1.setTeam(teamB);
Member findMember = teamA.getMember();
```

- 위의 코드는 teamB로 변경될 때 teamA → member1 관계가 제거되지 않았다는 문제가 있음
- 연관관계를 변경할 때는 기존 팀이 있으면 기존 팀과 회원의 연관관계를 삭제하는 코드를 추가해야 함

```java
public void setTeam(Team team) {
    if(this.team != null) {
        this.team.getMembers().remove(this);
    }
    this.team = team;
    team.getMembers().add(this);
}
```

### 추가 주의사항
- 양방향 매핑시에는 무한 루프에 빠지지 않도록 조심해야 함
- 무한 루프를 방지하기 위해 `@JsonIgnore`를 사용할 수 있음