---
layout: post
categories: [Spring]
title: 유효성 검증을 통해 커스텀 exception 생성하기
author: tngtied
date: 2024-02-26
---

최근 프로젝트에서 user API를 구현하다 보니, signup과 login시 특정 필드에 대해서 유효성 검사를 해야 하는 상황이 발생했다. 유효성 검증의 방법 및 유효성 검사에 실패했을 경우의 핸들링을 고민한 끝에, UserLoginDTO 및 UserSignupDTO, 그리고 CustomException 및 CustomErrorCode를 사용하기로 결정하였다.

# 유효성 검증

## 검증할 유효성 설정

유효성 검증은 UserSignupDTO에서 `jakarta.validation.constraints` 패키지를 사용하여 이루어진다.

```Java
@Data
@Getter
@RequiredArgsConstructor
public class UserSignupDTO {
	@JsonProperty
	@Column(unique = true, name = "USERNAME")
	@NotNull
	@Size(min = 2, max = 20)
	String username;

	@JsonProperty
	@Length(min = 8, max = 20)
	@NotNull
	String password;

	@JsonProperty
	@NotNull
	@Email
	@Column(unique = true, name = "EMAIL")
	String email;

	public UserSignupDTO(String username, String password, String email) {
		this.username = username;
		this.password = password;
		this.email = email;
	}
}
```

각 필드마다 `@NotNull`, `@Size`와 같은 어노테이션을 붙임으로써 검증할 유효성을 설정한다. `@Column(unique = true)`의 경우 데이터베이스 상에서 이루어지는 연산이므로 아래와 같이 user(member) entity 내부의 필드에 붙인다.

```Java
@Entity
@Table(name = "MEMBER")
@Getter
@AllArgsConstructor
public class Member implements UserDetails {

	@Id
	@GeneratedValue
	@Column(name = "USERID")
	private Long userid;

	@Column(unique = true, name = "USERNAME")
	@NotNull
	@Size(min = 2, max = 8)
	private String username;

	@Column(name = "PASSWORD")
	@NotNull
	private String password;

	@ElementCollection(fetch = FetchType.EAGER)
	private List<String> roles = new ArrayList<>();

	@NotNull
	@Email
	@Column(unique = true, name = "EMAIL")
	private String email;
```

이후 해당 유효성 검증은 컨트롤러의 `BindingResult` 파라미터를 통해 이루어진다.

## 유효성 검증

```Java
@PostMapping("/signup")
	public UserValidationErrorDTO signup(@RequestBody @Valid UserSignupDTO siteUser, BindingResult bindingResult) {
		UserValidationErrorDTO userValidationErrorDTO = new UserValidationErrorDTO();
		userValidationErrorDTO.objectErrorList = new ArrayList<String>();
		userValidationErrorDTO.fieldErrorList = new ArrayList<UserValidationFieldError>();

		if (bindingResult.hasErrors()) {
			System.out.println(">>BindingResult has errors");
			userValidationErrorDTO.setHasErr(true);

			for (ObjectError err : bindingResult.getAllErrors()) {
				if (err instanceof FieldError) {
					FieldError fieldError = (FieldError)err;
					if (fieldError.getField().equals("username")) {
						if (fieldError.getCode().toString().equals("Size")) {
							throw new CustomException(CustomErrorCode.INCORRECT_USERNAME_SIZE);
						}
					} else if (fieldError.getField().equals("password")) {
						if (fieldError.getCode().toString().equals("Length")) {
							throw new CustomException(CustomErrorCode.INCORRECT_PASSWORD_SIZE);
						}
					} else if (fieldError.getField().equals("email")) {
						throw new CustomException(CustomErrorCode.EMAIL_PATTERN_NOT_MATCH);
					}
				} else {
					userValidationErrorDTO.objectErrorList.add(
						Pattern.compile("(\\w*)$").matcher(err.getClass().toString()).group(1));
				}
			}
		} else {
			try {
				memberService.create(siteUser.getUsername(), siteUser.getEmail(), siteUser.getPassword());
			} catch (Exception e) {
				//regex pattern matching
				if (e.getClass().equals(DataIntegrityViolationException.class)) {
					if (e.getMessage().equals("USERNAME")) {
						throw new CustomException(CustomErrorCode.DUPLICATE_USERNAME);
					} else if (e.getMessage().equals("EMAIL")) {
						throw new CustomException(CustomErrorCode.DUPLICATE_EMAIL);
					}
				}
				userValidationErrorDTO.objectErrorList.add(
					Pattern.compile("(\\w*)$").matcher(e.getClass().toString()).group(1));
				return userValidationErrorDTO;
			}
			userValidationErrorDTO.setHasErr(false);
		}
		return userValidationErrorDTO;
	}
```

`BindingResult`의 `ObjectError`마다, 해당 에러가 `FieldError`일 경우 변환해, 내용을 확인하고 경우에 맞는 exception을 throw한다. 중복된 유저명이나 이메일의 경우, `BindingResult` 상에서 나타나지 않으므로 try-catch 문으로 `DataIntegrityViolationException`을 받는다. 이후, 중복된 필드가 무엇인지 확인하고 해당 필드에 맞는 ErrorCode의 CustomException을 throw한다.

# Custom Exception

Custom Exception은 총 다섯 가지의 파트로 구성된다. 자세히는 다음과 같다.

- 각 커스텀 에러 코드마다의 enum을 담은 `CustomErrorCode`
- 해당 에러코드를 `RuntimeException`에 담는 `CustomException`
- 해당 exception이 발생했을 때 핸들링하여 `ResponseEntity`를 반환하는 `CustomExceptionHandler`
- 에러에 대한 공통 응답 객체, `ErrorResponseEntity`

## CustomErrorCode

```Java
@Getter
@AllArgsConstructor
public enum CustomErrorCode {
	EMAIL_PATTERN_NOT_MATCH(HttpStatus.BAD_REQUEST, "이메일 형식에 부합하지 않습니다."),
	INCORRECT_USERNAME_SIZE(HttpStatus.BAD_REQUEST, "유저네임의 길이는 2글자 이상, 20글자 이하여야만 합니다."),
	INCORRECT_PASSWORD_SIZE(HttpStatus.BAD_REQUEST, "패스워드의 길이는 8글자 이상, 20글자 이하여야만 합니다."),
	DUPLICATE_USERNAME(HttpStatus.BAD_REQUEST, "해당하는 유저네임이 이미 존재합니다."),
	DUPLICATE_EMAIL(HttpStatus.BAD_REQUEST, "해당하는 이메일이 이미 존재합니다."),

	private final HttpStatus httpStatus;
	private final String message;
}
```

유효성 검증에 실패하는 요청은 모두 BAD_REQUEST로 취급하였다.

## CustomException

```Java
@AllArgsConstructor
@Getter
public class CustomException extends RuntimeException {
	CustomErrorCode customErrorCode;
}
```

간단하게 `CustomErrorCode`만을 담는 객체이다.

## CustomExceptionHandler

```Java
@ControllerAdvice
public class CustomExceptionHandler {
	@ExceptionHandler(CustomException.class)
	protected ResponseEntity<ErrorResponseEntity> handleCustomException(CustomException e) {
		return ErrorResponseEntity.toResponseEntity(e.getCustomErrorCode());
	}
}
```

`@ControllerAdvice`의 경우, 위에서 선언한 것과 같이 `@ExceptionHandler` 또는 `@ModelAttribute`를 선언할 수 있도록 특화된 `@Component`의 일종이다. 기존에 `@Component`가 그러한 것처럼 IoC(Inversion of Control) 컨테이너가 해당 클래스를 bean으로 등록하여 어플리케이션에서 사용 가능하도록 만든다.
해당 어노테이션이 붙은 클래스 내부에 구현된 `handleCustomException` 메서드에 `@ExceptionHandler(CustomException.class)`을 붙임으로써 `CustomException`클래스의 exception이 발생했을 때 해당 메서드로 핸들링한다는 것을 스프링에게 알려줄 수 있다.

## ErrorResponseEntity

```Java
@Data
@Builder
public class ErrorResponseEntity {
	private int status;
	private String code;
	private String message;

	public static ResponseEntity<ErrorResponseEntity> toResponseEntity(CustomErrorCode e) {
		return ResponseEntity
			.status(e.getHttpStatus())
			.body(ErrorResponseEntity.builder()
				.status(e.getHttpStatus().value())
				.code(e.name())
				.message(e.getMessage())
				.build()
			);
	}
}
```

기본적으로 `ResponseEntity`를 사용하며, 해당 클래스가 커스텀해서 작성된 `ErrorResponseEntity`를 body에 포함하도록 작성하였다. body에 들어가는 ErrorResponseEntity의 경우 입력값으로 받은 CustomErrorCode의 enum을 사용하도록 작성하였다.

# 결과물

```Java
@Test
@DisplayName("Test: post signup duplicate field [username]")
void signupTestDuplicateUsername() throws Exception {
	String jsonContents = objectToJson(makeValidUser());
	mockMvc.perform(post(base_path + "/user/signup")
			.content(jsonContents)
			.contentType(MediaType.APPLICATION_JSON)
			.accept(MediaType.APPLICATION_JSON))
		.andExpect(status().isOk())
		.andDo(print());
	mockMvc.perform(post(base_path + "/user/signup")
			.content(jsonContents)
			.contentType(MediaType.APPLICATION_JSON)
			.accept(MediaType.APPLICATION_JSON))
		.andExpect(status().isBadRequest())
		.andDo(print());
}
```

동일한 `UserSignupDTO`를 반복해서 POST하는 테스트를 작성하였다. 그 결과 아래와 같이 훌륭한 Bad Request 응답이 반환되는 것을 볼 수 있다.

```
MockHttpServletResponse:
           Status = 400
    Error message = null
          Headers = [Content-Type:"application/json"]
     Content type = application/json
             Body = {"status":400,"code":"DUPLICATE_EMAIL","message":"해당하는 이메일이 이미 존재합니다."}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
