# JPA(Java Persistence API)

JPA(Java Persistence API)는 자바에서 관계형 데이터베이스와 객체 지향 프로그래밍 언어 간의 패러다임 불일치를 해결하기 위한 ORM(Object-Relational Mapping) 표준 기술입니다.

객체와 테이블(릴레이션)을 매핑한다 라는 의미입니다.

실제 구현체로는 Hibernate, EclipseLink, OpenJPA 등이 있으며, 이 중 Hibernate가 가장 널리 사용됩니다. SQL에 의존하지 않고 DAO, JDBC 코드를 작성하는 시간을 줄여줍니다.

## 영속성 컨텍스트
- JPA의 핵심 개념 중 하나로, 엔티티 객체들을 관리하는 논리적인 공간입니다.
- 1차 캐시 역할을 하며, 동일한 트랜잭션 내에서 같은 식별자를 가진 엔티티는 동일한 객체임을 보장합니다.

## 지연 로딩
- 연관된 엔티티나 컬렉션을 실제로 사용하는 시점에 데이터베이스에서 조회하는 기능입니다.
- 성능 최적화에 도움이 됩니다.

## 핵심 구성 요소

- **Entity**: `@Entity` 어노테이션이 붙은 클래스로, 데이터베이스 테이블과 매핑되는 자바 객체입니다.
- **EntityManager**: 엔티티의 생명주기를 관리하고(한마디로 CRUD를 담당하는) 데이터베이스와의 상호작용을 담당하는 인터페이스입니다. `persist`, `find`, `merge`, `remove` 등의 메서드를 제공합니다.
- **Persistence Context**: 엔티티 인스턴스들이 관리되는 환경으로, 엔티티의 상태 변화를 추적하고 데이터베이스와 동기화를 담당합니다. -> 엔티티를 관리하는 1차 캐시

## 엔티티 생명주기

JPA 엔티티는 네 가지 상태를 가집니다:

1. **비영속(New/Transient)**: 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태입니다.
2. **영속(Managed)**: 영속성 컨텍스트에 관리되는 상태로, 데이터베이스와 동기화됩니다.
3. **준영속(Detached)**: 영속성 컨텍스트에 저장되었다가 분리된 상태입니다.
4. **삭제(Removed)**: 삭제하기로 결정된 상태입니다.

## 주요 어노테이션

- `@Entity`: 클래스를 엔티티로 지정합니다.
- `@Table`: 엔티티와 매핑할 테이블을 지정합니다.
- `@Id`: 기본 키를 지정합니다.
- `@GeneratedValue`: 기본 키 생성 전략을 지정합니다.
- `@Column`: 필드와 컬럼을 매핑합니다.
- `@OneToMany`, `@ManyToOne`, `@OneToOne`, `@ManyToMany`: 연관관계를 매핑합니다.

## 장점과 단점

**장점**:
- 생산성 향상
- 객체 지향적 개발 가능
- 표준화된 API
- Dialect(방언) 추상화된 인터페이스를 제공하여 특정 DB에 종속되지 않게 만들어준다

**단점**:
- 복잡한 쿼리 처리의 어려움
- 성능 튜닝의 복잡성
- 학습 곡선의 존재

## N+1 문제

JPA를 사용할 때 가장 주의해야하는 성능적 문제는 **N+1 문제**입니다.

N+1 문제는 하나의 쿼리로 부모 엔티티를 조회할 때 각 부모 엔티티의 연관된 자식 엔티티까지 N번의 쿼리로 조회하여 총 N+1번의 쿼리가 실행되는 성능 저하 문제입니다.

하나의 초기 쿼리로 특정 엔터티 목록을 조회한 후 해당 엔티티가 연관된 다른 엔티티를 참조할 때마다 N번의 추가 쿼리가 실행되는 문제!!

즉시로딩은 권장되지 않음.. 바로 문제 발생  
지연로딩도 연관 엔티티를 조회하는 시점에 문제 발생

그래서 `fetch join` 이나 `@EntityGraph` 방식을 제공하고 있슴

모든 연관관계에서 N+1 문제가 발생하는 것은 아닙니다. N+1 문제가 발생하는 대표적인 상황들을 정리하면 다음과 같습니다.

- 지연 로딩된 컬렉션 또는 엔티티 접근
- 즉시 로딩 시 JOIN이 아닌 SELECT로 접근하게 되는 경우

## Fetch Join

N+1 문제를 해결하는 가장 대표적인 방식이 Fetch Join (페치 조인)입니다.

페치 조인은 JOIN을 사용해서 연관된 엔티티를 한 번에 조회하는 방식입니다.
```java
em.createQuery("SELECT DISTINCT m FROM Member m JOIN FETCH m.orders", Member.class)
 .getResult();
```
일대다 관계를 조인하는 경우 중복 데이터 조회가 발생하게 되므로 반드시 DISTINCT 키워드를 사용해서 중복 데이터를 제거하는 것이 좋습니다.

페이징과 하이버네이트의 @BatchSize
페치 조인은 가장 간단하게 N+1 문제를 해결할 수 있는 방법이지만 만약 페이징을 사용하고 있다면 사용에 주의해야합니다.

다음 두 가지 문제로 인해 페치 조인을 페이징에 사용하는 것은 권장되지 않습니다.

일대다 관계에서 중복 데이터가 발생

중복 제거 후 결과를 메모리에서 페이징하여 DB에서 LIMIT/OFFSET 적용이 되지 않아 데이터 다량 조회 시 성능 저하

그래서 지연 로딩이나 페이징 상황에서는 하이버네이트에서 제공하는 @BatchSize 기능을 사용하여 N+1 문제를 해결하거나 페이징을 시도합니다.

@BatchSize는 하이버네이트 구현체에서만 동작하는 기능입니다.
```java
@BatchSize(size = 10)
@OneToMany(fetch = FetchType.Lazy)
private List<Order> orders = new ArrayList<>();
@BatchSize는 IN 쿼리를 사용해서 하나의 쿼리로 조회를 수행하게 만들어줍니다.
```
```sql
SELECT * FROM orders_table WHERE member_id IN (?, ?, ?, ..., ?)
```
위와같은 식으로 하나의 SELECT 쿼리를 이용해서 최대 size 개의 데이터를 조회하게 됩니다.

예시는 일대다 관계지만 일대일, 다대일에서도 사용할 수 있습니다.

만약 하이버네이트 구현체를 사용하지 않는다면 DTO를 만들어서 쿼리 결과를 DTO로 받아내는 방식을 사용합니다. 이 방법은 JPQL의 NEW를 참조해주세요.

@EntityGraph
@EntityGraph는 Spring Data JPA가 제공하는 N+1 문제 해결 방법입니다.

다음과 같이 Repository 인터페이스의 쿼리 메소드에 @EntityGraph를 붙여서 사용합니다.

```java
@EntityGraph(attributePaths = "orders") //attributePaths에는 엔티티 클래스 필드명
List<Member> findAll();
```
하이버네이트의 @Fetch
만약 JPA 구현체로 하이버네이트를 사용한다면 @Fetch를 사용하는 것도 하나의 방법입니다. @Fetch는 데이터 조회 시 부속 질의(서브 쿼리)를 사용하여 N+1 문제를 해결합니다.

@Fetch의 사용법은 다음과 같습니다.

```java
import org.hibernate.annotations.Fetch;

@Fetch(FetchMode.SUBSELECT)
@OneToMany(fetch = FetchType.Lazy)
private List<Order> orders = new ArrayList<>();
```
@Fetch는 즉시 로딩에서는 조회 시점, 지연 로딩에서는 연관된 엔티티(지연된 엔티티)를 사용하는 시점에 부속 질의가 수행되게 됩니다.

방법 정리
N+1 문제를 해결하는 방법을 정리하면 다음과 같습니다.

방법	상황
Fetch Join	순수 JPA 사용, 일반적인 일대일/다대일 연관관계
@EntityGraph	Spring Data JPA 사용
하이버네이트 구현체: @EntityGraph, @BatchSize
그 외: DTO 조회	다대일 페이징
최종 정리하면 가능한 지연 로딩 방식을 이용하고 페치 조인 or @EntityGraph를 이용해서 N+1 문제를 해결하여 최적화를 하는 것이 좋습니다.

Spring Data JPA라면 @EntityGraph를 우선 사용하고 복잡한 조회 쿼리가 있는 부분에선 JPQL에 페치 조인을 사용하는 것이 좋습니다.


## ✅ 도메인 설계를 어떻게 했을 때 발생할까?

- 연관관계가 **지연 로딩 (LAZY)** 으로 설정된 경우 (JPA의 기본)
- 연관 엔티티를 **반복문 내에서 직접 참조**하는 경우
- **양방향 연관관계에서 mappedBy를 기준으로 잘못 탐색**할 경우
- 컬렉션을 순회하며 지연 로딩 필드를 꺼내 쓸 경우
### 예시:

```java
@Entity
class Order {
    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;

```

`Order`를 조회한 뒤 `order.getMember().getName()`을 반복문에서 접근하면,

각각의 `Member`에 대해 개별 쿼리가 나감 → N 쿼리 발생

---

## ✅ 3. 해결 방법

| 방법 | 설명 |
| --- | --- |
| **Fetch Join** | JPQL에서 join fetch 사용해 연관 엔티티 한 번에 조회→ `N` 쿼리 방지 |
| **EntityGraph** | JPA의 @EntityGraph 어노테이션을 사용해 필요한 연관 엔티티를 함께 조회 |
| **Batch Size 설정** | 지연 로딩을 유지하면서 `IN` 쿼리로 일괄 조회(컬렉션이나 다대일 관계에 유용) |
| **DTO 직접 조회** | 필요한 데이터만 JPQL/QueryDSL로 DTO에 바로 조회하여 불필요한 연관관계 제거 |

### Fetch Join 예시:

```java
@Query("SELECT o FROM Order o JOIN FETCH o.member")
List<Order> findAllWithMember();

```

### EntityGraph 예시:

```java
@EntityGraph(attributePaths = {"member"})
List<Order> findAll();

```
