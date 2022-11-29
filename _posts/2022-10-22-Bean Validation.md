# Bean Validation (Bean 검증)
- 일일히 검증 로직 짜는건 비효율적 → `@NotBlank`, `@NotNull`, `@Range` 등 공통된 검증 사용 가능.
- 따라서, annotation으로 깔끔하게 해결.
- build.gradle 에 `implementation 'org.springframework.boot:spring-boot-starter-validation'` 추가.
- Field Error: How to?
    - 검증할 클래스 (ex.Item) 필드에 annotation 추가. (`@NotBlank`, `@NotNull`, `@Range(min= 1000, max= 1000000)` 등)
    - Controller 메소드에 `@Validated` 추가 → 스프링이 자동으로 global validator 등록, annotation보고 검증 수행.
    - [errors.properties](http://errors.properties) 에 마찬가지로 오류 메세지 바꿀 수 있음.

```java
# Bean validation
NotBlank= 값을 입력하세요.
Range= {1} ~ {2} 값을 허용합니다.
MAX= 최대 {1} 의 값까지 허용합니다.
```

- Object Error: How to?
    - `@ScriptAssert` 어노테이션 방법이 있긴 하지만, 실무에서는 다른 클래스 필드를 끌어오거나 디비에서 값을 조회해서 가져와야 되는 경우도 있기 때문에 선호되는 방법은 아님. (기능이 너무 단순하고 약함)
    - 추천하는 방법은 그냥 java 코드로 직접 검증하는 것.

        ```java
        // 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int result = item.getPrice() * item.getQuantity();
            if (result < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, result}, null);
            }
        }
        ```

- Bean Validation의 한계
    - 동일한 객체를 등록할 때와 수정할 때 검증방법이 다를 수 있음 → 공통적으로 해결하지 못함 (충돌이 발생).
        - ex) 상품 등록과 수정 시에 요구 사항이 다를 수 있음
            - 수정 시 요구사항 - quantity 수량 제한을 풀고, 상품 id도 필수로 필요할 것.
                - 해결책 → id: @NotNull 추가, quantity: @Max(9999) 제거? → 해결 X.
- 해결책 - 크게 2가지
    1. Bean validation의 groups 기능 사용.
        - SaveCheck, UpdateCheck 2개의 Interface class 만들어서 각 필드 어노테이션에 각각 조건을 걸기.
            - `@NotBlank(groups = {SaveCheck.class})`
            - `@Range(min= 1000, max= 1000000, groups = {SaveCheck.class, UpdateCheck.class})`
        - 그리고 Controller에 `@Validated(SaveCheck.class)` 로 수정. (SaveCheck 조건이 걸린 어노테이션만 골라 적용)
        - 그러나, 실무에서는 많이 쓰지 않음.
        - Why? 생각해보면 real world 에서는 등록과 수정의 구체적인 내용이 상당히 많이 다른 경우가 많고, 따라서 이런 경우엔 두 개의 객체로 각각 분리해서 받는게 훨씬 나음.
    2. 폼 전송 객체 분리 (ex. 등록용 폼 객체, 수정용 폼 객체)
        - 상품 등록, 수정에 각각의 필요에 맞게 따로 분리해서 객체를 생성
            - **폼 데이터 전달을 위한 별도의 객체 사용**
                - HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository
                - 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다는 번거로움이 있지만, 실무에서의 복잡한 상황에 대응하기 위해 가장 선호되는 방법.

        ```java
        @Data
        public class ItemSaveForm {

            @NotBlank
            private String itemName;

            @NotNull
            @Range(min= 1000, max= 1000000)
            private Integer price;

            @NotNull
            @Max(9999)
            private Integer quantity;

        }
        ```

        ```java
        @Data
        public class ItemUpdateForm {

            @NotNull
            private Long id;

            @NotBlank
            private String itemName;

            @NotNull
            @Range(min= 1000, max= 1000000)
            private Integer price;

            // 수정 시 수량은 제한 없음
            @NotNull
            private Integer quantity;
        }

        ```


- Api 호출 시 검증
    - `@RestController` 를 이용한 Json Api 통신 시에도 `@Validated` 검증을 사용할 수 있음.
    - 차이점
        - `@ModelAttribute` 대신, `@RequestBody`를 이용한다는 점이 차이.
        - 다만, `@ModelAttribute` 는 각각의 필드에 세밀하게 작용해 타입 오류가 발생해도 bindingResult에 넘기고 Controller호출은 가능했지만,
        - `@RequestBody` 를 이용한 Api 호출 시에는 타입 오류가 발생해 JSON 객체로 변경되지 못하면 아예 다음 단계 진행이 불가능. 즉, Controller 호출과 검증 모두 실행되지 않는다.
    - (복습) `@ModelAttribute`, `@RequestParam`, `@RequestBody`
        - `@ModelAttribute` - HTTP 요청 파라미터를 쉽게 받아주는 기능. 객체를 자동으로 생성해주고, 넘어온 파라미터를 객체 필드에 set까지 해줘 바로 사용할 수 있게 함.
        - `@RequestParam` - 마찬가지로 HTTP 요청 파라미터를 쉽게 받아주는 기능. 이 경우는 객체가 아니라 String, int 와 같은 단순한 타입의 파라미터를 바인딩 해주어 사용할 수 있게 함.
        - `@RequestBody` - HTTP 메세지의 Body 정보를 직접 조회하는 기능. Api 통신 시 Json형식으로 정보를 받고 넘길 수 있는데 이 경우에 사용함.
