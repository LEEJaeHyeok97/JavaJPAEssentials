# 소개

### JPA는 다양한 쿼리 방법을 지원

- JPQL
    - 실무에서 대부분 문제 해결이 가능하다
- QueryDSL
- jpa criteria
- 네이티브 sql
- jdbc api 직접 사용, mybatis, sprintjdbctemplate 함께 사용

### JPQL 소개

- 가장 단순한 조회 방법
    - EntityManager.find()
    - 객체 그래프 탐색 a.getB().getC()
- 나이가 18살 이상인 회원을 모두 검색하고 싶다면?

### JPQL

- JPA를 사용하면 엔티티 객체를 중심으로 개발
- 문제는 검색 쿼리
- 검색을 할 때도 **테이블이 아닌 엔티티 객체를 대상으로 검색**
- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요

- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리

- 테이블이 아닌 객체를 대상으로 하는 객체지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존x
- JPQL은 한마디로 객체 지향 SQL

### 크리테리아

- 실무에서 사용하지 않는다.(복잡+실용성이 없다)
- QueryDSL 사용 권장

### 네이티브 SQL 소개

- JPA가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
- 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

### 5프로 정도의 통계성 쿼리: SpringJdbcTemplate

- 직접 sql문으로 쿼리

# 기본 문법과 쿼리 API

### JPQL

- JPQL은 어떤 특정 데이터베이스 SQL에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.

### JPQL 문법

- select m from Member as m where m.age >18
- 엔티티와 속성은 대소문자 구분 O (Member, age)
- JPQL 키워드는 대소문자 구분X (SELECT, FROM, where)
- 엔티티 이름 사용, 테이블 이름이 아님(Member)
- 별칭은 필수(m) (as는 생략가능)

### TypeQuery, Query

- TypedQuery: 반환 타입이 명확할 때 사용
- Query: 반환 타입이 명확하지 않을 때

### 결과 조회

- query.getResultList(): 결과가 하나 이상일 때, 리스트 반환
    - 결과가 없으면 빈 리스트 반환
- query.getSingleResult(): 결과가 정확히 하나, 단일 객체 반환
    - 결과가 없으면: javax.persistence.NoResultException
    - 둘 이상이면: javax.persistence.NonUniqueResultException

# 프로젝션

- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입
- SELECT m FROM Member m → 엔티티 프로젝션
- SELECT [m.team](http://m.team) FROM Member m → 엔티티 프로젝션
- SELECT m.address FROM Member m → 임베디드 타입 프로젝션
- SELECT m.username, m.age FROM Member m → 스칼라 타입 프로젝션
- DISTINCT 사용 가능

### 프로젝션 - 여러 값 조회

- Query 타입으로 조회
- Object[]
- new 명령어로 조회

### 페이징

- 페이징 API
    - JPA는 페이징을 다음 두 API로 추상화
    - setFirstResult(int startPosition): 조회 시작 위
    - setMaxResults(int maxResult): 조회할 데이터 수
    

```java
//페이징 쿼리
String jpql = "select m from Member m order by m.name desc";
List<Member> resultList = em.createQuery(jpql, Member.class)
				.setFirstResult(10)
				.setMaxResults(20)
				.getResultList();
```

### 조인

- 내부 조인
    - 조인 하려는 두 테이블에서 조인하려는 두 부분 중 한 부분만 없어도 데이터를 없앤다.
- 외부 조인
    - 조인하려는 두 테이블에서 두 부분 중 한 부분이 없어도 데이터를 null로 넣고 조인을 한다.
- 세타 조인
    - 카테시안 곱으로 조인

- 조인 - ON절
    - 조인 대상 필터링
    - 연관관계 없는 엔티티 외부 조인

### 서브 쿼리

- 서브쿼리 지원 함수
    - EXIST
    - ALL | ANY | SOME
    - IN
- 예제)
    - 팀 A 소속인 회원
        
        ```java
        select m from Member m
        where exists (select t from m.team t where t.name = '팀A')
        ```
        

### JPA 서브 쿼리 한계

- FROM 절의 서브쿼리는 현재 JPQL에서 불가능
    - 조인으로 풀 수 있으면 풀어서 해결