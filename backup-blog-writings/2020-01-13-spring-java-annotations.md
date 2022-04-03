---
layout: post
title:  "Java/Spring Annotations"
date:   2020-01-13
tags: [Blog]
---

### `@PostConstruct`
- Deprecated Java 9, removed Java 11.
- 빈이 생성된 후 별도의 초기화 작업이 필요할 때, 초기화 작업을 수행하는 init 메서드에 애너테이션을 선언한다.
- 이 애너테이션이 선언된 메서드는 WAS가 띄워질 때 실행된다.
- xml의 bean element 안에 `init-method="메서드 이름"` 라는 attribute로 지정하는 것과 같다.
- Related: `@PreDestroy`

### `@PreDestroy`
- Deprecated Java 9, removed Java 11.
- 스프링 컨테이너에서 빈을 제거하기 전에 별도의 작업이 필요할 때, 그 작업을 수행하는 메서드에 애너테이션을 선언한다.
- 이 애너테이션이 선언된 메서드는 ApplicationContext가 close()하기 전에 실행된다.
- xml의 bean element 안에 `destroy-method="메서드 이름"` 라는 attribute로 지정하는 것과 같다.
- Related: `@PostConstruct`

### `@RequestMapping`
- 요청에서 쿼리 매개변수, 폼 매개변수 , 파일 등을 추출할 수 있게 해준다.

1. 쿼리 매개변수를 얻을 수 있다.
- 사용하고 싶은 쿼리 매개변수 앞에 애너테이션을 붙여준다.
- 단, 변수 이름과 쿼리 매개변수 이름은 같아야 한다.
- 예시

    아래는 `id`라는 쿼리 매개변수를 추출하여 출력하는 메서드이다.

    ```java
    @GetMapping("/api/foos")
    @ResponseBody
    public String getFoos(@RequestParam String id) {
        return "ID: " + id;
    }
    ```

    실행 결과는 다음과 같다.

    ```console
    http://localhost:8080/api/foos?id=abc
    ----
    ID: abc
    ```

2. 쿼리 매개변수의 이름을 직접 정할 수 있다.
- `name` attribute에 실제 쿼리 매개변수 이름을 입력하면, 변수 이름을 다르게 지정할 수 있다. 실제 쿼리 매개변수의 이름이 길다면 유용하다.
- 예시
    ```java
    @PostMapping("/api/foos")
    @ResponseBody
    public String addFoo(@RequestParam(name = "veryLongQueryParameter") String id) {
        return "ID: " + id;
    }
    ```

3. 선택적으로 입력받을 수 있도록 할 수 있다.
- `@RequestParam` 애너테이션이 붙은 메서드 매개변수는 기본적으로 필수로 입력되어야 한다. 예를 들어 `@RequestParam`이 붙은 `id` 매개변수가 입력되지 않으면 아래와 같이 400 Bad Request 에러가 난다.

    ```console
    GET /api/foos HTTP/1.1
    ----
    400 Bad Request
    Required String parameter 'id' is not present
    ```

- `id`가 반드시 필요한 메서드라면 이 동작이 맞지만, 그렇지 않은 경우에도 메서드가 실행되도록 하고 싶다면 `required` attribute 값을 `false`로 하면 된다. 이 경우, `id`가 입력되지 않아도 메서드가 실행되고, `id` 값은 null이 된다.
- 예시
    ```java
    @GetMapping("/api/foos")
    @ResponseBody
    public String getFoos(@RequestParam(required = false) String id) {
        return "ID: " + id;
    }
    ```

    `abc`라는 `id`가 입력되었을 경우,

    ```console
    http://localhost:8080/api/foos?id=abc
    ----
    ID: abc
    ```

    `id`가 입력되지 않았을 경우,

    ```console
    http://localhost:8080/api/foos
    ----
    ID: null
    ```
4. Java 8 Optional을 이용하는 경우
- Optional과 `orElseGet` 메서드를 이용하면 쿼리 매개변수가 입력되지 않아도 메서드 실행이 가능하다.

    ```java
    @GetMapping("/api/foos")
    @ResponseBody
    public String getFoos(@RequestParam Optional<String> id){
        return "ID: " + id.orElseGet(() -> "not provided");
    }
    ```

    이 경우, `required` attribute를 입력하지 않아도 된다.

    ```console
    http://localhost:8080/api/foos
    ----
    ID: not provided
    ```

- 하지만 null을 사용하는 보통의 컨벤션과 다르고, 가독성이 떨어져서 굳이 써야하는지 의문이 든다.

5. 기본값을 지정할 수 있다.
- `defaultValue` attribute 값으로 기본값을 지정할 수 있다.
- 예시

    ```java
    @GetMapping("/api/foos")
    @ResponseBody
    public String getFoos(@RequestParam(defaultValue = "test") String id) {
        return "ID: " + id;
    }
    ```

    실행 결과

    ```console
    http://localhost:8080/api/foos
    ----
    ID: test
    ```

- 이 경우, `required`가 자동으로 false가 된다.

### Related
- `@PathVariable`

### Reference

1. [StackOverflow: Understanding Spring @Autowired usage][1]
2. [Spring RequestParam][2]

[1]:https://stackoverflow.com/a/19419296
[2]:https://www.baeldung.com/spring-request-param
