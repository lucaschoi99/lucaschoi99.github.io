# ArgumentResolver
  - 웹과 관련된 공통된 관심사의 편리한 해결을 도움.
  - 개발자가 직접 custom-typed 어노테이션을 만들어 등록하여 사용할 수 있도록 도움.
  - 직접 어노테이션 만들어 등록 후 사용하면 의도한 기능을 더 편리하게 구현할 수 있음. (ex. `@Login`)
    - 로그인 기능 중 Session loginMember를 받아오는 과정에서 `@SessionAtrribute` 로 찾아 가져오는 긴 로직보다는 `@Login` 어노테이션을 직접 구현해서 한줄로 쓰는게 훨씬 깔끔한 코드를 만들 수 있도록 도움.

      ```java
      @Target(ElementType.PARAMETER)
      @Retention(RetentionPolicy.RUNTIME)
      public @interface Login {
      }
      ```

      ```java
      @Slf4j
      public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {
        ...
      ```
