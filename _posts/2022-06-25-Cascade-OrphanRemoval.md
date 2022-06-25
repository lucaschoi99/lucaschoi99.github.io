---
title: "[JPA ORM] 영속성 전이 (Cascade)와 고아 객체 제거(OrphanRemoval)"
date: 2022-06-25 23:45:44
categories: JPA ORM Cascade OrphanRemoval


---

## Cascade (영속성 전이) 란?

Entity를 persist(영속화)할 때 연관된 entity도 함께 persist하는 편리함을 제공하는 기능이다. 주의할 점은, 앞서 설명한 지연로딩(Lazy) 과는 아무 관련없는 다른 개념이다.

- - -

## Cascade 기능을 사용하는 이유

부모 Entity를 persist할 때 자식 entity도 함께 persist 하고 싶을 때 이용한다. 이로 인해서 부모와 자식의 persist, remove 를 함께 관리할 수 있다는 장점이 있다. 또한 부모 entity 만으로 자식 entity들을 모두 컨트롤할 수 있다.

- - -

## Cascade 사용 시 주의할 점

하지만 cascade 기능은 연쇄적인 개념이기 때문에 주의가 필요하다. 

반드시 Entity 서로 간의 단일 종속관계가 있을 때만 사용하고, 다른 Entity 와 복잡하게 매핑관계가 얽혀 있다면 사용하면 안된다. 예상치 못한 코드 흐름으로 영속성 컨텍스트에서 사용 중인 entity가 삭제될 수도 있고 생성될 수도 있는 위험이 있기 때문이다. 

그러나 부모 Entity를 생성할 때 자식 Entity들을 한꺼번에 persist하고, 삭제할때 함께 삭제할 수 있다는 점에서 매우 편리하고 실용적인 기능임은 분명하다.

- - -

## OrphanRemoval (고아 객체 제거)

고아 객체 제거란 부모 Entity와 연관관계가 끊어진 자식 Entity를 자동으로 삭제하는 기능이다. 즉, 참조가 제거된 Entity는 다른 곳에서 참조하지 않는 객체로 보고 삭제하는 것이다.

 **`orphanRemoval = true`** 로 기능을 켤 수 있다.

```
Parent parent1 = em.find(Parent.class, id);

 //자식 엔티티를 컬렉션에서 제거

parent1.getChildren().remove(0); 

```

위의 코드 대로 실행하면 자식 Entity는 부모 Entity의 자식 컬렉션에서 제거되고 영속성 컨텍스트에서도 관리되지 않고 삭제되게 되는 것이다.



이 기능도 Cascade 와 마찬가지로, 참조하는 곳이 하나일 때 사용해야 한다. 즉, 반드시 Entity 서로 간의 단일 종속관계가 있을 때만 사용해야 예상치 못한 객체 삭제를 피할 수 있다.

- - -

## Cascade + OrphanRemoval 함께 사용 시 얻는 이점

**`CascadeType.ALL, orphanRemoval=true`**

부모 Entity를 통해 자식 Entity들의 영속성 컨텍스트에서의 Lifecycle (Persist, removal) 을 관리할 수 있는 장점이 있다. 따라서 의도적으로 같이 관리하고 싶은 Entity 들간의 영속성을 편리하게 관리할 수 있게 도와주는 기능이라고 생각하면 된다.

- - -

**참고 강의안:** 인프런 김영한님 강의안, 자바 ORM 표준 JPA 프로그래밍 - 기본편 중 08.프록시와 연관관계 관리.pdf
