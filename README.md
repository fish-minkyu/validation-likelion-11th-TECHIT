# Validation(유효성 검사 & 사용자 지정 유효성 검사)

- 2024.01.23 ~ 24 `10주차`
- 01.23 Validation
- 01.24 사용자 지정 유효성 검사

`1월 23일` 수업 내용은 Validation이었다. 
- @Valid
- @NotNull
- @NotEmpty
- @NotBlank
- @Min
- @Size
- @Email

위의 어노테이션을 중점으로 학습하였다.  
`value` 속성으로 유효성 검사의 기준을 변경할 수 있고, `message`로 에러 메시지도 수정할 수 있다.  
그리고 `groups`속성으로 한 Dto의 객체를 특정 작업에 따라 분류하여 유효성 검사를 할 수도 있다.  
하지만, `안티 패턴 문제`가 발생이 되어 잘 고려하여 사용하여야 한다.

또한 `@Validated`의 2가지 기능에 대해 학습하였다.  
- A. 클래스에 붙이면 메서드 개별 파라미터의 검증을 진행할 수 있다.  
- B. 필요한 부분만 따로 검사해서 실행 

`@Validated`은 `RequestParam`과 합쳐지면 에러 상태 코드는 `500` 에러 코드를 반환한다.  
사용자가 잘못했음에도 `500` 에러 코드를 응답하므로 반드시 수동으로 에러 처리를 해줘야 한다.  

결론적으로 총 3단계를 통해 유효성 검사를 실시할 수가 있다.
1. Controller 단위에서 유효성 검사  
2. Service 단위에서 유효성 검사
3. DB 제약사항을 통해 유효성 검사  


또한, 들어온 데이터가 서버가 정한 형식에 적합한지만 검사를 해야한다.  
그 외 검사는 서비스 계층에서 하는 것이 맞다.  
(Ex. 날짜란 데이터를 받을 때 Integer인지만 확인, 그 날짜가 31일 넘는지 아닌지는 서비스 계층에서 확인한다.)

\
`1월 24일` 수업 내용은 사용자 지정 유효성 검사이다.  
이 때, 필요한 것은 다음과 같다.  
1. 설정할 어노테이션 
2. 그 어노테이션의 유효성 검사 설정을 할 클래스

Anotation을 만들 때, 독립적으로 만들어야 한다.
예를 들어, @Email이란 어노테이션이 당연히 이메일은 null이 아니라고 생각한다면 위험해질 수 있다.
따라서, 앞서 @EmailWhiteList를 만들 때, 따로 Null 체크를 하는 조건문을 넣어줬다.


# 스택

- Spring Boot 3.2.2
- Spring web
- Lombok
- Validation

# Key Point

`01/23`
1. @valid 유효성 검사
```java
// 클래스 단위에다가 @Validated를 붙이지 않았을 떼 메서드 Validation
@PostMapping("/validate-dto")
public String validateDto(
// @Valid
// : 요청 받을 때, 이 데이터는 입력을 검증해야 한다.(Dto는 입력받는 단계에서 검증하는 과정을 거친다.)
// 검증하고 싶은 순간 @Valid 어노테이션이 있어야지만 validation을 할 수 있다.
@Valid
@RequestBody
    UserDto dto // Error 시, MethodArgumentNotValid 발생
      ) {
      // 개발자가 원하는대로 사용자가 입력값을 주지 않을 수 있다.
      log.info(dto.toString());
      return "done";
      }
```
2. @Validated 2가지 기능  
2-1. 클래스 단위에서 작동, 메소드 파라미터 유효성 검사  
[UserController](/src/main/java/com/example/validation/UserController.java)
```java
@Slf4j
// @Validated
// : 클래스 단위에다가 @Validated를 붙이고
// 메서드의 매개변수에다가 내가 검증하고 싶은 기준을 넣어주면 된다.
// 즉, 메서드의 파라미터를 검증하고 싶다면 @Validated를 사용한다.
// 기능 A. 클래스에 붙이면 메서드 개별 파라미터의 검증을 진행할 수 있다.
// 기능 B. 필요한 부분만 따로 검사해서 실행
@Validated
@RestController
@RequiredArgsConstructor
public class UserController {
  private UserService service;
  
  // 클래스 단위에다가 @Validated를 적용했을 때 메서드 Validation
  // @Validated가 붙은 클래스의 메서드 파라미터는 검증이 가능하다.
  // /validate-params?age=14
  @GetMapping("/validate-params")
  public String validateParams(
    // Validated + Param을 사용하는 상황에서 발생하는 예외는 조금 다른 예외가 발생한다.
    // 이 예외는 Spring Boot에서 자동으로 처리되는 예외가 아니다.
    // => 사용자가 잘못했지만 서버가 잘못했다는 500에러가 발생하므로 반드시 수동으로 에러 처리를 해줘야한다.
    @Min(14)
    @RequestParam("age")
    Integer age // Error 시, ConstraintViolationException 발생
  ) {
    log.info(age.toString());
    // @Validated가 어느 클래스든 사용이 가능하다고 테스트하기 위해 service의 메소드 호출
    service.printAge(age);
    return "done";
  }
}
```

2-2. Dto별 검사 분리방법  
UserpartialDto - UserController - MandatoryStep
[UserPartialDto](/src/main/java/com/example/validation/dto/UserPartialDto.java)
```java
// UserDto와 구분짓기 위해 생성
// Validated의 2번째 기능을 알아보기 위한 Dto
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class UserPartialDto {
  // 이 두가지(username, password)는 회원가입 단계에서 반드시 첨부해야 하는 데이터
  // 단, 회원 정보 업데이트 단계에서는 반드시는 아니다.
  @Size(min = 8, groups = MandatoryStep.class) // .class: 클래스 정의 자체를 타입으로 전달한다.
  @NotNull(groups = MandatoryStep.class)
  private String username;
  @Size(min = 10, groups = MandatoryStep.class)
  @NotNull(groups = MandatoryStep.class)
  private String password;
  // => groups 항목에다가 MandatoryStep.class를 하게되면
  // 하나의 그룹으로 묶이게 된다.

  // 안티 패턴 문제 발생
  // : Dto 자체는 데이터만 상관이 있어야 하는데 어떠한 과정에서도 검증이 되는지도 알고 있어야 하게 되었다.
  // 따라서 데이터를 담기 위한 용도, 어느 단계에서 사용될 것임을 알아야 하는 용도 등이 되어서
  // 관심사 분리가 완벽히 되지 못했다.

  // 이 두가지(email, phone)는 회원가입 완료 후 추가 정보 기입 단계에서 첨부하는 데이터
  // 단, 추가정보 기입시에는 반드시 포함해야 한다.
  @NotNull
  @Email
  private String email;
  @NotNull
  private String phone;
}
```
[UserController](/src/main/java/com/example/validation/UserController.java)
```java
  @PostMapping("/validate-man") // 1차적으로 반드시 필요한 정보를 기입하는 메소드
  public String validateMan(
    // 기능 B. 필요한 부분만 따로 검사해서 실행
    // : Dto의 일부분만 따로 검사를 하고 사용할 수 있다.
    @Validated(MandatoryStep.class)
    @RequestBody
    UserPartialDto dto
  ) {
    log.info(dto.toString());
    return "done";
  }
```
[MandatoryStep](/src/main/java/com/example/validation/groups/MandatoryStep.java)
```java
package com.example.validation.groups;

// 어떤 항목 그룹에 속해있는 것을 나타내기 위한 인터페이스
public interface MandatoryStep {
}

```

3. 검증 실패시 응답하기 1 & 2번  
[UserController](/src/main/java/com/example/validation/UserController.java)
```java
  // 검증 실패시 응답하기 1번 - @Valid
  // MethodArgumentNotValidException는 400에러를 반환한다.
  @ExceptionHandler(MethodArgumentNotValidException.class)
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public Map<String, Object> handleValidationException(
    final MethodArgumentNotValidException exception
  ) {
    Map<String, Object> errors = new HashMap<>();
    // 예외가 가진 데이터를 불러오기
    // : 예외가 발생하면서 어떠한 데이터 인해 예외가 발생했는지 확인
    List<FieldError> fieldErrors
      = exception.getBindingResult().getFieldErrors();
    // getBindingResult()
    // : 데이터를 객체로 옮기는 과정에서 생겨난 결과를 모아놓은 곳

    // 각각의 에러에 대해서 순회하며
    for (FieldError error: fieldErrors) {
      String fieldName = error.getField();
      String errorMessage = error.getDefaultMessage();
      // Map에 추가하기
      errors.put(fieldName, errorMessage);
    }

    // 여러개의 에러가 있다면 한번에 응답을 해주게 된다.
    return errors;
  }
```
```java
  // 검증 실패시 응답하기 2번 - @Validated
  // ConstraintViolationException은 500 에러로 처리되므로 반드시 예외처리를 직접 해줘야 한다.
  // (클라이언트 잘못인데 서버 잘못으로 응답이 되므로 해줘야 한다.)
  @ExceptionHandler(ConstraintViolationException.class)
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  public Map<String, Object> handleConstraintException(
    final ConstraintViolationException exception
  ) {
    Map<String, Object> errors = new HashMap<>();
    Set<ConstraintViolation<?>> violations // ?: 와일드 카드로서 어떤 타입이든 들어올 수 있다.
      = exception.getConstraintViolations();

    for (ConstraintViolation<?> violation: violations) {
      String property = violation.getPropertyPath().toString();
      String message = violation.getMessage();
      errors.put(property, message);
    }
    return errors;
  }
```
---

`01/24`

1. `Anotation`
[EmailWhiteList](src/main/java/com/example/validation/constraints/anotations/EmailWhiteList.java)
```java
// @Target
// : 이 어노테이션(= 주석)이 어디에 붙일 수 있는지
@Target(ElementType.FIELD) // FIELD는 클래스의 속성에 붙인다는 의미다.
// @Retention
// : Retention(= 유지)은 언제까지 어노테이션이 남아있는지
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EmailWhiteListValidator.class)
public @interface EmailWhiteList {
  // Bean validation을 할 때, validation 라이브러리에서
  // constraint를 만들고 싶으면 이러한 조건들을 붙여줘야 한다.
  String message() default  "Email not in whiteList";
  Class<?>[] groups() default {};
  // 거의 안쓰기 때문에 필요하다라고만 알아두고 똑같이 넣어주면 된다.
  // 궁금하면 찾아보기 (그렇게 중요하지 않음)
  Class<? extends Payload>[] payload() default {};
}
```
2. `유효성 검사 설정 Class`
[EmailWhiteListValidator](src/main/java/com/example/validation/constraints/EmailWhiteListValidator.java)
```java
// ConstraintValidator Class
// 어노테이션이 붙은 타겟이 어떤 조건을 만족할지 validation
public class EmailWhiteListValidator implements ConstraintValidator<EmailWhiteList, String> {

  private final Set<String> whiteList;

  // 생성자
  public EmailWhiteListValidator() {
    this.whiteList = new HashSet<>(); // Set을 쓰는 것이 성능적으로 좋다.
    this.whiteList.add("gmail.com");
    this.whiteList.add("naver.com");
    this.whiteList.add("kakao.com");
  }


  // EmailWhiteList 어노테이션이 붙은 대상의 데이터가
  // 검사를 통과하면 true를
  // 검사에 실패하면 false를
  // 반환하도록 만들면 된다.
  @Override
  public boolean isValid(
    // value: 실제로 사용자가 입력한 내용이 여기 들어온다.
    String value,
    // 일단 ConstraintValidatorContext란 타입이 있구나 하고 넘어가기
    ConstraintValidatorContext context
  ) {
    // value가 null인지 체크하고, (null이면 false)
    if (value == null) return false;
    // value에 @가 포함되고 있는지 확인하고, (아니면 false)
    if (!value.contains("@")) return false;
    // value에 @을 기준으로 자른 뒤, 재일 뒤가 'this.whiteList'에
    // 담긴 값 중 하나인지를 확인한다.
    String[] split = value.split("@");
    String domain = split[split.length -1];
    return whiteList.contains(domain);
  }
}
```

# 복습
~~2024.01.24 Validation 복습~~  
~~2024.01.27 사용자 지정 유효성 검사 복습~~
