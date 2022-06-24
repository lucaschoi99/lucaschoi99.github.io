---
title: "[JPA ORM] 상속관계 매핑"
date: 2022-06-24
categories: JPA ORM Mapping

---

## DB에서 상속관계를 표현하는 전략
관계형 데이터베이스 구조 상 상속관계가 존재하지 않는다. 그러나 Java 객체지향 언어의 큰 장점을 DB 구현에 적용시키지 못한다면, 그것은 객체지향 언어를 사용하는 큰 의미가 사라진다고 할 수 있다.

다행히도, 슈퍼타입 서브타입 관계라는 모델링 기법을 통해 객체 상속과 유사하게 설계를 할 수 있다. 즉, 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 동일하게 생각해 구현하면 된다.

실제 모델로 구현하는 방법은 크게 3가지 방법이 존재한다.

   - **JOINED 전략:** 공통된 속성을 하나의 abstract class로 받고, 이를 상속한 각각의 테이블로 구현
   - **SINGLE_TABLE 전략:** 모든 속성을 하나의 커다란 통합 테이블로 구현
   - **TABLE_PER_CLASS 전략:** 중복된 속성이 있어도 각각의 구현 클래스마다 테이블 만들기
- - -

## TABLE_PER_CLASS 전략

결론부터 말하자면, TABLE_PER_CLASS 전략은 실무에서, 그리고 개인 프로젝트 시에도 사용하면 안된다. DB 설계와 ORM 구조 모두에서 장점보다 단점이 더 크기 때문이다.

여러 테이블을 함께 조회할때 UNION 이 필요하기 때문에 성능이 떨어지고, 하나의 필드를 기준으로 쿼리할 때에도 마찬가지로 모든 테이블을 모두 조회해야 하기 때문에 상당히 비효율적이다. 물론, 아무 신경쓰지 않고 명확하게 구분할 수 있으며 모든 필드에 not null을 설정할 수 있다는 장점은 존재한다. 그러나 단점이 훨씬 커보이는 것은 사실이다.

또, 단순히 개발자적 관점에서 볼때 중복 필드를 모으지 않고 따로 처리하는 것은 엄청난 시간적, 공간적 낭비가 아닐 수 없다.

<img width="662" alt="image-20220625014649007" src="https://user-images.githubusercontent.com/73485743/175604771-98d8137d-7c41-430b-bdc4-1d7e97297de1.png">

**[TABLE_PER_CLASS 전략: 사용하지 말 것]**

- - -
## JOINED 전략

   그렇다면 JOINED 전략, SINGLE_TABLE 전략 중에서는 무엇을 택해야 하는가? 
   
   이 두 가지 전략 중 어느 것이. 더 낫다라고 하기는 힘들다. 서비스나 프로젝트의 상황에 따라 장점과 단점을 잘 생각해보고, 그에 맞게 전략을 선택하면 된다.

   먼저 JOINED 전략은 JPA 상속 구조와 가장 유사하다. 상속받는 클래스와 각 테이블의 필드를 PK, FK로 연결시켜 구현하면 된다. Spring boot 에서 이 전략을 사용하려면 상속받을 클래스에 `@Inheritance(strategy=InheritanceType.JOINED)` 어노테이션을 달아주면 된다. 또한 DTYPE으로 각 테이블을 구별해 줄 필요가 있다면, 예를 들어 상속받는 클래스 ITEM의 필드에 DTYPE 필드를 넣어주어야 상속하는 클래스들 (ALBUM, MOVIE, BOOK ...) 의 데이터를 관리할 수 있는 설계라면, 상속받을 클래스 (여기서는 abstract class ITEM)에 `@DiscriminatorColumn` 어노테이션을 달아주면 해결된다.

   또한 이 전략은 저장공간을 효율적으로 관리할 수 있다는 장점도 있다. 다만 단점은 데이터 저장 시에 INERT 쿼리가 2번씩 나가고, 조회 시에 JOIN 쿼리를 많이 사용하게 되어 성능 저하 이슈가 있다는 점이다. 그러나 테이블끼리 JOIN 쿼리가 체계적으로 나가도록 설계를 하면 그렇게 성능에 critical한 문제는 되지 않을 것 같다.
   
<img width="650" alt="image-20220625011918525" src="https://user-images.githubusercontent.com/73485743/175604979-67ec7f88-f5d5-4f55-93b6-a887413bc1e9.png">

**[JOINED TABLE 전략]**

- - -
## SINGLE_TABLE 전략
   단일 테이블 전략은 말 그대로 모든 테이블에 들어갈 필드를 하나의 큰 테이블에 모두 모아 만드는 방법이다. 이렇게 되면 테이블끼리의 JOIN 쿼리가 필요 없으므로 조회 성능이 빠르다는 장점이 있다. 그러나 특정 테이블에 없는 필드는 모두 null을 허용하고 넣어주어야 한다.
   
   마찬가지로 Spring boot 에서는 `@Inheritance(strategy=InheritanceType.SINGLE_TABLE)` 어노테이션을 달아주면 되고, 이 전략의 경우에는 앞선 JOINED 전략과는 다르게 DTYPE으로 반드시 각 테이블 데이터를 구별할 필요가 있다는 점이다. 따라서 이 전략을 취하게 되면 따로 어노테이션을 달지 않아도 default로 DTYPE 필드가 생성된다.

   아무래도 하나의 테이블에 모든 데이터를 저장하기 때문에 프로젝트 규모가 커지게 되면 테이블도 따라서 매우 커질 수 있다. 따라서 상황에 따라서 조회 성능이 느려질 수 있는 경우가 생기는데 그 상황이 오면 앞의 JOINED 전략과 SINGLE_TABLE의 전략 중에서 장점과 단점을 살펴보고 무엇을 선택할지 다시 고려해보는 것이 좋을 것이다.

<img width="574" alt="image-20220625012714493" src="https://user-images.githubusercontent.com/73485743/175605068-49610a28-f04e-4d42-9b06-4fa8f913550c.png">

**[SINGLE_TABLE 전략]**

- - -
## MappedSuperclass
   마지막으로 Spring에서 단순하게 공통 매핑 정보를 위해서 상속관계 형태의 매핑을 사용하고 싶다면 `@MappedSuperclass` 어노테이션을 사용하면 된다. 
   
   이 방법은 부모 클래스의 필드를 상속 받는 자식 클래스에 매핑 정보만 제공하는 것이다. 따라서 상속할 BaseEntity class를 abstract class로 만들고 공통으로 사용할 매핑 정보를 모아주면 된다. (이 클래스는 @Entity와 전혀 다른 개념이기 때문에 상위 클래스로 조회, 검색 (ex. em.find(BaseEntity) 이런 식의 코드 작성은 불가능하다.)

<img width="514" alt="image-20220625013332705" src="https://user-images.githubusercontent.com/73485743/175605139-da22477a-96d2-4ce2-93d5-4abb2c2666b2.png">

**[`@MappedSuperclass` 를 사용하는 경우]**

- - -
**참고 강의안:** 인프런 김영한님 강의안, 자바 ORM 표준 JPA 프로그래밍 - 기본편 중 07.고급 매핑.pdf
