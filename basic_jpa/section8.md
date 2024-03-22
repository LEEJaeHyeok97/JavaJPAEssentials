# 상속관계 매핑

- 관계형 데이터베이스는 상속관계 X
- 슈퍼타입 서브타입 논리 모델을 실제 구현하는 방법

### 단일 테이블 전략

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/b0a32f87-a9a1-4ebb-b458-0a02b17bc0fa/Untitled.png)

- @Inheritance(strategey=InheritanceType.SINGLE_TABLE)
- 단점
    - 자식 엔티티가 매핑한 컬럼은 모두 null 허용
    - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있어서 조회 성능이 오히려 느려질 수 있다.

### 구현 클래스마다 테이블 전략

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/bb528d96-5371-421f-8803-20f50ee8b96d/Untitled.png)

- name, price를 각각의 테이블이 다 갖는 방식
- 쓰면 안되는 전략이다.
    - 데이터베이스 설계자와 ORM 전문가 둘 다 비추천하는 전략이다.

### 조인 전략

- 장점
    - 테이블 정규화
    - 외래 키 참조 무결성 제약조건 활용가능
    - 저장공간 효율화
- 단점
    - 조회 시 조인을 많이 사용, 성능 저하
    - 조회 쿼리가 복잡함
    - 데이터 저장 시 INSERT sql 2번 호출

→ 조인 전략이 정석이고 객체랑도 잘 맞고 설계 입장에서 좋다.

<aside>
💡 단일테이블 전략이랑 조인 전략 두가지를 트레이드 오프로 고민에 둬야한다.

</aside>

**→ 단순한 것에는 (데이터도 얼마 안될 때) 단일 테이블로 간다.**

**비즈니스적으로 중요하고 복잡하다면 조인 전략으로 간다.**

# Mapped Superclass - 매핑 정보 상속

### @MappedSuperclass

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/112886f4-0c77-4226-911b-79f68894c0f8/Untitled.png)

- 공통 매핑 정보가 필요할 때

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/d585b519-8573-4b79-a92a-7a04a0718b84/a806942d-ece9-4a03-adaa-009577fc38dc/Untitled.png)

- 속성만 상속받아 사용할 수 있다.
- 상속관계 매핑이 아님.
- 엔티티 X, 테이블과 매핑 X
- 자식 클래스에 매핑 정보만 제공
- 조회, 검색불가
- 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
- @Entity나 @MappedSuperclass로 지정한 클래스만 상속할 수 있다