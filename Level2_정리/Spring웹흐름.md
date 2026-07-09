# 🌱 Spring 웹 흐름

> **목적**: 요청–응답 파이프라인에 **횡단 관심사(인증 같은)를 어디에 끼우나** — 끼우는 **위치가 곧 능력**을 정한다.
> **트레이드오프/원리**: Filter(컨테이너 앞단) vs Interceptor(DispatcherServlet 내부) — 어디에 사느냐가 무엇을 할 수 있는지를 가른다.

## 요청 하나의 여정

HTTP 요청이 Controller에 닿기까지 지나는 관문들:

```
HTTP 요청
  → Tomcat (WAS)                     : raw 요청을 HttpServletRequest로 바인딩
  → Filter                           : 컨테이너 앞단 (서블릿 밖)
  → DispatcherServlet                : 여기서부터 Spring MVC
       → HandlerMapping              : 요청에 맞는 Handler(Controller 메서드) 찾기
       → HandlerAdapter              : 호환되는 어댑터 선택
       → Interceptor.preHandle       : 어댑터 진입 직전
       → ArgumentResolver            : @RequestBody 등 파라미터 바인딩 (여기서 역직렬화)
       → @Valid                      : 검증
       → Controller 메서드 실행
       → ReturnValueHandler          : @RestController면 Jackson 직렬화 / 아니면 View 렌더링
```

- **WAS vs WS**: 정적 파일만 주는 웹 서버(WS)와 달리, 앱을 실행하는 것이 WAS. Tomcat이 그 WAS다.
- **역직렬화 시점**: `HttpServletRequest`는 raw `InputStream`을 들고 있을 뿐이다. 실제 Java 객체 변환은 **ArgumentResolver 단계**에서 일어난다 — 그 전엔 "어떤 타입으로 변환할지"를 알 수 없기 때문.
- **@Valid 타이밍**: 타입 변환(역직렬화)이 **먼저**라, 타입이 안 맞으면 그 단계에서 이미 실패하고 **@Valid까지 가지도 못한다**.

## 횡단 관심사를 어디 끼우나 — 위치가 능력을 정한다

인증 검사를 파이프라인 어디에 끼울지 골라야 했고, 그 **위치가 할 수 있는 일을 갈랐다**.

- **Interceptor는 부적합했다**: DispatcherServlet **내부**에 살아서, 내부 forward(`RequestDispatcher.forward`)를 따라가지 못한다. `/public` 요청이 통과한 뒤 컨트롤러가 `/admin`으로 forward하면, Interceptor가 다시 안 돌아 **인증 없이 AdminController가 실행**될 수 있다.
- **그래서 Filter로 갔다**: Filter는 **서블릿 컨테이너 앞단**이라 forward 같은 dispatch를 커버할 수 있다. 위치가 더 바깥이라 능력이 더 넓다.

## 인증은 "요청당 한 번" (OncePerRequestFilter)

- 인증 필터는 `OncePerRequestFilter`를 상속해 **요청당 한 번만** 실행한다. 원리: 필터가 실행되면 요청에 "이미 처리됨" 표시(request attribute)를 남기고, forward로 다시 불려도 그 표시를 보고 건너뛴다.
- **왜 한 번이 맞나 — 인증 vs 인가로 갈라 보면 풀린다**:
  - **인증(누구인가)**: 내부 forward가 일어나도 사용자 **신원은 안 바뀐다** → 다시 확인할 필요 없음 → 한 번이 정답. forward에서 건너뛰는 건 버그가 아니라 **의도**다. (Spring Security 필터들이 모두 OncePerRequestFilter를 상속하는 이유.)
  - **인가(이 경로 접근 가능한가)**: 경로마다 다르지만, 내부 forward는 앱이 스스로 넘긴 **신뢰된 이동**이라 보통 다시 인가하지 않는다.
- 그래서 `DispatcherType.FORWARD`와 "한 번만"은 부딪히는 게 아니다 — 인증엔 forward 재실행이 필요 없어서 **"한 번"이 이기는 게 옳다**. (정말 forward마다 매번 검사해야 하는 드문 경우라면, OncePerRequestFilter가 오히려 틀린 도구다.)

## Filter는 앞단이라 못 하는 것

- Filter는 DispatcherServlet **앞단**이라, 그 내부의 `@ExceptionHandler`·`HttpMessageConverter` 선택 메커니즘이 동작하지 않는다.
- 그래서 Filter에서 에러 응답을 만들려면 `ObjectMapper`로 **직접 JSON 직렬화**하고 `response.setContentType()`도 직접 지정해야 한다.

## 재방문 예정 (Level 2에서 못 판 것)

- **URL 입력 → 서버 도착까지의 네트워크 여정**: DNS로 IP 조회 → 라우터 → WAS 도착 → Tomcat이 하는 일. 이 칸(애플리케이션 파이프라인)보다 한 층 **아래(네트워크)**라 별도 세션에서 판다. WS/WAS·서블릿 컨테이너 vs Servlet 객체의 정확한 구분도 여기서.

## 대표 로그

- [#15 Filter vs Interceptor·OncePerRequestFilter](../logs/15_Filter-Interceptor.md) · [#21 Filter–DispatcherServlet 경계·역직렬화 시점](../logs/21_Filter-DispatcherServlet경계.md) — 정본
- [#01 Content-Type](../logs/01_Content-Type.md) · [#10 ProblemDetail 에러 응답](../logs/10_ProblemDetail-에러응답.md) · [#11 요청 처리·타입 변환·@Valid](../logs/11_요청처리-타입변환Valid.md)
