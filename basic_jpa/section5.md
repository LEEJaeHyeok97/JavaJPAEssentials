## 영속성 컨텍스트 1

### JPA에서 가장 중요한 2가지

- 객체와 관계형 데이터베이스 매핑하기
- 영속성 컨텍스트

### 엔티티 매니저 팩토리와 엔티티 매니저

- 엔티티 매니저 팩토리는 고객의 요청이 올 때마다 엔티티매니저를 각각의 요청마다 생성을 한다.
- 엔티티 매니저는 내부적으로 데이터베이스 커넥션을 통해 db를 사용하게 된다.

### 영속성 컨텍스트

- JPA를 이해하는데 가장 중요한 용어
- “엔티티를 영구 저장하는 환경”이라는 뜻
- EntityManager.persist(entity);
    - persist 메소드는 DB에 저장하는게 아니라 ‘영속성 컨텍스트’에 저장한다.

### 엔티티 매니저? 영속성 컨텍스트?

- 영속성 컨텍스트는 논리적인 개념
- 눈에 보이지 않는다.
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근

### 엔티티의 생명주기

- 비영속(new/transient)
    - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속(managed)
    - 영속성 컨텍스트에 관리되는 상태
- 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed)
    - 삭제된 상태

### 비영속 상태

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/1cd99398-7f60-4f6e-9c5a-f48e5571549d/Untitled.png)

```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```

### 영속 상태

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/8999ef99-8f4c-4cd0-91ec-513bcabb17c9/Untitled.png)

```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

//객체를 저장한 상태(영속)
em.persist(member);
```

### 준영속, 삭제

```java
//회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
em.detach(member);

//객체를 삭제한 상태
em.remove(member);
```

=== 영속성 컨텍스트로 얻게되는 이점 ===

- 어플리케이션과 DB사이에 영속성 컨텍스트가 존재함으로써 얻어지는 이점이 있다
    - 버퍼링, 캐싱이 가능해진다.

## 영속성 컨텍스트2

### 엔티티 조회, 1차 캐시

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/5dfb0dfd-544f-4a19-8ce2-34124b951272/Untitled.png)

```java
//엔티티를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

//엔티티를 영속
em.persist(member);
```

### 1차 캐시에서 조회

```java
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

//1차 캐시에 저장됨
em.persist(member);

//1차 캐시에서 조회
Member findMember = em.find(Member.class, "member1");
```

### 데이터베이스에서 조회

```java
Member findMember2 = em.find(Member.class, "member2");
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/94c4507d-0f19-4b44-92de-50b8b80a820c/Untitled.png)

- member2가 1차 캐시에 없으면 DB에서 조회를 한다.
- member2를 1차 캐시에 저장 후 1차 캐시에 있는 member2를 반환한다.
- 영속성 컨텍스트는 트랜잭션 단위로 만들고 트랜잭션이 끝날때 종료시킨다.
    - 1차 캐시도 다 날아간다.⇒ 찰나의 순간에 이득이 있다.
- 비즈니스 로직이 아주 복잡한 경우가 아니라면 성능에 이점을 얻기는 어렵다.

<aside>
💡 2차캐시: 어플리케이션 전체에서 공유하는 캐시

</aside>

### 영속 엔티티의 동일성 보장

```java
//영속
Member.findMember1 = em.find(Member.class, 101L);
Member.findMember2 = em.find(Member.class, 101L);

System.out.println("result = " + (findMember1 == findMember2));
// true가 보장된다. 1차 캐시가 똑같은 것처럼 해준다.
```

<aside>
💡 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

</aside>

### 엔티티 등록: 트랜잭션을 지원하는 쓰기 지연

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/4cdba6c7-6d58-471b-a17e-5a58bf0a5a77/Untitled.png)

- 영속성 컨텍스트 안에는 1차 캐시 뿐만 아니라 **쓰기 지연 SQL 저장소**가 있다.

### 엔티티 수정. 변경 감지

- JPA는 자바 컬렉션 쓰듯이 쓴다.
    - set을 하고 다시 persist 하지 않는다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/46d45e36-ebed-45db-8f78-4050d2da1c97/Untitled.png)

- JPA는 커밋 시 flush가 실행된다.
- 스냅샷에 1차 캐시에 들어온 상태를 떠놓고 값을 변경할 때 커밋 시점에 flush가 내부적으로 실행되고 내부적으로 모든 데이터를 비교 후 쓰기 지연 SQL 저장소에 저장을 하고 DB에 커밋을 수행한다.

### 엔티티 삭제

```java
//삭제 대상 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

em.remove(memberA); //엔티티 삭제
```

*플러시: 영속성 컨텍스트의 변경내용을 데이터베이스에 반영

ex)

```java
if(member.getName().equals("ZZZ") {
	em.update(member);
}

// 위는 잘못된 코드 예시
```

## 플러시

- 데이터베이스 transaction이 커밋될 때 flush가 일어난다.
- 영속성 컨텍스트 내의 변경 사항과 DB의 내용을 일치시키는 작업이다.

### 플러시 발생

- 변경 감지(더티 체킹)
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송(등록, 수정, 삭제 쿼리)

<aside>
💡 플러시가 발생한다고 해서 transaction이 커밋되는 것은 아니고 transaction.commit() 실행 시 커밋된다.

</aside>

### 영속성 컨텍스트를 플러시하는 방법

- em.flush() - 직접 호출
- 트랜잭션 커밋 - 플러시 자동 호출
- JPQL 쿼리 실행 - 플러시 자동 호출

```java
Member member = new Member(200L, "member200");
em.persist(member);

em.flush(); // 커밋 되기 전에 플러시 하고 싶다면 직접 호출한다.

tx.commit();
```

*플러시 한다고 1차 캐시가 지워지거나 하진 않는다. 오직, 쓰기 지연 SQL 저장소의 변경사항이 DB에 반영되는 것 뿐이다.

- 커밋 직전에만 플러시 하면된다.

## 준영속 상태

- 영속 → 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
    - 더티 체킹 등이 사용 불가

```java
//영속
Member member = em.find(Member.class, 150L);
member.setName("AAAAA");

em.detach(member); // JPA가 더이상 관리하지 않는다.

Member member = em.find(Member.class, 150L); // 1차 캐시에 정보가 없으므로 쿼리문이 새로 날아가 DB에서 데이터를 1차 캐시에 저장
```

- em.detach(entity)
    - 특정 엔티티만 준영속 상태로 전환
- em.clear()
    - 영속성 컨텍스트를 완전히 초기화
- em.close()
    - 영속성 컨텍스트를 종료