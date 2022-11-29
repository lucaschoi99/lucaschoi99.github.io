# 검증 (Validation)
- 타입, 필드, 각종 요구사항 서버 검증
- 어떤 값, 필드가 잘못 입력됐는지 친절하게 알려주는 것이 포인트. (친절한 서비스)
- 스프링에서 “BindingResult” 로 편리하게 검증 오류를 제공받을 수 있음.
    - 오류 정보를 BindingResult에 담아서 컨트롤러를 정상 호출해주는 것 까지 해줌.
    - FieldError, ObjectError 와 함께 사용.
        - TypeError 발생시 → FieldError가 값을 BindingResult에 저장해주어 rejectedValue를 사용할 수 있게 해줌.
        - 오류 메세지 처리(개발자가 직접 설정) → [messages.properties](http://messages.properties) 파일 처리한 것처럼 [errors.properties](http://errors.properties) 파일 만들어 관리해 줄 수 있음. (같은 원리)
            - FieldError에 “codes”, “arguments” 부분에 넣어주면 됨.

                `bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1000, 1000000}, null));`

            - (개선) 사실 BindingResult는 어떤 객체를 담당하는지 이미 알고 있음. 따라서 FieldError 사용할 필요 없이 .rejectValue, .reject 이용해서 중복 줄여 개발할 수 있음.

                `bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);`

                - MessageCodesResolver : 구체적인 것부터 범용적인 것까지 codes 배열로 만들어주기 때문에 가능한 것. (스프링이 자동으로 해 줌)

                    **FieldError** rejectValue("itemName", "required")
                    다음 4가지 오류 코드를 자동으로 생성

                    ```
                    <우선순위>
                    required.item.itemName (code.객체명.필드명)
                    required.itemName (code.필드명)
                    required.java.lang.String (code.필드타입)
                    required (code)

                    ```

        - 오류 메세지 처리(스프링이 직접 오류 처리 ex.타입 에러) → [errors.properties](http://errors.properties) 에 추가해서 마찬가지로 바꿔줄 수 있음.

            `rejected value [ㅃㅃ]; codes [typeMismatch.item.price,typeMismatch.price,typeMismatch.java.lang.Integer,typeMismatch]`

            ```
            [errors.properties](http://errors.properties) 

            #추가
            typeMismatch.java.lang.Integer=숫자를 입력해주세요.
            typeMismatch=입력한 타입이 맞지 않습니다.
            ```

    - Thymeleaf → `“#fields”` 로 접근 가능, `“th:errors”`, `“th:errorclass”` 활용 가능.

- 검증 로직 분리 → 따로 class 만들어 관리.

    ```java
    @Component
    public class ValidatorItem implements Validator {

        @Override
        public boolean supports(Class<?> clazz) {
            return Item.class.isAssignableFrom(clazz);
        }

        @Override
        public void validate(Object target, Errors errors) {

            Item item = (Item) target;

            // 검증 로직
            ...
        }
    }
    ```

    - Validator 불러올 때 →

         Controller에 `@InitBinder` 추가, 사용하려는 메소드에서 `@Validated` 넣어줘서 검증기 호출

        ```java
        @InitBinder
        public void init(WebDataBinder dataBinder) {
            dataBinder.addValidators(validatorItem);
        }
        ```

        ```java
        @PostMapping("/add")
        public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

            ...
        }
        ```
