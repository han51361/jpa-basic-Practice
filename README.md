 # ORM 표준 JPA 프로그래밍 

- entityManager 는 쓰레드간에 공유하지 않는다. 
- #### `JPA 의 모든 데이터 변경,생성은 트랜잭션 안에서 실행되어야한다. `


- ### EntityManager
    -   데이터 수정시 : persist를 안해도 바뀌어서 저장된다.
    -   JPA를 통해서 데이터를 가져온다면 Commit 시점에서 다 관리를 해준다.
    -   Commit 시점 전에 데이터의 변경이 있다면 JPA 가 Update Query 를 쏜다.
    -   그 이후 커밋
    
- ### JPQL
    - JPA 를 사용하면 `엔티티 객체를 대상`으로 개발(SQL = DB table을 대상으로)
    - 문제 : 검색 쿼리
    - 검색할 때 테이블이 아닌, 엔티티 객체를 대상으로 검색
    - 모든 DB 데이터를 객체로 변환하여 검색하는 것 = 불가능
    - 결국 : 검색 조건이 포함된 SQL 이 필요
    
- ## JPA 에서 중요한 2가지
    - ### 객체와 관계형 데이터 베이스 매핑하기
    - ### 영속성 컨텍스트 
    
    
- ## 영속성 컨텍스트
    - 엔티티를 영구 저장하는 환경
    - EntityManager.persist(entity); 
        - 엔티티 매니저를 통해서 영속성 컨텍스트에 접근 
        - EntityManager (N) : 1 PersistenceContext (스프링 프레임워크)
        
    - 엔티티의 생명주기
        - 비영속(new/transient)
            - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
            
        - 영속(managed)
            - 영속성 컨텍스트에 관리되는 상태
            - EntityManager에 persist 하는 것 
            - persist 햿다고 DB 에 저장되진 않는다. 
            
        - detached(준영속)
            - 영속성 컨텍스트에 저장되었다가 분리
        - remove
            - 삭제
    - 엔티티와 1차캐시
        - entityManger 는 데이터를 찾을 때, 1차 캐시를 먼저 조회 후, 
         데이터가 없다면, DB 조회하고 1차 캐시에 저장한다. 
            - (1 트랜잭션 안에서만 캐시 유지 )
    - 엔티티 등록
        - 트랜잭션을 지원하는 쓰기 지연
        - persist를 할 때, insert 쿼리문을 보내는 것이 아니라 insert문을 생성 후 , 보관해놓는다. 
        - commit(); 하는 순간 DB에 insert SQL을 보낸다. 
        
    - Dirty Checking(변경 감지)
         - #### `flush() : 영속성 컨텍스트의 변경내용을 DB에 반영` 
            -   변경 감지 
             
                수정된 엔티티 쓰기 지연 SQL저장소 등록 
              
                쓰기 지연 SQL 저장소 쿼리를 DB에 전송 
                              
    - 플러시 : 영속성 컨텍스트의 변경내용을 DB에 반영
        - 영속성 컨텍스트를 플러시해도 1차캐시가 변하지는 않는다. 
        
        
   - 영속성 컨텍스트를 플러시하는 방법
        - em.flust()
        - 트랜잭션 커밋
        - JPQL 쿼리 실행 
        
        
   - 준영속 상태
        - em.detach();
        - 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
    
    
   - ### 객체와 테이블 매핑 
    - @Entity  
        - 기본 생성자 필수 (파라미터 없는 Public / protected 생성자)
        - final , enum, interface, inner class X
        - 저장할 필드에 final 사용 x
        
   - DB 스키마 자동 생성 - 속성(pom.xml - hibernate.hbm2ddl.auto)
        - create :   기존 테이블 삭제 후 다시 생성(Drop + create )
        - create-drop :  create 와 같으나 종료시점에 table drop
        - update : 변경분만 반영(운영 DB에 사용하면 안된다. )
        - validate : 엔티티와 테이블이 정상 매핑되었는지만 확인
        - none : 사용하지 않음 
   
   - DB 스키마 자동생성 - 주의점
        - `운영장비에는 절대 create, create-drop, update 사용하면 안된다. ` 
            - 잘못하면 데이터 다 날라간다. (* alter 와 drop 기능을 분리하는 것이 맞다.)
        - 개발 초기 : create , update
        - test 서버 : update / validate
        - 스테이징과 운영 서버 : validate/ none
        
   - DDL 생성 기능은 DDL을 자동생성할 때만 사용되고, JPA 실행 로직에는 영향을 주지 않는다. 
   
   - #### @Column
        - name : 테이블 네임 
        - insertable / updatable : 등록/변경가능 여부
        - nullable : null 값 허용 여부 (false 설정시, ddl 생성할 때 notnull 제약)
        - unique : @Table 의 uniqueConstraints와 같지만 한 칼럼에 제약할 때
        - columnDefinition : DB 칼럼정보를 직접 준다. 
        - length : 문자길이 제약조건 / String 타입에만
        - precision, scale : BigDecimal 타입에 사용
            - precision : 소수점을 포함한 전체 자릿수
            - scale : 소수의 자릿수 
   - #### @Enumerated 
        - ##### `ordinal 사용 X`
        
   - 기본 키 매핑 어노테이션
     - @Id 
        - 직접 할당이면, @Id 만 사용
        
     - @GeneratedValue용(자동생성)
        - IDENTITY : DB에 위임
        - SEQUENCE : DB 시퀀스 오브젝트 사용 (@SequenceGenerator 필요)
        - TABLE : 키 생성용 테이블 사용 (@TableGenerator 필요)
        - AUTO : 방언에 따라 자동 지정 / 기본 값 - 위에 3개 중 하나를 알아서 설정해준다. 
   
   - TABLE 전략
        : 키생성 전용 테이블을 하나 만들어 DB 시퀀스 흉내내는 전략
        - 장점 : 모든 테이블에 적용 가능
        - 단점 : 성능 
        
   - 권장하는 식별자 전략 
        - 기존 키 제약 조건 : not null, 유일, 변하면 X
        - 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자. 
        - ##### `권장 : Long 형 + 대체키 + 키 생성전략 사용 `
        
   - IDENTITY 전략
        - 기본 키 생성이 DB에 들어가져야 PK 값이 지정된다. 
        - `JPA 가 1차 캐시에 들어가야 PK 를 찾는데, identity는 그렇지 못하니, 예외적으로 identity는 persist 를 호출하는 시점에, db에 insert 쿼리를 날린다. `
        - 따라서 한번에 모아서 insert 하는 것이, identity 전략에서는 불가능하다. 

  --------------------------
  
  - ## 연관관계 매핑 
  
  
   -  객체간의 관계는 단방향이 더 좋다.
   
   - mappedBy : 객체와 테이블간에 연관관계를 맺는 차이를 이해해야 한다. 
   
   객체 간 : 객체간 연관관계가 단방향이 2개이다. -> 이걸 억지로 양방향이라 하는 거다.
   
    
   테이블 간 : 테이블 간 연관관계는 FK로 설정된 칼럼을 통한 연관관계 1개!
   
   - 연관관계의 주인 은 : DB입장에서 Foreign 키를 관리할 객체를 정해야한다!  -> 이것이 주인의 개념
   - 연관관계의 주인만이 외래 키를 관리(등록, 수정)
   - 주인이 아닌 쪽은 읽기만 가능
   - 주인은 MAppedBy 속성 사용 X
   - mappedby 는 조회만 가능 (등록 수정 불가)
   
   
   >  ### 외래 키가 있는 곳을 주인으로 정하라! 
> 외래키가 있는 곳이 -> many 



 - 양방향 연관관계 주의점
    - 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정해줘야한다. 
    - 연관관계 편의 메소드를 생성하자. -> Member 의 changeTeam 메소드 확인 ! / Team의 addMember
    - 양방향 매핑시 무한루프 조심 
        -   ex) toString() / lombok /JSON 
    - 단방향 매핑만으로도 이미 연관관계 매핑 완료 : 양방향 매핑은 최대한 자제하라
    - JPQL에서 역방향으로 탐색할 일이 많다. 
    - 단방향 매핑을 잘하고 양방향은 필요할 때 추가해도 됨
    
    
  
  - ## Controller 단에서는 entity를 절대 반환하지 말라
  
  - 단방향 / 양방향
  
    * 테이블
        - 외래 키 하나로 양쪽 조인 가능
        - 방향이라는 개념이 없음
        
    * 객체
        - 참조용 필드가 있는 쪽으로만 참조 가능
        - 한쪽만 참조하는 단방향
        - 양쪽이 서로 참조하면 양방향 
        
        
  - 다대일 양방향 vs 다대일 단방향 
  
  - 다대일 양방향 
    - 외래 키가 있는 쪽 : 연관관계 주인
    - 양쪽을 서로 참조하도록 개발 