# 프록시

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/83567c8e-23f8-462e-a706-f4ce2043f66d/Untitled.png)

- 멤버를 출력할 때는 팀을 함께 출력했다가 어느날 갑자기 팀을 출력하지 않게된다면 멤버를 호출할 때 팀까지 호출하면 낭비다.

⇒ JPA는 지연로딩, 프록시로 해결한다.

### 프록시 기초

- em.getReference(): 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/7e10071a-0050-484c-8f7a-3908c1866014/Untitled.png)

- id는 파라미터로 존재하므로 조회 시 DB에서 가져오지 않아도 알고 있는 상태
- username을 조회할 때는 username을 호출하는 시점에 jpa가 DB에 쿼리를 날려 findMember에 값을 채운다.
- 프록시 클래스의 형태는
    - 엔티티명$HibernateProxy$odcVHpjy 와 같은 형태이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/7e779886-6635-49a9-b2d4-335bcba308ef/Untitled.png)

### 프록시 특징

- 실제 클래스를 상속 받아서 만들어짐(하이버네이트가 처리)
- 실제 클래스와 겉 모양이 같다.
- 사용하는 입장에서 진짜 객체인지 프록시인지 구분하지 않고 사용하면 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/c597408c-dd1b-4252-92da-f3305230c53b/Untitled.png)

- 프록시는 실제 객체의 참조(target)를 보관
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/5aec412a-d1bc-4847-8603-ece1c1dcfc0d/Untitled.png)

- 영속성 컨텍스트를 통해서 **초기화를 요청한다(진짜 엔티티를 만드는 과정)**
- JPA 표준 스펙에는 존재하지 않는다.

- 프록시 객체는 처음 사용할 때 한번만 초기화
- 프록시 객체를 초기화 할 때, **프록시 객체가 실제 엔티티로 바뀌는 것은 아님**, 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크 시 주의해야함(==비교 실패, 대신 instance of 사용)
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일때, 프록시를 초기화하면 문제 발생.
    - no session → 영속성 컨텍스트가 없다.

# 즉시 로딩과 지연 로딩

### 프록시와 즉시로딩 주의

- 가급적 지연로딩만 사용
- 즉시 로딩을 사용하면 예상치 못한 SQL이 발생
- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
- @ManyToOne, @OneToOne은 기본이 즉시 로딩
    - → LAZY로 설정
- @OneToMany, @ManyToMany는 기본이 지연 로딩

### 지연 로딩 활용

- 이론적인 부분은 자주 사용한다면 즉시로딩.
- 실무는 무조건 지연로딩이 기본

# 영속성 전이(CASCADE)와 고아객체

### 영속성 전이: 저장

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/e7f9248c-d810-4111-a4f7-a1287c2aff84/Untitled.png)

### 영속성 전이: CASCADE - 주의!

- 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없다.
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐

### CASCADE 종류

- **ALL: (영속+ 삭제)**
- **PERSIST: 영속**
- **REMOVE: 삭제**

<aside>
💡 하나의 부모가 오직 한 곳의 자식들을 관리하는 구조일 경우 쓸 수 있다.
예를들어 첨부파일, 게시판
단일 엔티티에 종속적일때(라이프사이클이 같을 때)

</aside>

### 고아 객체

- 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
- orphanRemoval = true
- Parent parent1 = em.find(Parent.class, id);
- parent1.getChilderen().remove(0); //자식 엔티티를 컬렉션에서 제거
- DELETE FROM CHILD WHERE ID = ?

### 고아 객체 - 주의

- 참조하는 곳이 하나일 때 사용해야함!
- **특정 엔티티가 개인 소유할 때 사용!**
- @OneToMany @OneToOne만 가능
- CASCADE.REMOVE 처럼 작동