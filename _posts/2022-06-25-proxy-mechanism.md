---
title: "[JPA ORM] 프록시(Proxy) 매커니즘"
date: 2022-06-25 07:10:28
categories: JPA ORM Proxy


---



## 프록시(Proxy)란 무엇인가?

JPA에서 Entity manager(em)로 데이터베이스에서 조회해서 객체를 참조할 때 실제 객체를 가져오지 않고 진짜 객체를 상속 받은 가짜객체 (속성과 필드는 동일하고 값은 없는 껍데기 객체) 를 가져오는 방법이다.

JPA 구현체인 Hibernate 에서 프록시를 동작하는 방법을 기준으로 알아보았다.
- - -

## 왜 프록시를 사용하는가?

왜 진짜 객체를 데이터베이스에서 바로 꺼내오지 않고 가짜 객체를 내세워 역할을 하게 하는 것일까? 말 그대로 데이터베이스 조회를 미룸으로써 얻는 이점이 있기 때문이다.

예를 들어, Member class와 Team class가 서로 연관관계 매핑이 되어 있지만 비즈니스 로직 상 Member를 접근할 때 Team을 함께 사용할 일이 매우 적다고 가정해보자.

`em.find(Member.class, ...)` 이런 식으로 Member 객체를 가져오게 된다면 Team 테이블도 함께 JOIN 되어 데이터베이스에서 가져와지지만 사용되지 않는다. 이런 작업이 계속 반복되게 되면 성능에 악영향을 주게 될 것이다. 

따라서 가짜 객체를 내세워서 역할을 하게 한 뒤, 실제로 진짜 객체 값이 필요할 때 영속성 컨텍스트를 통해 데이터베이스에서 객체를 참조해 값을 주입해주면 되는 것이다.

<img width="613" alt="image" src="https://user-images.githubusercontent.com/73485743/175618674-d73bff9f-5c0d-4241-ab7a-12cd3a806f7d.png">

- - -

## 프록시의 특징과 사용 시 조심할 점

- `em.getReference(Member.class, ...)` 이런 식으로 프록시 객체를 만들어낼 수 있고, 진짜 객체의 값이 필요할 때 영속성 컨텍스트를 통해 초기화를 진행하게 된다.
- 프록시 객체가 초기화 되면 진짜 객체로 대체되는 것이 아니고, 실제 객체의 필드 값을 그대로 사용할 수 있게 되는 것이다.
- 프록시 객체는 진짜 객체를 상속받았기 때문에 둘의 타입이 다르다. 따라서 타입 비교를 할 때는 == 비교 대신 instanceof 를 사용해야 한다.
- 만약 같은 Transaction 내에서, 프록시 객체가 먼저 만들어져 초기화 됐다면 후에 `em.find()`로 객체를 가져와도 프록시 객체가 가져와진다. 반대로 `em.find()`로 객체를 먼저 참조했다면 후에 `em.getReference()`로 객체를 가져오려고 해도 프록시 객체가 아닌 진짜 객체가 가져와 진다.
- 위의 특징은 같은 트랜잭션 내에서 영속성 컨텍스트를 유지하기 위해 JPA가 제공하는 기본적인 매커니즘을 따른 것이다. (그리고 만약 영속성 컨텍스트 안에 이미 진짜 객체가 등록되어 있다면 굳이 프록시 객체를 다시 만들어 사용할 필요도 없다.)
- 어떠한 이유로 영속성 컨텍스트가 해제된 준영속 상태의 entity의 경우에는 프록시 객체를 초기화하면 `LazyInitializationException` 예외가 터진다. 영속성 컨텍스트에서 더이상 관리하지 않는 객체이기 때문이다. 영속성 컨텍스트 해제에는 `em.close()`, `em.clear()`, `em.detach(객체)` 등의 경우가 있다.

- - -
**참고 강의안:** 인프런 김영한님 강의안, 자바 ORM 표준 JPA 프로그래밍 - 기본편 중 08.프록시와 연관관계 정리.pdf
