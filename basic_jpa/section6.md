# 객체와 테이블 매핑

## 엔티티 매핑 소개

### 객체와 테이블 매핑

- @Entity
    - @Entity가 붙은 클래스는 JPA가 관리하는 엔티티다.
    - JPA를 사용해서 테이블과 매핑할 클래스는 @Entity가 필수이다.
    - 주의
        - 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
        - final 클래스, enum, interface, inner 클래스 사용X
        - 저장할 필드에 final 사용X
    
- @Entity 속성 정리
    - 속성: name
        - JPA에서 사용할 엔티티 이름을 지정한다.
        - 기본값: 클래스 이름을 그대로 사용(예: Member)
        - 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.
    - 보통은 기본값을 쓴다.

- @Table
    - 엔티티와 매핑할 테이블 지정한다.

# 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 > 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 생성된 DDL은 개발 장비에서만 사용
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

## 자동 생성 속성

| 옵션 | 설명 |
| --- | --- |
| create | 기존테이블 삭제 후 다시 생성(DROP + CREATE) |
| create-drop | create와 같으나 종료시점에 테이블 DROP |
| update | 변경분만 반영(운영DB에서는 사용하면 안됨) |
| validate | 엔티티와 테이블이 정상 매핑 되어 있는지만 확인 |
| none | 사용하지 않음 |

## 주의점

- **운영장비에는 절대 create, create-drop, update 사용하면 안된다.**
- 개발 초기 단계에는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none

## DDL 생성 기능

- 제약조건 추가: 회원 이름은 필수, 10자 초과X
    - @Column(nullable = false, length = 10)
- 유니크 제약조건 추가
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.

# 필드와 컬럼 매핑

- 큰 데이터를 넣을 때는 @Lob 을 사용한다

## @Column

- name: 필드와 매핑할 테이블의 컬럼 이름(기본값: 객체의 필드 이름)
- length: 문자 길이 제약조건, String 타입에 사용한다.
- etc..

## @Enumerated

- 자바 enum 타입을 매핑할 때 사용
- **ORDINARY 사용X**
    - ORDINARY를 쓰면 ENUM 타입에 값이 추가될 때 기존 값이 변경이 안되므로 운영 상황에서 심각한 문제가 발생한다. → String으로 문자열을 저장하도록 한다.

| 속성 | 설명 | 기본값 |
| --- | --- | --- |
| value | - EnumType.ORDINARY: enum 순서를 데이터베이스에 저장
- EnumType.STRING: enum 이름을 데이터베이스에 저장 | EnumType.ORDINARY |

## @Temporal

- java 8부터는 최신 하이버네이트에서 지원하므로 잘 쓰지 않는다.

## @Lob

- 지정할 수 있는 타입이 없다.
- CLOB: String, char[], java.sql.CLOB
- BLOB: byte [], java.sql.BLOB

## @Transient

- 데이터베이스에 저장 X, 조회X
- 메모리에만 임시로 어떤 값을 보관하고 싶을 때 사용한다.

# 기본 키 매핑

## 기본 키 매핑 방법

- 직접 할당: @Id 만 사용
- 자동 생성(@GeneratedValue)

## IDENTITY 전략 - 특징

- 기본 키 생성을 데이터베이스에 위임한다.
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
- IDENTITY 전략은 DB에 저장되는 순간 PK값을 알 수 있기 때문에 필요한 시점에 em.persist로 DB에 저장해야 조회된다.

## SEQUENCE 전략 - 매핑

- 주로 오라클 같은 곳에서 사용한다
- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다.
- 오라클, PostgreSQL, DB2, H2 에서 사용

## TABLE 전략

- Sequence 전략을 흉내내는 것
- 테이블을 하나 만들어서 키 생성 전용 테이블로 사용
- 성능이 떨어진다.

## 권장하는 식별자 전략

- 기본 키 제약 조건: null 아님, 유일, 변하면 안된다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자.
- 예를 들어 주민등록번호도 기본 키로 적절하지 않다.
- 권장: Long형 + 대체키 + 키 생성전략 사용